# Phase 4: Testing & QA — Process Flowcharts

The phase is split into nine per-step flowcharts so each can be navigated, embedded in step-specific docs, or printed independently. The underlying process, sub-stages, and gate criteria live in [PROCESS.md](./PROCESS.md) and [QUALITY-GATES.md](./QUALITY-GATES.md); the diagrams here mirror that source-of-truth and chain end-to-end.

## Table of Contents

- [Step 0: One-Time Setup](#step-0-one-time-setup)
- [Step 1: Test Plan Creation](#step-1-test-plan-creation)
- [Step 2: Unit Testing](#step-2-unit-testing)
- [Step 3: Integration Testing](#step-3-integration-testing)
- [Step 4: E2E Testing (Web)](#step-4-e2e-testing-web)
- [Step 5: Mobile App Testing](#step-5-mobile-app-testing)
- [Step 6: Load Testing](#step-6-load-testing)
- [Step 7: Bug Intake & Triage](#step-7-bug-intake--triage)
- [Step 8: Bug Fix Loop](#step-8-bug-fix-loop)
- [Daily QA Workflow](#daily-qa-workflow)
- [Four Human Gates](#four-human-gates)

## Legend

| Symbol | Meaning |
|--------|---------|
| 🤖 | AI/tool-driven action (Claude Code, Playwright, k6, CodeRabbit, BrowserStack) |
| 🔌 | Claude Code calling the **Linear MCP** server (read or write) |
| 🔁 | Auto-transition driven by Linear ↔ git integration (PR open / merge) |
| 👤 | Human-led action |
| Diamond | Decision point or quality gate |
| Dark navy node | Phase / step entry or exit |
| Purple node | One-time setup callout (Step 0) |
| Blue node | Linear MCP write action |
| Green node | Auto-transition driven by Linear ↔ git integration |
| Red node | Bug-pipeline action |
| Amber node | Fallback / escalation branch |

## Abbreviations

| Abbreviation | Meaning |
|--------------|---------|
| AC | Acceptance Criteria |
| ADR | Architecture Decision Record |
| AI | Artificial Intelligence |
| CI | Continuous Integration |
| DB | Database |
| E2E | End-to-End (testing) |
| MCP | Model Context Protocol |
| NFR | Non-Functional Requirement |
| OAuth | Open Authorization |
| OpenAPI | Open API Specification |
| p95 / p99 | 95th / 99th percentile latency |
| PII | Personally Identifiable Information |
| PR | Pull Request |
| PRD | Product Requirements Document |
| QA | Quality Assurance |
| RC | Release Candidate |
| RCA | Root Cause Analysis |
| RPS | Requests Per Second |
| S0 / S1 / S2 / S3 | Severity tiers (Critical / High / Medium / Low) |
| SAST | Static Application Security Testing |
| SR | Screen Reader |
| WCAG | Web Content Accessibility Guidelines |

---

## Step 0: One-Time Setup

Phase 4 adds no new MCP server. The Linear MCP connection from Phase 1 (with Phase 3's widened scopes) is the only integration this phase needs, because bugs are filed by people rather than by an automated error feed. Step 0 therefore confirms the carry-over, widens Claude Code's Linear tool permissions to cover Resolved/Done transitions and regression-test sub-issues, then creates the Phase 4 labels and the Bug Triage saved view that Step 7 runs on.

```mermaid
flowchart TD
    S0_START([Start: Phase 4 prerequisites<br/>Linear MCP from Phase 1 with Phase 3 scopes<br/>+ Claude Code tool-permission settings]) --> S0_MCP

    S0_MCP[Confirm the Linear MCP carry-over<br/>claude mcp list shows linear connected<br/>else re-add at --scope project and approve OAuth<br/>smoke test - list open bug issues for the team<br/>🔌 Claude Code + Linear MCP + 👤 each developer/QA]
    S0_MCP --> S0_PERM

    S0_PERM[Tech Lead - Claude Code tool-permission settings:<br/>Linear - widen to allow Resolved/Done transitions<br/>and create_issue with parentId + regression-test label<br/>👤 Tech Lead]
    S0_PERM --> S0_LABELS

    S0_LABELS[QA Lead - Phase 4 workspace labels<br/>bug, source:qa, source:user,<br/>severity:S0 to severity:S3,<br/>regression-test, coverage-gap, flaky-test<br/>👤 QA Lead]
    S0_LABELS --> S0_VIEW

    S0_VIEW[QA Lead - Bug Triage saved view<br/>filter state = Triage<br/>confirm Triage Intelligence is on for the team<br/>👤 QA Lead + 🤖 Triage Intelligence]
    S0_VIEW --> S0_VERIFY

    S0_VERIFY{Verification checklist:<br/>linear connected and smoke-tested,<br/>tool permissions widened,<br/>Phase 4 labels created,<br/>Bug Triage view live,<br/>Triage Intelligence on,<br/>audit logs on?}

    S0_VERIFY -- No --> S0_MCP
    S0_VERIFY -- Yes --> S0_END([Setup complete<br/>Ready for Step 1: Test Plan Creation])

    style S0_START fill:#1B3A5C,color:#fff
    style S0_END fill:#1B3A5C,color:#fff
    style S0_MCP fill:#3D6B9F,color:#fff
    style S0_PERM fill:#5C2E8A,color:#fff
    style S0_LABELS fill:#5C2E8A,color:#fff
    style S0_VIEW fill:#5C2E8A,color:#fff
```

---

## Step 1: Test Plan Creation

Entry point is the Phase 1 PRD Document and the Phase 2 architecture/ADRs/OpenAPI on Linear. Sub-stages 1.1 → 1.5 read inputs via Linear MCP, draft the test plan in Claude Code, run a self-review gap-analysis, publish the plan as a Linear Document attached to the Project, and assign test ownership inside the Document. Gate 1 is QA Lead + Tech Lead approval — on No, the loop returns to 1.2 to regenerate. See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-test-plan-approved).

```mermaid
flowchart TD
    TP_IN([From Phase 3 handoff: PRD + architecture<br/>+ ADRs + OpenAPI + accepted backlog]) --> TP1

    TP1[1.1 Pull inputs via Linear MCP<br/>read PRD Document + ADRs + OpenAPI by URL<br/>do not paste - keep trace clean<br/>🔌 Claude Code + Linear MCP] --> TP2
    TP2[1.2 test-plan-generation prompt<br/>scope, strategy per type, AC↔test map,<br/>test data, environments, entry/exit criteria,<br/>risk-based prioritisation - chat output only<br/>🤖 Claude Code] --> TP3
    TP3[1.3 test-plan-gap-analysis<br/>flag ACs without mapped tests,<br/>NFRs without load coverage,<br/>boundaries without contract tests,<br/>missing edge cases<br/>🤖 Claude Code] --> TP4

    TP4[1.4 test-plan-to-linear-document<br/>create Linear Document attached to Project<br/>title Test Plan v1<br/>stable section anchors §1 Scope, §4 AC Map ...<br/>🔌 Claude Code + Linear MCP] --> TP5
    TP5[1.5 Map test ownership inside the Document<br/>devs own unit + integration,<br/>QA/dev pairs own E2E,<br/>perf owner owns load<br/>👤 QA Lead] --> G1

    G1{GATE 1: Test Plan approved?<br/>QA Lead + Tech Lead sign off in the Linear Document}
    G1 -- No --> TP2
    G1 -- Yes --> TP_OUT([To Steps 2-6 in parallel<br/>Inputs: approved Test Plan v1])

    style TP_IN fill:#1B3A5C,color:#fff
    style TP_OUT fill:#1B3A5C,color:#fff
    style TP1 fill:#3D6B9F,color:#fff
    style TP4 fill:#3D6B9F,color:#fff
```

---

## Step 2: Unit Testing

Entry point is each Phase 3 development task. Sub-stages 2.1 → 2.5 keep tests next to code (Phase 3 DoD), generate with Claude Code, review line-by-line for fake assertions, enforce coverage in CI, and audit gaps at sprint end. There is no dedicated gate at Step 2 — the per-PR coverage gate is part of Phase 3 review; Phase 4 audits at cycle close. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-per-sprint--test-coverage).

```mermaid
flowchart TD
    UT_IN([From each Phase 3 dev task<br/>implementation code + AC]) --> UT1

    UT1[2.1 Tests live with code<br/>part of Phase 3 story DoD<br/>👤 developer] --> UT2
    UT2[2.2 unit-test-generation<br/>source file + framework + example test<br/>covers happy + error + edge + AC mapping<br/>🤖 Claude Code] --> UT3

    UT3[2.3 Review tests line-by-line<br/>reject:<br/>always-true assertions,<br/>testing implementation not behaviour,<br/>missing edge cases,<br/>unrealistic test data<br/>👤 developer] --> UT4
    UT4[2.4 CI enforcement<br/>test runner coverage threshold<br/>greater than or equal to 80 percent on new code<br/>fails the CI job below target<br/>report published as a CI artefact<br/>🤖 CI] --> UT_END

    UT_END --> UT5{Sprint end?}
    UT5 -- Yes --> UT5A[2.5 test-coverage-gap-analysis<br/>across changed modules<br/>gaps become Linear issues<br/>label coverage-gap + phase:qa<br/>🔌 Claude Code + Linear MCP]
    UT5A --> UT_OUT
    UT5 -- No --> UT_OUT([Continuous - feeds Step 7<br/>if a test catches a regression])

    style UT_IN fill:#1B3A5C,color:#fff
    style UT_OUT fill:#1B3A5C,color:#fff
    style UT5A fill:#3D6B9F,color:#fff
```

---

## Step 3: Integration Testing

Entry point is the Phase 2 OpenAPI spec, the data model, and implementation code. Sub-stages 3.1 → 3.5 generate per-endpoint integration tests, enforce real dependencies via Testcontainers (no mocked DBs), add Pact contract tests for microservices, hit the coverage target (≥ 60% endpoints, 100% on auth/payments/PII), and run in staging-like CI. There is no dedicated gate — feeds the Gate 3 release-candidate validation.

```mermaid
flowchart TD
    IT_IN([From Phase 2 OpenAPI + data model<br/>+ implementation code]) --> IT1

    IT1[3.1 integration-test-generation<br/>per endpoint - covers all status codes,<br/>pagination, filtering, auth/permissions,<br/>persistence verification<br/>🤖 Claude Code] --> IT2
    IT2[3.2 Real dependencies via Testcontainers<br/>real PostgreSQL/Redis/Kafka in Docker<br/>NEVER mock the DB at this layer<br/>reject AI tests that mock the DB<br/>👤 developer + 🤖 Testcontainers] --> IT3

    IT3{Microservices?}
    IT3 -- Yes --> IT3A[3.3 Pact contract tests<br/>at service boundaries<br/>+ Schemathesis/Dredd vs OpenAPI spec<br/>🤖 Pact + Schemathesis]
    IT3A --> IT4
    IT3 -- No --> IT4

    IT4[3.4 Coverage target<br/>greater than or equal to 60 percent endpoints<br/>happy + one error case<br/>100 percent on auth, payments, PII<br/>👤 QA Lead audit] --> IT5
    IT5[3.5 Run in staging-like CI<br/>per-PR with Testcontainers<br/>nightly against staging itself<br/>🤖 CI] --> IT_OUT([Continuous - feeds Gate 3<br/>release-candidate validation])

    style IT_IN fill:#1B3A5C,color:#fff
    style IT_OUT fill:#1B3A5C,color:#fff
```

---

## Step 4: E2E Testing (Web)

Entry point is the prioritised critical user journeys from the Test Plan and a staging environment URL. Sub-stages 4.1 → 4.5 identify journeys (revenue-impacting and safety-critical only), generate Playwright tests with role-based locators and auto-waiting, run in CI on every PR, manage flakiness aggressively, and debug failures with Claude Code via the trace file. The escalation branch (E_FB) covers persistent flake > 2% by evaluating an AI-native E2E platform. No dedicated gate at Step 4 — feeds Gate 3.

```mermaid
flowchart TD
    E_IN([From Test Plan: critical user journeys<br/>+ staging URL + test credentials]) --> E1

    E1[4.1 Identify critical journeys<br/>revenue-impacting + safety-critical only<br/>login, checkout, signup, password reset,<br/>data sync, billable feature happy paths<br/>👤 QA Lead] --> E2
    E2[4.2 playwright-e2e-generation<br/>role-based locators NOT CSS,<br/>auto-waiting NOT page.waitForTimeout,<br/>one assertion per logical step,<br/>isolated test data setup/teardown<br/>🤖 Claude Code] --> E3

    E3[4.3 Run in CI on every PR<br/>trace on-first-retry for replay<br/>failures block merge to main<br/>🤖 CI + Playwright] --> E4

    E4{Test flaky<br/>greater than 2 percent?}
    E4 -- Yes --> E4A[4.4 Quarantine via Linear issue<br/>label flaky-test<br/>fix or delete within 2 cycles<br/>👤 QA + 🔌 Linear MCP]
    E4A --> E5
    E4 -- No --> E5

    E5{Failure during run?}
    E5 -- Yes --> E5A[4.5 playwright-failure-debug<br/>Claude reads trace,<br/>identifies divergence,<br/>proposes fix to test or code<br/>🤖 Claude Code]
    E5A --> E_FB
    E5 -- No --> E_OUT

    E_FB{Persistent flake<br/>greater than 2 percent globally<br/>after hardening?}
    E_FB -- Yes --> ESC[Evaluate AI-native E2E platform<br/>Momentic / QA.tech for self-healing<br/>see Tools Evaluation report<br/>👤 QA Lead]
    ESC --> E_OUT
    E_FB -- No --> E_OUT([Continuous - feeds Gate 3<br/>full regression suite for RC])

    style E_IN fill:#1B3A5C,color:#fff
    style E_OUT fill:#1B3A5C,color:#fff
    style ESC fill:#7A5C1B,color:#fff
```

---

## Step 5: Mobile App Testing

Entry point is the project's mobile applicability — skip if web-only. Sub-stages 5.1 → 5.4 generate framework-appropriate tests (Appium / Espresso / XCUITest / Detox), run on BrowserStack App Automate real devices using analytics-driven device matrix, leverage the AI Self-Healing Agent for locator drift, and route failures through the Test Failure Analysis Agent. No dedicated gate — feeds Gate 3.

```mermaid
flowchart TD
    M_IN([From project scope<br/>does the project ship native iOS/Android?]) --> M_CHECK

    M_CHECK{Native mobile?}
    M_CHECK -- No, web-only --> M_SKIP[Skip Step 5<br/>use Playwright mobile emulation in Step 4<br/>👤 QA Lead]
    M_SKIP --> M_OUT_SKIP([Skip to Step 6: Load Testing])

    M_CHECK -- Yes --> M1[5.1 Choose framework<br/>Appium cross-platform / Espresso Android /<br/>XCUITest iOS / Detox React Native<br/>👤 QA Lead]
    M1 --> M2[5.2 mobile-test-generation<br/>journeys + framework + BrowserStack creds<br/>+ device matrix from analytics<br/>🤖 Claude Code]
    M2 --> M3[5.3 Run on BrowserStack App Automate<br/>real devices,<br/>AI Self-Healing Agent updates locators on UI change,<br/>screenshot + video on failure<br/>🤖 BrowserStack]
    M3 --> M4[5.4 Test Failure Analysis Agent categorises<br/>production bug / automation error / environment issue<br/>~95 percent triage time reduction<br/>🤖 BrowserStack]
    M4 --> M_OUT([Continuous - feeds Gate 3<br/>RC must pass mobile matrix])

    style M_IN fill:#1B3A5C,color:#fff
    style M_OUT fill:#1B3A5C,color:#fff
    style M_OUT_SKIP fill:#1B3A5C,color:#fff
    style M_SKIP fill:#7A5C1B,color:#fff
```

---

## Step 6: Load Testing

Entry point is the NFRs from Phase 1, the OpenAPI spec, and expected traffic patterns. Sub-stages 6.1 → 6.5 generate k6 scripts with realistic traffic profiles, run in a performance-matched staging environment (not the standard staging), wire results to the release-candidate gate, and add multi-region distributed load only when local k6 cannot generate enough load. Distributed-testing escalation branches via Grafana Cloud k6.

```mermaid
flowchart TD
    L_IN([From Phase 1 NFRs + OpenAPI spec<br/>+ expected traffic patterns]) --> L1

    L1[6.1 k6-load-generation<br/>OpenAPI + RPS at peak + ramp/steady/ramp-down<br/>+ user behaviour + NFR targets p95/p99/error rate<br/>🤖 Claude Code] --> L2
    L2[6.2 Model realistic traffic<br/>work backwards from real data<br/>do NOT load-test paranoia<br/>👤 perf owner] --> L3
    L3[6.3 Run in performance-matched staging<br/>NOT the standard staging<br/>provision and tear down for the run<br/>👤 SRE + 🤖 k6] --> L4

    L4{Local k6 insufficient<br/>or geographic distribution matters?}
    L4 -- Yes --> L_DIST[Grafana Cloud k6 from $29/mo<br/>multi-region load generation<br/>🤖 Grafana Cloud k6]
    L_DIST --> L5
    L4 -- No --> L5

    L5[6.4 Wire results to RC gate<br/>p95/p99/error rate/concurrent-user vs NFR<br/>failure may be the test, not the code -<br/>Tech Lead decides<br/>👤 Tech Lead] --> L_OUT([Per release - feeds Gate 3<br/>RC validation])

    style L_IN fill:#1B3A5C,color:#fff
    style L_OUT fill:#1B3A5C,color:#fff
    style L_DIST fill:#7A5C1B,color:#fff
```

---

## Step 7: Bug Intake & Triage

Linear is the single bug ledger — QA-discovered bugs, tests that failed on main, and user or support reports all land in the same Triage queue, and nothing files a bug automatically. Sub-stages 7.1 → 7.5 file the bug with a source label, capture the evidence in the issue body at filing time, let Triage Intelligence collapse duplicates and propose labels, then hand the QA Lead the human gate at 7.4 where severity is assigned from scratch on business impact. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-per-sprint--test-coverage).

```mermaid
flowchart TD
    B_SOURCE{Bug source}

    B_SOURCE -- Manual or exploratory QA --> B71A[7.1 Filed in Linear Triage<br/>labels bug + source:qa<br/>👤 QA]
    B_SOURCE -- Failing test on main --> B71B[7.1 Filed in Linear Triage<br/>labels bug + source:qa<br/>👤 QA or the developer on the failing run]
    B_SOURCE -- User or support report --> B71C[7.1 Filed in Linear Triage<br/>labels bug + source:user<br/>👤 PM/support]

    B71A --> B72
    B71B --> B72
    B71C --> B72

    B72[7.2 Evidence in the issue body at filing time<br/>expected vs actual, reproduction steps,<br/>environment + release version,<br/>stack trace / logs / screenshot / HAR<br/>Bug Report Template<br/>👤 reporter]
    B72 --> B73

    B73[7.3 Triage Intelligence<br/>semantic similarity vs existing issues<br/>surfaces Merge as related suggestion<br/>+ proposes labels and priority<br/>🤖 Triage Intelligence]
    B73 --> B74

    B74[7.4 QA Lead human gate<br/>open Bug Triage saved view - state = Triage<br/>per issue:<br/>assign severity from scratch on business impact,<br/>bounce back if the evidence is not actionable,<br/>accept or reject the duplicate suggestion,<br/>confirm or override assignee,<br/>add severity:S0/S1/S2/S3<br/>👤 QA Lead]
    B74 --> B_CONTEST

    B_CONTEST{Severity contested?}
    B_CONTEST -- Yes --> B75[7.5 bug-triage prompt<br/>structures the impact assessment<br/>QA Lead still owns the call<br/>🤖 Claude Code]
    B75 --> B_DECIDE
    B_CONTEST -- No --> B_DECIDE

    B_DECIDE{Severity}
    B_DECIDE -- S0 Critical --> B_S0[Fix immediately<br/>rollback if no fix in 1h<br/>escalate to on-call by hand -<br/>nothing pages off a Linear label]
    B_DECIDE -- S1 High --> B_S1[This cycle<br/>release blocker]
    B_DECIDE -- S2 Medium --> B_S2[Within 2 cycles]
    B_DECIDE -- S3 Low --> B_S3[Backlog]

    B_S0 --> B_OUT
    B_S1 --> B_OUT
    B_S2 --> B_OUT([To Step 8: Bug Fix Loop<br/>or backlog for S3])
    B_S3 --> B_OUT

    style B_SOURCE fill:#1B3A5C,color:#fff
    style B_OUT fill:#1B3A5C,color:#fff
    style B72 fill:#B91C1C,color:#fff
    style B74 fill:#B91C1C,color:#fff
```

---

## Step 8: Bug Fix Loop

Entry point is a triaged bug in Linear with severity, reproduction steps, and whatever evidence the reporter attached. Sub-stages 8.1 → 8.7 fetch the bug like any Phase 3 task, **reproduce locally with a failing test first**, fix the code with the debugging prompt, keep the regression test in the same PR, route through Phase 3 Step 4 review, auto-close the Linear issue on merge, verify in staging, and post a brief post-mortem for S0/S1. The strict rule enforced at PR review: regression test ships in the same diff as the fix.

```mermaid
flowchart TD
    BF_IN([From Step 7: Triaged bug in Linear<br/>+ severity + reproduction steps<br/>+ attached evidence]) --> BF1

    BF1[8.1 linear-next-task on the bug<br/>same prompt as Phase 3 Step 2<br/>update_issue In Progress + assignee=me<br/>git checkout branchName<br/>🔌 Claude Code + Linear MCP] --> BF2

    BF2[8.2 bug-reproduction<br/>Claude proposes a failing test<br/>that captures the bug<br/>WRITE THIS TEST FIRST<br/>will not reproduce - back to the reporter<br/>🤖 Claude Code] --> BF3

    BF3[8.3 debugging prompt<br/>read code paths, trace data flow,<br/>propose targeted fix + state blast radius<br/>🤖 Claude Code] --> BF4

    BF4[8.4 Add regression test<br/>the failing test from 8.2 now passes - keep it<br/>SAME DIFF as the fix - reviewers reject otherwise<br/>👤 developer] --> BF5
    BF5[8.5 Open PR through Phase 3 Step 4<br/>title - ENG-XXX Fix bug summary<br/>body - Closes ENG-XXX + brief post-mortem<br/>🤖 Claude Code pr-description] --> BF_AUTO1

    BF_AUTO1[Linear auto - In Review<br/>🔁 Linear ↔ git auto] --> BF_REVIEW[Phase 3 Step 4 review gate<br/>CI + CodeRabbit + human]
    BF_REVIEW --> BF_AUTO2[On merge - Linear auto - Done<br/>🔁]
    BF_AUTO2 --> BF6

    BF6[8.6 QA verifies in staging<br/>posts Verified in staging comment<br/>on the Linear issue - the close-out record<br/>👤 QA] --> BF7

    BF7{Severity S0 or S1?}
    BF7 -- Yes --> BF7A[8.7 post-mortem comment on Linear<br/>root cause one paragraph,<br/>why tests missed it,<br/>regression + monitoring to add,<br/>prevention owner<br/>🤖 Claude Code post-mortem prompt + 👤]
    BF7A --> BF_OUT
    BF7 -- No --> BF_OUT([To next bug or back to Step 7<br/>or to Gate 3 RC validation])

    style BF_IN fill:#1B3A5C,color:#fff
    style BF_OUT fill:#1B3A5C,color:#fff
    style BF1 fill:#3D6B9F,color:#fff
    style BF_AUTO1 fill:#2E8B57,color:#fff
    style BF_AUTO2 fill:#2E8B57,color:#fff
    style BF4 fill:#B91C1C,color:#fff
    style BF7A fill:#B91C1C,color:#fff
```

---

## Daily QA Workflow

The daily loop runs in parallel with development. Manual testing, CI failures, and user reports all feed the same Linear triage queue — nothing arrives on its own, so the morning sweep is what makes intake reliable.

```mermaid
flowchart TD
    QA_M([Morning]) --> QA_M1[Check CI status<br/>any test failures overnight?<br/>👤]
    QA_M1 --> QA_M2[Sweep inbound reports<br/>support and user reports filed into Linear Triage<br/>anything raised in chat gets an issue<br/>👤]
    QA_M2 --> QA_M3[Clear the Bug Triage saved view<br/>validate severity + assignee for each<br/>👤 QA Lead]
    QA_M3 --> QA_PER([Per Story - parallel with dev])

    QA_PER --> QA_P1[Unit tests with Claude Code<br/>alongside the code<br/>🤖]
    QA_P1 --> QA_P2[Integration tests for new endpoints<br/>real DB via Testcontainers<br/>🤖]
    QA_P2 --> QA_P3[Playwright E2E for new critical flows<br/>🤖]
    QA_P3 --> QA_P4[PR includes prod code + tests + regressions<br/>👤]
    QA_P4 --> QA_W([Weekly])

    QA_W --> QA_W1[Run full regression suite<br/>automated nightly<br/>🤖]
    QA_W1 --> QA_W2[Review flakiness report<br/>fix or quarantine via flaky-test label<br/>🔌 Linear MCP + 👤]
    QA_W2 --> QA_W3[Load test latest build vs staging<br/>🤖 k6]
    QA_W3 --> QA_REL([Per Release])

    QA_REL --> QA_R1[All Gate 3 checks pass before RC<br/>👤 Tech Lead + QA Lead]
    QA_R1 --> QA_R2[Archive load test + coverage report<br/>👤]

    style QA_M fill:#1B3A5C,color:#fff
    style QA_PER fill:#1B3A5C,color:#fff
    style QA_W fill:#1B3A5C,color:#fff
    style QA_REL fill:#1B3A5C,color:#fff
```

---

## Four Human Gates

The flow has four explicit human gates so that no AI-triaged bug or AI-generated test reaches main without sign-off:

1. **Gate 1 — Test Plan approved.** QA Lead + Tech Lead sign off in the Linear Document. Required before any test code is written. See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-test-plan-approved).
2. **Gate 2 — Per-PR + per-cycle test coverage.** New code has unit tests; coverage ≥ 80%; integration tests for new endpoints; E2E tests for new critical flows; flakiness < 2%. Enforced continuously through Phase 3 Step 4. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-per-sprint--test-coverage).
3. **Gate 3 — Release candidate validation.** All test types green, load tests meet NFRs, 0 S0 / 0 S1 open, regression suite green, flake < 2%. Tech Lead + QA Lead approve. See [QUALITY-GATES.md → Gate 3](./QUALITY-GATES.md#gate-3-release-candidate-testing).
4. **Gate 4 — Phase Handoff.** All artefacts present and Linear-linked, coverage report published as a CI artefact, post-mortems filed for every S0/S1. See [QUALITY-GATES.md → Gate 4](./QUALITY-GATES.md#gate-4-phase-handoff).

Linear's git integration handles the auto-transitions; humans validate, prioritise, and review — they do not transcribe.

---

## Related Documents

- [Process Definition →](./PROCESS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Prompt Templates →](./PROMPTS.md)
- [Test Plan Template →](../templates/test-plan-template.md)
- [Bug Report Template →](../templates/bug-report-template.md)
- [Phase 1 Linear MCP setup (Step 0) →](../01-requirement-gathering/PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp)
- [Phase 3 Development PROCESS →](../03-development/PROCESS.md)
