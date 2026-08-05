# Phase 6: CI/CD & DevOps — Quality Gates

The **six** gates below align with the eight [PROCESS.md](./PROCESS.md) steps. Step 0 is setup, verified by its own checklist. Steps 1–3 fold into Gate 1 (infrastructure foundation); Steps 4–7 each have their own gate (Gates 2–5); Gate 6 covers the handoff to Phase 7. Pass criteria are absolute — items unchecked block progression.

---

## Gate 1: Infrastructure Foundation

Before any workload is deployed to cloud resources.

### Tech stack & cost (Steps 1–2)
- [ ] Cloud provider, IaC tool & licence, state backend, orchestration, observability decisions documented as ADRs
- [ ] Terraform (primary) or OpenTofu (CLI-compatible fallback) chosen, with an explicit **BSL 1.1 vs MPL 2.0** rationale in the tech-stack ADR naming who reviewed the licence
- [ ] Initial cost estimate generated and within budget (low / expected / peak scenarios) — **human-run against a cited price source. No Phase 6 helper produces this figure**, and `cost-guardrails-bringup` refuses to
- [ ] Cost-on-PR active: **Infracost `diff`** posts a comment on every IaC PR; Infracost AutoFix enabled
- [ ] Cloud-native budget alerts set at 50% / 80% / 100%
- [ ] Unit-economics metric (cost per active user / cost per request) wired to a dashboard
- [ ] `cost-guardrails-bringup` skill run; its output (CI job, breakdown baseline, budget alerts, unit-economics metric, reviewer checklist) committed and the PR comment verified on a real IaC PR. **It answers four of this block's five substantive items** — the initial cost estimate above is the one it refuses, so a run that reports the whole cost block green has produced a number nobody priced

### IaC repo and modules (Step 3)
- [ ] Terraform project bootstrapped with `required_version` and `required_providers` pinned (`~>`), and `.terraform.lock.hcl` committed
- [ ] One directory per environment (`/envs/dev`, `/envs/staging`, `/envs/prod`) with a shared module library (`/modules/*`); workspace-vs-directory choice documented as an ADR
- [ ] `terraform fmt -recursive` clean and `terraform validate` passing in CI
- [ ] Every shared module has at least one native `terraform test` case (`.tftest.hcl`) using `mock_provider`, and it runs in CI — **no run block uses `command = apply`**
- [ ] `AGENTS.md` committed at repo root and referenced by the Phase 6 subagents and skills — mandatory tags, forbidden resource types, region defaults, production-approval requirement all filled in
- [ ] `terraform-iac-engineer` subagent committed under `.claude/agents/`, every placeholder filled, and smoke-tested (`validate` + `plan` on the real repo, then a refused apply request)
- [ ] **0 `cannot determine from the plan` rows left open** in the subagent's conventions self-check. That self-check returns *three* verdicts — met / not met / **cannot determine from the plan** — for values the plan renders as `(known after apply)` or that a provider default supplies. **An undetermined row is not a pass**: each one is closed by a named human who read the real value, before this gate is checked. A self-check summarised as clean while carrying undetermined rows reads identically to one that verified them

### State backend — bring-up work that used to come free with a control plane (Step 3.4)
- [ ] Remote state backend provisioned **in the project's own cloud account** (S3 / Azure Blob / GCS) — no managed control plane required, $0 licence
- [ ] **Bucket / container versioning is ON** — this is the only recovery path from a corrupted or force-unlocked state
- [ ] **Encryption at rest is ON with a customer-managed key** (SSE-KMS / CMEK / customer-managed SSE)
- [ ] **State locking verified working** — DynamoDB lock table exists (or S3 native lockfile on Terraform ≥ 1.10, or Blob lease) and a deliberate concurrent-apply attempt was **observed to block**
- [ ] **State bucket denies public access** (S3 Block Public Access on all four settings / Azure disable-public-blob-access / GCS public-access prevention enforced)
- [ ] **Cloud-native audit covers the state bucket** — CloudTrail data events / Azure Monitor + Storage logging / GCP Cloud Audit Logs (data access) — **and** the GitHub audit log is enabled. Together these replace a vendor control plane's audit log; there is no single console consolidating them, so both must be confirmed on.
- [ ] **CI writes via OIDC; humans hold read-only** — the state bucket's IAM policy grants write to exactly one role, assumable only by the GitHub Actions OIDC identity (`sub`-scoped to this repository and environment); every human principal has read-only or no access
- [ ] State file is **never** committed to git, never uploaded as a CI artefact, never printed to a log — verified with `git ls-files` that none is already tracked
- [ ] No `*.tfvars` file containing a secret value is committed; secrets come from the secret manager or OIDC
- [ ] `iac-state-backend-bringup` skill committed under `.claude/skills/` and smoke-tested end-to-end on a non-production environment

### Apply path — no laptop applies, enforced by IAM (Steps 3.5–3.6)
- [ ] `terraform apply` against staging or prod runs **only** in CI under the OIDC role — verified by attempting an apply from a developer machine and confirming it fails on **authorisation**, not on convention
- [ ] The apply job consumes a **saved plan file** produced by the reviewed `plan` run — no re-plan inside apply
- [ ] Production apply sits behind a GitHub Environment with required reviewers
- [ ] The OIDC trust policy is scoped to repository **and branch and environment** — not to the repository alone
- [ ] Secrets that OIDC cannot cover live in the project's secret manager (AWS Secrets Manager / Azure Key Vault / GCP Secret Manager / Vault / Doppler / 1Password), scoped per environment, each with a named owner and a rotation date
- [ ] Scheduled drift workflow (`terraform plan -detailed-exitcode`) exists, **has run successfully at least once**, and its exit-2 path opens a Linear issue and pages on-call for production
- [ ] `ci-identity-and-secrets-bringup` skill committed under `.claude/skills/` and smoke-tested

**Pass:** All items checked. IaC baseline established with a $0 control plane.

---

## Gate 2: CI/CD Pipeline Operational

Before the team relies on automation for deploys.

### Pipeline coverage
- [ ] Lint + format runs on every PR (< 2 min)
- [ ] Build runs on every PR
- [ ] Unit + integration tests run on every PR with the test runner's native coverage threshold failing the build below 80% on new code; coverage report published as a CI artefact
- [ ] Security scans on every PR (CodeQL SAST + Dependabot + ggshield + Claude Code `/security-review`)
- [ ] `terraform plan` runs on every IaC PR with the plan output and an Infracost delta comment posted to the PR
- [ ] E2E tests run against staging on merge to main
- [ ] Auto-deploy to staging on merge (no manual steps)
- [ ] Manual approval gate on production deploys (GitHub Environment with required reviewers)

### AI integration in CI
- [ ] Anthropic Claude Code Action committed, version pinned to an **exact** release (not a floating `@v1`)
- [ ] `@claude` mention triggers fix-on-mention workflow that opens a follow-up commit (separate workflow from any reviewer)
- [ ] The project's AI PR reviewer choice ([Phase 3 Step 4.3](../03-development/PROCESS.md#step-4-code-review) — CodeRabbit, the Claude PR review bot, or both) is recorded in the tech-stack ADR and wired on the repo
- [ ] Auto-review posts within ~3 min, severity-bucketed, and **never** approves the PR
- [ ] AI review is **not** a required status check — a provider outage cannot freeze the merge queue

*If the Claude PR review bot is one of the configured reviewers, additionally:*

- [ ] Committed per [PR-REVIEW-BOT.md](../03-development/PR-REVIEW-BOT.md) — workflow + prompt template + committed checklist
- [ ] Review submitted as `REQUEST_CHANGES` / `COMMENT`, never `APPROVE`; no AI attribution in the body
- [ ] Size gate trips on an oversized PR and posts a skip notice rather than failing silently
- [ ] Review job is `continue-on-error`; a forced failure posts a degraded-mode notice that clears on the next successful pass
- [ ] Second-pass test done: a finding marked `❌ Disagreed` is not re-raised on the next push
- [ ] At least one Copilot Agentic Workflow committed and dry-run passing (stale-issue triage / release-notes / flaky-test repair / lint-debt fix)
- [ ] GitLab Duo Root Cause Analysis enabled (if on GitLab fallback)

### Pipeline quality
- [ ] Total PR pipeline duration < 15 min
- [ ] Caching configured (deps + Docker layers + BuildKit cache mounts)
- [ ] Parallel jobs where dependencies allow
- [ ] All cloud auth via OIDC; every other secret resolved from the project's secret manager or a GitHub Environment-scoped encrypted secret; no raw credentials in workflow YAML
- [ ] `cicd-pipeline-bringup` skill committed under `.claude/skills/` and smoke-tested — the generated pipeline runs green on a real PR, the AI review job is **not** a required check, and no workflow uses `pull_request_target`. **The AI-integration block above is only partly its work, and it refuses the rest by name**: it wires `@claude` fix-on-mention (exact pin, bot-actor gate, spend cap) and will not commit auto-review-on-PR-open. **Choosing and wiring the AI PR reviewer — and every item in the review-bot sub-block below — has no Phase 6 helper behind it**; it is [Step 4.2a](./PROCESS.md#step-4-cicd-pipeline-setup), built by a human from [PR-REVIEW-BOT.md](../03-development/PR-REVIEW-BOT.md). A run that reports the AI-integration block green has claimed work the skill declined to do
- [ ] Cloud auth uses OIDC, not long-lived keys
- [ ] Claude Code Action gated against bot-PR token burn (`if: github.actor != 'dependabot[bot]'`)
- [ ] Anthropic API monthly spend cap configured (or routed to Bedrock / Vertex / Foundry per existing cloud commits)

### Branch protection
- [ ] `main` requires PR + all checks passing + AI review acknowledged (whichever reviewer is configured) + 1 human approval
- [ ] Force-push to `main` disabled
- [ ] Linear identifier required in PR title (Phase 3 convention)
- [ ] High-blast-radius changes (auth, schema migrations, IAM) require 2 humans

**Pass:** All items checked. Pipeline is production-ready.

---

## Gate 3: Containerization Standards

For every container image built by the project.

- [ ] Multi-stage Dockerfile (build stage + runtime stage)
- [ ] Runtime base image is distroless or Alpine — not full OS
- [ ] Base image pinned by digest (or specific version, **never** `latest`)
- [ ] Non-root user in runtime stage; `READONLY_ROOTFS=true` where possible
- [ ] Docker `HEALTHCHECK` directive present
- [ ] `.dockerignore` excludes `.git`, `node_modules`, tests, docs, secrets
- [ ] BuildKit cache mounts and CI layer cache configured
- [ ] Image size minimised (compare against previous release; flag > 20% growth)
- [ ] Docker Gordon `docker ai "rate my Dockerfile"` reviewed for each service
- [ ] `container-image-engineer` subagent committed under `.claude/agents/`, placeholders filled, and used (or reviewed against) for every service Dockerfile. **It self-certifies seven of the nine criteria above** — the image-size comparison and the Gordon review are the two it cannot verify and must hand back by name; a run that claims all nine has fabricated two

### If Kubernetes
- [ ] Manifests generated with Claude Code's `k8s-manifests-generation` prompt
- [ ] Deployment + Service + Ingress + ConfigMap + Secret (External Secrets Operator) + HPA + PDB + NetworkPolicy present
- [ ] K8sGPT operator running in-cluster; MCP mode connected to Claude Code
- [ ] Helm or Kustomize choice documented as ADR

**Pass:** All items checked per service container.

---

## Gate 4: Observability & Alerting

Before production traffic reaches the service.

### Instrumentation
- [ ] OpenTelemetry SDK integrated for metrics, traces, logs
- [ ] Auto-instrumentation enabled for the runtime (Java / Node / Python / Go / .NET)
- [ ] Structured JSON logs with correlation IDs
- [ ] Custom business metrics exposed (Prometheus exposition or OTLP)
- [ ] Distributed tracing covers request path end-to-end (ingress → backend → DB → cache → external API)

### Backend choice committed
- [ ] **Datadog + Bits AI SRE** **OR** **Grafana Cloud + Assistant + Sift** chosen and live (ADR'd in Step 1)
- [ ] Datadog / Grafana MCP servers connected for incident workflows

### Dashboards (versioned as code)
- [ ] Service Health dashboard (latency, errors, traffic, resource usage, dependency health)
- [ ] SLO Tracking dashboard (SLI vs target + error budget remaining)
- [ ] Business KPI dashboard (signups / orders / active users — product-relevant)
- [ ] Dashboards committed at `/observability/dashboards/`

### Alerts
- [ ] One SLO per Phase-1 NFR (latency, availability, error rate)
- [ ] Burn-rate alerts: fast burn (2% / 1h → page) + slow burn (10% / 6h → ticket)
- [ ] Alert routing configured (PagerDuty / Opsgenie / incident.io)
- [ ] Every alert links to a runbook at `/docs/runbooks/alerts/<alert-name>.md` — verifiable by script, since `<alert-name>` is the backend's alert name kebab-cased
- [ ] Alerts tested — verified fire + verified resolve
- [ ] `observability-bringup` skill committed under `.claude/skills/` and smoke-tested — dashboards, SLOs, alerts and runbooks all produced by one chained run, not four ad-hoc ones. **The Instrumentation block above is not the skill's** — OpenTelemetry integration is application code and belongs to the implementation specialists. **Every runbook it produces is a draft, and a draft is not a pass**: the runbook line above is satisfied only once an SRE has signed off each one, which the skill refuses to do on its own behalf and which is what wiring the alert to production waits on
- [ ] Bits AI SRE / Sift configured to triage alerts before on-call paging
- [ ] No alerts that don't require human action (no noise)

**Pass:** All items checked. Observability production-ready.

---

## Gate 5: Deployment & Rollback

Every production deploy must satisfy.

- [ ] Deployment strategy ADR (blue/green / canary / rolling) committed
- [ ] Health checks (`/healthz` + `/readyz`) block traffic during deploy
- [ ] Phase-4 Playwright smoke suite runs post-deploy before traffic fully routes
- [ ] Automatic rollback wired: SLO breach within 10 min → `repository_dispatch` → rollback workflow re-deploys the previous image digest and, for IaC changes, re-applies the last known-good commit under the CI OIDC role
- [ ] Rollback drill covers **both** paths — image-digest rollback and IaC re-apply — and the runbook states which IaC changes are not revertible by re-applying the parent commit
- [ ] `deploy-and-rollback-bringup` skill committed under `.claude/skills/` and smoke-tested against staging. **It answers six of this block's nine substantive items** — the health checks, the smoke-suite traffic gate, the automatic rollback, both drill lines, and the deploy annotation. **It answers none of the other three, and two of them have no Phase 6 helper at all**: the deployment-strategy ADR is a recorded decision it refuses to make, and the Bits AI SRE / Sift post-deploy watch and the team-channel announcement are operator configuration a named human wires. A run reported as closing Gate 5 has claimed three items nobody did
- [ ] Manual rollback drilled and works within 5 minutes (quarterly cadence)
- [ ] Deploy posts annotation to Datadog / Grafana for incident correlation
- [ ] Bits AI SRE / Sift configured to watch first 30 min after deploy
- [ ] Deploy announced in team channel (Slack / Teams)

**Pass:** All items checked. Deploys are safe.

---

## Gate 6: Phase Handoff

Before handing off to Phase 7 — Delivery & Handoff.

- [ ] All Gate 1–5 items complete
- [ ] Production environment fully provisioned via Terraform (or OpenTofu) IaC
- [ ] Disaster recovery plan documented and tested
- [ ] Infrastructure runbook (`/docs/infrastructure.md`) covers: provision from scratch, make changes, restore state from a prior bucket version, clear a stuck lock safely, handle drift, rotate secret-manager secrets and the OIDC trust policy
- [ ] Deployment runbook (`/docs/deployment.md`) covers: standard deploy, hotfix, rollback, rollback drill
- [ ] Observability runbook exists for every alert
- [ ] Cost analysis complete and within budget
- [ ] All secrets resolved through the project's secret manager or CI OIDC; no hardcoded values anywhere
- [ ] Drift workflow's **last run succeeded** (not merely that the workflow file exists)
- [ ] AGENTS.md current and referenced by all AI tools
- [ ] Audit logs enabled and confirmed in **both** places that replace a vendor control plane's audit log — cloud-native audit on the state bucket (CloudTrail data events / Azure Monitor / GCP Cloud Audit Logs) and the GitHub audit log — plus Datadog / Grafana
- [ ] The two Phase 6 subagents are committed under `.claude/agents/` and **listed by `/agents`**; the six bring-up skills are committed as **folders** under `.claude/skills/` and each appears in the session's available-skills list under its slug. **`/agents` does not list skills** — checking only `/agents` leaves six skills unverified, and a skill whose folder name and `name:` field disagree loads silently never

**Pass:** All items checked. Phase 6 complete.

---

## Metrics to Track

| Metric | Target | Measurement |
|--------|--------|-------------|
| PR pipeline duration | < 15 min | CI analytics |
| Deployment frequency | Multiple per day or weekly per org maturity | Deploy event count |
| Lead time for changes | < 24h from commit to prod | Commit → deploy timestamps |
| Change failure rate | < 15% | Rollbacks / total deploys |
| Mean time to restore | < 1h (target driven by Bits AI SRE / Sift triage) | Incident resolution logs |
| Infrastructure cost vs budget | Within ±10% | Monthly cloud bill |
| Cost per active user / per request | Trend downward | Custom metric |
| Alert signal-to-noise | > 70% actionable | Post-incident review of fired alerts |
| AI-suggested-vs-actual root-cause variance | Track quarterly | Bits AI SRE / Sift retro |
| `terraform-iac-engineer` PR acceptance rate | > 60% accepted with ≤ 2 fixup commits | Subagent retro per cycle |
| Plan-to-apply divergence | 0 — every apply matches the plan a human reviewed | Count applies that re-planned, or whose actual changes differed from the reviewed plan |
| Anthropic API spend (Claude Code Action) | Within monthly cap | Anthropic console / cloud-provider billing |

These are the DORA Four plus AI-specific telemetry — track them all monthly.

---

## AI-Specific DevOps Standards

| Standard | Rationale |
|----------|-----------|
| **Review every AI-generated IaC line** | Cloud mistakes are expensive — wrong security group is a production outage |
| **The IaC agent never holds an apply credential** | The vendor-side "review-mode" org policy that used to enforce this is gone. `terraform-iac-engineer` can `init -backend=false` / `validate` / `plan` / `fmt` / `test` and write files; `apply` exists only as a CI job under an OIDC role no human and no agent can assume locally. Credential separation is a stronger control than a product setting, and unlike a product setting it is verifiable by trying to break it |
| **State is treated as a secret** | `terraform.tfstate` carries every resource ID and, for many providers, provider secrets in plaintext. Encrypted bucket, restrictive IAM, versioning on, never an artefact, never in git. No scanner in the prescribed stack inspects state |
| **AI-generated pipeline configs run in non-prod first** | Pipeline errors break deploys for the whole team |
| **AI-generated Dockerfiles are reviewed against Gate 3 before merge** | AI may suggest vulnerable, unpinned, or root-running base images — and no image scanner runs in the prescribed stack, so the digest-pin / minimal-base / non-root bar is enforced by review |
| **Never grant CI more cloud permissions than necessary** | AI may generate broad IAM policies — tighten to least-privilege; use OIDC |
| **Cost-estimate every IaC change** | Infracost `diff` on every PR prevents surprise bills — native Terraform / OpenTofu support, no vendor bridge to keep calibrated |
| **Bits AI SRE / Sift hypotheses are starting points, not diagnoses** | First responder still owns the incident and validates before action |
| **Track AI-suggested-vs-actual root-cause variance** | Confidence calibration improves over cycles; under-performing sources get demoted |
| **Audit log every AI-driven state change** | The forensic trail is now two surfaces, not one console: cloud-native audit on the state bucket (CloudTrail data events / Azure Monitor / GCP Cloud Audit Logs) plus the GitHub Actions run and audit log. Confirm **both** are on at Gate 1 — nothing consolidates them for you |
| **MCP scopes inherit from the connecting human** | Off-boarding is unchanged — revoke OAuth grants and the agent loses access. The Phase 6 MCP roster is GitHub + observability only; neither can mutate infrastructure |
