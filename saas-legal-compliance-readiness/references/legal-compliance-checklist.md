# SaaS Legal & Compliance Readiness Checklist

Use this checklist to turn transcript-derived legal/business advice into practical pre-launch and ongoing compliance work. This is a first-pass assessment tool, not legal advice — items that need a lawyer, broker, or auditor are marked clearly.

## 1. Privacy Policy Accuracy

The most common gap: the privacy policy describes a generic app, not this app.

Audit for:

- A privacy policy copied from a template or generator that references things the app doesn't do (e.g. "cookies" only) while omitting things it does do (e.g. analytics scripts, third-party AI model calls, email marketing tools).
- Data actually collected but never listed: IP addresses in logs, device fingerprints in analytics, precise location from API calls, data sent to third-party analytics/ad services.
- A claim like "we do not sell user data" while analytics or marketing tools are configured to share data with third parties that meet the legal definition of a "sale" in some jurisdictions.

Fix:

- List every category of personal data the app actually touches and where it goes (self-hosted DB, analytics vendor, email provider, AI model provider, ad network).
- Update the privacy policy to match that list exactly, not a generic template.
- Re-audit any time a new third-party script, SDK, or integration is added — that's the most common source of drift between policy and reality.

Implementation prompts:

- "List every third-party service this app sends any user data to (analytics, email, AI APIs, ad networks, error tracking) and what data each one receives."
- "Compare that list against the current privacy policy and flag every mismatch — data collected but not disclosed, or claims made that the data flow contradicts."

Get a professional for: final policy language if the product handles sensitive categories (health, financial, children's data) or operates in multiple regulatory regimes.

## 2. GDPR / CCPA Basics

If the app has any EU or California users — which most public web/mobile apps do — these aren't optional.

Audit for:

- No mechanism for a user to request their data be deleted.
- No mechanism for a user to opt out of data sale/sharing (required under CCPA).
- Consent implemented as a single "accept all cookies" banner with no explanation of what's being agreed to.
- No defined data retention period — data kept indefinitely with no policy decision behind it.

Fix:

- Build an actual deletion pipeline, not just a UI button: deleting an account must remove or anonymize the user's data from the primary database, backups (or apply retention limits to backups), logs, and any third-party tool it was copied to (analytics, email marketing, support ticketing).
- Add an explicit opt-out mechanism for data sale/sharing where CCPA applies.
- Write consent language in plain terms describing what is collected and why, not just "we use cookies."
- Decide and document a retention period per data type (e.g. logs retained 90 days, financial records retained per tax law, marketing data retained until opt-out).

Implementation prompts:

- "Add a 'delete my account' flow that removes the user's personal data from the primary database, applies deletion to future backups, and documents what happens to data already in existing backups."
- "Add a data-sale opt-out control if the app shares any data with third-party ad or analytics platforms."

Get a professional for: confirming which specific regulations apply to your user base and data types, and reviewing the final consent/retention language.

## 3. SOC 2 Readiness (If Targeting Enterprise Customers)

The first enterprise prospect will ask for a SOC 2 report. Not having one stalls the deal; having a plan for one keeps it moving.

Audit for:

- No access control review process, no logging of who has access to what.
- No documented incident response procedure.
- No vendor/sub-processor management (who has access to customer data outside your own team).
- No automated evidence collection — everything would have to be manually reconstructed if an auditor asked.

Fix:

- Start SOC 2 Type 1 prep now if enterprise is a real goal: this proves controls are designed correctly at a point in time and can realistically be reached in ~60 days with the right tooling.
- Use a continuous-compliance platform (Vanta, Drata, Secureframe) connected to your infra (cloud provider, GitHub, Google Workspace) to automate evidence collection: access reviews, patch/vulnerability scan cadence, backup restore verification.
- Plan for SOC 2 Type 2 (controls operating effectively over 6-12 months) as the next step once Type 1 evidence trail is established — this is what most large enterprise procurement teams actually require.

Implementation prompts:

- "Set up automated quarterly access reviews and monthly dependency vulnerability scans, and document both as part of a SOC 2 evidence trail."
- "List every sub-processor (hosting, email, analytics, AI providers) that touches customer data, for a vendor management record."

Get a professional for: engaging an actual SOC 2 auditor, and legal review of any data processing agreements with sub-processors.

## 4. Cyber Liability Insurance and Personal Exposure

Handling other people's data without insurance means personal financial exposure, not just company exposure — especially for solo founders and small LLCs.

Audit for:

- No cyber liability insurance in place while the app stores user PII, payment data, or health data.
- No awareness that most underwriters require baseline security practices (access controls, encryption at rest, vulnerability scanning) before they'll issue or price a policy.

Fix:

- Get a cyber liability quote before launch, not after an incident — for most small SaaS platforms this is a modest annual cost relative to the exposure it covers (breach notification, remediation, legal fees, damages).
- Treat the underwriter's required security baseline as a forcing function: if they won't insure you without access controls and encryption, that's a signal those aren't optional regardless of insurance.

Implementation prompts:

- "List the current security controls in place (access control, encryption at rest, vulnerability scanning, backup testing) as a checklist to bring to a cyber liability insurance quote."

Get a professional for: the actual insurance policy — this requires a licensed broker, not a coding agent.

## 5. Platform / Vendor Terms of Service Liability

Every platform a product is built on (Supabase, Vercel, Stripe, AWS, OpenAI/Anthropic, etc.) has terms that cap what they owe you when something goes wrong.

Audit for:

- Assuming a managed platform "handles" data-loss or downtime risk without having read the actual liability cap in its ToS.
- A limitation-of-liability clause that caps the vendor's exposure at what was paid them recently (often startlingly low relative to what a failure could cost the business).

Fix:

- Read the limitation-of-liability section of every critical platform's ToS (hosting, database, payments, AI provider) and note the actual cap.
- Treat that gap (platform's cap vs. your actual exposure) as a risk to cover with your own backups, monitoring, and insurance — not something the vendor absorbs for you.
- Re-check ToS when a vendor changes pricing tiers or terms; caps are often tied to the plan you're on.

Implementation prompts:

- "Summarize the limitation-of-liability clause in [platform]'s Terms of Service and how it compares to what a data loss or outage would actually cost this business."

Get a professional for: negotiating custom liability terms with an enterprise-tier vendor contract, if volume justifies it.

## 6. App Store / Play Store Submission Risk

Review is getting stricter specifically because of AI-generated and vibe-coded apps.

Audit for:

- Missing privacy manifest (Apple) or missing data-safety disclosure (Google Play).
- App uses AI (generation, recommendations, chat) but hasn't disclosed how it handles user data per current AI-transparency requirements.
- App only tested on the happy path — edge cases (empty input, malformed data, poor connectivity) that trigger crashes during review, which read as quality/privacy risk to reviewers.

Fix:

- Complete the privacy manifest / data-safety form accurately against the actual data-collection audit from section 1 — don't leave it as a boilerplate "no data collected" if that's not true.
- If the app uses AI in any user-facing way, add the required AI-transparency disclosure describing what's AI-generated and how input data is used.
- Test and fix edge cases (bad input, slow network, permission denial) before submission — a crash during review is treated as a quality signal, and repeated review cycles cost real time.

Implementation prompts:

- "Fill out the App Store privacy manifest / Play Store data safety form using the actual data-collection audit, not a generic template."
- "Test the app with malformed input, no network, and denied permissions, and fix any crash before submission."

## 7. Prioritization Matrix

If time before launch is limited, work in this order:

1. **Privacy policy vs. actual behavior mismatch** — the fastest, cheapest fix and the most common trigger for complaints/regulatory attention.
2. **Data deletion pipeline** — required by law in most regions with users, and often completely missing in vibe-coded apps.
3. **Cyber liability insurance quote** — cheap relative to uninsured personal exposure; get the quote even if you don't buy immediately, since it reveals your security gaps.
4. **Platform ToS liability caps** — a one-time read that tells you exactly how much risk you're actually carrying versus what you assumed the vendor covers.
5. **SOC 2 evidence trail** — only urgent if enterprise deals are an active goal, but the earlier it starts, the cheaper Type 2 becomes later.
6. **App Store submission prep** — do this right before submission, using the data audit from step 1.

## 8. Red Flags

Treat these as signals the business (not just the product) is not launch-ready:

- A privacy policy that was copied from a generator and never updated to match what the app actually collects or which third parties receive data.
- No way for a user to actually delete their data — "delete account" only hides the UI, data remains in the database/backups/analytics.
- No cyber liability insurance while storing PII, payment data, or health data.
- Nobody on the team has ever read the limitation-of-liability clause of the primary hosting/database/payment platform.
- An enterprise prospect asked for SOC 2 and the answer was "we don't have that" with no remediation plan or timeline.
- An app using AI features with no AI-transparency disclosure in its store listing.
- A submission tested only on the happy path, with no edge-case (bad input, no network, denied permission) testing before review.
