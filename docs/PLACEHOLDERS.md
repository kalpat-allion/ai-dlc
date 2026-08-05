# Placeholder Reconciliation

> **What this is.** One row per **repository fact**, listing every placeholder name the shipped templates use for it. Its job is not to hold your values — the per-directory READMEs do that, and they are what an operator has open while instantiating. Its job is to stop the same fact being asked for twice under two names.
>
> **The problem it solves.** The six shipped phases carry **58 distinct placeholders** across **38 templates**. Before Phase 2, **literal overlap between phases was zero** — no placeholder name appeared in more than one phase — and what existed instead was **six pairs that name the same repo fact differently**. An operator instantiating two phases fills the same value twice under two names, and nothing catches an inconsistent fill. That defect is invisible from inside any single README, which is why this file is central.
>
> **Phase 2 is the first phase to reuse rather than re-coin, and that is this file working.** It uses **16** placeholders, of which **8 are existing names adopted from Phases 3 and 6** — `{{ORM}}`, `{{DATASTORE}}`, `{{MIGRATIONS_PATH}}`, `{{AUTH_STACK}}`, `{{TEAM_STACK}}`, `{{CLOUD_PROVIDER}}`, `{{FRONTEND_FRAMEWORK}}`, `{{FRONTEND_ROOT}}` — and only 8 are new. **The pair count is still six, not fourteen.** Two near-pairs were caught at authoring time and are recorded below rather than minted. Phase 2 is also the first phase whose placeholders literally overlap another phase's, which is the intended end state, not a regression.
>
> **Phase 3's four new artifacts carry 10 placeholders and coined only 3** — `{{DEFAULT_BRANCH}}`, `{{REQUIREMENTS_DIR}}`, `{{API_SPEC_PATH}}`. Seven are reused: `{{TEST_RUNNER}}`, `{{PR_TITLE_FORMAT}}`, `{{LINEAR_PROJECT}}`, `{{TEAM_PREFIX}}`, `{{TEAM_STACK}}`, `{{ADR_DIR}}`, `{{TECH_DEBT_LABEL}}`. **The pair count is still six**, but the build added a **seventh row of a different kind** — not a synonym, but one fact two templates both act on. See below.
>
> **Phase 4's four artifacts carry 17 placeholders and coined 8** — `{{E2E_TEST_DIR}}`, `{{E2E_COMMAND}}`, `{{STAGING_BASE_URL}}`, `{{COVERAGE_REPORT_PATH}}`, `{{FLAKY_TEST_LABEL}}`, `{{COVERAGE_GAP_LABEL}}`, `{{LOAD_TEST_DIR}}`, `{{PERF_TEST_ENV}}` — reusing 9. **The pair count is still six**, and two more near-pairs were caught at authoring time. The build's real contribution to this file is a third kind of case: **two names were drafted, then withdrawn before they shipped**, because the fact they named is written as a literal in the sibling templates. That is rule 4 firing during authoring rather than in a post-hoc sweep — see [the withdrawn pair](#a-third-kind-of-case--two-names-drafted-then-withdrawn).
>
> **Phase 5's five artifacts carry 8 placeholders and coined only 3** — `{{SECURITY_DOCS_DIR}}`, `{{DEPENDENCY_MANIFESTS}}`, `{{CODEQL_LANGUAGES}}` — reusing 5: `{{ARCHITECTURE_DIR}}`, `{{ADR_DIR}}`, `{{DEFAULT_BRANCH}}`, `{{TEAM_STACK}}`, `{{SECRETS_MANAGER}}`. **The best reuse ratio of any phase so far, and the pair count is still six.** Its contribution to this file is a **fourth kind of case**: three names were drafted and withdrawn not because the fact is a literal elsewhere, but because **filling the value disables a stop-and-ask guard** — see [the pre-fillable guards](#a-fourth-kind-of-case--names-that-would-disable-a-guard-by-being-filled).
>
> *(The counts above were also recounted rather than incremented. The header said 44 and the inventory said "All 45"; the templates carried **44**, so the inventory heading was one over before this build and is corrected to **58** now, across **38** templates.)*

---

## The reconciliation table

These are the rows that matter. Each is one fact about the repository, asked for under more than one name.

| Repo fact | Asked for as | In | Must agree because |
|---|---|---|---|
| **How secrets reach the runtime** | `{{SECRETS_MECHANISM}}` | `software-architect` | The architect designs against the mechanism the IaC actually references. Two answers means designs that cite a secret store the infrastructure does not use |
| | `{{SECRETS_MANAGER}}` | `terraform-iac-engineer`, `ci-identity-and-secrets-bringup`, `secret-leak-response` | **Phase 5 adds an incident-time consumer to a bring-up-time name.** The rotated replacement is created in whatever this names; filled with a store the runtime does not read, the rotation "succeeds" and the service keeps authenticating with the credential you were trying to kill |
| **Metrics / traces / logs backend** | `{{OBSERVABILITY_STACK}}` | `software-architect` | The dashboards and SLOs are built on the same backend the design assumes instrumentation reports to |
| | `{{OBS_BACKEND}}` | `observability-bringup`, `deploy-and-rollback-bringup` | |
| **How tests are run** | `{{TEST_RUNNER}}` | `frontend-engineer`, `backend-engineer`, `refactor-specialist`, `software-architect`, `open-pull-request`, `/load-task-context`, `/write-module-readme`, `e2e-and-coverage-engineer`, `author-test-plan`, `reproduce-and-diagnose-bug` | CI runs the command; the specialists run the runner. If they disagree, tests pass locally and the pipeline runs nothing |
| | `{{TEST_COMMAND}}` | `cicd-pipeline-bringup` | **This one carries the coverage gate, and the bad fill is silent in the good-looking direction.** Filled without its coverage flag — `pnpm test` rather than `pnpm test -- --coverage` — the job passes every PR and the pipeline's coverage checkbox reports green over a threshold that never ran. Same failure direction as `{{PERF_TEST_ENV}}`: a pass made *reachable* that should not be, so nobody investigates. The skill is now required to observe one **failing** run rather than trust the fill |
| | `{{E2E_COMMAND}}` | `e2e-and-coverage-engineer` | A third name for a related but **different** fact — see the near-pair note below. Recorded here so nobody reconciles the three into one |
| **The service's directory** | `{{FRONTEND_ROOT}}` / `{{BACKEND_ROOT}}` | `frontend-engineer`, `backend-engineer` | The Docker build context must be a directory the code actually lives in. In a multi-service repo these are genuinely different values — **check, do not assume** |
| | `{{SERVICE_ROOT}}` | `container-image-engineer` | |
| **The Linear team** | `{{TEAM_PREFIX}}` — its *key*, e.g. `ENG` | `linear-task-agent`, `run-sprint-planning`, `/load-task-context` | Two representations of one object. A prefix from a different team than the one the Project sits under produces issues nobody's saved view shows |
| | `{{LINEAR_TEAM}}` — its *name*, e.g. `Engineering` | `publish-prd-to-linear` | |
| **Language and framework** | `{{TEAM_STACK}}`, `{{FRONTEND_FRAMEWORK}}`, `{{BACKEND_FRAMEWORK}}` | the P3 specialists | Not synonyms, but **constrained by each other**: a `{{RUNTIME}}` of `Node 22` under a `{{TEAM_STACK}}` of `FastAPI + Python 3.12` is a contradiction that ships a broken image |
| | `{{RUNTIME}}` — language + version, e.g. `Node 22` | `container-image-engineer` | |

**Fill the left column once, then propagate.** Working fact-first rather than file-first is the whole point; going template-by-template is what produces the inconsistent fill.

### A seventh row that is not a synonym pair — one name, one fact, two actors

`{{PR_TITLE_FORMAT}}` now has **two consumers that both compose a PR title**: `linear-task-agent` and `open-pull-request`. Same name, same fact — so this is **not** the failure this table was built for. **It has the same failure mode anyway.**

| Repo fact | Asked for as | In | Must agree because |
|---|---|---|---|
| **The PR title convention** | `{{PR_TITLE_FORMAT}}` — one name, **two composers** | `linear-task-agent`, `open-pull-request` | Filled inconsistently, the skill composes one title while the agent's own convention says another. **The mismatch surfaces as a de-linked PR, not as a bad fill** — the tracker's git integration matches on the identifier in the title, so the symptom is an issue that never moves to In Review and a PR nobody can trace back |
| **How tests are run** | `{{TEST_RUNNER}}` — one name, **ten consumers**, which run it, read its file convention, read its reports, and publish its name | `frontend-engineer`, `backend-engineer`, `refactor-specialist`, `software-architect`, `open-pull-request`, `/load-task-context`, `/write-module-readme`, `e2e-and-coverage-engineer`, `author-test-plan`, `reproduce-and-diagnose-bug` | Already a reconciliation row against `{{TEST_COMMAND}}`. The Phase 3 additions make it the same shape as the row above: `/load-task-context` needs the runner's **test-file convention** to locate tests and `/write-module-readme` needs an invocation it can **scope to one module and actually run**. **Filled with the whole-suite CI command, both degrade silently** — one finds no tests, the other documents a command that does not test the module. **Phase 4 adds two more failure directions**: `e2e-and-coverage-engineer` uses the runner identity to recognise the tests that are **not its own** — filled wrong, it starts writing at a layer it is forbidden to touch — and `author-test-plan` writes the name into a **published** plan, where a wrong value is quoted for the rest of the release |

**This argues the table's framing should widen** from *"synonym pairs"* to *"any repo fact that two or more templates both act on"*. The original framing caught the case where one fact has two names; it does not catch the case where one name has two actors whose behaviour must agree. **Both fail the same way — an inconsistent fill that surfaces as a behaviour bug rather than as a configuration error** — and only the first kind was getting a row. The widening is recorded here rather than applied as a restructure: the six existing rows are still correct as synonym pairs, and re-cutting the whole table is a change to make when a third row of this kind appears, not on the second.

### Two near-pairs, caught before they were minted

Neither is a reconciliation row, because in both cases the second name was **not created**. They are recorded because the reasoning is the thing worth reusing.

- **`{{AUTH_STACK}}` now carries two related but distinct facts.** `software-architect` means the auth *provider* (`Clerk`); `api-contract-freeze` needs the *wire security scheme* for the OpenAPI `securitySchemes` block. Coining `{{AUTH_SCHEME}}` would have been the seventh pair, so the naming rule was applied as written — *reuse the existing name even if you would have named it differently* — and the skill is phrased as "a security scheme for `{{AUTH_STACK}}`". **If a repo ever has a provider and a scheme that genuinely disagree, this is the row that splits**, under converge-on-next-touch.
- **`{{A11Y_SCAN_COMMAND}}` deliberately does *not* reuse `{{TEST_RUNNER}}` or `{{TEST_COMMAND}}`.** Adjacent, but a different fact: the auditor's PASS condition requires a command that renders the page. Filling it with the unit-test command makes PASS permanently unreachable, and the failure looks like a strict agent rather than a bad fill.
- **`{{DEFAULT_BRANCH}}` deliberately does *not* adopt the spelling `__BASE_BRANCH__`.** The near-hit is in [`03-development/PR-REVIEW-BOT.md`](../aidlc-phases/03-development/PR-REVIEW-BOT.md), and it is **not a template placeholder at all** — it is a GitHub Actions **runtime** substitution, filled by the workflow from `github.event.pull_request.base.ref` at each run. **An operator never fills it and must never try.** Adopting its spelling would imply the two are the same mechanism, and the first person to "reconcile" them would either hard-code a branch name into a workflow or leave a template placeholder unfilled on the assumption that something substitutes it. Different lifetime, different filler, different name.
- **`{{E2E_COMMAND}}` deliberately does *not* reuse `{{TEST_RUNNER}}` or `{{TEST_COMMAND}}`** — the same reasoning as `{{A11Y_SCAN_COMMAND}}`, one row further down the pyramid. It drives a browser against a **deployed** environment. Filled with the unit-test command, `e2e-and-coverage-engineer` runs the wrong suite, passes, and reports it as an end-to-end result — a green gate line measuring nothing it claims to.
- **`{{PERF_TEST_ENV}}` deliberately does *not* reuse `{{STAGING_BASE_URL}}`, and this is the sharpest fill in the phase.** They are the same shape and **must never hold the same value**: `load-test-engineer`'s entire `PASS WITHHELD` rule exists to stop a load test being read as production evidence when it ran against ordinary staging, and an operator who fills both from one value has disabled that rule *by configuring it*. Nothing in the template can detect it — the agent is told which environment is performance-matched, it cannot measure whether that is true. **The only defence is the fill instruction**, which is why both READMEs carry it in bold.
- **`{{API_SPEC_PATH}}` deliberately does *not* reuse `{{API_BASE_PATH}}`.** A file path (`docs/api/openapi.yaml`) and a URL prefix (`/api/v1`) are different facts that happen to share three letters. Reusing the name would produce a spec written to `/api/v1` and a mock server pointed at a directory.

### A third kind of case — two names drafted, then withdrawn

`{{E2E_FRAMEWORK}}` (`Playwright`) and `{{LOAD_TEST_TOOL}}` (`k6`) were drafted into the two Phase 4 skills and **withdrawn before they shipped**. The two Phase 4 subagents write both tools as **literals**, because their rules are true of those tools and of nothing else — trace-file diagnosis, role-based locators and `waitForTimeout` for one; `thresholds`, `check()` and a virtual-user ramp for the other. Parameterising a body like that produces a template that *claims* to work with any tool while enforcing rules only one of them has.

Shipping the names anyway would have created the **one fact, one literal and one name** case this file already calls its blindest spot — with the additional property that the inconsistent fill **looks correct on both sides**. An operator filling `Cypress` gets a published test plan naming Cypress and an installed agent writing Playwright, each internally consistent, with no surface anywhere showing the two disagree.

**Resolved by removing the names, not by adding rows.** The skills now write `Playwright` and `k6` as literals, matching their siblings, and [`04-testing-and-qa/skill-prompts/README.md`](../aidlc-phases/04-testing-and-qa/skill-prompts/README.md) records that a team on a different tool **edits these templates rather than filling a value** — which is the honest description of what the phase actually requires. This is the first build where **rule 4 fired during authoring instead of in a sweep afterwards**, which is the cheap moment: withdrawing a name costs one edit, while retiring a shipped one breaks every repo that already instantiated it.

**Phase 5 applied the same test to `CodeQL` and `Dependabot` and reached the same answer.** `.github/codeql/`, the `qlpack.yml` and `.qls` wiring, `@problem.severity`, dataflow classes over AST matching, the two-fixture run, and *"the Dependabot alerts view is the single source of truth for what CVEs are open"* are rules that are true of those two tools and of nothing else. Both are written as literals across all five templates, and both directory READMEs say a different tool means editing the template. **Three consecutive builds have now reached this verdict on a prescribed tool — the test is stable enough to state as a rule:** if the body's rules would be false under a different fill, the name is a claim of portability the template cannot honour.

### A fourth kind of case — names that would disable a guard by being filled

Phase 5 drafted and withdrew three names for a reason unlike any above. `{{DATA_CLASSIFICATION}}` and `{{COMPLIANCE_SCOPE}}` (for `threat-modeler`) and `{{ENTRY_POINTS}}` (for `dependency-risk-analyst`) all name real, stable-looking repository facts. Each was withdrawn because **the template's value comes from stopping to ask for that exact input, and a placeholder is a way to pre-answer it once and never be asked again.**

- `threat-modeler` refuses to start until data sensitivity, user base, compliance scope and threat actors are answered, because **severity is computed from those four**. An operator who fills them at install time gets a model rated against whatever was true in month one — internally consistent, confidently prioritised, and wrong about which findings block a launch.
- `dependency-risk-analyst` confirms its entry-point list with a human **per run**. A list filled at install omits every route added since, and the CVEs reachable only from those routes come back `not reachable` carrying real import evidence. **That is a Gate 3 pass made reachable that should not be** — the `{{PERF_TEST_ENV}}` failure shape, arriving through staleness rather than through a wrong value.

The general form: **a stop-and-ask guard and a placeholder for the same fact cannot both exist.** The placeholder wins, silently, on the first run and every run after. Where the guard is what the artifact is for, the name does not get minted — and where the fact genuinely is stable repo configuration, the guard was never load-bearing and should be deleted rather than duplicated. **This is a different test from rules 1-4**, which all ask *what is the fact called elsewhere*; this one asks *what happens to the template's own refusals once the fact has a value*.

---

## Full inventory

All 58, grouped by what they describe. **Where each is consumed** is authoritative — it is generated from the templates, not remembered.

### Tracker and conventions

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{TEAM_PREFIX}}` | `ENG` | `linear-task-agent`, `run-sprint-planning`, `/load-task-context`, `reproduce-and-diagnose-bug` |
| `{{LINEAR_TEAM}}` | `Engineering` | `publish-prd-to-linear` |
| `{{LINEAR_PROJECT}}` | `TimeSync v2 — Scheduling` | `linear-task-agent`, `run-sprint-planning`, `author-test-plan` |
| `{{LINEAR_INITIATIVE}}` | `TimeSync v2` | `publish-prd-to-linear` |
| `{{LABEL_CONVENTIONS}}` | `feature, bug, tech-debt, chore` | `linear-task-agent` |
| `{{TECH_DEBT_LABEL}}` | `tech-debt` | `linear-task-agent`, `/write-module-readme`, `reproduce-and-diagnose-bug` |
| `{{BUG_LABEL}}` | `bug` | `linear-task-agent`, `reproduce-and-diagnose-bug` — the second consumer **reads** it as an entry condition rather than writing it; a value that does not match what triage actually applies makes the skill refuse every real bug |
| `{{FLAKY_TEST_LABEL}}` | `flaky-test` — the label a quarantined flaky spec carries | `e2e-and-coverage-engineer` |
| `{{COVERAGE_GAP_LABEL}}` | `coverage-gap` — suggested on each audit entry; the agent **never files it** | `e2e-and-coverage-engineer` |
| `{{PR_TITLE_FORMAT}}` | `[ENG-XXX] <imperative summary>` — **two composers; see the reconciliation note above** | `linear-task-agent`, `open-pull-request` |
| `{{DEFAULT_BRANCH}}` | `main` — the branch PRs target, the branch you rebase onto, the branch a regression test must be proven to fail on, *and* the branch a release candidate's mitigations are verified against. **Not** `__BASE_BRANCH__`, which is a workflow runtime substitution and not an operator-filled value | `open-pull-request`, `reproduce-and-diagnose-bug`, `threat-model-verifier` |

### Stack and runtime

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{TEAM_STACK}}` | `Next.js 14 + TypeScript` | `frontend-engineer`, `backend-engineer`, `software-architect`, `solution-architect`, `run-sprint-planning` — **estimates sized against the wrong stack are confidently wrong** — and `dependency-risk-analyst`, for which the stack decides *what counts as an unfollowable call path*: reflection, dynamic dispatch and plugin loading are the reasons a verdict comes back `undetermined` rather than `not reachable` |
| `{{CODEQL_LANGUAGES}}` | `javascript-typescript, python` — must match what `.github/workflows/codeql.yml` actually builds. **A query written for a language the workflow does not build produces no results and no error** | `codeql-query-author` |
| `{{FRONTEND_FRAMEWORK}}` | `React 18` | `software-architect`, `accessibility-auditor` |
| `{{BACKEND_FRAMEWORK}}` | `NestJS` | `software-architect` |
| `{{RUNTIME}}` | `Node 22` | `container-image-engineer` |
| `{{AUTH_STACK}}` | `Clerk` — the auth **provider** for the architect, and the **wire security scheme** the OpenAPI spec declares. See the near-pair note above | `software-architect`, `api-contract-freeze` |
| `{{REALTIME_STACK}}` | `Socket.IO`, or `none` | `software-architect` |
| `{{ASYNC_JOB_STACK}}` | `BullMQ`, or `none` | `software-architect` |

### Paths

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{FRONTEND_ROOT}}` | `apps/web` | `frontend-engineer`, `accessibility-auditor`, `e2e-and-coverage-engineer` |
| `{{BACKEND_ROOT}}` | `services/api` | `backend-engineer` |
| `{{SERVICE_ROOT}}` | `services/api`, or `.` | `container-image-engineer` |
| `{{COMPONENT_LIBRARY_PATH}}` | `apps/web/src/components/ui` | `frontend-engineer`, `software-architect` |
| `{{API_CLIENT_PATH}}` | `apps/web/src/lib/api` | `frontend-engineer` |
| `{{MIGRATIONS_PATH}}` | `services/api/migrations` | `backend-engineer`, `data-model-design` |
| `{{IAC_DIR}}` | `infra/` | `terraform-iac-engineer`, `iac-state-backend-bringup`, `ci-identity-and-secrets-bringup`, `cost-guardrails-bringup` |
| `{{SCHEMA_PATH}}` | `prisma/schema.prisma` | `data-model-design` |
| `{{ADR_DIR}}` | `docs/adrs/` — **the first name to span Phases 2 and 3**, and now three phases; the value `/adr` writes into is the value `/load-task-context` reads back out and the value `threat-model-verifier` reads to tell Landed from Partial | `/adr`, `solution-architect`, `architecture-reviewer`, `/load-task-context`, `threat-modeler`, `threat-model-verifier` |
| `{{ARCHITECTURE_DIR}}` | `docs/architecture/` | `solution-architect`, `architecture-reviewer`, `threat-modeler` |
| `{{SECURITY_DOCS_DIR}}` | `docs/security/` — the **directory** only. The filenames inside it stay literals in the templates (`threat-model.md`, `ai-threat-review.md`, `mitigation-verification-<release>.md`, the post-mortem seed) because the phase's gates and handoff read those exact names; a repo that renames them has edited the templates, not filled a value | `threat-modeler`, `threat-model-verifier`, `secret-leak-response` |
| `{{DEPENDENCY_MANIFESTS}}` | `services/api/package.json + pnpm-lock.yaml` — the manifest and lockfile pairs defining the **shipped** dependency tree. In a monorepo, every workspace that reaches production. **See the hard requirements below** | `dependency-risk-analyst` |
| `{{A11Y_REPORT_PATH}}` | `docs/accessibility/wcag-aa-report.md` | `accessibility-auditor` |
| `{{REQUIREMENTS_DIR}}` | `docs/prd/` — if requirements live in the tracker rather than the repo, point it at the tracker Document set and say so in the fill. **`load-test-engineer` needs the NFR numbers to be *citable* here**, so a fill pointing at a directory that does not hold them makes it refuse — visibly, which is the good failure | `/load-task-context`, `load-test-engineer` |
| `{{E2E_TEST_DIR}}` | `tests/e2e` — the agent's entire write scope | `e2e-and-coverage-engineer` |
| `{{LOAD_TEST_DIR}}` | `tests/load` — the agent's entire write scope | `load-test-engineer` |
| `{{COVERAGE_REPORT_PATH}}` | `coverage/coverage-summary.json` — the audit refuses to report any number it did not read from this file | `e2e-and-coverage-engineer` |
| `{{API_SPEC_PATH}}` | `docs/api/openapi.yaml` — the spec **file**. Deliberately **not** `{{API_BASE_PATH}}`, which is a URL prefix and a different fact. Both Phase 3 commands consume it for opposite reasons: one reads a section from it, the other refuses to hand-copy anything that renders from it; `load-test-engineer` takes endpoints, payloads and auth from it and invents none of the three | `/load-task-context`, `/write-module-readme`, `load-test-engineer` |

### Data and testing

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{ORM}}` | `Prisma` | `backend-engineer`, `software-architect`, `data-model-design` |
| `{{DATASTORE}}` | `PostgreSQL 16` — **include the version**, it changes the available DDL | `backend-engineer`, `software-architect`, `data-model-design` |
| `{{TEST_RUNNER}}` | `vitest` — **the runner, never the CI command with coverage flags.** `/load-task-context` needs its test-file convention to locate tests; `/write-module-readme` needs an invocation it can scope to one module and actually run; `e2e-and-coverage-engineer` reads its reports and uses its identity to recognise the tests it must **not** write; `author-test-plan` publishes its name | `frontend-engineer`, `backend-engineer`, `refactor-specialist`, `software-architect`, `open-pull-request`, `/load-task-context`, `/write-module-readme`, `e2e-and-coverage-engineer`, `author-test-plan`, `reproduce-and-diagnose-bug` |
| `{{TEST_COMMAND}}` | `pnpm test -- --coverage` | `cicd-pipeline-bringup` |
| `{{E2E_COMMAND}}` | `npx playwright test` — drives a browser against a deployed environment. **Not the unit-test command**; see the near-pair note above | `e2e-and-coverage-engineer` |
| `{{STAGING_BASE_URL}}` | `https://staging.example.com` — what the E2E suite targets. **Never a production URL** | `e2e-and-coverage-engineer` |
| `{{PERF_TEST_ENV}}` | `perf.example.com (provisioned per run)` — the **performance-matched** environment. **Must never hold the same value as `{{STAGING_BASE_URL}}`**; filling both alike disables the `PASS WITHHELD` rule by configuration, and nothing in the template can detect it | `load-test-engineer` |
| `{{A11Y_SCAN_COMMAND}}` | `pnpm dlx @axe-core/cli http://localhost:3000` — must run against a **rendered** page. Not the unit-test command; see the near-pair note above | `accessibility-auditor` |

### Design tooling and contracts

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{ERASER_MCP_SERVER}}` | `eraser`, or a distinct per-project name such as `eraser-acmeco` — must match the name registered in the repo's committed `.mcp.json` | `render-design-diagrams` |
| `{{API_BASE_PATH}}` | `/api/v1` | `api-contract-freeze` |
| `{{MOCK_SERVER}}` | `Prism`, or `Mockoon` for Windows-heavy teams | `api-contract-freeze` |

### Infrastructure

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{TF_CLI}}` | `terraform` or `tofu` | `terraform-iac-engineer`, `ci-identity-and-secrets-bringup`, `deploy-and-rollback-bringup`. **Deliberately not `iac-state-backend-bringup`** — see the note below the table |
| `{{CLOUD_PROVIDER}}` | `AWS` | `terraform-iac-engineer`, `iac-state-backend-bringup`, `ci-identity-and-secrets-bringup`, `cost-guardrails-bringup` |
| `{{DEPLOY_TARGET}}` | `ECS Fargate` | `cicd-pipeline-bringup`, `deploy-and-rollback-bringup` |
| `{{SECRETS_MECHANISM}}` | `AWS Secrets Manager` | `software-architect` |
| `{{SECRETS_MANAGER}}` | `AWS Secrets Manager` | `terraform-iac-engineer`, `ci-identity-and-secrets-bringup` |
| `{{CLAUDE_ACTION_VERSION}}` | `v1.0.158` — **exact, never a floating `@v1`** | `cicd-pipeline-bringup` |

> **`{{TF_CLI}}` was withdrawn from `iac-state-backend-bringup` on the Phase 6 re-check, and the reason is the Phase 5 rule reaching a fourth name.** That skill's step 1 **refuses** until the cloud and CLI choices are *recorded decisions with a named reviewer* — the licence rationale is a Gate 1 line in its own right. `{{TF_CLI}}` appeared nowhere else in its body, so the name existed **only inside the guard it disabled**: instantiated, the agent reads `terraform` and never asks whether anyone chose it. Same shape as `{{ENTRY_POINTS}}`, `{{DATA_CLASSIFICATION}}` and `{{COMPLIANCE_SCOPE}}` — **a stop-and-ask guard and a placeholder over the same fact cannot coexist; the placeholder wins, silently, on the first run and every run after.** `{{CLOUD_PROVIDER}}` stays in that skill because steps 3 and 4 genuinely vary on it (lock table vs native lockfile vs blob lease), and step 1 now says in words that a configured value is not a recorded decision.

### Observability and alerting

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{OBSERVABILITY_STACK}}` | `OpenTelemetry + Grafana` | `software-architect` |
| `{{OBS_BACKEND}}` | `Grafana` — one only | `observability-bringup`, `deploy-and-rollback-bringup` |
| `{{ALERT_ROUTER}}` | `PagerDuty` | `observability-bringup` |
| `{{ALERT_CHANNEL}}` | `#platform-oncall` | `cost-guardrails-bringup` |

---

## The hard requirements

Each is already stated in the per-directory READMEs and repeated here because its cost is not recoverable. *(This section was headed "Two" while listing three; it is now headed for the list rather than the count, which is why adding a fifth needed no edit to the heading.)*

- **`{{CLOUD_PROVIDER}}` and `{{TF_CLI}}` must be filled before any infrastructure helper runs.** A guessed infrastructure setting is the one mistake in this framework with no cheap undo.
- **`{{CLAUDE_ACTION_VERSION}}` must be an exact released version.** A floating tag makes the CI behaviour of every repo change without a commit.
- **`{{A11Y_SCAN_COMMAND}}` must render the page.** Filled with a unit-test command, the accessibility auditor can never reach PASS — and the symptom is an agent that looks pedantic rather than a placeholder that is wrong, so nobody goes looking for the real cause.
- **`{{PERF_TEST_ENV}}` must not be the ordinary staging environment.** This is the mirror image of the row above and the more dangerous direction: a bad `{{A11Y_SCAN_COMMAND}}` makes a verdict **unreachable**, which someone eventually investigates, while a `{{PERF_TEST_ENV}}` pointed at staging makes a PASS **reachable that should not be** — and a PASS is the verdict that stops anyone else looking. The agent is *told* which environment is performance-matched; it cannot measure whether that is true.
- **`{{DEPENDENCY_MANIFESTS}}` must list every workspace that ships.** Same direction as `{{PERF_TEST_ENV}}`, arriving by omission rather than by a wrong value: a monorepo fill naming only the primary service leaves the other workspaces' CVEs outside the triage entirely, so **Gate 3's "0 Critical CVEs in production dependencies" passes against a tree nobody counted.** There is no failure, no warning and no empty section — the omitted packages simply never appear, and a short clean report is exactly what the reader was hoping for. The one defence is re-reading this fill whenever a workspace is added, which is why it names workspaces rather than a directory.

---

## One hardcoded counterpart — worse than a synonym pair

**[`02-system-design/skill-prompts/api-contract-freeze/SKILL.md`](../aidlc-phases/02-system-design/skill-prompts/api-contract-freeze/SKILL.md) writes `docs/api/openapi.yaml` as a literal**, in its step 3 (where the spec is generated) and its step 5 (where the mock server is pointed at it). Phase 3's `{{API_SPEC_PATH}}` names the same fact as a placeholder.

**This is a worse failure than a synonym pair, and it is worth being precise about why.** A synonym pair at least appears in the reconciliation table above, so an operator working fact-first meets both names and fills them consistently. **A literal appears nowhere** — not in this table, not in the skill's placeholder list, not in the directory README's fill instructions. A repo that keeps its spec somewhere else gets a skill that writes to the wrong path and a mock server pointed at a file that does not exist, **with nothing anywhere signalling that a value was assumed on its behalf.** The operator has no surface on which to notice.

**Recorded, not fixed.** That file is outside the scope of the change that found it, and more importantly: **renaming inside a vendored template breaks every repo that already instantiated it**, which is the same converge-on-next-touch rule the six pairs below are held under. The correct moment is the next substantive edit to `api-contract-freeze`, where the two literals become `{{API_SPEC_PATH}}` and the skill's fill list gains a row. Until then this note is the only place the assumption is visible.

> **The general form is worth a sweep somebody has not run yet.** The reconciliation table catches *one fact, two names*. The new row above catches *one name, two actors*. **Neither catches *one fact, one literal and one name*** — and a literal is invisible to every check this file performs, because every check starts by finding `{{...}}`. Grepping the shipped templates for hardcoded paths that another template parameterises is a different routine from anything currently documented here.

---

## Naming rule for new placeholders

1. **Check this file first.** If the fact already has a name, **reuse that name** — even if it was coined in another phase, and even if you would have named it differently.
2. If it is genuinely new, name it after **the fact, not the consumer**: `{{DATASTORE}}`, not `{{BACKEND_DB}}`.
3. Add the row here in the same PR as the template. A placeholder that ships without a row is how the seventh synonym pair gets created.
4. **Check whether the fact is already written as a literal somewhere.** A name that duplicates an existing hardcoded path is not a synonym pair and gets no row from rules 1-3 — see the hardcoded-counterpart section above. **A literal is invisible to a grep for `{{...}}`**, which is how every other check in this file works.
5. **If two templates will both *act* on the fact, say so in the row** — not just where it is consumed. `{{PR_TITLE_FORMAT}}` and `{{TEST_RUNNER}}` both have consumers whose behaviour must agree, and an inconsistent fill surfaces as a de-linked PR or a test suite that finds nothing, not as a configuration error anyone traces back.
6. **Check whether the template stops to ask for this fact.** If it does, do not mint the name — a placeholder pre-answers the question the guard exists to force, and it wins on every run thereafter. See [the pre-fillable guards](#a-fourth-kind-of-case--names-that-would-disable-a-guard-by-being-filled). Rules 1-4 all ask what the fact is called elsewhere; this one asks what happens to the template's own refusals once the fact has a value.

> **On renaming to converge.** The six pairs above are *not* being renamed today, deliberately. These templates are vendored by copy into consuming repos, so a rename sweep breaks every repo that already instantiated them and buys nothing an operator using this table does not already get. **Converge on next touch:** when a template is edited for another reason, adopt the canonical name then, and update this file's row in the same commit. Revisit a coordinated rename only if the pair count grows past what one table comfortably holds.
