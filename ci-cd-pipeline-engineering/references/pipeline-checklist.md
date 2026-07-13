# CI/CD Pipeline Engineering Checklist

Use this checklist to turn transcript-derived CI/CD and deployment advice into practical fixes.

## 1. CI Minutes and Cost Management

Free-tier CI minutes (e.g. GitHub Actions' 2,000/month) scale with team size and pipeline complexity faster than most teams expect — a solo project using 80 minutes can hit the cap within weeks of adding a second team member, integration tests, and a staging/deploy step.

Audit for:

- No visibility into current CI minute usage until the pipeline actually stops running.
- Every commit running every test (unit, integration, lint, full build) regardless of what files changed.
- No self-hosted runner evaluated despite consistently high CI usage.

Fix:

- Check CI minute consumption weekly (most platforms show this in settings), and set a team alert at 75% of the monthly allocation — this gives time to optimize before the pipeline stops running mid-sprint.
- Add path-based/conditional triggers: a change to docs/README doesn't need integration tests; a change to the marketing site doesn't need the backend build. Run only the pipeline stages relevant to what changed.
- Evaluate self-hosted runners once usage is consistently high: a fixed monthly cost (e.g. ~$20/month for a small server) running unlimited minutes is often dramatically cheaper than pay-as-you-go overage charges (which can run several dollars per extra minute on some runner types), at the cost of managing the runner yourself.
- Break pipeline stages into parallel lanes where possible (unit tests, integration tests, and linting don't need to run sequentially) — a "45 minute pipeline" is often nine sequential steps that could run concurrently in a fraction of the time.
- Cache dependencies between builds (e.g. node_modules) instead of reinstalling from scratch on every run.

Implementation prompts:

- "Add path-based triggers to the CI pipeline so documentation-only or marketing-page-only changes skip the integration test and backend build stages."
- "Set up a weekly check (or automated alert) on CI minute usage at 75% of the monthly cap."
- "Break the test suite into parallel jobs (unit, integration, lint) instead of running them sequentially, and enable dependency caching between runs."

## 2. Environment Separation

A huge share of "it broke in production" incidents trace back to development, staging, and production all being the same environment — often literally the developer's laptop.

Audit for:

- Testing new features directly against the production database.
- No staging URL that mirrors production before a change goes live.
- Schema/database migrations run directly against production with no test on a copy first.

Fix:

- Set up three real environments: development (local, or an ephemeral branch that resets regularly), staging (mirrors production, used for review and demos), and production (untouchable except through the deploy pipeline).
- Adopt a simple branch strategy first: `main` = production, a `dev` branch = working/staging area. Never push directly to `main`. Work and break things in `dev`, merge to `main` only when verified.
- Use preview deployments per branch (Vercel, Netlify, and most modern platforms support this natively and for free) — every pushed branch gets its own live URL to test against before merging.
- Test schema changes on a branched copy of the database (Neon and PlanetScale both support database branching) before running them against production.
- Only adopt a more complex branching model (Git Flow with release/hotfix branches) once the pain of the simple model actually shows up — most teams over-adopt complexity before they need it and spend more time managing branches than writing code.

Implementation prompts:

- "Set up a dev branch and a main branch, configure preview deployments per branch, and document that all work happens in dev with main reserved for production-ready merges."
- "Set up database branching (or an equivalent staging copy) so schema migrations are tested before running against the production database."

## 3. Deployment Safety

Deploying is a gamble unless every deploy is small, previewed, and reversible.

Audit for:

- Large deploys bundling many unrelated changes, making it hard to know what caused a break.
- No documented or tested rollback procedure.
- Deploys going to 100% of traffic instantly with no canary or gradual rollout.

Fix:

- Ship one change at a time. Small, frequent deploys make it obvious what broke when something breaks.
- Preview every deploy before it's live: most platforms build a preview URL automatically on push — click through it, verify the change, then promote to production.
- Document and test the rollback procedure before it's needed: it should take under two minutes and not require remembering commands under pressure.
- For teams with meaningful traffic, use canary/gradual rollout (route a small percentage of traffic to the new version, watch error rates for a fixed window, auto-promote or auto-rollback based on a threshold) instead of an all-at-once cutover.
- Use feature flags to decouple "deployed" from "live" — ship code disabled by default, turn it on for a subset of users, and kill it instantly (without a rollback) if something's wrong.

Implementation prompts:

- "Set up a canary/gradual rollout for deploys so new versions go to a small percentage of traffic first, with an automatic rollback if error rates spike within a fixed monitoring window."
- "Document the exact rollback steps for this app's current deployment platform, and time how long it actually takes to execute."

## 4. Platform Limits

Every deployment platform markets "scales automatically," but every platform has a ceiling — and hitting it on launch day is entirely avoidable by reading the limits page before choosing the platform.

Common limits to check per platform before committing:

- **Execution/function timeout**: e.g. Vercel serverless functions time out after a fixed number of seconds depending on plan. A feature that needs longer (AI processing, file generation, batch jobs) will fail silently — the user just sees an endless spinner, not a useful error.
- **Concurrent execution ceiling**: platforms cap how many function instances can run simultaneously. Beyond that, requests queue, cold-start, or error out — this is a platform limit, not a code bug, and shows up first under real launch-day traffic.
- **Payload/bandwidth limits**: request body size caps (e.g. a few megabytes) will silently fail a large file upload; a bandwidth cap can be burned through quickly by an image-heavy app.
- **Function bundle size limits**: a serverless function that exceeds the platform's package size ceiling can fail to deploy, sometimes without an obvious error message.

Fix:

- Before choosing or committing to a platform, read its actual limits documentation page (not the marketing page, not a tutorial) for execution time, concurrency, payload size, and bundle size.
- Match the platform to the workload: front-end frameworks and short edge functions fit serverless/edge platforms well; long-running background jobs, persistent connections, and heavier compute fit platforms built for persistent processes (a small VPS, Railway, Render, Fly.io).
- If a feature's run time exceeds the platform's execution ceiling, move it to a background job with a webhook/polling callback instead of trying to force it into a synchronous request.
- Request a concurrency or rate-limit increase from the platform before launch day if projected traffic is close to the default ceiling — not during the traffic spike.

Implementation prompts:

- "Check this platform's documented execution timeout, concurrency limit, and payload size limit, and compare them against what this app's slowest feature actually needs."
- "If [feature] exceeds the platform's execution time limit, move it to a background job with a callback instead of a synchronous request."

## 5. Prioritization Matrix

If time is limited, fix in this order:

1. **Environment separation (dev/staging/prod with real branches)** — the single highest-leverage fix; prevents most "we tested it and it still broke" incidents.
2. **Rollback procedure documented and tested** — cheap insurance that turns a bad deploy from a crisis into a two-minute fix.
3. **Platform limits read and matched to workload** — a five-minute read that prevents an entire category of launch-day surprises.
4. **CI minute monitoring with an alert threshold** — prevents the pipeline from silently stopping mid-sprint.
5. **Path-based/conditional CI triggers** — meaningful cost and speed win once the test suite has grown.
6. **Canary/gradual rollout** — worth adding once there's real traffic where a bad deploy would hit meaningful numbers of users at once.

## 6. Red Flags

Treat these as signals the pipeline is not production-ready:

- No staging environment; testing happens directly against production.
- Direct pushes to `main`/production with no pull request or review step.
- Nobody has checked CI minute usage until the pipeline stopped running.
- Full test suite runs on every commit regardless of what files changed.
- No documented rollback procedure, or one that's never been tested.
- A deployment platform chosen without reading its execution time, concurrency, or payload limits.
- Deploys bundling many unrelated changes at once, making post-incident diagnosis slow.
- Schema/database migrations run directly against production with no tested copy first.
