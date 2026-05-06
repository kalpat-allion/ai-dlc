# Phase 4: Testing & QA — Process Definition

## Overview

This document defines the AI-assisted workflow for the Testing & QA phase, using the consolidated AI-DLC tool stack. Linear is the spine — every test plan, test gap, bug, and regression is tracked as a Linear artefact, and Claude Code / Sentry agents drive the lifecycle through MCP. The Sentry Agent for Linear closes the loop from production error to triaged bug to fix PR without humans copy-pasting stack traces.

**Phase Duration:** Runs in parallel with Development (continuous) + a dedicated hardening cycle before each release
**Phase Owner:** QA Lead / Senior Developer
**Tools Used:** Vitest / Jest / pytest / JUnit 5 (unit), Supertest + Testcontainers (integration), **Playwright** (E2E), **k6** (load), Claude Code (test generation, debugging), CodeRabbit + SonarQube (already in Phase 3 stack), **Linear** (project management spine, integrated via MCP), **Sentry** (production error tracking, integrated via MCP and the Sentry Agent for Linear), BrowserStack App Automate (mobile, optional)

> **Tool Philosophy:** Testing is the cheapest phase of the AI-DLC. Frameworks are free OSS. AI test generation is already paid for in the Development stack (Claude Code + Cursor). Bug triage uses tools the team already runs (Linear, Sentry, CodeRabbit). **Linear and Sentry are wired together** through the Sentry Agent for Linear — production errors become triaged Linear issues with Seer root-cause analysis attached, with no human shovelling. The QA Lead's job is calibration and prioritisation, not transcription.

---

## Tool Stack

| Layer | Tool | Purpose | Cost |
|-------|------|---------|------|
| **Unit Testing (JS/TS)** | Vitest (new) / Jest (existing) | Fast unit test framework | Free OSS |
| **Unit Testing (Python)** | pytest | Python standard | Free OSS |
| **Unit Testing (Java)** | JUnit 5 | Java standard | Free OSS |
| **Integration Testing** | Framework + Supertest + Testcontainers | Real dependencies in CI | Free OSS |
| **E2E Testing (Web)** | Playwright | De facto standard in 2026 | Free OSS |
| **Load Testing** | k6 (JS/TS) or Locust (Python) | Developer-friendly load tests | Free OSS |
| **Mobile Testing** (if applicable) | BrowserStack App Automate + Appium | Real-device mobile testing | From $199/mo per parallel |
| **Test Generation** | Claude Code + Cursor | AI-generated tests (already in Phase 3 stack) | Already paid |
| **Test Review** | CodeRabbit | AI review of test code alongside production code | Already in Phase 3 stack |
| **Bug Tracking + AI Triage** | **Linear** + **Triage Intelligence** | Issue tracker + duplicate detection + label/priority suggestions | $10/user/mo Business+ |
| **Error Tracking + RCA** | **Sentry** + **Seer** + **Sentry Agent for Linear** | Production error tracking, AI root-cause analysis, auto-creates Linear issues | From $26/mo |
| **Code Quality in CI** | SonarQube Community | Static analysis + coverage gates | Free OSS |

---

## Process Steps

### Step 0: One-Time Setup — Linear MCP + Sentry MCP + Sentry Agent for Linear

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | Linear workspace (already connected from Phase 1 — see [`01-requirement-gathering/PROCESS.md` Step 0](../01-requirement-gathering/PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp)), Sentry organisation admin, Anthropic Connectors admin (Team/Enterprise) |
| **Tools** | **Linear MCP** (`https://mcp.linear.app/mcp`), **Sentry MCP** (`https://mcp.sentry.dev/mcp`), **Sentry Agent for Linear** (in-app, no MCP — it is a Linear Agent) |
| **Output** | Claude Code can read & write Linear issues + Sentry issues; Sentry can auto-file bug issues into Linear with Seer RCA pre-attached |
| **Human** | Workspace admin enables connectors; QA Lead configures Sentry Agent triage rules; each user authorises once via OAuth |

Phase 4 builds on the Phase 1 Linear MCP setup but adds **two new pieces**: the Sentry MCP server (so Claude Code can query Sentry directly during debugging) and the Sentry Agent for Linear (so production errors auto-flow into Linear's Triage with root-cause analysis attached).

#### Path A — Sentry MCP for Claude Code

1. **Add the Sentry MCP server** at user scope:
   ```bash
   claude mcp add --transport http --scope user sentry https://mcp.sentry.dev/mcp
   ```
2. **Authenticate:** open Claude Code (`claude`) → `/mcp` → select **sentry** → approve OAuth in the browser.
3. **Verify:** `claude mcp list` should show `sentry: connected`. In a session: `Via Sentry MCP, find the top 5 unresolved issues in project <slug> in the last 24h, with stack traces.` Claude should return real Sentry data with affected-user counts.
4. **Tool scopes:** Sentry MCP exposes `find_issues`, `get_issue`, `get_event`, `seer_status`, `seer_run`, `update_issue` (resolve, ignore, assign), and project-level reads. Phase 4 needs **read + Seer + update** — no need to wrap the REST API.
5. **Revoke when done:** `claude mcp remove sentry`, revoke OAuth in Sentry under **Settings → Integrations → Authorised Apps**.

#### Path B — Sentry Agent for Linear (in-app, no CLI)

The Sentry Agent for Linear is a **Linear Agent** (April 2026 onwards) — same primitive as Linear's own AI agents. It is assignable, mentionable, and authenticated via OAuth between Linear and Sentry. Once enabled, production errors flowing through Sentry triage rules can land in Linear automatically with Seer's root-cause analysis pre-attached.

1. **Linear admin:** **Settings → Integrations → Sentry → Connect**. Approve the OAuth scopes.
2. **Sentry admin:** in the Sentry organisation that mirrors the Linear workspace, **Settings → Integrations → Linear**. Map Sentry projects to Linear teams (one-to-one is cleanest — `sentry:web` → `linear:Web` team).
3. **Configure triage rules in Sentry** (Sentry → Alerts → Issue Alerts):
   - Trigger: **A new issue is seen** (or **An issue affects N+ users**, depending on signal-to-noise tolerance).
   - Action: **Create a Linear issue via the Sentry Agent**, in the mapped team's **Triage** state, label `source:sentry` + `severity:auto` + `bug`.
4. **Enable Seer auto-run on the agent.** In Linear → Settings → Integrations → Sentry → **Auto-run Seer RCA on issue creation**: ON. **Auto-create PR**: OFF (we want the human gate at Phase 4 Step 7). **Auto-update status**: ON (so the agent can mark the Linear issue *In Review* when Seer finishes).
5. **Smoke test:** in a non-production project, throw a deliberate error. Within ~60 seconds a Linear issue should appear in Triage with: stack trace, affected users, environment, release version, **Seer RCA** as a comment, and severity proposal. The QA Lead reviews this in Step 7.

#### Tool-permission policy (Anthropic Connectors, Team/Enterprise only)

For **Linear** in the Anthropic admin console, Phase 4 keeps the Phase 3 widening (`update_issue`, `assign_issue`, `create_issue`, `create_comment`) and additionally permits:

- `update_issue` with state transitions to `Resolved` / `Done` (so Claude Code can close bugs after the fix lands)
- `create_issue` with `parentId` + label `regression-test` (for regression-test stories filed when a bug is fixed — see Step 8)

For **Sentry**, enable: `find_issues`, `get_issue`, `get_event`, `seer_status`, `seer_run`. Keep `delete_event` and bulk operations **disabled**.

#### Verification checklist

- [ ] `claude mcp list` shows both `linear: connected` and `sentry: connected`
- [ ] Sentry → Settings → Integrations → Linear shows Connected with project↔team mapping
- [ ] Linear → Settings → Integrations → Sentry: Auto-run Seer ON, Auto-PR OFF, Auto-status ON
- [ ] Smoke test: deliberate error in staging produced a Linear Triage issue with Seer RCA attached
- [ ] Audit logging on in both Linear and Sentry
- [ ] Anthropic Connectors policy reviewed (Team/Enterprise) — see Risks & Guardrails

> **Permission inheritance:** Claude inherits the connecting user's Linear and Sentry permissions. The Sentry Agent for Linear inherits the OAuth scopes the workspace admin approved at integration time — review them; do not approve "all" scopes by default.

---

### Step 1: Test Plan Creation

> Visual: [Step 1 flowchart](./FLOWCHART.md#step-1-test-plan-creation)

| Attribute | Detail |
|-----------|--------|
| **Input** | Approved PRD (Linear Document from Phase 1), user stories with AC, architecture (Phase 2), OpenAPI spec, NFRs |
| **Tool** | **Claude Code** with Linear MCP — reads PRD + architecture, drafts plan, publishes plan to Linear |
| **Output** | Test Plan v1 published as a **Linear Document** attached to the Linear Project, with sections: scope, strategy by test type, AC↔test mapping, test data, environments, entry/exit criteria |
| **Human** | QA Lead validates plan, prioritises critical flows, assigns test ownership |

**Workflow:**

**1.1 — Pull inputs.** In a Claude Code session with Linear MCP enabled, fetch the Phase 1 PRD Document and the Phase 2 architecture/ADRs by URL. **Do not paste** — let Claude read via MCP so trace stays clean.

**1.2 — Draft test plan in Claude.** Run the [`test-plan-generation`](./PROMPTS.md#test-plan-generation) prompt. Claude produces: scope, strategy per test type (unit / integration / E2E / load / mobile if applicable), test cases mapped to every AC, test data strategy, environment needs (dev / staging / perf), entry/exit criteria, risk-based prioritisation. The draft stays in chat as Markdown — no Linear write yet.

**1.3 — Self-review with Claude.** Run the [`test-plan-gap-analysis`](./PROMPTS.md#test-plan-gap-analysis) prompt against the draft. Claude flags ACs without mapped tests, NFRs without load coverage, integration boundaries without contract tests, missing edge cases. Resolve every Critical/High before publishing.

**1.4 — Publish to Linear.** Run the [`test-plan-to-linear-document`](./PROMPTS.md#test-plan-to-linear-document) prompt. Claude Code creates a **Linear Document** attached to the Phase 1 Linear Project titled `Test Plan v1`. Stable section anchors (`§1 Scope`, `§4 AC Map`, etc.) let bugs and test issues deep-link back into the right plan section.

**1.5 — Map test ownership.** QA Lead works through the AC↔test map and assigns each row: developers own unit + integration; QA or developer pairs own E2E; performance specialist (or senior dev) owns load. Ownership is annotated **inside the Linear Document** under each section — not a separate spreadsheet.

> **Gate 1 (Test Plan approved) must pass before tests are written.** See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-test-plan-approved). The Linear Document marked `Approved v1.0` is the contract; Phase 4 work cites a section of it.

**Escalation:** If the PRD has ambiguous acceptance criteria that cannot be translated into test cases, **loop back to Phase 1** — comment on the original PRD Document, do not write tests against ambiguity.

---

### Step 2: Unit Testing (Continuous, Parallel with Development)

> Visual: [Step 2 flowchart](./FLOWCHART.md#step-2-unit-testing)

| Attribute | Detail |
|-----------|--------|
| **Input** | Implementation code from each Phase 3 development task |
| **Tool** | **Vitest / Jest / pytest / JUnit 5** + **Claude Code** for generation + **CodeRabbit** for review |
| **Output** | Unit test suite with ≥ 80% coverage on new code, every AC mapped to ≥ 1 test |
| **Human** | Developer reviews AI-generated tests for correctness; QA Lead audits test plan coverage at sprint end |

**Workflow:**

**2.1 — Tests live with code.** Unit tests are part of the Phase 3 story Definition of Done — a story without unit tests does not pass Phase 3 Gate 1. Phase 4 audits and supplements; it does not start a parallel test track.

**2.2 — Generate with Claude Code.** Run the [`unit-test-generation`](./PROMPTS.md#unit-test-generation) prompt. Provide: source file, test framework, an example test from the project for style. Claude produces tests covering happy path, error handling, edge cases, and one test per acceptance criterion.

**2.3 — Review the AI tests line by line.** Common AI test failure modes: assertions that always pass (`expect(result).toBeDefined()` when result is always defined), testing implementation instead of behaviour (snapshotting internals), missing edge cases (empty arrays, nulls, concurrency), unrealistic test data (`"foo"`, `"test123"`). Reject and rewrite — a test that always passes is worse than no test.

**2.4 — CI enforcement.** SonarQube enforces coverage ≥ 80% on **new** code per PR. Drop in coverage on existing code is permitted but logged; concentrated drop > 5% triggers a `tech-debt` ticket.

**2.5 — Sprint-end coverage audit.** Each sprint close, the QA Lead runs the [`test-coverage-gap-analysis`](./PROMPTS.md#test-coverage-gap-analysis) prompt against the changed modules. Gaps become Linear issues labelled `phase:qa` + `coverage-gap`, scheduled for the next cycle.

**Coding standard:** AI-generated tests must be reviewed by a human. Tests that always pass are worse than no tests.

---

### Step 3: Integration Testing

> Visual: [Step 3 flowchart](./FLOWCHART.md#step-3-integration-testing)

| Attribute | Detail |
|-----------|--------|
| **Input** | API contracts (Phase 2 OpenAPI), data model, implementation code |
| **Tool** | **Same framework as unit tests** + **Supertest** (HTTP) + **Testcontainers** (real DB / brokers in Docker) |
| **Output** | Integration test suite covering API endpoints and service boundaries |
| **Human** | QA Lead validates critical integration paths, reviews test data strategy |

**Workflow:**

**3.1 — Generate per endpoint.** Run the [`integration-test-generation`](./PROMPTS.md#integration-test-generation) prompt. Feed the OpenAPI spec section and the schema entities. Claude produces tests covering: full request/response cycle, all status codes (200/201/400/401/403/404/500), pagination on list endpoints, filter/sort query params, auth + permissions, data persistence verification.

**3.2 — Real dependencies, not mocks.** Use Testcontainers to spin up a real PostgreSQL/Redis/Kafka in Docker for the test run. **Do not mock the database in integration tests** — that is what unit tests are for. Mocking the database in integration tests is one of the most common AI generation mistakes; reject those tests.

**3.3 — Contract tests for microservices.** For multi-service projects, add **Pact** (free OSS) consumer-driven contract tests at service boundaries. Pair with the OpenAPI contract test (Schemathesis or Dredd) running against the OpenAPI spec — together they catch both consumer breakage and spec drift.

**3.4 — Coverage target.** ≥ 60% of API endpoints have an integration test covering happy path + one error case. Endpoints handling money, auth, or PII are 100% — no exceptions.

**3.5 — Run in staging-like CI.** Integration tests run in CI on every PR to main, against a Testcontainers stack. Once-a-night, run them against staging itself to catch infrastructure drift.

---

### Step 4: End-to-End Testing (Web)

> Visual: [Step 4 flowchart](./FLOWCHART.md#step-4-e2e-testing-web)

| Attribute | Detail |
|-----------|--------|
| **Input** | User flows from PRD, wireframes from Phase 2, staging environment URL |
| **Tool** | **Playwright** (framework) + **Claude Code** (test generation + debugging) |
| **Output** | E2E test suite covering critical user journeys |
| **Human** | QA Lead validates test flows match real behaviour, debugs flaky tests |

**Workflow:**

**4.1 — Identify critical journeys.** Not every flow gets E2E tests. From the test plan (Step 1) take the **revenue-impacting and safety-critical paths only**: login, checkout, data sync, signup, password reset, the critical happy path of every billable feature. Comprehensive E2E coverage is the most expensive test layer; spend it where regressions are unacceptable.

**4.2 — Generate with Claude Code.** Run the [`playwright-e2e-generation`](./PROMPTS.md#playwright-e2e-test-generation) prompt. Provide: user story, deployed staging URL, test user credentials (or env-var pattern), seed data approach. Claude produces a test that:
   - Uses **role-based locators** (`page.getByRole('button', { name: 'Submit' })`) — not CSS classes
   - Relies on Playwright's **auto-waiting** — no `page.waitForTimeout`
   - Has **one assertion per logical step** with clear messages
   - Is **isolated** — each test creates and cleans its own data

**4.3 — Run in CI on every PR.** Failing E2E tests block merge to main. Configure `trace: 'on-first-retry'` so Trace Viewer can replay failures during debugging.

**4.4 — Manage flakiness aggressively.** Any test failing > 2% of runs over a week is **fixed or quarantined** with a tracking ticket. Never merge with flaky tests in the suite — flake erodes trust until the team starts ignoring real failures.

**4.5 — Debug failures with Claude Code.** When a test fails, run the [`playwright-failure-debug`](./PROMPTS.md#playwright-failure-debug) prompt with the trace file path. Claude reads the trace, identifies the divergence, and proposes a fix to either the test or the code under test.

**Escalation:** If flakiness exceeds 2% globally after hardening, evaluate an AI-native E2E platform (Momentic, QA.tech) for self-healing locators. See the [Phase 4 Tools Evaluation](../../docs/tools-evaluation/4.TestingQA_Phase_Tools.md).

---

### Step 5: Mobile App Testing (if applicable)

> Visual: [Step 5 flowchart](./FLOWCHART.md#step-5-mobile-app-testing)

| Attribute | Detail |
|-----------|--------|
| **Input** | Native iOS/Android app builds, critical mobile user flows |
| **Tool** | **BrowserStack App Automate** + **Appium** (or Espresso / XCUITest / Detox) |
| **Output** | Mobile E2E suite running on the target real-device matrix |
| **Human** | QA Lead validates coverage across OS versions and device types |

**Workflow:**

**5.1 — Skip if not applicable.** Web-only projects → use Playwright's mobile emulation (Step 4) and skip to Step 6. Native iOS/Android only.

**5.2 — Generate Appium tests.** Run the [`mobile-test-generation`](./PROMPTS.md#mobile-test-generation-appium--browserstack) prompt. Choose framework based on platform: Appium (cross-platform), Espresso (Android-only), XCUITest (iOS-only), Detox (React Native).

**5.3 — Run on BrowserStack App Automate.** Real devices covering the top OS/device combinations the production user base actually has — pull this from analytics, not assumptions. BrowserStack's **AI Self-Healing Agent** auto-updates locators when UI changes (the most common mobile test failure cause).

**5.4 — Wire to CI.** New app build → BrowserStack triggers the suite → results post back to the PR. **Test Failure Analysis Agent** (BrowserStack feature) categorises failures as production bug / automation error / environment issue — cuts triage time by ~95%.

**Cost note:** $199–597/month for 1–3 parallel sessions (annual billing). Add only if mobile is in scope.

---

### Step 6: Load Testing

> Visual: [Step 6 flowchart](./FLOWCHART.md#step-6-load-testing)

| Attribute | Detail |
|-----------|--------|
| **Input** | NFRs from Phase 1, OpenAPI spec, expected traffic patterns |
| **Tool** | **k6** (JS/TS) or **Locust** (Python) |
| **Output** | Load test suite validating performance against NFR targets |
| **Human** | Performance owner defines realistic traffic patterns, interprets results |

**Workflow:**

**6.1 — Generate k6 scripts.** Run the [`k6-load-generation`](./PROMPTS.md#k6-load-test-generation) prompt. Feed: OpenAPI spec, expected RPS at peak, ramp-up/steady/ramp-down durations, user behaviour pattern (e.g., `login → browse → search → checkout`), NFR targets (p95 latency, p99 latency, error rate).

**6.2 — Model realistic traffic.** Work backwards from real traffic data — **do not load-test paranoia**. A site with 100K daily visitors over 10 hours has a different concurrency profile from the same volume in 2 hours. The load test that proves the NFR is the one that mirrors production behaviour.

**6.3 — Run in a staging environment that mirrors prod.** Load tests in a tiny dev environment prove nothing. Provision performance-matched infrastructure for the test, then tear it down.

**6.4 — Wire to release-candidate gate.** Phase 4 Gate 3 fails the release candidate if p95 / p99 / error rate / concurrent-user capacity fall below NFR. The decision belongs to the Tech Lead, not the script — a load test failure may be the test, not the code.

**6.5 — Distributed testing (optional).** Grafana Cloud k6 from $29/month for multi-region load. Add when local k6 cannot generate enough load or geographic distribution matters (CDN, latency-sensitive APIs).

---

### Step 7: Bug Intake & Triage — Sentry → Linear via the Sentry Agent

> Visual: [Step 7 flowchart](./FLOWCHART.md#step-7-bug-intake--triage)

| Attribute | Detail |
|-----------|--------|
| **Input** | Bugs from manual QA, automated test failures, **production errors flowing in via Sentry**, user reports |
| **Tool** | **Linear** (tracker) + **Triage Intelligence** (duplicate detection, label/priority suggestions) + **Sentry** (production error tracking) + **Sentry Agent for Linear** (auto-creates Linear issues with Seer RCA) + **Claude Code** (Sentry MCP queries during investigation) |
| **Output** | Triaged bug backlog with severity, owner, fix plan; every bug carries Seer RCA or Claude RCA; duplicates collapsed |
| **Human** | QA Lead validates severity and assignment, decides fix priority, verifies fixes |

The bug pipeline is the most-automated workflow in Phase 4. The Sentry Agent for Linear handles steps **1–4** below without human input; the QA Lead enters at step 5 to validate.

**Workflow:**

**7.1 — Production error fires in Sentry.** Sentry's existing alert rules detect a new issue (or one crossing affected-user threshold).

**7.2 — Sentry Agent files a Linear issue.** The triage rule configured in Step 0.B.3 routes the alert to the Sentry Agent for Linear. The agent calls Linear's `create_issue` API to file a new issue in the mapped team's **Triage** state, with: title from Sentry's issue summary, description containing stack trace + affected users + environment + release version, labels `source:sentry` + `bug` + auto-severity.

**7.3 — Seer RCA runs.** Because **auto-run Seer** is enabled (Step 0.B.4), the Sentry Agent kicks off Seer root-cause analysis immediately. Within a few minutes Seer posts a comment on the Linear issue: probable root cause, the line of code suspected, the fix proposal. The Linear issue moves from *Triage* to *In Review*.

**7.4 — Triage Intelligence checks for duplicates.** Linear's Triage Intelligence runs semantic similarity against existing issues. If a strong match appears, it surfaces a "merge as related" suggestion on the issue. The QA Lead either accepts (and the duplicate is closed pointing at the original) or rejects.

**7.5 — QA Lead reviews and validates** (the human gate). Open the Linear `Bug Triage` saved view (filter: `state = Triage OR (label = source:sentry AND state = In Review)`). For each issue:
   - **Validate severity.** Sentry's auto-severity is a starting point — adjust against the [bug severity scale](#bug-severity-scale) below using business impact, not just affected-user count.
   - **Validate Seer's RCA.** If correct, accept; if wrong, run the [`debugging`](./PROMPTS.md#debugging-bug-investigation) prompt in Claude Code (which uses Sentry MCP to fetch fresh stack traces and Linear MCP to comment back).
   - **Assign.** Auto-assignee suggestions from Linear are usually right — confirm or override.
   - **Label.** Add `phase:qa`, `severity:S0/S1/S2/S3`. Remove `severity:auto`.
   - **Set fix urgency.** S0 → fix immediately; S1 → this cycle; S2 → next cycle; S3 → backlog.

**7.6 — Manual bugs (non-Sentry).** QA-discovered or user-reported bugs are filed in Linear directly with label `source:qa` or `source:user`. Linear's Triage Intelligence still runs duplicate detection and label suggestions on these. Otherwise the same triage flow applies from 7.5.

#### Bug severity scale

- **S0 (Critical):** Production down, data loss, security breach. Fix immediately; rollback if no fix in < 1h.
- **S1 (High):** Major feature broken, no workaround. Fix this cycle; release blocker.
- **S2 (Medium):** Feature broken with workaround, or minor feature broken. Fix within 2 cycles.
- **S3 (Low):** Cosmetic, nice-to-have. Backlog.

---

### Step 8: Bug Fix Loop — Claude Code + Sentry MCP + Linear MCP

> Visual: [Step 8 flowchart](./FLOWCHART.md#step-8-bug-fix-loop)

| Attribute | Detail |
|-----------|--------|
| **Input** | Triaged bug in Linear (with Seer RCA and stack trace attached) |
| **Tool** | **Claude Code** with both Linear MCP and Sentry MCP enabled |
| **Output** | Fix PR through Phase 3 Step 4 review gate, regression test added, Linear issue auto-closed on merge |
| **Human** | Developer reviews the fix and the regression test before opening the PR; QA verifies in staging after merge |

**Workflow:**

**8.1 — Pick up the bug as a Phase 3 task.** Run the same [`linear-next-task`](../03-development/PROMPTS.md#linear-next-task) prompt the developer uses for stories — bugs are issues, the workflow is the same. Claude Code transitions the issue to *In Progress*, self-assigns, checks out the Linear `branchName`.

**8.2 — Pull the Sentry context via MCP.** Run the [`sentry-context-pull`](./PROMPTS.md#sentry-context-pull) prompt. Claude Code uses Sentry MCP to fetch: the latest event for this issue, the stack trace, the affected-user list, the release version that introduced it, breadcrumbs leading to the error. This is fresh data — Seer's RCA on the Linear issue may be from hours ago.

**8.3 — Reproduce locally.** Use the [`bug-reproduction`](./PROMPTS.md#bug-reproduction) prompt with the stack trace and reproduction steps. Claude proposes a failing test that captures the bug — **write this test first**. A bug that cannot be reproduced is a bug that will recur.

**8.4 — Fix.** Apply Seer's proposed fix (if it was correct in step 7.5) or run the [`debugging`](./PROMPTS.md#debugging-bug-investigation) prompt. Claude reads the relevant code paths, traces data flow, proposes a targeted fix.

**8.5 — Add the regression test.** The failing test from 8.3 now passes — keep it. **Every bug fix PR includes the regression test in the same diff.** Tests added later drift; tests in the same commit as the fix do not.

**8.6 — Open the PR through the Phase 3 review gate.** Title: `[ENG-XXX] Fix <bug summary>`. Body: `Closes ENG-XXX` plus a short post-mortem (root cause, fix, regression test, prevention idea). Phase 3 Step 4 governs the review.

**8.7 — On merge:**
   - Linear auto-transitions the bug issue to **Done** via the git integration.
   - Claude Code (or the Sentry Agent — admin choice) calls Sentry's `update_issue` to mark the Sentry issue as **Resolved in version `<release>`**.
   - QA verifies in staging and adds a comment on the Linear issue: `Verified in staging by <user> on <date>`.

**8.8 — Post-mortem for S0/S1.** Every S0 and S1 bug closes with a brief post-mortem comment on the Linear issue: root cause (one paragraph), why tests missed it, what we will add (regression test + monitoring), owner of the prevention work. Run the [`post-mortem`](./PROMPTS.md#post-mortem) prompt to seed the draft.

---

## Phase Handoff

When Testing & QA is complete (all gates passed), the following artefacts hand off to **Phase 5: Security & Compliance**:

| Artefact | Format | Location |
|----------|--------|----------|
| Test Plan (final) | Linear Document attached to the Linear Project | **Linear Project → Documents** |
| Test Suites (unit / integration / E2E / load / mobile) | Code | Test directories in repo |
| Coverage Report | HTML + JSON | CI artefacts + SonarQube dashboard |
| Load Test Results | k6 reports | Archived in CI artefacts; summary in Linear comment on the cycle |
| Known Issues Log | Linear saved view (`label = bug AND state != Done`) | Linear |
| Regression Test Baseline | Code (test files committed alongside fixes) | Repo |
| Sentry Resolved Issues Log | Sentry view | Sentry organisation |
| Post-mortems (S0/S1) | Linear comments on the originating bug issues | Linear |

**Handoff Checklist:**
- [ ] Test plan v1 (or latest) marked **Approved** as a Linear Document
- [ ] All test types implemented per the test plan (unit, integration, E2E, load, mobile if applicable)
- [ ] Unit test coverage ≥ 80% on new code (SonarQube verified)
- [ ] Integration tests cover all API endpoints (happy path + error case); 100% on auth/payments/PII paths
- [ ] E2E tests cover all critical user journeys; flakiness rate < 2%
- [ ] Load test results meet NFR targets from Phase 1
- [ ] Mobile tests pass on the target device matrix (if applicable)
- [ ] **0 S0 / 0 S1** Linear bugs open on main
- [ ] All S2 bugs triaged with documented fix plan
- [ ] Every S0/S1 fix has a regression test in the same PR
- [ ] Every S0/S1 has a post-mortem comment on the Linear issue
- [ ] Sentry-Linear integration verified end-to-end on a real bug in this cycle
- [ ] Saved view `phase:qa AND state != Done` reviewed and triaged

---

## Linear Workspace Setup (Phase-4 additions)

Phase 1 and 3 setup carries over (Initiative → Project → Milestone → Issue, plus the standard labels). Phase 4 adds:

**Labels:**

| Label | Meaning |
|-------|---------|
| `bug` | Bug — anything Phase 4 owns the lifecycle for |
| `source:sentry` | Filed automatically by the Sentry Agent for Linear |
| `source:qa` | Filed by QA from manual or exploratory testing |
| `source:user` | Filed from a user/customer report |
| `severity:S0`, `severity:S1`, `severity:S2`, `severity:S3` | Triaged severity (after step 7.5) |
| `severity:auto` | Sentry's auto-severity, before human triage |
| `regression-test` | Sub-issue tracking the regression test for a specific bug |
| `coverage-gap` | Filed by the sprint-end coverage audit (Step 2.5) |
| `flaky-test` | Test quarantined as flaky pending fix (Step 4.4) |

**Saved views the QA Lead should configure:**

1. **Bug Triage** — `state = Triage OR (label = source:sentry AND state = In Review)` (the daily triage queue)
2. **Open S0/S1** — `label IN (severity:S0, severity:S1) AND state != Done` (release blockers)
3. **Open Bugs (all)** — `label = bug AND state != Done`
4. **Phase 4 Active** — `label = phase:qa AND state != Done`
5. **Coverage & Flake Backlog** — `label IN (coverage-gap, flaky-test) AND state != Done`

---

## Test-and-Bug Loop (end-to-end)

```
[Phase 3 handoff] Code merged to main, unit tests in place, Linear cycle closed
   │
[Step 1.1-1.5] Claude Code + Linear MCP → Test Plan v1 published as Linear Document
   │
   ▼  GATE 1: Test Plan approved (QA Lead + Tech Lead sign-off in the Linear Document)
   │
[Step 2-6 in parallel during the next development cycle]
   ├─ Unit tests (Step 2): with code, ≥ 80% coverage on new code
   ├─ Integration tests (Step 3): with each new endpoint
   ├─ E2E tests (Step 4): per critical journey
   ├─ Mobile tests (Step 5): if applicable
   └─ Load tests (Step 6): nightly + before each release candidate
   │
   ▼  GATE 2: Per-cycle test coverage gates pass
   │
[Step 7] Bug intake (continuous)
   ├─ Sentry → Sentry Agent for Linear → Linear Triage with Seer RCA attached
   ├─ Manual QA / user reports → Linear directly with label source:qa or source:user
   ├─ Triage Intelligence flags duplicates and proposes labels/priority
   └─ QA Lead validates severity, assignee, label (the human gate)
   │
[Step 8] Bug fix loop (Claude Code + Linear MCP + Sentry MCP)
   ├─ linear-next-task → fetch bug → In Progress → branchName checkout
   ├─ sentry-context-pull → fresh stack trace + breadcrumbs
   ├─ Reproduce with a failing test
   ├─ Fix; the failing test now passes (regression test, kept in the diff)
   ├─ PR through Phase 3 Step 4 review gate
   └─ On merge: Linear auto-Done; Sentry marked Resolved; QA verifies in staging
   │
   ▼  GATE 3: Release candidate validation
   ├─ All test types green
   ├─ Load tests meet NFRs
   ├─ 0 S0 / 0 S1 open
   └─ Full E2E regression suite green; flakiness < 2%
   │
   ▼  GATE 4: Phase Handoff
   │
[Phase 5: Security & Compliance]
```

Four explicit gates ensure that **no AI-triaged bug or AI-generated test reaches main without human validation**, every bug fix carries a regression test in the same diff, and every production error has an audit trail from Sentry → Linear → fix PR → resolved release.

---

## Risks & Guardrails

| Risk | Mitigation |
|------|------------|
| **AI tests that always pass** — Claude generates `expect(result).toBeDefined()` style assertions that look like tests but assert nothing | Step 2.3 mandates line-by-line review. CodeRabbit flags trivially-true assertions in 80%+ of cases. Sprint-end audit (Step 2.5) re-checks via the coverage-gap prompt. |
| **AI tests that mock the database in integration tests** — most common AI mistake on integration tests | Step 3.2 — reject these tests at PR review. Testcontainers must be running for the integration suite; the test must hit a real DB. |
| **Sentry alert spam → Linear flooded with auto-issues** — too-broad triage rules drown Triage Intelligence | Step 0.B.3 starts with the conservative trigger ("affects N+ users" not "any new issue"). Triage Intelligence collapses duplicates, but volume past ~30/day overwhelms the QA Lead. Re-tune triggers monthly. |
| **Seer RCA hallucination** — Seer proposes a fix for the wrong line of code | Step 7.5 — QA validates the RCA, not just the severity. If wrong, the developer runs the debugging prompt with Sentry MCP to investigate live, then fixes. Track Seer accuracy per project; if < 60% accept rate over a month, disable auto-run and use Seer on-demand only. |
| **Regression tests committed in a follow-up PR (and forgotten)** — the bug recurs | Step 8.5 / 8.6 — the regression test ships in the same diff as the fix. Reviewers reject fix PRs without the regression test. |
| **Flaky E2E tests merged with `.skip` and a TODO** — flake decay | Step 4.4 — quarantine via a tracking issue (`flaky-test` label), never `.skip` without that issue. The flake quarantine cannot exceed two cycles before fix-or-delete. |
| **Production-bug PRs auto-closing the wrong Linear issue** — wrong identifier in the title | Phase 3 Step 4.5's sanity check carries over. Bugs especially: Seer's auto-filed Linear issue and the developer's PR must reference the same identifier. The `[ENG-XXX]` title rule is non-negotiable. |
| **Sentry MCP scope overreach** — `delete_event` or bulk `update_issue` enabled and accidentally fires on the whole project | Step 0 admin policy keeps `delete_event` and bulk operations disabled. Off-board by revoking OAuth in Sentry. |
| **Linear comment spam on bug issues** — Sentry Agent + Seer + Claude all comment, the actual triage thread drowns | Convention: Seer posts one RCA comment; the developer posts one fix comment on PR open; QA posts one verification comment. Do not narrate progress on bug issues — the PR is the narrative. |
| **Load tests pass against an under-provisioned staging environment** — green tests, red production | Step 6.3 — staging must be performance-matched to production for load tests, not the standard staging. Provision and tear down for the run. |

---

## Related Documents

- [Prompt Templates →](./PROMPTS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Process Flowchart →](./FLOWCHART.md)
- [Test Plan Template →](../templates/test-plan-template.md)
- [Bug Report Template →](../templates/bug-report-template.md)
- [Code Review Checklist →](../templates/code-review-checklist.md)
- [Phase 1 Linear MCP setup (Step 0) →](../01-requirement-gathering/PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp)
- [Phase 3 Development PROCESS →](../03-development/PROCESS.md)

## External References

- [Linear MCP server docs](https://linear.app/docs/mcp) — server URL, OAuth, tool families
- [Linear Agent MCP support — Changelog (April 2026)](https://linear.app/changelog/2026-04-23-linear-agent-mcp-support)
- [Linear Triage Intelligence](https://linear.app/docs/triage-intelligence) — duplicate detection, label and priority suggestions
- [Linear Triage docs](https://linear.app/docs/triage)
- [Sentry MCP server](https://docs.sentry.io/ai/mcp/) — `https://mcp.sentry.dev/mcp`, OAuth, tools
- [Sentry Agent for Linear](https://docs.sentry.io/integrations/issue-tracking/sentry-linear-agent/) — assign to Sentry, auto Seer, triage rules
- [Sentry MCP cookbook — automate triage with Claude Code + Sentry](https://sentry.io/cookbook/performance-bot-sentry-claude/)
- [Playwright (test framework)](https://playwright.dev/) — auto-waiting, role locators, Trace Viewer
- [k6 (load testing)](https://k6.io/) — JS/TS-based load testing
- [Testcontainers](https://testcontainers.com/) — real dependencies in tests
- [Pact (consumer-driven contracts)](https://pact.io/) — service-boundary contract tests
- [BrowserStack App Automate](https://www.browserstack.com/app-automate) — real-device mobile testing + AI Self-Healing Agent
- [Phase 4 Tools Evaluation](../../docs/tools-evaluation/4.TestingQA_Phase_Tools.md)
