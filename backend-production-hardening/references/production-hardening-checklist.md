# Backend Production Hardening Checklist

Use this checklist to turn transcript-derived production advice into practical backend improvements.

## 1. Multi-Tenant Data Isolation

The single most common production incident: customer A sees customer B's data. It usually isn't a hack, it's a missing WHERE clause or a policy that exists but gets bypassed.

Audit for:

- Tables that store user or tenant data with no row-level security (RLS) policy at all.
- RLS policies that exist, but an API route queries the database using a service-role key or admin connection string, which bypasses every policy.
- Cache keys that don't include the tenant/user ID, so one tenant's cached response gets served to another.
- Application-level tenant filtering (a `WHERE tenant_id = ?` in app code) with no database-level backstop — one missed clause is a full data leak.

Upgrade with:

- RLS as the last line of defense: every table with user data gets at minimum a SELECT policy scoped to the authenticated user/tenant; sensitive tables also get INSERT/UPDATE/DELETE policies.
- Never call the database with a service-role/admin key from a general API route. If elevated access is required for a specific job, scope it narrowly and re-check ownership in code before acting.
- Add a tenant ID (or org ID) column to every table that holds tenant-scoped data, and index it — this is also the natural shard key later.
- Choose the isolation model deliberately: shared schema + RLS (cheapest, fine until one tenant's traffic affects others), schema-per-tenant (better performance isolation, migrations multiply per tenant), or database-per-tenant (full isolation, usually required for healthcare/finance/government compliance).
- Build monitoring that alerts the moment a tenant's query returns rows belonging to another tenant — don't wait for a support ticket to be the detection mechanism.

Implementation prompts:

- "Audit every table in the database for row-level security policies. List any table that stores user or tenant data but has no SELECT policy."
- "Find every server-side code path that uses the service-role/admin database key and verify it manually checks tenant/user ownership before returning data."
- "Add a tenant_id column with an index to every table that doesn't have one, and add an RLS policy that restricts rows to the caller's tenant_id."

Verification: log in as user/tenant A, then try to read or modify a record belonging to user/tenant B by changing an ID in the URL, request body, or JWT claim. It should fail every time, at the API layer and the database layer independently.

## 2. Database Reliability and Scaling

"The app is slow" is almost never solved by a bigger server. Diagnose before you spend money.

Diagnosis order (check in this sequence):

1. **Connection problem vs. capacity problem.** Most slowdowns are connection exhaustion, not compute. If requests queue or time out under load, check active connections against your database's connection limit before doing anything else.
2. **Query problem vs. volume problem.** Use `EXPLAIN ANALYZE` and query-stats tooling (e.g. `pg_stat_statements`) to find the query consuming the most *total* time — that's often a cheap query run 10,000 times a day, not the single slowest query.
3. **Read problem vs. write problem.** ~80% of database operations are reads. A read replica helps reads; it does nothing for write contention. If writes are the bottleneck, the fix is queueing, batching, or background jobs — not more replicas.

Fixes, cheapest first:

- Enable connection pooling (PgBouncer, Supavisor, or your platform's built-in pooler) before adding replicas or upgrading tiers. For serverless functions, use transaction-mode pooling, not session-mode — session mode exhausts connections in minutes because functions spin up and down constantly.
- Set your ORM/client's connection pool size to (allowed DB connections ÷ number of app instances), not a default value.
- Index the columns your slowest and most frequent queries filter or join on.
- Move files (images, PDFs, video) out of database columns and into object storage + a CDN; a database is not a file store.
- Only reach for read replicas, Citus-style sharding, or database-per-tenant once monitoring proves the single instance is the actual bottleneck — don't pre-optimize because of a conference talk.

Implementation prompts:

- "Check whether the app uses connection pooling. If not, add PgBouncer/Supavisor in transaction mode and set the ORM's pool size based on max connections divided by instance count."
- "Run EXPLAIN ANALYZE on the slowest reported query and pg_stat_statements to find the most frequently run expensive query. Fix whichever is cheaper to fix first."

## 3. Backups and Disaster Recovery

A backup nobody has restored is a belief, not a backup.

Decide explicitly (these are business decisions, not defaults):

- **Recovery point objective (RPO)**: how much data can you afford to lose? Daily backups mean up to 24 hours of loss — fine for a blog, not for a payments app. Point-in-time recovery captures every transaction continuously and most managed databases support turning it on.
- **Recovery time objective (RTO)**: how long can you be down while restoring? 10 minutes or 10 hours is a product decision, not a technical afterthought.
- **Location**: backups must live somewhere that survives the primary server dying — cross-region or off-site, never colocated with the primary.

Non-negotiable:

- Actually test the restore, quarterly at minimum: spin up a fresh instance, load the backup, verify the tables and the app both work against it. The worst time to learn your backup is broken is during the outage that needs it.

Implementation prompts:

- "Verify point-in-time recovery or continuous backups are enabled, not just periodic snapshots."
- "Set up a quarterly calendar reminder (or automated job) to restore the latest backup to a scratch environment and verify the app runs against it."

## 4. Caching Without Silent Staleness

Caching is a consistency decision, not just a performance feature. Every cache needs an explicit answer to "what is allowed to be wrong, and for how long."

Classify data before caching it:

- Cache aggressively, time-based: content that rarely changes and costs nothing when stale (company address, blog posts, static config).
- Never cache time-based, only event-driven invalidation: pricing, user permissions, inventory/stock counts, account status — being stale here costs money or creates a security gap (a deactivated user still has access; a customer buys something that's out of stock).

Guard against:

- **Cache stampede**: when a hot key expires, many concurrent requests can hit the database at the same instant. Use request coalescing, locking, or stale-while-revalidate so only one request refreshes the cache.
- **Multi-layer incoherence**: CDN edge, Redis/shared cache, and in-process memory can each hold a different "current" value with a different lifetime. Know which layer is authoritative and who clears which layer when data changes.
- **Missing tenant/user scoping in the cache key** (see section 1) — a cross-tenant data leak can come from the cache layer even if the database query itself was correct.

Implementation prompts:

- "For every cached value in the app, document what triggers invalidation: time-based expiry or an event on write. Flag any pricing, permissions, or inventory data that is only time-based."
- "Add cache hit-rate and database query-count metrics so we can verify the cache is actually reducing load, not just adding a stale-data risk."

## 5. Secrets Management

A leaked key is not an if, it's a when. Treat every key as pre-leaked and design for that.

Audit for:

- API keys or database credentials hard-coded in source, in a `.env` file committed to git, or shipped to the browser in a `NEXT_PUBLIC_`/`VITE_`-prefixed variable that doesn't need to be public.
- Git history: a key removed in a later commit still exists in the diff history forever. If a repo was ever public, assume any key that touched it has been scraped by bots.
- Static credentials with no rotation schedule — same key for months means an unnoticed leak has an unbounded blast radius.

Fixes:

- Move secrets to a dedicated secrets manager (Doppler, Infisical, AWS/GCP Secrets Manager, HashiCorp Vault) with versioning and audit logs, not just environment variables in a dashboard.
- Enable git secret scanning / push protection (GitHub push protection, GitGuardian, TruffleHog, git-secrets) so leaked patterns are blocked before they're committed.
- Scope every key to the minimum it needs: read-only where possible, restricted to specific endpoints/domains/IPs. A Stripe key scoped to "create checkout sessions from this domain" has a tiny blast radius compared to an unscoped key.
- Rotate on a schedule (e.g. every 30-90 days), not only after an incident. Use dual-key rotation (issue new key, deploy it, verify traffic, then revoke the old key) for zero-downtime rotation.
- Any key that was ever in front-end code or git history should be treated as already compromised and rotated immediately, regardless of whether you can prove it leaked.

Implementation prompts:

- "Search the entire git history, not just current files, for API keys, database URLs, and tokens. Rotate anything found, even if it was later removed."
- "Move all secrets currently in .env files to [secrets manager], and verify the app fetches them at runtime instead of bundling them at build/deploy time."
- "Enable GitHub push protection or an equivalent pre-commit secret scanner on this repo."

## 6. Webhooks and Payment Safety

Stripe (or any payment/webhook provider) solved PCI and tokenization. Everything after the webhook fires is the app's responsibility.

Non-negotiable rules:

- **Verify the webhook signature before acting on it.** Never trust an unauthenticated POST to a webhook endpoint.
- **Idempotency.** Every webhook can be delivered more than once (retries, network hiccups). Attach or check an idempotency key so the same event never double-charges, double-fulfills, or double-sends an email.
- **Don't return `200 OK` unless the business logic actually succeeded.** A handler that catches an internal error and still returns 200 tells the provider "all good," so it never retries — and the failure becomes invisible to every dashboard that only watches HTTP status codes.
- **Chain the full business workflow off a verified event**: mark invoice paid, activate subscription/access, update CRM, send receipt — a charge that clears but doesn't trigger the rest of the workflow is a support ticket waiting to happen.
- **Use a dead-letter queue** for events where the response was 200 but the downstream business logic actually failed, so they're queued for retry/investigation instead of silently vanishing.

Implementation prompts:

- "Verify every webhook handler checks the provider's signature before processing the payload."
- "Add an idempotency key check to every webhook handler so a retried event cannot create a duplicate charge, order, or email."
- "Audit webhook handlers for any code path that catches an error internally but still returns 200 to the provider — those failures are invisible today."

Verification: manually replay the same webhook event twice and confirm only one action occurs. Force a downstream failure (e.g. database write error) after signature verification succeeds and confirm it does NOT return 200.

## 7. Rate Limiting and Abuse Protection

An unprotected endpoint that calls a paid API (AI models, SMS, email) is an open invoice for anyone who finds the URL.

Layer rate limiting instead of relying on one flat cap:

- **Hard limits**: a fixed request cap per window as a safety net against abuse.
- **Adaptive limits**: tighten automatically under system load, loosen when healthy (token bucket / sliding window).
- **Tiered limits as pricing architecture**: free/pro/enterprise tiers double as your monetization lever, not just an abuse filter.

For AI/LLM endpoints specifically:

- Put an API gateway in front of every AI endpoint that requires a valid key before any request reaches the model.
- Validate payload size and reject oversized context windows before they reach an expensive model tier.
- Track spend per user/tenant and enforce daily/monthly caps at the gateway, not in application code that can be bypassed.

Implementation prompts:

- "Add rate limiting to authentication endpoints, AI/LLM endpoints, and any public data endpoint, with stricter limits on the expensive ones."
- "Add per-user token/request spend tracking to every AI endpoint and enforce a daily cap before the request reaches the model."

## 8. Production Observability

If you can't answer "how would we find out this broke" in under a minute, this is the gap to close first.

The three pillars, correlated (not siloed):

- **Logs** (what happened) — structured, not sentence-based: timestamp, severity, request/correlation ID, user ID, action. A correlation ID that threads through every service turns a multi-hour debugging session into a single query.
- **Metrics** (how often, how fast) — both infra metrics (CPU, memory, uptime) and business metrics (signups/hour, payments/hour, checkout completions/hour). A webhook can fail silently while every infra metric stays green — only a business-metric alert catches that.
- **Traces** (where in the request it happened) — connects logs and metrics across service boundaries.

Minimum production setup:

- External uptime/health checks from multiple regions (not the server asking itself if it's healthy).
- Error tracking (e.g. Sentry) capturing every unhandled exception, front-end and back-end, grouped by frequency.
- Synthetic transactions: automatically run the critical path (signup → checkout → payment → confirmation) every few minutes so failures are caught before a customer hits them.
- Alerts on business metrics, not just infra metrics — "payments dropped to zero while signups stayed normal" is a real, common, silent-failure signature.
- A documented incident runbook: what to check first (hosting dashboard, database, deploy logs, rollback), written while calm so it can be followed under pressure.

Implementation prompts:

- "Add structured logging with a correlation/request ID that threads through every service call, replacing ad hoc console.log statements."
- "Add error tracking (e.g. Sentry) to both front-end and back-end, and set up an external uptime check from a different region than the primary server."
- "Add an alert on business metrics (payments/hour, signups/hour) in addition to existing infra alerts, so a webhook that silently swallows errors is still caught."

## 9. Deployment Safety

Ship in a way that limits blast radius when something breaks, because something eventually will.

- Deploy behind a canary: route a small percentage of traffic to the new version, watch error rates for a fixed window, auto-promote or auto-rollback based on a threshold — don't cut over 100% of users at once.
- Gate risky features behind feature flags so a bad feature can be killed without a code rollback.
- Know the rollback procedure before you need it: it should take under two minutes and not require remembering commands under pressure.
- Test schema migrations against a branched/staging copy of production data before running them against the real database.

Implementation prompts:

- "Set up a canary or gradual rollout so new deploys go to a small percentage of traffic first, with an automatic rollback if error rates spike."
- "Document and test the rollback procedure for this app so it takes under two minutes to execute."

## 10. Prioritization Matrix

If time is limited, harden in this order:

1. **Tenant/data isolation** — a cross-tenant leak is the fastest way to end a company; fix RLS and service-key bypasses first.
2. **Secrets** — rotate and move anything ever exposed in git history or front-end code; this is often a five-minute fix with a huge blast-radius reduction.
3. **Backups tested with an actual restore** — cheap insurance against total data loss.
4. **Webhook idempotency and signature verification** — protects revenue and customer trust directly.
5. **Connection pooling** — the single highest-leverage fix for "the app is slow/falling over" at real user counts.
6. **Business-metric alerting** — closes the gap between "something broke" and "a customer told us."
7. **Cache invalidation policy** — prevents silent stale-data bugs once traffic and caching layers grow.
8. **Deployment safety (canary/rollback)** — reduces the cost of every future change.

## 11. Red Flags

Treat these as signals the backend is not production-ready:

- Any table with user/tenant data that has no RLS policy, or has one that's bypassed by a service-role key elsewhere in the code.
- A `.env` file with real secrets committed to git, at any point in history.
- No connection pooling on a database used by more than a handful of concurrent users.
- A backup strategy nobody has ever restored from.
- A webhook handler with no idempotency check and no signature verification.
- Monitoring that only watches infra metrics (CPU, uptime) with nothing watching business metrics (payments, signups).
- Cache invalidation strategy for pricing, permissions, or inventory that is purely time-based.
- A rollback plan that exists only in someone's memory.
- Front-end code making business-logic decisions (pricing, feature gating, role checks) that the API doesn't independently enforce.
