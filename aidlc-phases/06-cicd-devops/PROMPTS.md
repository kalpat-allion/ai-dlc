# Phase 6: CI/CD & DevOps — Prompt Templates

> **All prompts are for Claude Code, Cursor, GitHub Copilot, or Pulumi Neo / Copilot.** Pulumi (CLI), Docker, GitHub Actions, Pulumi Deployments, and observability backends run natively — the AI generates the configs and triages the runs.

The prompts are organised in the same order as the [PROCESS.md](./PROCESS.md) steps. Anchors below are referenced from PROCESS.md; keep the heading slugs stable.

---

## AGENTS.md Authoring

> Used in [PROCESS.md → Step 0.7](./PROCESS.md#07--author-an-agentsmd-at-the-repo-root). One-time, run from Claude Code at the repo root before any IaC or pipeline generation.

```
You are a platform-engineering lead authoring the canonical `AGENTS.md` for this repository. AGENTS.md is the project context file every AI coding agent (Pulumi Neo, Claude Code, Cursor, Copilot agent) reads as ground truth — be precise, terse, and project-specific. Do not pad with generic best-practice prose.

## Inputs
- **Project name and description:** [name + 1-line description]
- **Cloud provider(s) and regions:** [AWS us-east-1 / GCP europe-west1 / etc.]
- **Stack naming convention:** [e.g., `<project>-<env>-<resource>`, or paste an example]
- **Mandatory tags:** [Project, Environment, Owner, CostCenter — list the exact tag keys and the allowed values]
- **Forbidden resource types:** [e.g., no public S3 buckets, no IAM users (only roles), no `t1.micro`, etc.]
- **Allowed regions:** [list]
- **Compliance scope:** [GDPR / HIPAA / SOC 2 / PCI / etc., or "none"]
- **Production-deploy approval rule:** [who approves, via which mechanism — typically GitHub Environment with required reviewers]
- **Existing ADRs to reference:** [paths under `/docs/adr/` or equivalent]

## Output sections (Markdown, in this order)

### Project
One paragraph: what the project is, who owns it, the canonical Linear / Jira project key.

### Stack & naming conventions
The naming pattern (with one worked example), the stack-per-environment list, and how to derive resource names from the pattern.

### Mandatory tags
Table of tag key → allowed values → enforcement (CrossGuard policy pack name).

### Forbidden patterns
Bulleted list of forbidden resource types and configurations, each with a 1-line "why" and the CrossGuard rule that blocks it.

### Region restrictions
Allowed regions, with a 1-line rationale tied to compliance or latency.

### Compliance scope
Which frameworks apply; which controls are infra-relevant (encryption, audit logging, data residency); link to the ADR or compliance doc.

### Production-deploy approval
The approval rule in plain language; reference the GitHub Environment / Pulumi Cloud policy that enforces it.

### How to use this file
One paragraph telling AI agents (Pulumi Neo, Claude Code, Cursor): "Read this file before any change; if a conflict exists between this file and your generated output, this file wins; if this file does not cover a case, ask the human."

### Pointers
Links to: ADRs, infra runbook, deployment runbook, Phase 6 PROCESS.md.

Write in the imperative second person. Keep the file under 300 lines. Do not include implementation code — AGENTS.md is policy, not IaC. If any input is missing, ask before generating — fabricated conventions become binding once committed.
```

---

## Cloud Provider Comparison

> Used in [PROCESS.md → Step 1.1](./PROCESS.md#step-1-infrastructure-tech-stack-selection).

```
You are a senior cloud architect advising on provider selection. Be opinionated and honest about trade-offs — do not hedge by recommending whatever the team already uses.

## Project Context
- **Product:** [description]
- **Target users / regions:** [geography]
- **Expected scale at launch:** [users, data volume, RPS]
- **Expected scale at 12 months:** [projected]
- **Compliance requirements:** [GDPR / HIPAA / SOC 2 / FedRAMP / etc.]
- **Team expertise:** [existing AWS / GCP / Azure experience]
- **Budget at launch:** [monthly target]

## Requirements
- **Compute:** [API service / long-running / serverless / batch]
- **Database:** [relational / NoSQL / analytics / vector]
- **Storage:** [object / block / file]
- **Other:** [CDN / search / messaging / queue / AI inference]

Compare AWS, GCP, Azure, Cloudflare, Vercel (and Fly.io / Render / Railway for small projects) on:
1. **Feature fit** — Native services that match requirements
2. **Estimated cost at launch and 12 months** — Monthly spend with realistic usage; state the unit assumptions you used (RPS, GB stored, GB egress)
3. **Team readiness** — Learning curve given current expertise
4. **Ecosystem maturity** — Integrations, community, AI-tool support (Pulumi providers, Claude Code MCP, Docker compatibility)
5. **Compliance certifications** — Match against our requirements
6. **Lock-in risk** — How portable is the architecture if we want to move?
7. **Simplicity** — For this project size, is it overkill?

Output:
- A comparison table (rows = providers, columns = the seven dimensions, cells = brief verdict + 1-line rationale).
- A single recommendation with the top two reasons for and the top reason against.
- One sentence on the conditions under which you would change the recommendation.

If any of `Expected scale`, `Compliance`, or `Budget` is not given, ask before recommending — these flip the answer. Do not auto-pick AWS for safety — for many MVPs a managed PaaS (Vercel, Fly.io, Railway, Cloud Run) is simpler and cheaper.
```

---

## Cost Estimation

> Used in [PROCESS.md → Step 2.1](./PROCESS.md#step-2-cost-analysis--finops).

```
You are a FinOps engineer producing a defensible monthly cost estimate. The output drives a budget commitment — be explicit about assumptions and conservative when uncertain.

## Architecture
[Paste architecture overview from Phase 2]

## Expected Usage
- **Users:** [at launch] / [at 12 months]
- **Requests per second:** [p50] / [p99]
- **Data stored:** [GB total, growth rate]
- **Data transferred:** [GB/month egress]
- **Geographic distribution:** [regions]
- **AI workload (if any):** [tokens/month, model, batch vs realtime]

## Cloud Provider & Services
[List services: e.g., "AWS ECS Fargate + RDS Postgres + S3 + CloudFront + Bedrock"]

## Pricing source
[Cloud-provider price list URL, calculator export, or "use latest public list prices for <region>" — required so the estimate is reproducible]

Calculate monthly cost per service. For each line item state: unit price, units consumed per month, and the source you used for the price.
1. **Compute** — Instance types, count, reserved vs on-demand vs spot
2. **Database** — Instance type, storage, IOPS, backup, read replicas
3. **Storage** — Total GB + operations + tiering
4. **Networking** — Egress (often the surprise cost), inter-region, NAT gateway
5. **CDN / edge services**
6. **Observability** — Datadog or Grafana ingest (metrics + logs + traces volume)
7. **AI inference** — Per-token, per-request, dedicated capacity
8. **Other** — Remaining services

Provide three scenarios: Low (conservative) / Medium (expected) / High (peak). State the usage multipliers you used for Low and High vs Medium.

Highlight:
- Top 3 cost drivers (with $/month and % of total)
- Areas where costs scale non-linearly with usage
- Hidden costs often missed (egress, NAT gateway, cross-AZ traffic, observability ingest)
- Optimisation opportunities (reserved instances, Savings Plans, spot, caching, CDN)

If `Pricing source` is missing, or any usage input is missing for a service that materially drives cost (compute, egress, AI inference, observability ingest), ask before estimating. Do not invent prices — quote them from the stated source or refuse the line item and flag it as `pricing-unknown`.

## Worked example (illustrative, AWS us-east-1 list prices)

Inputs: 1M users, 200 RPS p50 / 800 RPS p99, 500 GB stored growing 100 GB/mo, 2 TB/mo egress, single region, no AI workload. Stack: ECS Fargate + RDS Postgres + S3 + CloudFront.

Output skeleton (numbers illustrative):

| Service | Unit price | Units / month | Cost / month | Source |
|---|---|---|---|---|
| Fargate vCPU | $0.04048 / vCPU-hr | 4 vCPU × 730 hr | $118 | AWS pricing page <date> |
| Fargate memory | $0.004445 / GB-hr | 8 GB × 730 hr | $26 | same |
| RDS db.t3.medium Multi-AZ | $0.136 / hr | 730 hr | $99 | same |
| RDS storage gp3 | $0.115 / GB-mo | 500 GB | $58 | same |
| S3 Standard | $0.023 / GB-mo | 500 GB | $12 | same |
| CloudFront egress | $0.085 / GB | 2,000 GB | $170 | same |
| NAT Gateway | $0.045 / hr + $0.045 / GB | 730 hr + 2,000 GB | $123 | same |
| **Medium total** | | | **~$606** | |

Scenarios: Low = 0.5× usage on egress + Fargate (~$430). High = 2× usage on egress + Fargate (~$960).

Top 3 drivers: CloudFront egress, NAT Gateway, Fargate compute. NAT Gateway is the classic surprise — flag explicitly.
```

---

## Pulumi Cost Delta

> Used in [PROCESS.md → Step 2.2](./PROCESS.md#step-2-cost-analysis--finops). Pulumi-on-PR cost-comment replacement for Infracost — Infracost does not yet support Pulumi natively. Runs as a Claude Code GitHub Action step on every IaC PR.

```
You are running as a Claude Code GitHub Action on a PR that modifies Pulumi IaC. Your output becomes a single PR comment that the reviewer relies on to approve or block the change.

## Inputs
- **Pulumi preview JSON output:** [`pulumi preview --json` output for the target stack]
- **Pulumi Insights resource catalogue:** [paste output of `pulumi insights search` for the org or fetch via the Pulumi MCP server's `resource-search` tool]
- **Cloud-provider price list:** [reference cloud-provider pricing API or static price snapshot in the repo]

## Tasks
1. Parse the Pulumi preview JSON: identify resources to create, replace, update, and delete.
2. For each create / replace, look up the unit cost from the price list.
3. For each update with cost-relevant property changes (instance type, storage size, IOPS, throughput), compute the cost delta.
4. Aggregate: total monthly $ delta = (created + replaced) - (deleted) + (updated deltas).
5. Flag anomalies: cost-impacting resources with no clear justification in the linked Linear issue, oversized instances vs the Phase-1 NFR target, missing tags from AGENTS.md.
6. Post a single PR comment with:
   - Total monthly delta in USD
   - Top 5 cost-driving changes (resource → delta)
   - Anomalies and questions for the reviewer

Format the comment as Markdown. Sign off as `*(via Claude Code + Pulumi Insights)*`.

If the price list does not cover a resource type that appears in the preview, list that resource under `Unpriced changes` with the resource address and ask the reviewer for an override price — do not guess.
```

---

## Cost Optimisation

> Used in [PROCESS.md → Step 2.3](./PROCESS.md#step-2-cost-analysis--finops).

```
You are a FinOps engineer reviewing a running infrastructure for savings. Recommend only opportunities you can justify against the supplied usage and constraint data — do not assume idle capacity that is not shown in the inputs.

Identify cost optimisation opportunities in our infrastructure.

## Current Infrastructure
[Paste Pulumi stack list + the top 50 resources by spend from cloud billing API or `pulumi insights search`]

## Current Monthly Spend
[From cloud billing]

## Usage Patterns
- **Peak hours:** [when]
- **Low hours:** [when]
- **Seasonal:** [yes/no]
- **Geographic distribution:** [regions]

## Constraints
- **Reliability requirements:** [99.9% / etc.]
- **Performance requirements:** [latency targets]
- **Compliance:** [data residency requirements]

Identify opportunities in these categories:
1. **Right-sizing** — Overprovisioned instances with < 30% utilisation
2. **Reserved / Savings Plans** — Steady-state workloads paying on-demand
3. **Spot / preemptible** — Workloads that tolerate interruption
4. **Storage tiering** — Cold data on expensive storage
5. **Egress reduction** — Cross-region / cross-AZ traffic, CDN opportunities
6. **Auto-scaling tuning** — Resources not scaling down at low-traffic hours
7. **Unused resources** — Orphaned disks, unattached IPs, idle load balancers
8. **Architectural** — Services that could use cheaper alternatives (Lambda vs EC2, S3 vs EBS, Cloud Run vs GKE)
9. **AI inference** — Batch vs realtime, prompt caching, smaller-model fallback

For each opportunity:
- **Estimated monthly savings**
- **Implementation effort** (low / medium / high)
- **Risk** (does this affect reliability/performance?)
- **Dependency** (what must be done first?)

Prioritise by savings/effort ratio. Output as a Pulumi-friendly Markdown list — each opportunity should be implementable as a Pulumi PR.
```

---

## Pulumi IaC Generation

> Used in [PROCESS.md → Step 3.2](./PROCESS.md#step-3-infrastructure-provisioning-iac-with-pulumi-ai). Run from Claude Code with the Pulumi MCP server connected.

```
You are a senior platform engineer authoring production-grade Pulumi IaC. Treat AGENTS.md as binding — every resource you generate must comply with its naming, tagging, and forbidden-resource rules.

Generate a Pulumi project in [TypeScript / Python / Go / .NET / Java].

## Cloud Provider & Region
[AWS / GCP / Azure / Cloudflare + region]

## Architecture
[Paste architecture overview from Phase 2]

## Resources Needed
[Bullet list — VPC, subnets, security groups, databases, services, queues, secrets, observability — with specific NFR-driven requirements]

## Project Conventions (from AGENTS.md)
- **Naming:** `<project>-<env>-<resource>` or [project-specific]
- **Mandatory tags:** Project, Environment, Owner, CostCenter
- **Forbidden resource types:** [e.g., no public S3 buckets, no IAM users — only roles]
- **Region restrictions:** [allowed regions]

## Output Requirements
1. **Component-resource structure** — `/components/vpc`, `/components/database`, `/components/service` (one component per reusable pattern)
2. **One stack per environment** — `dev`, `staging`, `prod`; environment-specific config in `Pulumi.<stack>.yaml`
3. **Pulumi ESC integration** — link `acme/<env>-ci` env via `pulumi config env add`; reference secrets via `pulumi.Config().requireSecret(...)`
4. **Tests** — generate Jest / pytest / Go test for each component
5. **Outputs** — important values exposed as stack outputs (URLs, ARNs, endpoints)
6. **Provider versions pinned** — `package.json` / `requirements.txt` / `go.mod` / `Pulumi.yaml`
7. **CrossGuard policy pack** — `/policy-packs/<project>` with required-tags, encryption-at-rest, no-public-S3, region-allowlist

## Pulumi best practices
- Use `ComponentResource` for reusable patterns; never copy-paste resources between stacks
- Validate inputs with TypeScript types / Python type hints / Go structs
- Avoid `Output<T>` lifting except where data flow requires it
- Comment non-obvious decisions inline

Run `pulumi preview` via the MCP server after generation to verify the stack resolves. If preview fails, fix and re-preview before reporting completion. Do not claim the stack is generated without an actual successful preview — paste the preview summary in your final response.

If `Architecture` or `AGENTS.md` content is not provided, ask before generating — fabricated conventions will violate org policy and waste a Neo / human review cycle.
```

---

## Infra Runbook

> Used in [PROCESS.md → Step 3.8](./PROCESS.md#step-3-infrastructure-provisioning-iac-with-pulumi-ai).

```
You are an SRE writing the canonical infrastructure runbook for this Pulumi project. The audience is the on-call engineer at 2am — every command must be copy-pasteable, every step must say what success looks like.

Generate `/docs/infrastructure.md` for this Pulumi project.

## Inputs
- **Stack list:** [output of `pulumi stack ls`]
- **AGENTS.md content**
- **ESC environments**
- **CrossGuard policy packs**

## Output sections (Markdown)

### Provision from Scratch
Numbered steps for: clone repo → install Pulumi + language toolchain → authenticate to cloud and Pulumi Cloud → `pulumi stack init <env>` → `pulumi config env add` → `pulumi up` for each environment in order. Include exact commands and expected duration.

### Make Infrastructure Changes
PR flow: branch → modify Pulumi code → `pulumi preview` locally → push → PR → Claude Code Action review + cost-delta + CrossGuard → human approval → merge → Pulumi Deployments runs `pulumi up` against the target stack.

### Recover from Corrupted State
Pulumi Cloud state recovery: `pulumi stack export` from last good revision → review diff → `pulumi stack import` → verify with `pulumi refresh --diff`.

### Handle Drift
Drift detection (Pulumi Deployments scheduled run) reports drift on prod → page on-call → SRE decides: import drift to code, refresh state, or `pulumi up` to remediate. Auto-remediation only on dev / ephemeral stacks.

### Rotate ESC Secrets
Pulumi ESC rotation flow: `pulumi env set <env> <key> --secret <new-value>` → ESC versions and immutably stores → CI picks up next run. Document upstream-store rotation for Vault / AWS Secrets Manager / Azure Key Vault.

### Disaster Recovery
Cross-region failover plan; RTO and RPO targets; backup-restore procedures; the quarterly DR drill date.

Write in second person ("you"). Include exact commands. Reference AGENTS.md for conventions.

If the stack list, ESC environments, or AGENTS.md content is not provided, ask before drafting — runbook fabrication leads to commands that fail when an engineer follows them under pressure.
```

---

## CI/CD Pipeline Generation

> Used in [PROCESS.md → Step 4.1](./PROCESS.md#step-4-cicd-pipeline-setup).

```
You are a CI/CD engineer generating a pipeline tailored to the project's actual stack. Include only the stages that apply — do not pad the workflow with stages that are not justified by the inputs (e.g., no Pulumi preview if the project has no IaC, no K8s steps if the deployment target is serverless).

## Platform
[GitHub Actions / GitLab CI]

## Project Context
- **Language / framework:** [stack]
- **IaC:** [Pulumi <language> / Terraform / OpenTofu / none]
- **Test framework:** [from Phase 4]
- **Security tools:** [list active tools from Phase 5 — typically SonarQube + Semgrep + Trivy + ggshield; add CrossGuard only if Pulumi]
- **Container registry:** [ECR / GHCR / Docker Hub / GAR / ACR / none if no containers]
- **Deployment target:** [ECS Fargate / Cloud Run / App Runner / K8s / Vercel / Fly.io / etc.]
- **Environments:** [dev / staging / prod — list only those that exist]
- **IaC runner (if applicable):** [Pulumi Deployments / self-hosted GitHub Actions runner]

## Pipeline Requirements
- **Branch strategy:** [trunk-based / git-flow]
- **Triggers:** [list — typically: PR, merge to main, scheduled drift check (IaC only), manual prod approval]
- **Required gates:** [list — e.g., tests pass + security pass + IaC preview clean + Claude Code Action review + 1 human approval]
- **Deployment strategy:** [blue/green / canary / rolling per ADR]

Generate workflow YAML covering the applicable subset of these stages, in this order. Skip a stage and state why if it does not apply:
1. **Lint & format** — Fast feedback (< 2 min)
2. **Build** — Compile + container image (skip image step if no containers)
3. **Unit + integration tests** with coverage gate
4. **Security scans** — match to the security tools listed above; CrossGuard only when IaC is Pulumi
5. **IaC preview** — `pulumi/actions@v6` for Pulumi, `infracost diff` + `terraform plan` for HCL, or omit if no IaC
6. **Cost delta comment** — Claude Code Action `pulumi-cost-delta` for Pulumi, `infracost comment` for HCL
7. **Deploy to staging** — IaC apply or platform deploy on merge to main
8. **E2E tests against staging** — from Phase 4
9. **Load test on RC** — only on tagged release candidate
10. **Deploy to production** — Manual approval gate via GitHub Environment, then deploy per the chosen strategy
11. **Post-deploy verification** — smoke tests + Datadog / Grafana deploy annotation

Include in every variant:
- **OIDC for cloud auth** — never long-lived keys
- **Secrets:** Pulumi ESC via `pulumi/esc-action@v1` if Pulumi is in use; otherwise platform-encrypted secrets with environment scoping
- **Caching** — `actions/cache` for deps, BuildKit cache mounts for Docker (when containers exist)
- **Parallel jobs** where dependencies allow
- **Required status checks** for branch protection
- **Notifications** to Slack / Teams on failure (success on prod-deploy only)
- **Claude Code Action gate** — `if: github.actor != 'dependabot[bot]'` to control API spend

If `Deployment target` or `IaC` is not specified, ask before generating — these flip half the pipeline.

Output the complete YAML file ready to commit, with comments on any stage where the included/skipped decision was non-obvious.
```

---

## Claude Code Action — PR Review Workflow

> Used in [PROCESS.md → Step 4.2](./PROCESS.md#step-4-cicd-pipeline-setup). Commit at `.github/workflows/claude.yml`.

```
You are a DevOps engineer wiring the official Anthropic Claude Code Action into this repo. Pin the version, scope permissions to the minimum, and gate against bot-driven API spend.

Generate `.github/workflows/claude.yml` using the official anthropics/claude-code-action@v1.

## Requirements
- **Auto-review on PR open:** trigger on `pull_request: [opened, synchronize, ready_for_review]`. Review the diff against AGENTS.md, the linked Linear issue's AC, ADRs, and relevant tests. Post severity-bucketed comments (Critical / High / Medium / Low). Skip drafts.
- **`@claude` mention fix-mode:** trigger on `issue_comment` and `pull_request_review_comment`. If the comment body contains `@claude`, run Claude Code with the comment as the prompt, scoped to the PR's branch. Open follow-up commits on the same branch.
- **Bot exclusion:** `if: github.actor != 'dependabot[bot]' && github.actor != 'renovate[bot]'`.
- **Concurrency:** cancel in-progress runs on the same PR when a new push arrives.
- **Auth:** `ANTHROPIC_API_KEY` repo secret (or AWS_BEDROCK_* / GCP_VERTEX_* / AZURE_FOUNDRY_* if cloud-routed).
- **Permissions:** `contents: write`, `pull-requests: write`, `issues: write` — minimum needed.
- **Timeout:** 30 min review, 60 min fix.
- **Spend cap:** environment variable `ANTHROPIC_MONTHLY_BUDGET_USD` checked at start; abort if over budget.

Output the complete YAML, with comments explaining each block.
```

---

## Agentic Workflow Templates

> Used in [PROCESS.md → Step 4.3](./PROCESS.md#step-4-cicd-pipeline-setup). Commit Markdown files at `.github/agentic-workflows/`. Each file declares trigger, agent engine (Claude Code), and safe outputs.

> **Reference outputs, not input prompts.** The four templates below are commit-ready starter workflows — copy them as-is into `.github/agentic-workflows/`, adjust the cron, the limits, and the project-specific phrasing, then commit. They are not meant to be pasted into a chat session.

### Triage stale issues

```
---
name: triage-stale-issues
on:
  schedule: { cron: "0 6 * * 1" }   # Mondays 06:00 UTC
agent: claude-code
safe-outputs: [add-comment, add-label, close-issue]
---
For every issue not updated in 60 days:
1. Read the issue body and last comment.
2. If the issue is still relevant: add a comment asking the author for an update.
3. If superseded by a closed PR or another issue: close with a reference comment.
4. If unclear: add label `needs-triage` and assign the team triage rotation.
Limit: 20 issues per run.
```

### Generate release notes

```
---
name: release-notes
on:
  push:
    tags: ["v*"]
agent: claude-code
safe-outputs: [create-pr]
---
Read the merged PRs since the previous tag. Group by Conventional Commit prefix (feat / fix / docs / chore). Open a PR that updates `CHANGELOG.md` with the grouped summary. Include Linear identifiers and PR links. Use the existing CHANGELOG style as the format reference.
```

### Repair flaky test

```
---
name: repair-flaky-test
on:
  schedule: { cron: "0 7 * * 2" }   # Tuesdays 07:00 UTC
agent: claude-code
safe-outputs: [create-pr]
---
1. Query CI for tests that failed at least 3 times and passed at least 3 times in the past 14 days (the flaky-test signature).
2. Pick the most-frequently-flaky test.
3. Read the test, the code under test, and the most recent failure logs.
4. Diagnose: race condition / timing / order-dependence / external-state leak.
5. Open a PR with the fix and a note explaining the diagnosis.
Limit: one PR per run.
```

### Fix lint debt

```
---
name: fix-lint-debt
on:
  schedule: { cron: "0 8 * * 5" }   # Fridays 08:00 UTC
agent: claude-code
safe-outputs: [create-pr]
---
1. Run the project linter and rank findings by `(severity × occurrences)`.
2. Take the top 1 rule with > 20 occurrences.
3. Open one PR that fixes that rule across the codebase.
4. Do NOT change behaviour — refactor only. PR title: `chore(lint): fix <rule-name>`.
Limit: one PR per run; reject if any source-test changes are required to keep tests passing.
```

---

## Dockerfile Generation

> Used in [PROCESS.md → Step 5.1](./PROCESS.md#step-5-containerization). Run via Docker Gordon (`docker ai`) for single services, or Claude Code for multi-service repos.

```
You are a container-platform engineer generating a production Dockerfile. Optimise for security and cache efficiency before image size; never use `:latest`; never run as root in the runtime stage.

Generate an optimised Dockerfile for this application.

## Application
- **Language / runtime:** [Node 22 / Python 3.13 / Go 1.23 / .NET 9 / Java 21]
- **Framework:** [Next.js / FastAPI / Spring Boot / etc.]
- **Build command:** [npm run build / etc.]
- **Start command:** [npm start / etc.]
- **Port:** [3000]

## Requirements
- **Security:** distroless or Alpine base, non-root user, READONLY_ROOTFS=true where possible
- **Size:** smallest practical image; pin base image by digest
- **Build cache:** structured for max layer reuse and BuildKit cache mounts
- **Health check:** Docker HEALTHCHECK directive

Generate a multi-stage Dockerfile with:
1. **Build stage** — Install deps, compile, run tests
2. **Runtime stage** — distroless / Alpine, only runtime deps, non-root, healthcheck
3. **Layer order** — dependency manifests first for cache efficiency
4. **Pinned versions** — base image by digest (use `docker buildx imagetools inspect` to get the digest)
5. **`.dockerignore`** — exclude `.git`, `node_modules`, tests, docs, secrets, IaC

Include comments explaining non-obvious decisions.
Output Dockerfile + .dockerignore, ready to commit.

If `Language / runtime` or `Start command` is missing, ask before generating — base image and entrypoint depend on them. Do not state a CVE count or scan result for the produced image — only Trivy / Docker Scout running in CI can do that.
```

---

## Kubernetes Manifests Generation

> Used in [PROCESS.md → Step 5.5](./PROCESS.md#step-5-containerization). Skip if not running on K8s.

```
You are a Kubernetes platform engineer producing manifests for a single service. Follow the supplied resource limits and replica counts exactly — do not invent capacity that was not specified.

Generate Kubernetes manifests for this service.

## Service
- **Name:** [service name]
- **Image:** [container image URL with digest]
- **Port:** [container port]
- **Resource requirements:** CPU / memory min and max
- **Replicas:** min / max for HPA
- **Persistent data:** [none / PVC / database connection]
- **External dependencies:** [databases, caches, APIs]

## Environment
- **Cluster:** [EKS / GKE / AKS / other]
- **Namespace:** [per-env or single]
- **Ingress controller:** [nginx / traefik / cloud-native]
- **Service mesh:** [none / Istio / Linkerd]
- **Secrets management:** External Secrets Operator pulling from Pulumi ESC

Generate:
1. **Deployment** — resource requests / limits, liveness / readiness / startup probes, graceful shutdown
2. **Service** — ClusterIP or LoadBalancer
3. **Ingress** — TLS termination, proper routing, AGENTS.md tag annotations
4. **ConfigMap** — non-sensitive config
5. **ExternalSecret** — pulls from Pulumi ESC via External Secrets Operator
6. **HorizontalPodAutoscaler** — CPU / memory / custom metric scaling
7. **PodDisruptionBudget** — for production resilience
8. **NetworkPolicy** — restrict traffic to necessary paths

Apply best practices:
- Non-root container user
- ReadOnlyRootFilesystem where possible
- Explicit resource limits (no noisy neighbour)
- Probes tuned for the app's startup characteristics
- Pod anti-affinity for HA across nodes / AZs
- All resources tagged per AGENTS.md

Output valid K8s YAML or Helm chart, depending on the project's overlay choice.
```

---

## Observability / Dashboard Generation

> Used in [PROCESS.md → Step 6.3](./PROCESS.md#step-6-observability--logging-monitoring-alerting).

```
You are an observability engineer authoring dashboards-as-code for one service. Use only the metrics listed under `Key metrics exposed` — do not invent metric names; if a panel needs a metric that is not exposed, list it under "Metrics to add" rather than guess the name.

Generate dashboards for this service.

## Service Context
- **Service name:** [name]
- **Backend:** Datadog OR Grafana
- **Language / runtime:** [stack]
- **Key metrics exposed:** [Prometheus metrics or OTLP attributes]
- **SLOs:**
  - Availability: [target %]
  - p95 latency: [target ms]
  - Error rate: [target %]
- **Business KPIs:** [signups/hour, active users, orders/minute]

Generate three dashboards (committed as code in `/observability/dashboards/`):

### Dashboard 1: Service Health
- Request rate by endpoint
- Error rate by endpoint and status code
- Latency p50 / p95 / p99
- Resource usage (CPU / memory / GC)
- Dependency health (database / cache / external APIs)
- Deploy markers from CI annotations

### Dashboard 2: SLO Tracking
- SLI current value vs SLO target (per SLO)
- Error budget remaining (1h / 6h / 24h / 7d windows)
- Burn-rate alert preview (fast burn / slow burn)

### Dashboard 3: Business KPIs
- Business metrics from instrumentation
- Funnel visualisations
- Trending indicators

Output: Datadog dashboard JSON (current Datadog API schema) or Grafana JSON (Grafana schema version 36+) — pick one based on the `Backend` input, do not produce both. Include a brief README.md per dashboard explaining the panels and how to extend them.

If `Backend`, `Key metrics exposed`, or any SLO target is missing, ask before generating.
```

---

## SLO and Alert Generation

> Used in [PROCESS.md → Step 6.4](./PROCESS.md#step-6-observability--logging-monitoring-alerting).

```
You are an SRE translating Phase-1 NFRs into SLOs and burn-rate alerts. One NFR maps to one SLO — do not collapse multiple NFRs, and do not invent SLOs that no NFR backs.

Generate SLOs and burn-rate alerts for this service.

## Inputs
- **NFRs from Phase 1:** [paste latency / availability / error-rate targets]
- **Service architecture:** [services + dependencies]
- **Backend:** Datadog OR Grafana / Prometheus AlertManager

For each NFR, generate:
1. **SLO definition** — SLI metric, target, rolling window (28d typical)
2. **Burn-rate alerts** — fast burn (2% budget in 1h → page) and slow burn (10% in 6h → ticket)
3. **Alert routing** — PagerDuty / Opsgenie / incident.io integration
4. **Runbook reference** — `/docs/runbooks/<alert-name>.md`

Output formats:
- Datadog: SLO + Monitor JSON
- Grafana: Prometheus AlertManager rules YAML + Grafana SLO config

Include error-budget burndown chart definitions for the SLO Tracking dashboard. Suggest an initial alert sensitivity that the team can tune in the first 4 weeks.

If `NFRs from Phase 1` is missing or contains no measurable target (latency / availability / error rate), ask before generating — vague NFRs produce noisy alerts.
```

---

## Runbook Generation from Alert

> Used in [PROCESS.md → Step 6.6](./PROCESS.md#step-6-observability--logging-monitoring-alerting).

```
You are an SRE drafting a starter runbook for a single alert. The output is a draft for human review — do not invent dashboard URLs, log queries, or remediation commands that you cannot ground in the supplied system context. Mark unverified items as `<TO VERIFY: ...>`.

Generate an incident response runbook for this alert.

## Alert
- **Name:** [alert name]
- **What triggers it:** [condition]
- **Severity:** [P0 / P1 / P2]
- **Service affected:** [service name]
- **Typical business impact:** [user-facing / internal]

## System Context
- **Architecture:** [brief summary]
- **Dependencies:** [databases, external services]
- **Recent incidents:** [related history if any]

Generate a runbook with:

### Title
[Alert name] — Response Runbook

### Summary
What this alert means in plain language + typical business impact.

### Initial Assessment (first 5 minutes)
1. Verify the alert is real (not a monitoring glitch)
2. Check Bits AI SRE / Sift hypothesis if available — accept it as a starting point, not a diagnosis
3. Check blast radius — all users or subset?
4. Check recent deploys — was this triggered by a deploy? (Deploy markers on Datadog / Grafana)

### Diagnosis Steps
Numbered steps with:
- Specific commands to run (with examples)
- Dashboards to check (with deep-links)
- Logs to query (with Loki / Datadog query syntax)
- Likely root causes ranked by probability

### Remediation
For each likely root cause:
- Immediate mitigation (stop the bleeding — feature-flag off, traffic shift, scale up)
- Full fix (address root cause)
- Verification (how to confirm fix worked — SLI returns to within target)

### Escalation
- When to escalate (if X doesn't work in Y minutes)
- Who to escalate to (on-call rotation + service owner)

### Post-Incident
- Post-mortem template reference
- Where to log learnings (`/docs/post-mortems/`)
- Bits AI SRE / Sift retro: was the AI hypothesis correct? track variance over time

Output as Markdown ready to commit at `/docs/runbooks/<alert-name>.md`.
```

---

## Self-Review Before Production Deploy

> Used in [PROCESS.md → Step 7.2](./PROCESS.md#step-7-deployment-automation--rollback) — run from the release captain's local Claude Code session before clicking the production-deploy approval.

```
Run a pre-prod-deploy self-review against this release candidate.

## Inputs
- **Release tag:** [v1.2.3]
- **Diff vs last prod release:** [git diff or PR list]
- **Pulumi preview against prod stack:** [paste output]
- **Linear issues closed in this RC:** [list]
- **Open SLO burn budgets:** [Datadog / Grafana export]

Checks:
1. **High-risk changes flagged** — auth, schema migrations, IAM, networking. Require dedicated review note for each.
2. **Pulumi preview is reviewed line-by-line** — flag any resource replacement (vs update), any DELETE, any IAM change.
3. **CrossGuard policy results clean** — zero violations, zero advisory warnings without resolution comment.
4. **Cost delta within tolerance** — ± 10% of expected baseline.
5. **Open incidents** — none on related services in past 24h.
6. **Rollback drill freshness** — last drill was within 90 days.
7. **Deploy window** — not Friday afternoon, not during a known traffic peak.
8. **Feature-flag posture** — risky changes behind flags, not deployed open by default.

Output a Markdown report: pass / fail per check, with severity. If any fail at High or Critical, recommend halting the deploy.
```
