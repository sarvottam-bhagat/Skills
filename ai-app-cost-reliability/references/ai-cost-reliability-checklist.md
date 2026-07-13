# AI App Cost & Reliability Checklist

Use this checklist to turn transcript-derived AI-engineering advice into practical fixes for any app that calls an LLM API.

## 1. Model Routing and Tiering

Running every request through your most expensive model is the single most common way an AI feature's bill spirals.

Audit for:

- Every request — FAQ answers, formatting, classification, complex reasoning — hitting the same top-tier model.
- No classification step between "request comes in" and "model gets called."

Fix:

- Build a lightweight complexity classifier (token count, task type, reasoning depth is often enough — ~10 lines of code).
- Route simple/repetitive tasks (formatting, classification, short FAQ-style answers) to a small/cheap model.
- Reserve the expensive/frontier model for genuinely complex, multi-step, or high-stakes requests.
- Route at the API gateway layer so the caller never needs to know which model answered.
- Re-run your eval suite per tier weekly — if the cheap model matches the expensive model's quality score on 80%+ of requests, that's most of your budget saved with no quality drop.

Implementation prompts:

- "Add a request classifier that scores incoming prompts by token count and task type, and routes simple requests to a cheaper model and complex ones to the current model."
- "Run the existing eval suite against both the cheap and expensive model on last week's real requests and report the quality delta per tier."

## 2. Caching to Avoid Redundant Calls

Most users ask semantically similar questions repeatedly. Paying full price for the same answer every time is pure waste.

Fix:

- **Semantic caching**: embed the intent of incoming queries; before calling the model, check if a semantically similar query was answered recently (e.g. in the last 24h) and serve that instead of calling the model again.
- **Prompt caching**: if your system prompt or context window repeats across requests (it almost always does), use the provider's prompt-caching feature so repeated context is billed at a fraction of the cost.
- **Batch APIs for non-urgent work**: route non-interactive workloads (nightly analysis, content pipelines, bulk processing) to batch endpoints, which are typically ~50% cheaper and don't need a synchronous response.
- Stack all three: cache repeated context, batch non-urgent work, and tier by complexity — these compound rather than compete.

Implementation prompts:

- "Add semantic caching in front of the model call: embed the query, check for a similar cached query from the last 24 hours, and skip the model call on a hit."
- "Enable prompt caching for the repeated system prompt/context in every model call."
- "Move [nightly report / bulk processing] job to the batch API instead of synchronous calls."

## 3. Output Validation, Retry, and Graceful Degradation

Raw model output should never reach a user unchecked. Models hallucinate, exceed expected length, or return malformed structure.

Fix:

- Validate every response against an expected schema, length boundary, and content policy before it reaches the user.
- On validation failure, retry with the specific failure reason fed back into the prompt (e.g. "previous response exceeded 200 words, respond in under 200 words") — cap at 2-3 attempts.
- If retries are exhausted, degrade gracefully: fall back to a simpler model, a canned/cached response, or a human handoff. Never show a raw error, a blank screen, or an unfiltered hallucination.
- Build this as one reusable validation/retry/degrade layer used by every AI call site, not a one-off per feature.

Implementation prompts:

- "Add an output validation layer that checks every AI response against [schema/length/content rules] before it's shown to the user."
- "When validation fails, retry the model call up to 2 times with the specific failure reason added to the prompt, then fall back to [simpler model / canned response] if it still fails."

## 4. Eval-Based Testing (Not Assertion-Based)

CI pipelines built for deterministic code (same input → same output) flake constantly on AI features, because AI output legitimately varies run to run.

Fix:

- Replace exact-string-match assertions with evaluation-based tests: score outputs on quality criteria (accuracy, tone, schema compliance) and set a pass/fail threshold per criterion.
- Use a second model call (or rubric-based check) to grade the first model's output — "model as judge" — as part of CI, not just manual eyeballing.
- Build the regression suite from real failures: every time a user reports a bad response, add that exact input to the test suite with the expected quality bar. Over time this becomes a suite of real edge cases, not synthetic ones.
- When comparing model or prompt versions, run the same eval suite against both and compare scores — don't rely on "it felt better."
- Add a cost check to the pipeline: estimate the token cost of a prompt/context change before it deploys, and flag anything that meaningfully increases per-request cost.
- Gate deploys on a canary quality score: route a small percentage of traffic to a new prompt/model version, monitor the eval score and latency for a fixed window, and auto-rollback if quality drops below threshold — don't rely on a human watching a dashboard.

Implementation prompts:

- "Replace the exact-match test assertions on AI outputs with an eval-based test that scores responses on accuracy, tone, and schema compliance against a threshold."
- "Every time a user reports a bad AI response, add that input and the expected quality bar to the regression test suite."
- "Add a cost estimate check to the deploy pipeline that flags any prompt/context change that meaningfully increases token cost per request."

## 5. Abuse Protection and Spend Caps

An AI endpoint with no auth or spend limit is an open invoice — one bot loop or one malicious user can produce a bill that ends the project in a weekend.

Fix:

- Put an API gateway in front of every AI endpoint; reject requests without a valid key before they reach the model.
- Validate payload size and context-window length at the gateway; reject oversized prompts before they hit an expensive model tier.
- Track token consumption per user/tenant and enforce daily/monthly caps at the gateway layer (not in application code that can be skipped).
- Set hard spend ceilings on the provider account itself, with alerts at 50%, 75%, and 90% of budget, so a runaway loop is caught in minutes, not at the end of the billing cycle.

Implementation prompts:

- "Put an API gateway in front of every AI endpoint that rejects requests without a valid API key before any tokens are spent."
- "Add per-user daily token/request caps enforced at the gateway, with different limits per pricing tier."
- "Set billing alerts on the model provider account at 50%, 75%, and 90% of the monthly budget."

## 6. Multi-Agent Architecture

Choosing the wrong coordination pattern for a multi-agent system leads to circular calls and unpredictable behavior.

Fix:

- Default to an **orchestrator pattern**: one central agent receives the request, decides which sub-agents to call, collects their outputs, and synthesizes the final response. Simple to reason about and debug.
- Only move to a **conductor pattern** (agents passing work to each other in a chain/graph with no single controller) for genuinely emergent workflows — deep research, multi-step reasoning chains — where the next step depends on what the previous agent discovered.
- Don't start with a conductor pattern because it "sounds more advanced." Start with orchestrator, validate outputs and error handling are solid, and only migrate a specific sub-workflow to conductor once data shows the orchestrator is the actual bottleneck.

Implementation prompts:

- "Design this multi-agent feature with a single orchestrator agent that calls sub-agents and synthesizes their outputs, rather than agents calling each other directly."

## 7. Agent Memory

Bolting memory onto an agent incorrectly breaks faster than having no memory at all.

Fix:

- **Short-term memory**: a rolling conversation buffer for the current session; summarize older exchanges into compressed context once the window gets long, keep recent turns verbatim.
- **Long-term memory**: a persistent vector store (e.g. embeddings in Postgres/pgvector, Pinecone, Weaviate) for user preferences, past decisions, and learned patterns, retrieved per query by semantic relevance — not loaded in full every time.
- Only store what changes the agent's future behavior (e.g. a user's stated preference or a past decision). Don't store casual, one-off asides — retrieval quality matters more than storage volume.
- Test memory by asking the agent to reference something from a past session; if it hallucinates a memory that was never stored, the retrieval pipeline needs work, not the prompt.

Implementation prompts:

- "Add a rolling conversation buffer that summarizes older turns once the context window exceeds [N] tokens, keeping recent turns verbatim."
- "Add a long-term memory store that saves only user preferences and past decisions that affect future behavior, retrieved by semantic relevance per query."

## 8. Prioritization Matrix

If time is limited, fix in this order:

1. **Spend caps and gateway auth** — an unprotected AI endpoint is the fastest way to an account-ending bill; fix this first regardless of anything else.
2. **Output validation before user-facing display** — prevents a hallucination or malformed response from becoming a support ticket or a trust incident.
3. **Model routing/tiering** — usually the single biggest recurring cost reduction with no quality loss.
4. **Semantic/prompt caching** — second biggest cost reduction, especially for apps with repeated or similar queries.
5. **Eval-based CI** — stops AI features from being an untested black box that only gets QA'd by production users.
6. **Multi-agent architecture and memory design** — only relevant once the feature has grown past a single model call.

## 9. Red Flags

Treat these as signals an AI feature is not production-ready:

- An AI endpoint reachable without authentication.
- No per-user or per-tenant spend cap anywhere in the request path.
- Every request, regardless of complexity, hitting the same top-tier/most expensive model.
- No caching layer despite users frequently asking similar questions.
- Raw model output rendered directly to the user with no schema/length/content validation.
- CI tests that assert exact string equality on AI-generated output (guaranteed to flake).
- A test suite built only from synthetic cases, with no real user-reported failures folded in.
- A multi-agent system built as agents-calling-agents (conductor) from day one, with no orchestrator baseline to compare against.
- Agent "memory" that stores every message verbatim forever, with no summarization or relevance-based retrieval.
