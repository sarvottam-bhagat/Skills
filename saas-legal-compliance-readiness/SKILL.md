---
name: saas-legal-compliance-readiness
description: "Assess and close the legal/compliance gaps that put a SaaS founder personally and financially at risk: privacy policy accuracy vs. actual data collection, GDPR/CCPA data deletion and consent, SOC 2 readiness for enterprise deals, cyber liability insurance, platform Terms of Service liability caps (Supabase/Vercel/Stripe), and App Store privacy/AI-transparency rejection risks. Use when the user is preparing to launch a product that collects user data, is talking to an enterprise prospect who asked for SOC 2, is about to submit to the App Store, or asks about privacy policy, terms of service, data deletion, compliance, or business liability for an app they're building."
---

# SaaS Legal & Compliance Readiness

## Overview

Use this skill when a product is technically ready to ship but the business protecting it is not. An AI coding agent can build the product and even draft a privacy policy, but it won't tell you what's legally required — most builders don't find out what those requirements are until the first incident teaches them. This skill is not a substitute for a lawyer; it's a way to surface the gaps early enough that getting real legal review is a cheap, calm decision instead of an expensive, panicked one.

## Workflow

1. Identify what the product actually does with data: what personal data it collects, where it stores it, whether it processes payments, whether it targets enterprise customers, and which regions its users are in (EU, California, etc.).
2. Audit against the checklist in `references/legal-compliance-checklist.md`.
3. Flag any place where the privacy policy, Terms of Service, or actual product behavior disagree with each other — that mismatch is the most common and most avoidable source of liability.
4. Produce a plain-language risk read and a prioritized action list; recommend real legal review for anything with real financial or regulatory exposure. This skill helps you find the gaps and draft first passes — it does not replace a lawyer or insurance broker.
5. Revisit before any material change: adding a new region, adding payments, adding an enterprise/SSO tier, or changing what data is collected.

## Audit Output

When asked to review legal/compliance readiness, produce:

- **Mismatch read**: Where the privacy policy or ToS says one thing and the product actually does another (e.g. policy says "we don't sell data" while analytics scripts send it to four ad networks).
- **Exposure read**: What could trigger a breach notification law, a regulatory fine, an uninsured personal liability, or an enterprise deal stalling out.
- **High-impact fixes**: 5-8 ranked changes with expected risk reduction and effort, clearly marked as "documentation/process fix" vs. "needs a lawyer/insurance broker."
- **Verification notes**: What to actually test (e.g. "click delete my account and verify data is gone from the database, backups, and analytics — not just the UI").

## Implementation Principles

- A privacy policy is not a template you copy from the internet — it's a description of what your app actually does. If it says something the code doesn't do (or omits something the code does do), that gap is the liability, not the paperwork itself.
- Data deletion is an architectural decision, not a button. "Delete my account" must actually remove or anonymize data across the primary database, backups, logs, and any third-party analytics/email tools it was copied to — not just hide it in the UI.
- Consent must be informed, not a checkbox. A banner that says "we use cookies" with an "accept" button is not GDPR-compliant consent if the user doesn't understand what they're agreeing to.
- Every platform you build on (Supabase, Vercel, Stripe, AWS, etc.) has a Terms of Service that caps their liability — often at what you paid them last month. If their outage or data loss costs you $10,000 and you paid them $25, their liability is $25; the rest is yours. Know this before you assume a managed platform "handles" your risk.
- Cyber liability insurance is cheap relative to the downside: a few hundred dollars a year vs. personal liability for notification costs, legal fees, and damages after a breach on an uninsured platform. Most underwriters also require basic security practices (access controls, encryption, vulnerability scans) before they'll issue a policy — so this connects directly to security work, it isn't separate from it.
- SOC 2 is a filter, not a wall. Type 1 (controls exist, judged at a point in time) can realistically be reached in ~60 days with the right automation (Vanta, Drata, Secureframe connected to your infra); Type 2 (controls operated effectively over 6-12 months) requires sustained evidence. If enterprise deals are the goal, start the evidence trail — access reviews, patch cadence, backup restore tests — long before a prospect asks for the report.
- App Store review is getting stricter specifically because of AI-generated apps: privacy manifests are mandatory, AI-transparency disclosures are required if the app uses AI and doesn't declare how it handles data, and the "happy path only" testing that vibe-coded apps tend to have is exactly what triggers rejections on edge cases.
- This skill produces a first-pass assessment and draft documents. It does not replace an actual lawyer, insurance broker, or compliance auditor — flag clearly which items are "safe to self-serve" (e.g. running a data audit, wiring up a deletion pipeline) vs. "get a professional" (e.g. finalizing ToS liability language, buying a specific insurance policy, passing a real SOC 2 audit).

## Reference

Read `references/legal-compliance-checklist.md` when preparing for launch, responding to an enterprise prospect's compliance questions, auditing a privacy policy against actual app behavior, preparing an App Store submission, or evaluating platform/vendor liability exposure.
