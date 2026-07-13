---
name: ci-cd-pipeline-engineering
description: "Design and fix CI/CD pipelines and deployment environments so they scale with a team instead of collapsing on the worst possible day: CI minutes cost management (self-hosted runners, conditional/path-based triggers, usage monitoring), dev/staging/production environment separation with a real branching strategy, and platform deployment limits (function timeouts, concurrency ceilings, bandwidth caps, payload limits) on Vercel, Netlify, Railway, AWS Lambda, and similar platforms. Use when Codex is setting up CI/CD, choosing or migrating a deployment platform, debugging a pipeline that got slower or hit a paywall, or when a team is still deploying straight to production with no staging environment."
---

# CI/CD Pipeline Engineering

## Overview

Use this skill to prevent two very common, very avoidable failures: a CI/CD bill or minutes-cap that surprises a team mid-sprint, and a deployment process where every push goes straight to production with no preview, no staging, and no rollback plan. Neither of these is a hard engineering problem — they're a small number of decisions that are cheap to make early and expensive to discover by accident.

## Workflow

1. Identify the current state: how many environments exist (just production? dev/staging/prod?), what deployment platform is used and what its free-tier/plan limits actually are, and whether CI runs the full test suite on every single push regardless of what changed.
2. Audit against the checklist in `references/pipeline-checklist.md`.
3. Choose a short list of fixes that compound: one environment-separation fix, one CI-cost fix, one platform-limit fix, and one deployment-safety fix.
4. Implement pipeline changes incrementally — a working pipeline that gets faster is safer to iterate on than a pipeline rebuilt from scratch.
5. Verify by simulating the failure: push a change that should only trigger a subset of pipeline stages, check current CI minute usage against the plan's cap, and confirm a deploy to staging doesn't touch production data.

## Audit Output

When asked to review or set up CI/CD, produce:

- **Cost/ceiling read**: What could cause a mid-month CI bill spike, a hit against a free-tier cap, or a silent pipeline stoppage.
- **Environment read**: Whether dev/staging/production are actually separate, or whether "staging" is just a synonym for "production with less traffic."
- **High-impact fixes**: 5-8 ranked changes with expected impact and implementation effort.
- **Implementation prompts**: Plain-English prompts the user can give to an AI coding agent for each fix.
- **Verification notes**: What to check (e.g. "confirm the readme-only change doesn't trigger the full integration suite").

## Implementation Principles

- Every CI/CD platform has a free tier, and every free tier has a trapdoor. The overrun always arrives during a sprint, not between them — check usage weekly, not after the cap is already hit.
- Not every commit needs every test. Path-based/conditional triggers (a README change doesn't need integration tests; a marketing-page change doesn't need a backend build) cut CI minutes without cutting safety.
- Self-hosted runners are a five-second math problem once CI minutes become the bottleneck: a fixed monthly cost for unlimited minutes on your own hardware, versus per-minute overage charges that can be an order of magnitude more expensive at scale.
- Main is always production. If a team pushes directly to main, there is no branching strategy — start with the simplest version that works (short-lived feature branches, pull requests before merge) and only add complexity (staging branches, release branches, hotfix branches) when the pain of the simple version actually shows up.
- Every push to a branch should get its own preview/staging URL before it ever reaches production. Test on that URL, review the diff, then merge — never edit or test directly against the live server.
- Deploy small and often, one change at a time. When a large deploy breaks, you don't know which of 47 changes caused it; when a small deploy breaks, you know instantly.
- Know your rollback procedure before you need it — it should take under two minutes and not require remembering commands under pressure. Practice it once when nothing is on fire.
- Every deployment platform markets "infinite scale," but every platform has a ceiling: execution time limits, concurrent execution caps, payload/bandwidth limits, and function bundle size limits. Read the actual limits page (not the marketing page or the tutorial) before committing to a platform, and match the platform to the workload — front-end-only frameworks on edge/serverless platforms, long-running background jobs on platforms built for persistent processes.

## Reference

Read `references/pipeline-checklist.md` when setting up CI/CD from scratch, debugging a pipeline that's grown slow or expensive, choosing between deployment platforms, or introducing a staging environment to a project that doesn't have one.
