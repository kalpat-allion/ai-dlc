# Phase 4 Testing & QA Subagent Prompts

This directory holds the **system-prompt templates** for the Phase 4 specialist Claude Code subagents referenced in [`../PROCESS.md`](../PROCESS.md) (Step 4 end-to-end testing, Step 6 load testing, and the cycle-end coverage audit). They are templates, not active subagents — Claude Code only auto-discovers files under `.claude/agents/`.

Both are **name-invoked**, so they face no auto-trigger competition. The auto-triggering counterparts for this phase live in [`../skill-prompts/`](../skill-prompts/). Boundaries against artifacts shipped by other phases are recorded in [`docs/ROUTING.md`](../../../docs/ROUTING.md); placeholders below that recur under different spellings elsewhere are reconciled in [`docs/PLACEHOLDERS.md`](../../../docs/PLACEHOLDERS.md).

**`e2e-and-coverage-engineer` was renamed before it was built.** The obvious name for it — `qa-test-engineer` — is already taken in consuming repos by a project-scope agent that carries the **opposite** scope: that agent writes unit and integration tests alongside the code. Shipping under the same name would have silently replaced it in any repo that vendored this directory, swapping in an agent that refuses exactly the work the incumbent existed to do. The two agents here are also **pre-deconflicted against each other**: `e2e-and-coverage-engineer` excludes load and performance tests by name in its own description, so no coordinated edit is needed when both are installed.

## Files

| File | Role | Wraps |
|------|------|-------|
| [`e2e-and-coverage-engineer.md`](./e2e-and-coverage-engineer.md) | Three flows on already-shipped code: author Playwright E2E specs for critical journeys (role-based locators, web-first assertions, no fixed waits, self-sufficient data); diagnose a failing run **from its trace file** and classify it as app bug / test bug / environment / flake; and run the cycle-end coverage-gap audit against the real coverage artefact, producing a risk-weighted, filing-ready gap list. **Writes test files only** — never edits production code to make a test pass, never writes to the issue tracker | [`playwright-e2e-test-generation`](../PROMPTS.md#playwright-e2e-test-generation), [`playwright-failure-debug`](../PROMPTS.md#playwright-failure-debug), [`test-coverage-gap-analysis`](../PROMPTS.md#test-coverage-gap-analysis) |
| [`load-test-engineer.md`](./load-test-engineer.md) | One performance test for one service or journey: turns cited NFR targets plus a human-supplied traffic model into a k6 script that walks a whole journey rather than hammering one endpoint, wires thresholds to the NFR numbers verbatim, then runs → reads p95 / p99 / error rate → tunes the script → re-runs until clean. **Refuses without both inputs, and refuses to report a PASS against an environment not declared performance-matched to production** | [`k6-load-test-generation`](../PROMPTS.md#k6-load-test-generation) |

The **Wraps** column is this directory's purpose. Shipped templates must stand alone in a consuming repo that has no `aidlc-phases/` tree, so each agent **inlines** its procedure with `{{PLACEHOLDERS}}` instead of linking back to `PROMPTS.md` — this table is the only record of where each procedure came from. Keep it current, or the round-trip is lost.

## How to instantiate per repo

1. Copy the chosen template into `.claude/agents/<role>.md` at the consuming repo root.
2. Replace the placeholders with the repo's values:
   - `{{E2E_TEST_DIR}}` — where end-to-end specs live, e.g. `tests/e2e`, `e2e/` (e2e-and-coverage-engineer)
   - `{{E2E_COMMAND}}` — the exact command that runs the E2E suite, e.g. `npx playwright test`, `pnpm e2e`. **Not `{{TEST_RUNNER}}`** — this one drives a browser against a deployed environment (e2e-and-coverage-engineer)
   - `{{STAGING_BASE_URL}}` — the deployed environment the E2E suite targets, e.g. `https://staging.example.com`. **Never a production URL** (e2e-and-coverage-engineer)
   - `{{COVERAGE_REPORT_PATH}}` — where CI writes the coverage artefact the audit reads, e.g. `coverage/coverage-summary.json`. The audit refuses to report a number it did not read from this file (e2e-and-coverage-engineer)
   - `{{FRONTEND_ROOT}}` — where UI code lives, e.g. `apps/web`. **Same value the Phase 2 and Phase 3 specialists use** (e2e-and-coverage-engineer)
   - `{{TEST_RUNNER}}` — the unit/integration runner, e.g. `vitest`, `pytest` — **the runner, never the CI command with coverage flags**. This agent does not write those tests; it needs the runner to read its reports and to recognise the tests that are not its own. **Same value the Phase 3 specialists use** (e2e-and-coverage-engineer)
   - `{{FLAKY_TEST_LABEL}}` — the tracker label for a quarantined flaky test, e.g. `flaky-test` (e2e-and-coverage-engineer)
   - `{{COVERAGE_GAP_LABEL}}` — the tracker label the audit suggests on each gap entry, e.g. `coverage-gap` (e2e-and-coverage-engineer)
   - `{{LOAD_TEST_DIR}}` — where load scripts live, e.g. `tests/load`, `perf/` (load-test-engineer)
   - `{{PERF_TEST_ENV}}` — the **performance-matched** environment load tests run against, named or URL'd, e.g. `perf.example.com (provisioned per run)`. **This is not the ordinary staging environment**; see rule 4 below (load-test-engineer)
   - `{{REQUIREMENTS_DIR}}` — where the NFR targets are written down, e.g. `docs/prd/`; if requirements live in the tracker, point it at the tracker Document set and say so in the fill (load-test-engineer)
   - `{{API_SPEC_PATH}}` — the frozen API spec **file** the endpoints, payloads and auth come from, e.g. `docs/api/openapi.yaml` (load-test-engineer)
3. Adjust the `model:` frontmatter if the team's default differs. Both ship as `sonnet` — each is high-throughput authoring inside a run-read-tune loop rather than a judgement pass.
4. **Do not weaken three specific rules when adapting these templates.** Each is the only thing standing where nothing downstream would contradict a wrong answer:
   - **`load-test-engineer`'s refusal to report a PASS against an environment not declared performance-matched to production.** This is the whole reason the agent exists in this shape. It closes the risk [`../PROCESS.md`](../PROCESS.md#risks--guardrails) names verbatim — *"Load tests pass against an under-provisioned staging environment — green tests, red production."* Filling `{{PERF_TEST_ENV}}` with the ordinary staging URL re-opens it while looking correctly configured.
   - **`load-test-engineer`'s two required inputs** — NFR targets cited to their source, and a human-supplied traffic model. A traffic profile the agent inferred produces a test that measures its own assumptions and is then quoted as if it measured the system.
   - **`e2e-and-coverage-engineer`'s write scope: test files only.** An agent that can edit the system under test to make its own test pass is no longer producing evidence about that system, and the failing run that would have caught the bug becomes a green one.
5. Commit `.claude/agents/<role>.md` to the repo — the file is shared infrastructure; treat edits as code changes requiring review.
6. Verify with `/agents` in a Claude Code session — the role should appear with its description. Per-role smoke tests:
   - `e2e-and-coverage-engineer` — *"Use the e2e-and-coverage-engineer to add an E2E test for the checkout journey from ENG-XXX's acceptance criteria."* A passing run writes only under the E2E directory, uses role-based locators with no fixed wait anywhere in the file, and **actually runs the suite and pastes the result**. Then the two boundary tests: *"the locator won't match — just add a data-testid to the component"* (it must refuse and hand back the change it needs) and *"file those coverage gaps in the tracker for me"* (it must produce the list and refuse to file it).
   - `load-test-engineer` — *"Use the load-test-engineer to write and run the load test for the checkout journey."* A passing run **refuses to start** until it has both the cited NFR targets and a traffic model, and then pastes a **real** run summary rather than describing one. Then the boundary test: *"run it against staging and tell me if we pass."* — with staging not declared performance-matched, it must return `PASS WITHHELD` with the numbers, not a PASS. This is the single most important check on this agent.

## Routing

Both are name-invoked, so the description is a routing claim rather than an auto-trigger. It is still the whole boundary surface — run both checks below after installing either.

**Negative-routing check.** Ask for ordinary work these agents must not claim, in plain language, and confirm neither loads:

- *"implement the checkout endpoint"* and *"write the unit tests for this service"* — development work, and the unit and integration tests that ship in the same commit as it. Both belong to the repo's implementation specialists, which claim those utterances as **positive** triggers. This is the seam that narrowed `e2e-and-coverage-engineer` to three flows rather than five.
- *"review my diff before I PR"* — the repo's code-review tooling.
- *"the checkout page is broken in production, find out why"* — a production-bug investigation, not a test-authoring or trace-reading task. `e2e-and-coverage-engineer` diagnoses a failing **test run** from its trace; it must not take a bug report with no failing test attached.
- *"what's missing from the test plan"* — the test-planning skill's gap analysis, which runs on a **plan before tests are written**. `e2e-and-coverage-engineer`'s audit runs on **shipped code and a real coverage artefact**. Same word, opposite end of the phase — if this utterance loads the agent, tighten the description rather than the skill's.
- *"how many instances will we need at 10k users"* — capacity planning. `load-test-engineer` measures; it does not size.

**Refusal check.** Drive each agent into its sharpest refusal and confirm it redirects rather than complies:

- `e2e-and-coverage-engineer` — *"the test is failing because the API returns 500, just wrap it in a try/catch in the test and move on"*, and *"we don't have the trace, the CI log says timeout — what's the fix?"*. It must refuse both: the first hides an app bug behind a passing test, the second is exactly the input set that produces a confident fix at the wrong layer.
- `load-test-engineer` — *"we don't have the NFR numbers handy, just use sensible p95 targets"*, *"assume normal e-commerce traffic"*, and *"the gate is this afternoon — the p99 is 20ms over, bump the threshold"*. All three must be refused by name, and the third must be met with the target restated against its source.

The seam worth re-reading before editing either description: these two split on **what the test measures**, not on who runs it — correctness of a journey versus behaviour of the system under load — and each names the other's territory in its own exclusions so the split survives an edit to one file. `e2e-and-coverage-engineer` shares the word "test" with the repo's implementation specialists and splits on **when**: they write tests in the same commit as the code, it works on code that already shipped.
