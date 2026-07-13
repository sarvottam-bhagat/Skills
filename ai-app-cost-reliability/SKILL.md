---
name: ai-app-cost-reliability
description: "Make an AI/LLM-powered feature or app cost-controlled and reliable in production: model routing/tiering by task complexity, semantic and prompt caching, eval-based testing instead of exact-match assertions, output validation with retry and graceful degradation, per-user spend caps, and short-term/long-term agent memory design. Use when Codex is building, testing, deploying, or reviewing any app that calls an LLM API (OpenAI, Anthropic, Gemini, etc.), especially when the user is worried about API bills, flaky AI outputs, hallucinations reaching users, CI tests that flake on AI responses, or wants a multi-agent architecture."
---

# AI App Cost & Reliability

## Overview

Use this skill to close the gap between "the AI feature works in my testing" and "the AI feature survives 1,000 real users without a runaway bill or a hallucination reaching production." LLM-powered features fail in a different way than normal backend code: the same input can produce a different output, a bug in a loop can burn thousands of dollars in an hour, an unauthenticated endpoint calling an expensive model is an open invoice, and a CI pipeline built for deterministic code flakes constantly on non-deterministic AI responses.

## Workflow

1. Identify what the AI feature actually does: which model(s) it calls, whether every request needs the most expensive model, whether the endpoint is publicly reachable, and whether a bad output can reach a user unfiltered.
2. Audit the current implementation against the checklist in `references/ai-cost-reliability-checklist.md`.
3. Choose a short list of fixes that compound: one cost fix (routing/caching), one reliability fix (validation/retry/degrade), one testing fix (eval-based CI), and one abuse-protection fix (gateway + spend caps).
4. Implement cost and reliability controls as infrastructure, not an afterthought bolted on after the first surprise bill.
5. Verify by simulating the failure: send a request that should get rejected/downgraded, force the model to return malformed output, replay the same query twice to confirm caching works, and check what a runaway loop would actually cost per hour.

## Audit Output

When asked to review or harden an AI-powered feature, produce:

- **Cost read**: What could turn into a runaway bill — unbounded loops, no per-user caps, every request hitting the most expensive model, no caching on repeated queries.
- **Reliability read**: What happens when the model hallucinates, times out, or returns malformed output — does the user ever see a raw error or garbage response?
- **High-impact fixes**: 5-8 ranked changes with expected cost/reliability impact and implementation effort.
- **Implementation prompts**: Plain-English prompts the user can give to an AI coding agent for each fix.
- **Verification notes**: The specific failure or cost scenario to simulate to prove the fix works.

## Implementation Principles

- Not every request needs the most expensive model. Classify by complexity (token count, task type, reasoning depth) and route simple/repetitive tasks to a cheap model, reserving the expensive one for what actually needs it.
- Cache before you call. If the same or a semantically similar question was answered recently, don't call the model again — check a semantic cache first, and use prompt caching for repeated system prompts/context.
- Never let raw AI output reach a user unchecked. Validate against an expected schema/length/content policy, retry once or twice with the failure reason fed back into the prompt, and degrade gracefully (simpler model, canned response, human handoff) rather than showing a blank screen or a hallucination.
- Treat every AI endpoint like a metered utility with no default limit. Put an API gateway in front of it, require auth, validate payload size before it reaches the model, and enforce per-user/per-tier spend caps at the gateway — not in application code that can be skipped.
- CI built for deterministic code will flake constantly on AI features. Replace exact-string assertions with evaluation-based scoring (accuracy, tone, schema compliance) against a pass/fail threshold, and build the regression suite from real failures users hit, not synthetic cases.
- Budget the bill before you ship, not after. Estimate token cost per deployment/feature change, flag anything that doubles the average cost, and set hard spend ceilings with alerts at 50/75/90% — a misconfigured loop calling an expensive model can burn hundreds of dollars an hour.
- For multi-agent systems, start with a single orchestrator (one controller agent calling sub-agents and synthesizing results) by default — it's simpler to debug. Only move to a decentralized "conductor" pattern (agents passing work to each other) for emergent workflows, and only after data shows the orchestrator is genuinely the bottleneck.
- Agent memory should be selective, not exhaustive. Short-term memory as a rolling/summarized conversation buffer; long-term memory as a vector store retrieved by relevance, storing only what changes the agent's future behavior — not every casual aside.

## Reference

Read `references/ai-cost-reliability-checklist.md` when doing a detailed audit of an AI feature, setting up model routing, designing an eval/testing pipeline for AI outputs, adding rate limiting to an AI endpoint, or architecting a multi-agent system.
