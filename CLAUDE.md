# CLAUDE.md — Gideon's Claude Code Standards

Behavioral guidelines derived from Karpathy's observations on LLM coding pitfalls, extended with production standards for agent systems and integrations.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial one-liners, use judgment.

---

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

## 5. Production Standards

**Build for production from day one — not for the demo.**

These apply to every workflow, agent, or integration.

### Before Writing Code

- Define what "success in production" looks like — not just "it runs"
- Define: what happens when the input is malformed, missing, or adversarial?
- Define: if this runs twice, what breaks? (Idempotency check — required, not optional)
- Define: what is the rollback plan and who triggers it?
- If you cannot answer all four, ask before proceeding.

### While Building

- Validate all inputs before any AI or agent processing touches them
- Add error handling and retries with exponential backoff on every external call
- Define a fallback for every step — what happens if this step fails entirely?
- Set a token and cost ceiling per run — never leave agentic loops uncapped
- Make every operation idempotent where possible (re-running = safe)
- No hardcoded secrets, keys, or environment-specific values in code

### Logging and Observability

- Log from day one: inputs, outputs, errors, latency, token usage, cost per run
- Logging is not alerting — define separately what triggers a human notification (error rate spike, output drift, cost anomaly)
- Structure logs for queryability from the start, not as an afterthought
- For agent systems: log which agent fired, what it received, what it returned, and how long it took

### Before Going Live

- Test with real messy data, not clean fixtures or happy-path examples
- Run in shadow mode until metrics pass — not until time passes:
  - Error rate below 2%
  - Output matches human baseline on spot-check sample
  - Cost per run within defined ceiling for 50+ consecutive runs
- Shadow mode is metric-gated. "2 weeks" is not a gate — it is a guess.
- When moving to live: reduce human review to a spot-check at a defined confidence threshold. Do not "remove" human review — that is a separate decision made with data, not assumed at launch.

### Cost and Scale

- Token budget is a first-class constraint, not an afterthought
- Every agentic workflow needs a cost ceiling per run and per day, with hard stops
- Define what an anomalous output pattern looks like and alert on it (not just errors)
- For multi-agent systems: trace cost per agent, not just total per run

---

## 6. Stack Defaults

Unless the task specifies otherwise:

- **Database:** Supabase (Postgres)
- **Auth:** Supabase Auth
- **Frontend:** React + Tailwind
- **Agent orchestration:** document agent handoffs and shared state explicitly
- **Secrets:** environment variables only — never in code or commits
- **API calls:** always handle rate limits, always set timeouts

---

## 7. Definition of Done

A task is done when:

- [ ] It works on messy real input, not just the happy path
- [ ] Error states are handled and logged
- [ ] Idempotency is confirmed or documented as a known exception
- [ ] No new secrets in code
- [ ] Cost ceiling is defined if this touches an AI or agent call
- [ ] A rollback path exists or is documented as unnecessary with reasoning

---

*These guidelines are working if: fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, clarifying questions come before implementation rather than after mistakes, and no agentic loop ships without a cost ceiling.*
