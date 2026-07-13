---
name: backend-production-hardening
description: "Harden a web or mobile app's backend so it survives real users, real data, and real attackers: multi-tenant data isolation (RLS, service-role key leaks), database reliability (connection pooling, backups/restore testing, cache invalidation, migrations), secrets management, webhook idempotency and payment safety, and production observability (structured logging, error tracking, business-metric alerting). Use when Codex is building, auditing, or shipping a Supabase/Postgres/Firebase-backed app, an API, a payment integration, or any multi-user system, especially when the user wants to go from a working prototype to something production-ready, secure, or safe to launch."
---

# Backend Production Hardening

## Overview

Use this skill to close the gap between "it works on my laptop with my data" and "it survives production with other people's data and money." Most AI-generated backends work perfectly in the demo and fail in one of a small number of predictable ways: a security policy that looks correct but is bypassed by application code, a database that runs out of connections instead of compute, a cache that goes stale silently, a webhook that gets charged twice, or a failure that nobody finds out about until a customer emails support.

## Workflow

1. Identify what the app touches: does it store other users' data (multi-tenancy), does it move money (payments/webhooks), does it call external APIs (rate limits, retries), and where does its database live (Supabase, Postgres, Firebase, etc.)?
2. Audit the current backend against the checklist in `references/production-hardening-checklist.md`.
3. Choose a short list of fixes that compound: one data-isolation fix, one database-reliability fix, one secrets fix, one payment/webhook fix (if applicable), and one observability fix.
4. Implement hardening as architecture, not afterthought. Enforce every rule at the layer that cannot be bypassed (database, not UI; API, not client).
5. Verify by trying to break it: attempt cross-tenant access, kill the database mid-request, replay a webhook twice, restore a backup, and check whether an on-call human would find out about a silent failure before a customer does.

## Audit Output

When asked to review or harden a backend, produce:

- **Risk read**: What could leak another user's data, lose money silently, lose data permanently, or go down without anyone noticing.
- **High-impact fixes**: 5-8 ranked changes with expected impact and implementation effort.
- **Implementation prompts**: Plain-English prompts the user can give to an AI coding agent for each fix.
- **Verification notes**: The specific attack or failure to simulate to prove the fix works (not just that the code compiles).

## Implementation Principles

- Enforce security at the layer that cannot be bypassed. A hidden button is not access control; an unchecked API route is not protected by a pretty UI. Row-level security (RLS) at the database is the last line of defense — but a service-role key or admin connection string bypasses RLS entirely, so any server code path that uses it must re-implement the tenant/ownership check itself.
- Treat "works for one user" and "works for one thousand users" as two different systems. The failure mode at scale is almost always connections, not compute — diagnose in this order: connection problem vs. capacity, query problem vs. volume, read problem vs. write problem.
- A backup that has never been restored is not a backup, it's a hope. Schedule quarterly restore tests, not just backup jobs.
- Caching without an invalidation plan is a second, wrong source of truth. Decide per data type what is allowed to be stale, for how long, and who clears it when it changes (event-driven for pricing/permissions/inventory, time-based for content that doesn't matter).
- Every webhook can fire more than once. Idempotency keys are not optional — one event must produce one action, even under retries.
- A `200 OK` response is not proof the business logic succeeded. Alert on business metrics (payments/hour, signups/hour) in addition to infra metrics (CPU, uptime), because a webhook can silently swallow failures while every dashboard stays green.
- Secrets are not "safe in `.env`" — they're safe in a secrets manager with rotation, scoping, and audit logs. Assume any key ever committed to git, even briefly, has already leaked; rotate it, don't just delete it.
- If you can't find out about an outage or a data leak faster than a customer can, you don't have monitoring, you have alert theater. Structured logs with correlation IDs, error tracking, and external uptime checks are the minimum, not the finish line.
- Multi-tenancy is a business decision, not a technical default. Match the isolation model (shared schema + RLS, schema-per-tenant, or database-per-tenant) to what the contract and the compliance requirements actually demand.

## Reference

Read `references/production-hardening-checklist.md` when doing a detailed backend audit, designing multi-tenant data access, adding payments/webhooks, choosing a database or ORM, setting up monitoring, or preparing for a production launch.
