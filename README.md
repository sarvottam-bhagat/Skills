# Skills

A personal collection of Codex skills.

## Available Skills

### mobile-app-ux-polish

Improves mobile app UX, retention, and perceived quality with practical polish patterns for iOS, Android, React Native, SwiftUI, Flutter, and mobile-first app experiences.

Use it when building, auditing, redesigning, or prompting improvements for apps that should feel more premium, memorable, interactive, and polished.

Includes:

- `SKILL.md` with the skill workflow and implementation principles.
- `references/ux-polish-checklist.md` with a detailed checklist for motion, haptics, visual identity, onboarding, retention surfaces, and App Store screenshots.

### backend-production-hardening

Hardens a web or mobile app's backend so it survives real users, real data, and real attackers.

Use it when building, auditing, or shipping a Supabase/Postgres/Firebase-backed app, an API, a payment integration, or any multi-user system that needs to go from a working prototype to production-ready.

Includes:

- `SKILL.md` with the skill workflow and implementation principles.
- `references/production-hardening-checklist.md` with a detailed checklist for multi-tenant data isolation (RLS, service-role key leaks), database reliability (connection pooling, backups/restore testing), caching invalidation, secrets management, webhook/payment idempotency, rate limiting, and production observability.

### ai-app-cost-reliability

Makes an AI/LLM-powered feature or app cost-controlled and reliable in production.

Use it when building, testing, deploying, or reviewing any app that calls an LLM API (OpenAI, Anthropic, Gemini, etc.), especially when worried about API bills, flaky AI outputs, hallucinations reaching users, CI tests that flake on AI responses, or when designing a multi-agent architecture.

Includes:

- `SKILL.md` with the skill workflow and implementation principles.
- `references/ai-cost-reliability-checklist.md` with a detailed checklist for model routing/tiering, semantic and prompt caching, eval-based testing, output validation/retry/degrade, spend caps, and agent memory design.

### saas-legal-compliance-readiness

Assesses and closes the legal/compliance gaps that put a SaaS founder personally and financially at risk.

Use it when preparing to launch a product that collects user data, talking to an enterprise prospect who asked for SOC 2, submitting to the App Store, or asking about privacy policy, terms of service, data deletion, compliance, or business liability. This is a first-pass gap-finder, not legal advice — it clearly flags what to self-serve versus what needs a lawyer, broker, or auditor.

Includes:

- `SKILL.md` with the skill workflow and implementation principles.
- `references/legal-compliance-checklist.md` with a detailed checklist for privacy policy accuracy, GDPR/CCPA, SOC 2 readiness, cyber liability insurance, platform ToS liability caps, and App Store submission risk.

### ci-cd-pipeline-engineering

Designs and fixes CI/CD pipelines and deployment environments so they scale with a team instead of collapsing on the worst possible day.

Use it when setting up CI/CD, choosing or migrating a deployment platform, debugging a pipeline that got slower or hit a paywall, or when a team is still deploying straight to production with no staging environment.

Includes:

- `SKILL.md` with the skill workflow and implementation principles.
- `references/pipeline-checklist.md` with a detailed checklist for CI minutes cost management, dev/staging/production environment separation, deployment safety (canary/rollback), and platform deployment limits.

### ai-generated-code-ownership

Turns AI-generated ("vibe-coded") code into code the team actually owns and can maintain.

Use it when shipping, handing off, or taking a security/code-quality pass on a codebase substantially built by an AI coding assistant, or when asking how to review, clean up, or make sense of AI-generated code before merging or launching.

Includes:

- `SKILL.md` with the skill workflow and implementation principles.
- `references/code-ownership-checklist.md` with a detailed checklist for reading/renaming AI-generated code, deleting unused scaffolding, auditing dependencies and unvetted community resources, and documenting architecture decisions for handoff.

## Use In A Project

Once installed, use a skill in any project by asking Codex something like:

```text
Use the mobile-app-ux-polish skill to audit this app and suggest high-impact UX polish improvements.
```

```text
Use the backend-production-hardening skill to audit this backend for security and reliability gaps before launch.
```

```text
Use the ai-app-cost-reliability skill to review this AI feature for cost and output reliability issues.
```

```text
Use the saas-legal-compliance-readiness skill to check this app's privacy policy and compliance gaps before launch.
```

```text
Use the ci-cd-pipeline-engineering skill to review our deployment pipeline and environment setup.
```

```text
Use the ai-generated-code-ownership skill to review this AI-generated codebase before we ship it.
```
