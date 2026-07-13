# AI-Generated Code Ownership Checklist

Use this checklist to turn a "vibe-coded" codebase into one the team actually understands, owns, and can maintain.

## 1. Read and Understand Before Shipping

The core failure mode: code that works but that nobody can explain.

Audit for:

- Files or functions that no one on the team (including the person who prompted them) can describe in one sentence.
- Core flows — authentication, payments, data access — that were never read line-by-line, only tested by clicking through the happy path.

Fix:

- Read every file in the codebase deliberately, not by skimming. If a function's purpose isn't clear in one sentence, stop and understand it before moving on.
- Prioritize reading order by blast radius: auth, payment/billing code, and anything that reads or writes user data first; UI polish and cosmetic components last.
- If a piece of code truly cannot be understood without a full rewrite, treat that as a signal to rewrite it deliberately rather than ship an unreadable dependency on it.

Implementation prompts:

- "Walk through [auth module / payment flow / data access layer] function by function and explain what each one does in one sentence, flagging anything unclear."

## 2. Rename for Clarity

Generic AI-generated names are a symptom of unclaimed code.

Audit for:

- Function and variable names like `processData`, `handleSubmit`, `fetchResults`, `doThing` that describe nothing about what the code actually does.
- Inconsistent naming for the same concept across different files (e.g. `user`, `usr`, `currentUser`, `authUser` all referring to the same thing).

Fix:

- Rename functions and variables to describe what they actually do in this app: `createUserAccount`, `validatePaymentAmount`, `getActiveSubscriptions` instead of generic placeholders.
- Renaming is also a comprehension check — if you can't come up with a specific name, you haven't understood the code yet (see section 1).

Implementation prompts:

- "Find every function with a generic name (process, handle, fetch, do + noun) and rename it to describe its specific behavior in this codebase."

## 3. Delete Unused Code

AI assistants routinely generate more than was asked for: backup functions, unused helpers, speculative abstractions.

Audit for:

- Functions that are never called anywhere in the codebase.
- Imports that are never used.
- Components that are never rendered.
- Configuration options or feature flags for functionality that was never actually built.

Fix:

- Run an unused-code/dead-export check (most languages have a linter or static analysis tool for this) and remove anything with zero call sites.
- Treat "might need it later" as a reason to check version control history later, not a reason to keep dead code in the working tree now — a smaller codebase is safer for the team and for users.

Implementation prompts:

- "Find every exported function, component, and import in the codebase with zero usages elsewhere, and remove them."

## 4. Dependency and Supply-Chain Audit

Every installed package is code you didn't write but are now responsible for defending.

Audit for:

- No recent dependency audit (`npm audit`, `pip-audit`, or equivalent) ever run.
- No committed lock file, meaning different environments can install different versions of the same "pinned" dependency.
- Dependencies that haven't been updated in a long time, especially ones handling security-sensitive functionality (auth, crypto, input parsing).
- No awareness of what the app's transitive dependencies actually are, beyond the handful directly imported.

Fix:

- Run the dependency audit tool for your ecosystem now, and put a recurring calendar reminder (e.g. monthly) to re-run it — fix critical/high findings immediately, schedule the rest.
- Commit the lock file. It exists specifically so every environment installs identical versions; don't delete or ignore it.
- For any dependency handling something security-sensitive, check when it was last maintained and whether it has known CVEs before trusting it in production.
- Reduce surface area deliberately: before adding a new package for something trivial, ask whether it's simpler to write the ~20 lines yourself rather than pull in a dependency with its own transitive tree.

Implementation prompts:

- "Run the dependency audit tool for this project's ecosystem and report critical/high severity findings."
- "Check whether the lock file is committed to version control; if not, commit it."
- "List any dependency that hasn't been updated in over a year and flag which ones handle auth, crypto, or input parsing."

## 5. Unvetted Community Resources (New Supply-Chain Risk)

The supply chain now includes more than installed packages — it includes any prompt, downloaded "skill" file, shared config, or copy-pasted system instruction that an AI coding assistant reads and acts on.

Audit for:

- Prompts, skill files, or configuration copied from a community source (a forum post, a shared repo, a downloaded template) without being read in full.
- Any resource an AI assistant is instructed to treat as trusted context, without a review step.

Fix:

- Apply the same discipline used for code dependencies: if you cannot read and understand every line of an external prompt/config/skill file, don't feed it to an AI assistant that has access to production systems or secrets.
- Isolate unvetted resources from production entirely — treat them the same as an unreviewed pull request from a stranger.
- Have a secrets-rotation plan ready to execute immediately if an external resource is later discovered to be compromised or malicious.

Implementation prompts:

- "List every external prompt, skill file, or shared configuration this project uses that wasn't authored by the team, and confirm each one has been read in full."

## 6. Documentation for Handoff

A project only one person (or one AI session) can run is not a maintainable product.

Audit for:

- No record of why key architecture decisions were made (why this database, why this framework, why this third-party service).
- Environment setup instructions that skip steps the original builder "stopped noticing," making local setup take much longer than documented.
- No documentation of failure modes: what happens when the database goes down, when a rate limit is hit, when a third-party API times out.

Fix:

- Write one paragraph per major architecture decision: what was chosen, why, and what the alternative would have cost.
- Write environment setup instructions by actually following them on a clean machine/environment, not from memory.
- Document known failure modes and what the first response should be for each — this matters most exactly when things are already breaking and nobody has time to investigate from scratch.

Implementation prompts:

- "Generate a one-paragraph explanation for each major architecture decision in this codebase (database choice, framework choice, key third-party integrations)."
- "Write step-by-step local environment setup instructions and verify them on a clean checkout."
- "Document what happens when [the database goes down / a rate limit is hit / a third-party API times out] and what the first thing to check is."

## 7. Prioritization Matrix

If time is limited, work in this order:

1. **Read and understand auth, payments, and data-access code** — highest blast radius if misunderstood or wrong.
2. **Dependency audit and lock file** — fast to run, catches known vulnerabilities immediately.
3. **Delete unused code** — reduces attack surface and maintenance burden with minimal risk.
4. **Rename for clarity** — cheap, compounding readability win, and a forcing function for actually understanding the code.
5. **Unvetted community resource review** — critical if the project uses any external prompts/skills/configs with production access.
6. **Handoff documentation** — matters most right before a handoff or when the original builder is about to be unavailable.

## 8. Red Flags

Treat these as signals a codebase hasn't been "claimed" yet:

- Core flows (auth, payments, data access) that were never read line-by-line, only click-tested.
- Widespread generic function/variable names (`processData`, `handleSubmit`) throughout the codebase.
- No dependency audit ever run, or a missing/uncommitted lock file.
- Dependencies handling auth or crypto that haven't been updated in over a year.
- Unused functions, imports, or components left in the working tree "just in case."
- External prompts, skill files, or configs fed to an AI assistant without being read in full.
- No documentation of why key architecture decisions were made, or what to check first when the system breaks.
- The only person who can explain how the system works is unavailable.
