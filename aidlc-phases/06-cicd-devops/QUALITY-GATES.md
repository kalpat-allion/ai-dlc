# Phase 6: CI/CD & DevOps — Quality Gates

The five gates below align with the seven [PROCESS.md](./PROCESS.md) steps. Steps 1–2 fold into Gate 1 (foundation); Steps 3–7 each have their own gate. Pass criteria are absolute — items unchecked block progression.

---

## Gate 1: Infrastructure Foundation

Before any workload is deployed to cloud resources.

### Tech stack & cost (Steps 1–2)
- [ ] Cloud provider, IaC tool, language, orchestration, observability decisions documented as ADRs
- [ ] Pulumi (or fallback OpenTofu) chosen with explicit rationale; HCL-only requirement documented if not Pulumi
- [ ] Initial cost estimate generated and within budget (low / expected / peak scenarios)
- [ ] Cost-on-PR active: **Pulumi Insights + `pulumi-cost-delta`** Claude Code action for Pulumi stacks **OR** Infracost AutoFix for Terraform/OpenTofu
- [ ] Cloud-native budget alerts set at 50% / 80% / 100%
- [ ] Unit-economics metric (cost per active user / cost per request) wired to a dashboard

### IaC repo and state (Step 3)
- [ ] Pulumi project bootstrapped with TypeScript / Python / Go / .NET / Java
- [ ] One stack per environment (`dev`, `staging`, `prod`); component resources for reusable patterns
- [ ] `AGENTS.md` committed at repo root and referenced by Pulumi Neo / Claude Code / Cursor
- [ ] Pulumi Cloud (or self-managed S3 / Azure Blob / GCS) configured for remote state
- [ ] State file is **never** committed to git
- [ ] Pulumi ESC environments created per stack; secrets pull from upstream stores (Vault / Secrets Manager / Key Vault / 1Password)
- [ ] CrossGuard policy packs cover: required tags, encryption-at-rest, no-public-S3, IAM least-privilege, region restrictions, instance-size caps
- [ ] Pulumi Cloud audit log enabled
- [ ] Pulumi Neo (if licensed) configured to **review-mode** for prod stacks
- [ ] `pulumi up` runs only via Pulumi Deployments or CI with OIDC — no laptop applies to staging or prod

**Pass:** All items checked. IaC baseline established.

---

## Gate 2: CI/CD Pipeline Operational

Before the team relies on automation for deploys.

### Pipeline coverage
- [ ] Lint + format runs on every PR (< 2 min)
- [ ] Build runs on every PR
- [ ] Unit + integration tests run on every PR with coverage gate (≥ 80% on new code)
- [ ] Security scans on every PR (SonarQube + Semgrep + Trivy + ggshield + CrossGuard)
- [ ] `pulumi preview` runs on every IaC PR with cost-delta + policy-result comment
- [ ] E2E tests run against staging on merge to main
- [ ] Auto-deploy to staging on merge (no manual steps)
- [ ] Manual approval gate on production deploys (GitHub Environment with required reviewers)

### AI integration in CI
- [ ] Anthropic Claude Code Action committed at `.github/workflows/claude.yml`, version pinned to `@v1`
- [ ] Auto-review on PR open posts within ~3 min, severity-bucketed (Critical / High / Medium / Low)
- [ ] `@claude` mention triggers fix-on-mention workflow that opens a follow-up commit
- [ ] At least one Copilot Agentic Workflow committed and dry-run passing (stale-issue triage / release-notes / flaky-test repair / lint-debt fix)
- [ ] GitLab Duo Root Cause Analysis enabled (if on GitLab fallback)

### Pipeline quality
- [ ] Total PR pipeline duration < 15 min
- [ ] Caching configured (deps + Docker layers + BuildKit cache mounts)
- [ ] Parallel jobs where dependencies allow
- [ ] All secrets resolved via Pulumi ESC; no raw credentials in workflow YAML
- [ ] Cloud auth uses OIDC, not long-lived keys
- [ ] Claude Code Action gated against bot-PR token burn (`if: github.actor != 'dependabot[bot]'`)
- [ ] Anthropic API monthly spend cap configured (or routed to Bedrock / Vertex / Foundry per existing cloud commits)

### Branch protection
- [ ] `main` requires PR + all checks passing + Claude Code Action review acknowledged + 1 human approval
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
- [ ] **Trivy** scan in CI passes — 0 Critical CVEs in the image
- [ ] **Docker Scout** AI-suggested base-image upgrades reviewed (and merged or explicitly declined)
- [ ] Image size minimised (compare against previous release; flag > 20% growth)
- [ ] Docker Gordon `docker ai "rate my Dockerfile"` reviewed for each service

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
- [ ] Sentry + Seer wired for application errors; Sentry MCP server connected to Claude Code
- [ ] Datadog / Grafana / Sentry MCP servers connected for incident workflows

### Dashboards (versioned as code)
- [ ] Service Health dashboard (latency, errors, traffic, resource usage, dependency health)
- [ ] SLO Tracking dashboard (SLI vs target + error budget remaining)
- [ ] Business KPI dashboard (signups / orders / active users — product-relevant)
- [ ] Dashboards committed at `/observability/dashboards/`

### Alerts
- [ ] One SLO per Phase-1 NFR (latency, availability, error rate)
- [ ] Burn-rate alerts: fast burn (2% / 1h → page) + slow burn (10% / 6h → ticket)
- [ ] Alert routing configured (PagerDuty / Opsgenie / incident.io)
- [ ] Every alert links to a runbook in `/docs/runbooks/`
- [ ] Alerts tested — verified fire + verified resolve
- [ ] Bits AI SRE / Sift configured to triage alerts before on-call paging
- [ ] No alerts that don't require human action (no noise)

**Pass:** All items checked. Observability production-ready.

---

## Gate 5: Deployment & Rollback

Every production deploy must satisfy.

- [ ] Deployment strategy ADR (blue/green / canary / rolling) committed
- [ ] Health checks (`/healthz` + `/readyz`) block traffic during deploy
- [ ] Phase-4 Playwright smoke suite runs post-deploy before traffic fully routes
- [ ] Automatic rollback wired: SLO breach within 10 min → `repository_dispatch` → `pulumi/actions@v6` rollback
- [ ] Manual rollback drilled and works within 5 minutes (quarterly cadence)
- [ ] Deploy posts annotation to Datadog / Grafana for incident correlation
- [ ] Bits AI SRE / Sift configured to watch first 30 min after deploy
- [ ] Deploy announced in team channel (Slack / Teams)

**Pass:** All items checked. Deploys are safe.

---

## Gate 6: Phase Handoff

Before handing off to Phase 7 — Delivery & Handoff.

- [ ] All Gate 1–5 items complete
- [ ] Production environment fully provisioned via Pulumi IaC
- [ ] Disaster recovery plan documented and tested
- [ ] Infrastructure runbook (`/docs/infrastructure.md`) covers: provision from scratch, make changes, recover state, rotate ESC secrets, handle drift
- [ ] Deployment runbook (`/docs/deployment.md`) covers: standard deploy, hotfix, rollback, rollback drill
- [ ] Observability runbook exists for every alert
- [ ] Cost analysis complete and within budget
- [ ] All secrets resolved through Pulumi ESC; no hardcoded values anywhere
- [ ] AGENTS.md current and referenced by all AI tools
- [ ] Audit logs enabled in Pulumi Cloud, GitHub, Datadog / Grafana, Sentry

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
| Pulumi Neo PR acceptance rate (if licensed) | > 60% accepted with ≤ 2 fixup commits | Neo retro per cycle |
| Anthropic API spend (Claude Code Action) | Within monthly cap | Anthropic console / cloud-provider billing |

These are the DORA Four plus AI-specific telemetry — track them all monthly.

---

## AI-Specific DevOps Standards

| Standard | Rationale |
|----------|-----------|
| **Review every AI-generated IaC line** | Cloud mistakes are expensive — wrong security group is a production outage |
| **Pulumi Neo runs in review-mode for prod stacks** | Autonomous-mode `pulumi up` against prod must require human approval |
| **AI-generated pipeline configs run in non-prod first** | Pipeline errors break deploys for the whole team |
| **AI-generated Dockerfiles are Trivy + Docker Scout scanned before merge** | AI may suggest vulnerable base images |
| **Never grant CI more cloud permissions than necessary** | AI may generate broad IAM policies — tighten to least-privilege; use OIDC |
| **Cost-estimate every IaC change** | Pulumi Insights / Infracost on PR prevents surprise bills |
| **Bits AI SRE / Sift hypotheses are starting points, not diagnoses** | First responder still owns the incident and validates before action |
| **Track AI-suggested-vs-actual root-cause variance** | Confidence calibration improves over cycles; under-performing sources get demoted |
| **Audit log every AI-driven state change** | Pulumi Cloud + GitHub audit logs are the forensic trail when AI acts unexpectedly |
| **MCP scopes inherit from the connecting human** | Off-boarding is unchanged — revoke OAuth grants and the agent loses access |
