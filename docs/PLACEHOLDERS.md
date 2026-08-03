# Placeholder Reconciliation

> **What this is.** One row per **repository fact**, listing every placeholder name the shipped templates use for it. Its job is not to hold your values — the per-directory READMEs do that, and they are what an operator has open while instantiating. Its job is to stop the same fact being asked for twice under two names.
>
> **The problem it solves.** The four shipped phases carry **47 distinct placeholders** across **29 templates**. Before Phase 2, **literal overlap between phases was zero** — no placeholder name appeared in more than one phase — and what existed instead was **six pairs that name the same repo fact differently**. An operator instantiating two phases fills the same value twice under two names, and nothing catches an inconsistent fill. That defect is invisible from inside any single README, which is why this file is central.
>
> **Phase 2 is the first phase to reuse rather than re-coin, and that is this file working.** It uses **16** placeholders, of which **8 are existing names adopted from Phases 3 and 6** — `{{ORM}}`, `{{DATASTORE}}`, `{{MIGRATIONS_PATH}}`, `{{AUTH_STACK}}`, `{{TEAM_STACK}}`, `{{CLOUD_PROVIDER}}`, `{{FRONTEND_FRAMEWORK}}`, `{{FRONTEND_ROOT}}` — and only 8 are new. **The pair count is still six, not fourteen.** Two near-pairs were caught at authoring time and are recorded below rather than minted. Phase 2 is also the first phase whose placeholders literally overlap another phase's, which is the intended end state, not a regression.
>
> **Phase 3's four new artifacts carry 10 placeholders and coined only 3** — `{{DEFAULT_BRANCH}}`, `{{REQUIREMENTS_DIR}}`, `{{API_SPEC_PATH}}`. Seven are reused: `{{TEST_RUNNER}}`, `{{PR_TITLE_FORMAT}}`, `{{LINEAR_PROJECT}}`, `{{TEAM_PREFIX}}`, `{{TEAM_STACK}}`, `{{ADR_DIR}}`, `{{TECH_DEBT_LABEL}}`. **The pair count is still six**, but the build added a **seventh row of a different kind** — not a synonym, but one fact two templates both act on. See below.
>
> *(The counts above were also recounted rather than incremented. The header said 44 and the inventory said "All 45"; the templates carried **44**, so the inventory heading was one over before this build and is corrected to **47** now.)*

---

## The reconciliation table

These are the rows that matter. Each is one fact about the repository, asked for under more than one name.

| Repo fact | Asked for as | In | Must agree because |
|---|---|---|---|
| **How secrets reach the runtime** | `{{SECRETS_MECHANISM}}` | `software-architect` | The architect designs against the mechanism the IaC actually references. Two answers means designs that cite a secret store the infrastructure does not use |
| | `{{SECRETS_MANAGER}}` | `terraform-iac-engineer`, `ci-identity-and-secrets-bringup` | |
| **Metrics / traces / logs backend** | `{{OBSERVABILITY_STACK}}` | `software-architect` | The dashboards and SLOs are built on the same backend the design assumes instrumentation reports to |
| | `{{OBS_BACKEND}}` | `observability-bringup`, `deploy-and-rollback-bringup` | |
| **How tests are run** | `{{TEST_RUNNER}}` | `frontend-engineer`, `backend-engineer`, `refactor-specialist`, `software-architect`, `open-pull-request`, `/load-task-context`, `/write-module-readme` | CI runs the command; the specialists run the runner. If they disagree, tests pass locally and the pipeline runs nothing |
| | `{{TEST_COMMAND}}` | `cicd-pipeline-bringup` | |
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
| **How tests are run** | `{{TEST_RUNNER}}` — one name, **five consumers**, three of which run it and two of which read its file convention | `frontend-engineer`, `backend-engineer`, `refactor-specialist`, `software-architect`, `open-pull-request`, `/load-task-context`, `/write-module-readme` | Already a reconciliation row against `{{TEST_COMMAND}}`. The Phase 3 additions make it the same shape as the row above: `/load-task-context` needs the runner's **test-file convention** to locate tests and `/write-module-readme` needs an invocation it can **scope to one module and actually run**. **Filled with the whole-suite CI command, both degrade silently** — one finds no tests, the other documents a command that does not test the module |

**This argues the table's framing should widen** from *"synonym pairs"* to *"any repo fact that two or more templates both act on"*. The original framing caught the case where one fact has two names; it does not catch the case where one name has two actors whose behaviour must agree. **Both fail the same way — an inconsistent fill that surfaces as a behaviour bug rather than as a configuration error** — and only the first kind was getting a row. The widening is recorded here rather than applied as a restructure: the six existing rows are still correct as synonym pairs, and re-cutting the whole table is a change to make when a third row of this kind appears, not on the second.

### Two near-pairs, caught before they were minted

Neither is a reconciliation row, because in both cases the second name was **not created**. They are recorded because the reasoning is the thing worth reusing.

- **`{{AUTH_STACK}}` now carries two related but distinct facts.** `software-architect` means the auth *provider* (`Clerk`); `api-contract-freeze` needs the *wire security scheme* for the OpenAPI `securitySchemes` block. Coining `{{AUTH_SCHEME}}` would have been the seventh pair, so the naming rule was applied as written — *reuse the existing name even if you would have named it differently* — and the skill is phrased as "a security scheme for `{{AUTH_STACK}}`". **If a repo ever has a provider and a scheme that genuinely disagree, this is the row that splits**, under converge-on-next-touch.
- **`{{A11Y_SCAN_COMMAND}}` deliberately does *not* reuse `{{TEST_RUNNER}}` or `{{TEST_COMMAND}}`.** Adjacent, but a different fact: the auditor's PASS condition requires a command that renders the page. Filling it with the unit-test command makes PASS permanently unreachable, and the failure looks like a strict agent rather than a bad fill.
- **`{{DEFAULT_BRANCH}}` deliberately does *not* adopt the spelling `__BASE_BRANCH__`.** The near-hit is in [`03-development/PR-REVIEW-BOT.md`](../aidlc-phases/03-development/PR-REVIEW-BOT.md), and it is **not a template placeholder at all** — it is a GitHub Actions **runtime** substitution, filled by the workflow from `github.event.pull_request.base.ref` at each run. **An operator never fills it and must never try.** Adopting its spelling would imply the two are the same mechanism, and the first person to "reconcile" them would either hard-code a branch name into a workflow or leave a template placeholder unfilled on the assumption that something substitutes it. Different lifetime, different filler, different name.
- **`{{API_SPEC_PATH}}` deliberately does *not* reuse `{{API_BASE_PATH}}`.** A file path (`docs/api/openapi.yaml`) and a URL prefix (`/api/v1`) are different facts that happen to share three letters. Reusing the name would produce a spec written to `/api/v1` and a mock server pointed at a directory.

---

## Full inventory

All 47, grouped by what they describe. **Where each is consumed** is authoritative — it is generated from the templates, not remembered.

### Tracker and conventions

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{TEAM_PREFIX}}` | `ENG` | `linear-task-agent`, `run-sprint-planning`, `/load-task-context` |
| `{{LINEAR_TEAM}}` | `Engineering` | `publish-prd-to-linear` |
| `{{LINEAR_PROJECT}}` | `TimeSync v2 — Scheduling` | `linear-task-agent`, `run-sprint-planning` |
| `{{LINEAR_INITIATIVE}}` | `TimeSync v2` | `publish-prd-to-linear` |
| `{{LABEL_CONVENTIONS}}` | `feature, bug, tech-debt, chore` | `linear-task-agent` |
| `{{TECH_DEBT_LABEL}}` | `tech-debt` | `linear-task-agent`, `/write-module-readme` |
| `{{BUG_LABEL}}` | `bug` | `linear-task-agent` |
| `{{PR_TITLE_FORMAT}}` | `[ENG-XXX] <imperative summary>` — **two composers; see the reconciliation note above** | `linear-task-agent`, `open-pull-request` |
| `{{DEFAULT_BRANCH}}` | `main` — the branch PRs target *and* the branch you rebase onto. **Not** `__BASE_BRANCH__`, which is a workflow runtime substitution and not an operator-filled value | `open-pull-request` |

### Stack and runtime

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{TEAM_STACK}}` | `Next.js 14 + TypeScript` | `frontend-engineer`, `backend-engineer`, `software-architect`, `solution-architect`, `run-sprint-planning` — **estimates sized against the wrong stack are confidently wrong** |
| `{{FRONTEND_FRAMEWORK}}` | `React 18` | `software-architect`, `accessibility-auditor` |
| `{{BACKEND_FRAMEWORK}}` | `NestJS` | `software-architect` |
| `{{RUNTIME}}` | `Node 22` | `container-image-engineer` |
| `{{AUTH_STACK}}` | `Clerk` — the auth **provider** for the architect, and the **wire security scheme** the OpenAPI spec declares. See the near-pair note above | `software-architect`, `api-contract-freeze` |
| `{{REALTIME_STACK}}` | `Socket.IO`, or `none` | `software-architect` |
| `{{ASYNC_JOB_STACK}}` | `BullMQ`, or `none` | `software-architect` |

### Paths

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{FRONTEND_ROOT}}` | `apps/web` | `frontend-engineer`, `accessibility-auditor` |
| `{{BACKEND_ROOT}}` | `services/api` | `backend-engineer` |
| `{{SERVICE_ROOT}}` | `services/api`, or `.` | `container-image-engineer` |
| `{{COMPONENT_LIBRARY_PATH}}` | `apps/web/src/components/ui` | `frontend-engineer`, `software-architect` |
| `{{API_CLIENT_PATH}}` | `apps/web/src/lib/api` | `frontend-engineer` |
| `{{MIGRATIONS_PATH}}` | `services/api/migrations` | `backend-engineer`, `data-model-design` |
| `{{IAC_DIR}}` | `infra/` | `terraform-iac-engineer`, `iac-state-backend-bringup`, `ci-identity-and-secrets-bringup`, `cost-guardrails-bringup` |
| `{{SCHEMA_PATH}}` | `prisma/schema.prisma` | `data-model-design` |
| `{{ADR_DIR}}` | `docs/adrs/` — **the first name to span Phases 2 and 3**; the value `/adr` writes into is the value `/load-task-context` reads back out | `/adr`, `solution-architect`, `architecture-reviewer`, `/load-task-context` |
| `{{ARCHITECTURE_DIR}}` | `docs/architecture/` | `solution-architect`, `architecture-reviewer` |
| `{{A11Y_REPORT_PATH}}` | `docs/accessibility/wcag-aa-report.md` | `accessibility-auditor` |
| `{{REQUIREMENTS_DIR}}` | `docs/prd/` — if requirements live in the tracker rather than the repo, point it at the tracker Document set and say so in the fill | `/load-task-context` |
| `{{API_SPEC_PATH}}` | `docs/api/openapi.yaml` — the spec **file**. Deliberately **not** `{{API_BASE_PATH}}`, which is a URL prefix and a different fact. Both Phase 3 commands consume it for opposite reasons: one reads a section from it, the other refuses to hand-copy anything that renders from it | `/load-task-context`, `/write-module-readme` |

### Data and testing

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{ORM}}` | `Prisma` | `backend-engineer`, `software-architect`, `data-model-design` |
| `{{DATASTORE}}` | `PostgreSQL 16` — **include the version**, it changes the available DDL | `backend-engineer`, `software-architect`, `data-model-design` |
| `{{TEST_RUNNER}}` | `vitest` — **the runner, never the CI command with coverage flags.** `/load-task-context` needs its test-file convention to locate tests; `/write-module-readme` needs an invocation it can scope to one module and actually run | `frontend-engineer`, `backend-engineer`, `refactor-specialist`, `software-architect`, `open-pull-request`, `/load-task-context`, `/write-module-readme` |
| `{{TEST_COMMAND}}` | `pnpm test -- --coverage` | `cicd-pipeline-bringup` |
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
| `{{TF_CLI}}` | `terraform` or `tofu` | `terraform-iac-engineer`, `iac-state-backend-bringup`, `ci-identity-and-secrets-bringup`, `deploy-and-rollback-bringup` |
| `{{CLOUD_PROVIDER}}` | `AWS` | `terraform-iac-engineer`, `iac-state-backend-bringup`, `ci-identity-and-secrets-bringup`, `cost-guardrails-bringup` |
| `{{DEPLOY_TARGET}}` | `ECS Fargate` | `cicd-pipeline-bringup`, `deploy-and-rollback-bringup` |
| `{{SECRETS_MECHANISM}}` | `AWS Secrets Manager` | `software-architect` |
| `{{SECRETS_MANAGER}}` | `AWS Secrets Manager` | `terraform-iac-engineer`, `ci-identity-and-secrets-bringup` |
| `{{CLAUDE_ACTION_VERSION}}` | `v1.0.158` — **exact, never a floating `@v1`** | `cicd-pipeline-bringup` |

### Observability and alerting

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{OBSERVABILITY_STACK}}` | `OpenTelemetry + Grafana` | `software-architect` |
| `{{OBS_BACKEND}}` | `Grafana` — one only | `observability-bringup`, `deploy-and-rollback-bringup` |
| `{{ALERT_ROUTER}}` | `PagerDuty` | `observability-bringup` |
| `{{ALERT_CHANNEL}}` | `#platform-oncall` | `cost-guardrails-bringup` |

---

## Two hard requirements

Both are already stated in the per-directory READMEs and are repeated here because they are the only two whose cost is not recoverable:

- **`{{CLOUD_PROVIDER}}` and `{{TF_CLI}}` must be filled before any infrastructure helper runs.** A guessed infrastructure setting is the one mistake in this framework with no cheap undo.
- **`{{CLAUDE_ACTION_VERSION}}` must be an exact released version.** A floating tag makes the CI behaviour of every repo change without a commit.
- **`{{A11Y_SCAN_COMMAND}}` must render the page.** Filled with a unit-test command, the accessibility auditor can never reach PASS — and the symptom is an agent that looks pedantic rather than a placeholder that is wrong, so nobody goes looking for the real cause.

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

> **On renaming to converge.** The six pairs above are *not* being renamed today, deliberately. These templates are vendored by copy into consuming repos, so a rename sweep breaks every repo that already instantiated them and buys nothing an operator using this table does not already get. **Converge on next touch:** when a template is edited for another reason, adopt the canonical name then, and update this file's row in the same commit. Revisit a coordinated rename only if the pair count grows past what one table comfortably holds.
