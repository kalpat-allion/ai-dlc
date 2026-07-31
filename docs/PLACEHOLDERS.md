# Placeholder Reconciliation

> **What this is.** One row per **repository fact**, listing every placeholder name the shipped templates use for it. Its job is not to hold your values — the per-directory READMEs do that, and they are what an operator has open while instantiating. Its job is to stop the same fact being asked for twice under two names.
>
> **The problem it solves.** The three shipped phases carry **36 distinct placeholders** across 18 templates, and **literal overlap between the phases is zero** — no placeholder name appears in more than one phase. That sounds clean and is not: what exists instead is **six pairs that name the same repo fact differently**. An operator instantiating two phases fills the same value twice under two names, and nothing catches an inconsistent fill. That defect is invisible from inside any single README, which is why this file is central.

---

## The reconciliation table

These are the rows that matter. Each is one fact about the repository, asked for under more than one name.

| Repo fact | Asked for as | In | Must agree because |
|---|---|---|---|
| **How secrets reach the runtime** | `{{SECRETS_MECHANISM}}` | `software-architect` | The architect designs against the mechanism the IaC actually references. Two answers means designs that cite a secret store the infrastructure does not use |
| | `{{SECRETS_MANAGER}}` | `terraform-iac-engineer`, `ci-identity-and-secrets-bringup` | |
| **Metrics / traces / logs backend** | `{{OBSERVABILITY_STACK}}` | `software-architect` | The dashboards and SLOs are built on the same backend the design assumes instrumentation reports to |
| | `{{OBS_BACKEND}}` | `observability-bringup`, `deploy-and-rollback-bringup` | |
| **How tests are run** | `{{TEST_RUNNER}}` | `frontend-engineer`, `backend-engineer`, `refactor-specialist`, `software-architect` | CI runs the command; the specialists run the runner. If they disagree, tests pass locally and the pipeline runs nothing |
| | `{{TEST_COMMAND}}` | `cicd-pipeline-bringup` | |
| **The service's directory** | `{{FRONTEND_ROOT}}` / `{{BACKEND_ROOT}}` | `frontend-engineer`, `backend-engineer` | The Docker build context must be a directory the code actually lives in. In a multi-service repo these are genuinely different values — **check, do not assume** |
| | `{{SERVICE_ROOT}}` | `container-image-engineer` | |
| **The Linear team** | `{{TEAM_PREFIX}}` — its *key*, e.g. `ENG` | `linear-task-agent` | Two representations of one object. A prefix from a different team than the one the Project sits under produces issues nobody's saved view shows |
| | `{{LINEAR_TEAM}}` — its *name*, e.g. `Engineering` | `publish-prd-to-linear` | |
| **Language and framework** | `{{TEAM_STACK}}`, `{{FRONTEND_FRAMEWORK}}`, `{{BACKEND_FRAMEWORK}}` | the P3 specialists | Not synonyms, but **constrained by each other**: a `{{RUNTIME}}` of `Node 22` under a `{{TEAM_STACK}}` of `FastAPI + Python 3.12` is a contradiction that ships a broken image |
| | `{{RUNTIME}}` — language + version, e.g. `Node 22` | `container-image-engineer` | |

**Fill the left column once, then propagate.** Working fact-first rather than file-first is the whole point; going template-by-template is what produces the inconsistent fill.

---

## Full inventory

All 36, grouped by what they describe. **Where each is consumed** is authoritative — it is generated from the templates, not remembered.

### Tracker and conventions

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{TEAM_PREFIX}}` | `ENG` | `linear-task-agent` |
| `{{LINEAR_TEAM}}` | `Engineering` | `publish-prd-to-linear` |
| `{{LINEAR_PROJECT}}` | `TimeSync v2 — Scheduling` | `linear-task-agent` |
| `{{LINEAR_INITIATIVE}}` | `TimeSync v2` | `publish-prd-to-linear` |
| `{{LABEL_CONVENTIONS}}` | `feature, bug, tech-debt, chore` | `linear-task-agent` |
| `{{TECH_DEBT_LABEL}}` | `tech-debt` | `linear-task-agent` |
| `{{BUG_LABEL}}` | `bug` | `linear-task-agent` |
| `{{PR_TITLE_FORMAT}}` | `[ENG-XXX] <imperative summary>` | `linear-task-agent` |

### Stack and runtime

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{TEAM_STACK}}` | `Next.js 14 + TypeScript` | `frontend-engineer`, `backend-engineer`, `software-architect` |
| `{{FRONTEND_FRAMEWORK}}` | `React 18` | `software-architect` |
| `{{BACKEND_FRAMEWORK}}` | `NestJS` | `software-architect` |
| `{{RUNTIME}}` | `Node 22` | `container-image-engineer` |
| `{{AUTH_STACK}}` | `Clerk` | `software-architect` |
| `{{REALTIME_STACK}}` | `Socket.IO`, or `none` | `software-architect` |
| `{{ASYNC_JOB_STACK}}` | `BullMQ`, or `none` | `software-architect` |

### Paths

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{FRONTEND_ROOT}}` | `apps/web` | `frontend-engineer` |
| `{{BACKEND_ROOT}}` | `services/api` | `backend-engineer` |
| `{{SERVICE_ROOT}}` | `services/api`, or `.` | `container-image-engineer` |
| `{{COMPONENT_LIBRARY_PATH}}` | `apps/web/src/components/ui` | `frontend-engineer`, `software-architect` |
| `{{API_CLIENT_PATH}}` | `apps/web/src/lib/api` | `frontend-engineer` |
| `{{MIGRATIONS_PATH}}` | `services/api/migrations` | `backend-engineer` |
| `{{IAC_DIR}}` | `infra/` | `terraform-iac-engineer`, `iac-state-backend-bringup`, `ci-identity-and-secrets-bringup`, `cost-guardrails-bringup` |

### Data and testing

| Placeholder | Example | Consumed by |
|---|---|---|
| `{{ORM}}` | `Prisma` | `backend-engineer`, `software-architect` |
| `{{DATASTORE}}` | `PostgreSQL 16` | `backend-engineer`, `software-architect` |
| `{{TEST_RUNNER}}` | `vitest` | `frontend-engineer`, `backend-engineer`, `refactor-specialist`, `software-architect` |
| `{{TEST_COMMAND}}` | `pnpm test -- --coverage` | `cicd-pipeline-bringup` |

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

---

## Naming rule for new placeholders

1. **Check this file first.** If the fact already has a name, **reuse that name** — even if it was coined in another phase, and even if you would have named it differently.
2. If it is genuinely new, name it after **the fact, not the consumer**: `{{DATASTORE}}`, not `{{BACKEND_DB}}`.
3. Add the row here in the same PR as the template. A placeholder that ships without a row is how the seventh synonym pair gets created.

> **On renaming to converge.** The six pairs above are *not* being renamed today, deliberately. These templates are vendored by copy into consuming repos, so a rename sweep breaks every repo that already instantiated them and buys nothing an operator using this table does not already get. **Converge on next touch:** when a template is edited for another reason, adopt the canonical name then, and update this file's row in the same commit. Revisit a coordinated rename only if the pair count grows past what one table comfortably holds.
