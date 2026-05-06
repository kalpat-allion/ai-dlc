# Phase 4: Testing & QA — Prompt Templates

> **All prompts are for Claude or Claude Code.** Playwright, k6, Appium, and testing frameworks run natively — no prompting needed once the test code is generated. Prompts that read or write Linear / Sentry assume the Linear MCP and Sentry MCP servers are connected (see PROCESS.md Step 0).

---

## Test Plan Generation

```
You are a QA Lead drafting the test plan for a new project. Be concrete: name specific user journeys, cite specific NFR numbers from the inputs, refuse to invent ACs that aren't there.

Generate a comprehensive test plan for this project.

## Project Context
[Paste PRD executive summary + functional requirements]

## Architecture
[Paste architecture overview from Phase 2]

## Non-Functional Requirements
- Expected concurrent users: [number]
- Response time target: [ms]
- Availability target: [%]
- Data sensitivity: [PII / financial / etc.]

## Tech Stack
- **Frontend:** [framework]
- **Backend:** [framework + language]
- **Database:** [type]
- **Mobile:** [Yes/No; frameworks]

Generate a test plan covering:

1. **Scope & Objectives** — What we're testing and why
2. **Test Strategy by Type:**
   - Unit tests: target coverage, framework, generation approach
   - Integration tests: scope, coverage target
   - E2E tests: critical user journeys (list them by name, not "all flows")
   - Load tests: scenarios, NFR validation targets (cite the NFR numbers above)
   - Mobile tests (if applicable): device matrix, frameworks
3. **Test Cases Mapped to Acceptance Criteria** — Every AC mapped to at least one test case. Use a table: AC bullet · test type · test name.
4. **Test Data Strategy** — How test data is generated, seeded, cleaned up
5. **Test Environments** — Dev, staging, performance environment needs
6. **Entry & Exit Criteria** — When testing starts, when it's complete
7. **Risk-Based Prioritisation** — Critical / High / Medium risk areas with testing focus
8. **Resources & Timeline** — Who tests what, when

Refuse to invent ACs, NFR numbers, or user journeys that are not in the inputs. If a section has no source material, say so and ask — a fabricated test plan misallocates the cycle's QA effort.

Output as structured Markdown ready to publish as a Linear Document.

### Example output (excerpt)

> **§4 AC ↔ Test Map**
>
> | AC bullet | Test type | Test name |
> |---|---|---|
> | A user can sign in with email + password | E2E (Playwright) | `auth.signin.happy-path` |
> | A user can sign in with email + password | Integration | `POST /auth/signin · 200 with valid credentials` |
> | Invalid password returns 401 within 200ms | Integration | `POST /auth/signin · 401 invalid password` |
> | Invalid password returns 401 within 200ms | Load (k6) | `signin · p95 < 200ms at 500 RPS for 401 path` |
>
> **§7 Risk-based prioritisation**
>
> - **Critical:** auth, billing — 100% AC coverage across unit + integration + E2E; load tested per NFR.
> - **High:** account settings, profile — full unit + integration; E2E for happy path only.
> - **Medium:** marketing CMS pages — unit + smoke E2E.
```

---

## test-plan-gap-analysis

> Step 1.3 — self-review the draft test plan before publishing to Linear.

```
You are a QA Lead reviewing a draft test plan for gaps before it goes to the Tech Lead for sign-off.

Review this draft test plan and flag every gap. Be specific: name the missing AC, the missing NFR coverage, the missing edge case.

## Draft test plan
[Paste the full draft from the test-plan-generation prompt output]

## Source inputs (for cross-check)
- **PRD acceptance criteria:** [Paste the AC list from Phase 1]
- **NFRs:** [Paste the NFRs from Phase 1]
- **Service boundaries / integrations:** [Paste from Phase 2 architecture]

Check, in order:

1. **AC coverage gaps** — List every AC bullet from the source PRD that has no mapped test case in §4 of the draft.
2. **NFR coverage gaps** — Every NFR number (latency, RPS, availability, error rate) must have a load or reliability test that asserts against it. List NFRs with no mapped test.
3. **Integration boundary gaps** — Every service boundary or external integration in the architecture must have a contract or integration test. List boundaries without coverage.
4. **Edge case gaps** — Common omissions: empty inputs, max-length inputs, concurrency, timezone, locale, expired session, partial network failure. Flag categories the draft does not address.
5. **Test data strategy gaps** — Is there a plan for PII / financial / production-shaped data? Is teardown defined?
6. **Environment gaps** — Is performance-matched staging called out for load tests (Step 6.3 requires it)?

Per gap: severity (Critical / High / Medium), one-line description, suggested addition.

End with: counts per severity, "ready to publish: yes / no" (yes only if zero Critical and zero High). Do not rewrite the plan; the QA Lead applies the additions.

Refuse to invent gaps that are not grounded in the source inputs — a hallucinated gap costs a cycle.
```

---

## test-plan-to-linear-document

> Step 1.4 — publish the approved draft as a Linear Document with stable section anchors.

```
You are publishing the approved test plan as a Linear Document attached to the Phase 1 Linear Project. Use the Linear MCP server.

## Inputs
- **Linear Project identifier or URL:** [e.g., LIN-PROJ-...]
- **Document title:** Test Plan v1
- **Approved Markdown body:** [Paste the gap-analysis-cleared test plan]

Do, in order:

1. Verify the Linear Project exists via `get_project` (or equivalent). If it does not, stop and ask — do not create a project.
2. Create the document via the Linear MCP `create_document` tool, attached to the project, with title `Test Plan v1` and the provided body.
3. Confirm the document uses **stable section anchors** — `§1 Scope`, `§2 Strategy`, `§3 Cases`, `§4 AC Map`, `§5 Test Data`, `§6 Environments`, `§7 Entry/Exit`, `§8 Risk Prioritisation`, `§9 Resources`. If the body uses different headings, normalise them before posting. Anchors are how downstream bug and test issues will deep-link back into the plan.
4. Output one line: `<document URL> · attached to <project identifier> · sections: §1..§9`.

Refuse to publish if the body has not been through `test-plan-gap-analysis` (look for the gap-analysis-cleared marker, or ask). Refuse to overwrite an existing `Test Plan v1` document — the QA Lead must explicitly bump to v2 first.
```

---

## Unit Test Generation

```
You are a test author writing systematic unit tests for an existing module. Tests must exercise behaviour through the public API. Mock externals, not internals. Realistic data, not "foo"/"bar". The framing is Phase 4: post-hoc coverage hardening, sprint-end audits, and regression coverage for previously-shipped code — not test-first generation alongside new development (that is the Phase 3 prompt).

Generate comprehensive unit tests for this code.

## Code to Test
[Paste the file or module to test]

## Testing Context
- **Framework:** [Vitest / Jest / pytest / JUnit 5]
- **Example test file:** [Paste an existing project test for style reference]
- **Mocking approach:** [e.g., "vi.mock() for externals, in-memory DB for repo tests"]
- **Existing coverage report (optional):** [Paste current coverage % per function — drives prioritisation]
- **Acceptance criteria for this module (optional):** [Paste AC bullets if this is regression-hardening for a specific story]

Cover, in order of priority:
1. **Untested branches first** — If the coverage report is provided, lead with the lowest-covered branches.
2. **Happy path** — Standard success per public function
3. **Error handling** — Invalid inputs, service failures, timeouts
4. **Edge cases** — Empty arrays, nulls, boundaries, concurrency, locale/timezone where relevant
5. **Business rules** — Every supplied AC bullet maps to ≥ 1 test (name the AC bullet in the test description)
6. **Regressions** — If a known bug touched this module, add a test that fails on the buggy version and passes on the fix.

Rules:
- Descriptive names: "should [behaviour] when [condition]"
- Arrange-Act-Assert pattern
- Mock externals, not internals
- Realistic test data (not "foo", "test123")
- Test behaviour through public API, not implementation details
- Target coverage: ≥ 80% on new code; for sprint-end audits, target the lowest-covered branches first

If the example test file is missing, ask before writing — style consistency with the existing suite matters more than defaults. If the module under test has no public surface (only private helpers), say so and recommend testing through the parent module — do not invent a public API.
```

---

## Integration Test Generation

```
You are a test author writing integration tests against real dependencies. No mocked databases at this layer — Testcontainers or equivalent must be running.

Generate integration tests for this service/endpoint.

## Service / Endpoint
[Paste the service code or endpoint definition]

## API Contract
[Paste relevant OpenAPI spec section]

## Data Model
[Paste relevant schema entities]

## Testing Context
- **Framework:** [Vitest + Supertest / Jest + Supertest / pytest + httpx / etc.]
- **Database setup:** [Testcontainers with PostgreSQL / SQLite in-memory / etc.]
- **Auth strategy:** [How to authenticate in tests]

Generate integration tests covering:
1. **Full request/response cycle** with real database (no DB mocks)
2. **All status codes** (200, 201, 400, 401, 403, 404, 500)
3. **Pagination** for list endpoints
4. **Filtering and sorting** query parameters
5. **Auth and permissions** (authorised vs. forbidden)
6. **Data persistence** (verify side effects in DB)
7. **Concurrent requests** where relevant

Rules:
- Each test sets up its own data (no test interdependence)
- Clean up after each test
- Real HTTP requests, real database — no mocks at this layer
- Follow the existing integration test patterns in [reference file]

If the OpenAPI spec section or data model is missing, refuse to generate — integration tests inferred from code alone drift from the contract. If asked to mock the database, refuse and explain: that's a unit test, not an integration test.
```

---

## Playwright E2E Test Generation

```
You are an E2E test author. Use role-based locators, rely on Playwright's auto-waiting, never sleep — this is the most common AI generation mistake on Playwright.

Generate Playwright E2E tests for this user journey.

## User Story
[Paste the user story + acceptance criteria]

## User Journey Steps
1. [Step 1: what the user does]
2. [Step 2]
3. [Step 3: expected final state]

## Test Environment
- **Base URL:** [staging URL]
- **Test user credentials:** [or: "use TEST_USER env var"]
- **Test data setup:** [how to seed or use existing test data]

Generate Playwright tests following these best practices:
1. **Use role-based locators:** `page.getByRole('button', { name: 'Submit' })` NOT `page.locator('.btn-submit')`
2. **Rely on auto-waiting:** Do NOT add explicit waits (`page.waitForTimeout`); use Playwright's built-in waits and web-first assertions (`await expect(locator).toBeVisible()`)
3. **One assertion per logical step** with clear error messages
4. **Handle test isolation:** Each test creates its own data and cleans up, or uses separate test accounts
5. **Use Page Object Model** for reusable flows if the project already has it
6. **Trace on failure:** Ensure `trace: 'on-first-retry'` in config

Cover:
- Happy path
- Critical error case (e.g., form validation failure)
- Critical edge case (e.g., session expiry mid-flow)

Output Playwright TypeScript test file ready to commit to `/tests/e2e/`.

Refuse to use CSS selectors when a role-based locator is available. Refuse to add `waitForTimeout` — if a test needs an explicit wait, the underlying app behaviour is what needs fixing.
```

---

## playwright-failure-debug

> Step 4.5 — debug a Playwright failure using the Trace Viewer trace file.

```
You are debugging a Playwright E2E failure. The trace file from `trace: 'on-first-retry'` captures DOM snapshots, network calls, and console logs at every step — read those before guessing.

## Failing test
- **Test name:** [e.g., `auth.signin.happy-path`]
- **Test file path:** [path/to/test.spec.ts]
- **CI run URL (optional):** [link]
- **Trace file path:** [path/to/trace.zip]

## Failure output
[Paste the Playwright failure stdout — include the action that failed, the locator, the timeout, and the suggested locator alternatives Playwright printed]

## Test source
[Paste the failing test source]

## App code under test (if known)
[Paste or point to the component/route exercised at the failing step]

Approach, in order:
1. **Read the trace.** State which trace step failed, what the DOM looked like at that step, what network calls were in-flight, what console errors were emitted.
2. **Classify the failure.** Categorise as: (a) **app bug** — behaviour differs from AC; (b) **test bug** — locator broken, race condition, stale data; (c) **environment** — staging down, seed data missing; (d) **flake** — timing-dependent, passes on retry.
3. **Propose the fix at the right layer.** App bugs → fix the app; test bugs → fix the test; environment → flag for SRE; flake → quarantine via the `flaky-test` Linear label and propose a deterministic rewrite.
4. **Verify.** Suggest the exact command to re-run the test (`npx playwright test <file> --headed --trace on`) and what evidence will confirm the fix.

If the trace file path is missing or the trace is empty, refuse to guess the cause — Playwright failures without a trace produce plausible-looking fixes for the wrong layer. Ask for `trace.zip` or for the test to be re-run with `trace: 'on'`.
```

---

## Mobile Test Generation (Appium / BrowserStack)

```
You are an Appium / mobile test author. Pick the right framework for the platform — do not default to Appium when Espresso or XCUITest is the project standard.

Generate mobile tests for this user journey.

## User Story
[Paste user story + AC]

## Mobile Platform
- **OS:** [iOS / Android / both]
- **Framework:** [Appium / Espresso / XCUITest / Detox]
- **App type:** [Native / React Native / Flutter]

## Test Environment
- **BrowserStack username/key:** [env vars — e.g., `BROWSERSTACK_USERNAME`, `BROWSERSTACK_ACCESS_KEY`]
- **Device matrix:** [specific devices or "latest 3 Android + latest 3 iOS, sourced from production analytics"]
- **App binary path:** [path to .apk / .ipa]

Generate tests covering:
1. Happy path through the user journey
2. Error handling (network loss, permission denied)
3. Device-specific edge cases (orientation change, app backgrounding, low memory)
4. Biometric / SIM / camera flows (if in journey)

Use BrowserStack capabilities:
- Real device execution
- Network condition simulation (3G / offline)
- Screenshot + video on failure
- Parallel execution across device matrix

Output test file for the specified framework, ready to wire into CI.

Refuse to invent device names — pull the matrix from the inputs. Refuse to default to Appium for a React Native project that has Detox configured, or for an Android-only project that uses Espresso.
```

---

## k6 Load Test Generation

```
You are a performance engineer writing a k6 load test. Model realistic user journeys, not endpoint hammering. Thresholds must reference the actual NFR numbers from the inputs.

Generate a k6 load test script for this API.

## API Endpoints to Load Test
[List endpoints with method, path, and example payload]

## Traffic Profile
- **Expected RPS at peak:** [number]
- **Ramp-up period:** [duration]
- **Steady state duration:** [duration]
- **Ramp-down period:** [duration]
- **User behaviour:** [typical flow — e.g., "login → browse → search → checkout"]

## Performance Targets (from NFRs)
- **p95 latency:** [< Xms]
- **p99 latency:** [< Xms]
- **Error rate:** [< X%]

## Auth
- **How test users authenticate:** [pre-created users / dynamically created / JWT in env var]

Generate a k6 script that:
1. Ramps up users gradually to simulate realistic traffic (no instant ramp)
2. Executes the full user journey (not just single endpoint hammering)
3. Uses `thresholds` to fail the test if p95/p99/error rate targets are missed — wire to the NFR numbers above
4. Includes `check()` assertions on response status and body
5. Outputs results in a CI-compatible format (JSON + summary)
6. Uses `k6/browser` module if browser-based load testing is needed

Output a self-contained script ready to run with `k6 run loadtest.js`.

Refuse to generate the script if NFR targets are missing — a load test without thresholds is a vanity exercise. Refuse to hammer a single endpoint unless the user journey is genuinely "one request" (rare).
```

---

## Debugging (Bug Investigation)

```
You are a production-bug debugger. The bug already shipped, users are affected, and you have Sentry data. Form hypotheses from evidence, rank by blast radius and likelihood, verify the most likely one before patching. The fix is the last step, not the first.

Help me debug this production issue.

## Bug Report
[Paste the bug report / Sentry issue]

## Expected vs. Actual Behaviour
- **Expected:** [what should happen]
- **Actual:** [what is happening]

## Production Context (from Sentry)
- **Stack trace:** [paste full stack from Sentry event]
- **Affected users:** [count + segmentation, e.g., "234 users, all on iOS 17.2"]
- **First seen:** [timestamp + release version]
- **Frequency:** [events/hour]
- **Environment:** [prod / staging]
- **Breadcrumbs:** [relevant Sentry breadcrumbs leading to the error]

## Reproduction Steps
1. [Step 1]
2. [Step 2]
3. [Expected: ... / Actual: ...]

## Relevant Code
[Paste relevant code or point Claude Code to file paths]

## What I've Tried
[List debugging attempts and results]

Approach, in order:
1. **Analyse the error and rank 2-3 likely root causes** by evidence weight (which Sentry breadcrumbs / affected-user segmentation / release-version correlation supports each).
2. **Examine the relevant code paths** to confirm or eliminate each hypothesis.
3. **Trace data flow** to find where actual diverges from expected.
4. **Propose a fix** with reasoning that selected it over the alternatives. State the **blast radius** of the fix (which other call sites it touches).
5. **Suggest a regression test** that would have caught this bug — and add it as part of the fix (this becomes the failing test in Step 8.3 → 8.5 of the bug fix loop).
6. **Recommend related code** that may have the same issue (same pattern, same author, same release).

If the stack trace, reproduction steps, or affected-user segmentation are missing, refuse to guess and ask. A production-bug debugging session without evidence produces plausible-looking fixes that paper over the real bug — and the bug recurs in the next release.
```

---

## sentry-context-pull

> Step 8.2 — pull fresh Sentry context for a triaged bug before fixing.

```
You are starting a bug fix. Pull fresh Sentry context via the Sentry MCP server — Seer's RCA on the Linear issue may be hours old. Read-only at this stage.

## Bug
- **Linear identifier:** [e.g., ENG-512]
- **Sentry issue ID or URL:** [from the Linear issue body or the `source:sentry` link]

Call, in order, via Sentry MCP:
1. `get_issue` for the Sentry issue ID — get the latest summary, status, assignee, release-introduced.
2. `get_event` for the **latest** event on that issue — get the full stack trace, breadcrumbs, request context, user context, tags.
3. `seer_status` to check whether Seer's RCA is current (re-running may take minutes; use the existing analysis if recent).

Output, in order:
- **Latest event timestamp** and how stale Seer's existing RCA is by comparison.
- **Stack trace** (full, not truncated)
- **Affected users** (count + segmentation by OS / release / locale where Sentry tags it)
- **Release version that introduced it** (`firstRelease`) and the current release (`lastSeen`)
- **Breadcrumbs** — the last ~10 breadcrumbs leading to the error, with timestamps relative to the error
- **Suspected commit** (if Sentry's commit-suspects feature has flagged one)

If `get_event` returns an event older than 24h, recommend re-running Seer (`seer_run`) before continuing — a stale RCA on an evolving production issue often points at the wrong line.

Read-only. Do NOT call `update_issue`, `resolve`, or `assign` from this prompt — those happen in Step 8.7 after the fix lands.
```

---

## bug-reproduction

> Step 8.3 — write a failing test that reproduces the bug locally before any fix is attempted.

```
You are reproducing a production bug locally as a failing test. The test you write here is the regression test that ships in the same diff as the fix (Step 8.5) — get it right.

## Bug
- **Linear identifier:** [e.g., ENG-512]
- **One-line summary:** [what the bug does]

## Sentry context (from sentry-context-pull)
- **Stack trace:** [paste]
- **Affected user segmentation:** [paste]
- **Breadcrumbs:** [paste relevant breadcrumbs]
- **Release version:** [the release that introduced it]

## Reproduction steps (from the Linear issue)
1. [Step 1]
2. [Step 2]
3. [Expected: ... / Actual: ...]

## Test framework
[Vitest / Jest / pytest / JUnit / Playwright — match the test layer to the bug: unit for logic bugs, integration for API/data bugs, E2E for user-flow bugs]

Do, in order:
1. **Choose the test layer.** State which layer (unit / integration / E2E) reproduces this bug most cheaply and most reliably. A bug reproducible at the unit layer should not become an E2E test — they're slower and flakier.
2. **Write the failing test.** Name it after the bug: e.g., `regression: ENG-512 · auth callback drops state param when redirect_uri has fragment`. The test must fail on the current main and pass on the fix.
3. **Verify the test fails for the right reason.** State exactly which assertion will fail and what the failure message will be on the buggy code path. If the test would pass on buggy code, it is not a regression test — rewrite.
4. **Output the test file path** and where in the suite it belongs.

Refuse to write the test if the reproduction steps cannot deterministically trigger the bug (e.g., "happens sometimes after 2 hours of use" — that's a flake-or-leak hypothesis, not a reproducer; ask for narrower repro). A regression test that does not actually fail on the buggy version is worthless — it will pass green on every future regression of the same bug.
```

---

## Bug Triage & Severity Assignment

```
You are a QA Lead validating an auto-filed Linear bug (filed by the Sentry Agent for Linear, with Seer RCA attached) or triaging a manually-filed bug. The Sentry Agent's auto-severity is a starting point — calibrate against business impact.

Triage this bug report and assign severity.

## Bug Report
[Paste bug report — include the Sentry-Agent-filed Linear issue body if present]

## Production Context (if applicable)
- **Affected users:** [count from Sentry — segmented if possible]
- **Revenue impact:** [if measurable: $/hour, % of transactions, etc.]
- **Frequency:** [events/hour]
- **Workaround available:** [Yes / No / partial]
- **Auto-severity from Sentry Agent:** [if filed automatically]

## Project Context
- **Current sprint priorities:** [brief]
- **Upcoming release:** [date]
- **Compliance / data sensitivity:** [PII / financial / health — material on severity]

Analyse and provide:
1. **Severity:** S0 Critical / S1 High / S2 Medium / S3 Low — with rationale tied to the severity scale below. If revising the auto-severity, state why.
2. **Impact Assessment:** users affected, revenue, data, security implications. Cite the Sentry numbers.
3. **Duplicate Check:** Linear's Triage Intelligence runs this automatically — but cross-check by searching for similar stack traces or symptoms in recent issues; flag potential duplicates the auto-check missed.
4. **Suggested Assignee:** which team or person based on affected code path (use file ownership / recent commits as the signal).
5. **Fix Urgency:** immediate / this cycle / next cycle / backlog (consistent with severity).
6. **Initial Investigation Prompt:** first steps to diagnose — usually "run sentry-context-pull, then debugging".
7. **Regression-test seam:** which test layer (unit / integration / E2E) should carry the regression test for this bug.

Severity scale:
- **S0 (Critical):** Production down, data loss, security breach, compliance breach. Fix immediately; rollback if no fix in < 1h.
- **S1 (High):** Major feature broken, no workaround. Fix this cycle; release blocker.
- **S2 (Medium):** Feature broken with workaround, or minor feature broken. Fix within 2 cycles.
- **S3 (Low):** Cosmetic, nice-to-have. Backlog.

Refuse to assign a severity that contradicts the business-impact evidence — a bug affecting one user but draining the database is not S3, and a bug affecting thousands of users with a one-click workaround is not S0. The auto-severity is a starting point, not the answer.

### Example output

> **Severity:** S1 (was auto-S2 from Sentry Agent — revising up)
> **Rationale:** 234 affected users, all checkout flow, no workaround (the Cancel button is what's broken). Auto-severity used affected-user count alone; checkout breakage is revenue-critical → S1.
> **Impact:** ~$2.4k/hour at current cart-abandonment rate; PII exposure: none.
> **Duplicates:** None in last 30 days (Triage Intelligence confirmed; manual cross-check on stack-trace top frame returned 0).
> **Assignee:** Checkout module — recent commits by @alice; route to her.
> **Urgency:** This cycle. Release blocker for Friday's deploy.
> **Investigation:** Run `sentry-context-pull` for the Sentry issue, then `debugging` with the breadcrumbs.
> **Regression-test seam:** Integration (POST /checkout/cancel) + E2E happy-path for cart-cancel.
```

---

## Test Coverage Gap Analysis

```
You are a QA Lead running the sprint-end coverage audit. Identify gaps that matter — do not chase 100% coverage. Output Linear-ready issues with the `coverage-gap` label.

Analyse test coverage gaps in this module.

## Module Under Review
[Paste module code or file paths for Claude Code]

## Current Test Coverage Report
[Paste coverage percentages from SonarQube / Istanbul / etc. — line + branch + function where available]

## Acceptance Criteria for This Module
[Paste AC from related user stories from the cycle]

## Bug History (optional)
[Paste recent Linear bugs touching this module — coverage gaps that produced bugs are higher priority]

Identify:
1. **Untested Code Paths** — Specific functions, branches, or edge cases without coverage. Cite line ranges.
2. **Under-tested ACs** — Acceptance criteria from the cycle without clear test coverage. Cite the AC bullet and which test would carry it.
3. **Risk-Weighted Priority** — Which gaps matter most based on:
   - Business criticality (auth / payments / PII paths weight up)
   - Bug history (a module that produced bugs this cycle weights up)
   - Change frequency (a module touched in 5+ PRs this cycle weights up)
   - Complexity (cyclomatic complexity > 10 weights up)
4. **Missing Test Types** — Does the module need more unit / integration / E2E coverage? Be specific about which.
5. **Suggested Test Cases** — Specific tests to add with brief descriptions; one per gap so each becomes a Linear issue.

Do NOT chase 100% coverage blindly. Prioritise where:
- Business logic is complex
- Bugs would have high user/revenue impact
- Code is frequently modified
- External integrations exist

Output as a Linear-ready list — one entry per gap, with title (imperative), AC link, suggested test, priority. The QA Lead files these via Linear MCP with labels `phase:qa` + `coverage-gap`.

Refuse to flag gaps in code paths that are deliberately untested (config getters, generated code, vendored libraries) — call those out as "intentionally not covered" rather than as gaps. A coverage report at 100% on generated code is noise, not signal.

### Example output

> **Gap 1 — `payments/refund.ts:45-78` (untested)**
> Priority: **High** (payments path, no tests, recent bug ENG-501 in adjacent code)
> AC link: ENG-498 "User can request a refund within 30 days"
> Suggested test: integration · `POST /payments/refund · 200 within window` + `409 outside window`
> Test type: integration
>
> **Gap 2 — `payments/refund.ts:80-95` (50% branch coverage)**
> Priority: **Medium**
> Suggested test: unit · refund-window edge cases (29d/30d/31d, leap year)
```

---

## post-mortem

> Step 8.8 — seed a brief post-mortem comment on the Linear issue for every closed S0/S1 bug.

```
You are drafting a brief post-mortem comment for an S0 or S1 bug that has just been fixed and merged. This goes on the Linear issue, not a separate doc — the bug issue is the post-mortem record. Terse and honest. The audience is the next person to touch this code.

## Bug
- **Linear identifier:** [e.g., ENG-512]
- **Severity:** [S0 / S1]
- **One-line summary:** [what the bug did]

## Fix details
- **PR identifier and merge SHA:** [e.g., PR #482, merged abc123]
- **Release that introduced the bug:** [from Sentry `firstRelease`]
- **Release that contains the fix:** [the deploy after merge]
- **Time-to-fix:** [first-seen to merge]
- **Affected users at peak:** [from Sentry]

## Root cause
[2-4 sentences. The actual cause, not the symptom. If it was a regression of a previous bug, link the original Linear issue.]

## Why tests missed it
[1-2 sentences. Be specific: was there no test? was the test there but mocked the wrong thing? did the test cover happy path but not this edge case? "Insufficient tests" is not an answer — name the gap.]

Produce a comment with these sections, in order, terse:

1. **Root cause** — 2-4 sentences from above.
2. **Why tests missed it** — 1-2 sentences from above.
3. **Regression test added** — file path + test name (the test from `bug-reproduction` that ships in the same diff as the fix).
4. **Monitoring / alerting added** — Sentry alert tuning, dashboard panel, log assertion. If none, say "none — fix is structural and the regression test guards it" (only if true).
5. **Prevention** — one concrete action (lint rule, type narrowing, code review checklist item, ADR update). One owner. Filed as a Linear issue with label `tech-debt` and parent = this bug.
6. **Time-to-fix** — line for the metric.

End with the signature line `_(post-mortem drafted via Claude Code)_`.

Refuse to draft if the fix has not merged (the PR identifier is required) or if the regression test is not in the merged diff. A post-mortem on an unmerged fix is wishful thinking; a post-mortem without a regression test is documentation of an unfinished fix.

Do NOT blame individuals. The post-mortem is a system question, not a people question. If the bug was caused by a missed code review or a missed AC, name the system gap (no AC for this case, no review checklist item) — not the reviewer.

### Example output

> **Root cause:** The OAuth callback handler stripped query-string parameters when `redirect_uri` contained a fragment. The URL parser used (`url.parse`) silently drops fragments, so the `state` parameter was lost on round-trip and the CSRF check rejected every callback.
>
> **Why tests missed it:** Integration tests covered the happy-path `redirect_uri` (no fragment). The AC did not specify fragment handling, and no test exercised it.
>
> **Regression test added:** `tests/integration/auth.callback.spec.ts · "preserves state when redirect_uri has fragment"` (in PR #482).
>
> **Monitoring added:** Sentry alert tuned to fire on `OAuthCallbackError` at >5 events/hour (was 50). Dashboard panel for callback success rate.
>
> **Prevention:** ADR-0014 to standardise on `URL` (WHATWG) over `url.parse` (legacy) project-wide. Filed as ENG-518, owner @alice. Lint rule via `no-restricted-imports` on `node:url`'s legacy API.
>
> **Time-to-fix:** 3h 12m (S1 target: this cycle).
>
> _(post-mortem drafted via Claude Code)_
```
