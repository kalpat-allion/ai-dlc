# Phase 6: CI/CD & DevOps — Process Definition

## Overview

This document defines the AI-assisted workflow for the CI/CD & DevOps phase of the AI-DLC. It is the phase where infrastructure decisions become running production systems and where the productivity gains from AI tooling are largest — modern AI agents now author IaC, generate pipelines, scan and fix Dockerfiles, draft dashboards, and triage incidents end-to-end. The process below is built around **Terraform (with OpenTofu as the CLI-compatible fallback)** as the IaC engine — authored by the **`terraform-iac-engineer` subagent** and brought up by the **`iac-state-backend-bringup`** and **`ci-identity-and-secrets-bringup`** skills — with **GitHub Actions + Copilot agentic workflows + the official Anthropic Claude Code Action** as the CI/CD spine, **Docker Gordon + Claude Code** for containerization, **Datadog Bits AI SRE** (or **Grafana Assistant**) for observability, and **Infracost** for cost.

**Phase Duration:** Initial setup 1–2 weeks + continuous refinement alongside Development
**Phase Owner:** DevOps Engineer / Platform Engineering Lead
**Tools Used:**

- **IaC:** Terraform (primary) or OpenTofu (CLI-compatible fallback) — HCL, remote state in the project's own cloud account
- **CI/CD platform:** GitHub Actions + GitHub Copilot Agentic Workflows + Anthropic Claude Code Action (or GitLab CI + GitLab Duo as fallback)
- **IaC apply runner:** GitHub Actions with cloud OIDC (primary — $0 control plane) or Spacelift Intelligence / HCP Terraform (fallback, only when a managed runner is a hard requirement)
- **Containerization:** Docker Desktop with Gordon AI + Claude Code
- **Cost:** Infracost CLI / Cloud (native Terraform + OpenTofu support) + cloud-native calculators; CAST AI / Kubecost for K8s
- **Observability:** Datadog + Bits AI SRE (primary commercial) **or** Grafana Cloud + Grafana Assistant + Sift (primary OSS-friendly)
- **AI authoring:** Claude Code (terminal agent + IDE extension) — connected to GitHub MCP and the observability MCP server; the Phase 6 subagents (`terraform-iac-engineer`, `container-image-engineer`) and the six bring-up skills committed in-repo

> **Tool Philosophy.** DevOps is now AI-native. We treat the `terraform-iac-engineer` subagent as a junior platform engineer that reads the architecture, writes the HCL, runs `terraform plan`, and stops — it holds no apply credential, so a human reviewing the plan is the only path to production. We treat Bits AI SRE / Grafana Sift as the first responder on every alert. Claude Code remains the orchestrator that ties the GitHub MCP and observability MCP servers, the six bring-up skills, and the two subagents into one session — same model the team already uses in Phase 3. Four tool families, zero overlap, every state-changing action gated behind a human approval **and** a credential the agent does not have.

---

## Tool Stack

| Layer | Primary | Fallback | Cost |
|-------|---------|----------|------|
| **IaC engine** | Terraform (BSL 1.1) | OpenTofu (MPL 2.0) — CLI-compatible, same HCL, same state format; the fallback when the BSL is unacceptable | Free CLI (both) |
| **AI IaC authoring** | `terraform-iac-engineer` subagent in Claude Code (`validate` + `plan` loop, no apply credential) | Claude Code with the [`terraform-iac-generation`](./PROMPTS.md#terraform-iac-generation) prompt directly | Foundation (in Max) |
| **IaC state + locking** | Cloud-native remote backend in the project's own account: S3 + versioning + SSE-KMS + DynamoDB lock table (or S3 native lockfile, TF ≥ 1.10) · GCS + CMEK + object-generation locking · Azure Blob + lease | Spacelift Intelligence or HCP Terraform — only when a managed runner / control plane is a hard requirement | **$0 control plane** (pennies of storage) |
| **IaC apply runner** | GitHub Actions with cloud OIDC, behind a GitHub Environment approval | Spacelift Intelligence / HCP Terraform | Free tier |
| **IaC RBAC + audit** | Cloud IAM policy on the state bucket (separate read vs write roles; CI writes via OIDC, humans read-only) + CloudTrail data events / GCP Cloud Audit Logs / Azure Monitor + the GitHub audit log | Managed-runner RBAC (Spacelift / HCP) | Free (cloud audit ingest may bill) |
| **Secrets in CI/CD** | Cloud OIDC for cloud auth (no stored credential) + the project's secret manager for everything else (AWS Secrets Manager / Azure Key Vault / GCP Secret Manager / Vault / Doppler / 1Password) | Platform-native GitHub/GitLab encrypted secrets with environment scoping | Cloud-native pricing; OIDC free |
| **CI/CD platform** | GitHub Actions + Copilot Agentic Workflows | GitLab CI + GitLab Duo | GH Free tier + Copilot Pro $10/user/mo + Copilot CI seats $4/seat |
| **AI in CI** | Anthropic `anthropics/claude-code-action@v1` (PR review, fix-on-mention, automation) | GitLab Duo Root Cause Analysis | API usage-based (Anthropic / Bedrock / Vertex / Foundry) |
| **Containerization** | Docker Desktop 4.61+ with Gordon AI + the `container-image-engineer` subagent in Claude Code | GitHub Copilot inline + K8sGPT (in-cluster operator + MCP) | Docker Personal free; Pro $9/user/mo |
| **Cost analysis** | Infracost CLI / Cloud + AutoFix — native Terraform and OpenTofu support, no vendor bridge | Cloud-native calculators + billing-API export | Infracost free OSS; Cloud tiers priced per org |
| **K8s cost (if used)** | CAST AI (autonomous bin-packing, savings-based) | Kubecost (OSS) | CAST AI savings-based; Kubecost OSS free |
| **Observability (commercial)** | Datadog + Bits AI SRE (autonomous alert triage, GA Dec 2025, 2× faster Q1 2026) | New Relic AI | Datadog from $15/host + usage |
| **Observability (OSS-leaning)** | Grafana Cloud + Grafana Assistant (free tier, April 2026) + Sift (Asserts.ai-powered RCA) | Self-hosted Prometheus + Loki + Tempo | Grafana Cloud Free tier; Pro $8/user/mo |
| **Incident management + AI** | incident.io (AI summary + PagerDuty-compatible) | PagerDuty AIOps | from $20/responder/mo |

**Optional upgrades:**
- **Datadog Bits AI for Dev / Security** — for autonomous triage on application performance and AppSec, on top of Bits AI SRE.
- **A managed IaC control plane (Spacelift Intelligence or HCP Terraform)** — only when a managed apply runner, a vendor-side policy engine, or a single consolidated IaC audit surface is a hard organisational requirement. The prescribed cloud-native path costs $0 and passes every gate in this phase; buy a control plane to remove ops burden, not to reach compliance.

---

## Process Steps

### Step 0: One-Time Setup — Wire AI Tools into the DevOps Loop

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | Cloud account with permission to create a state bucket and IAM roles, GitHub org with Copilot Pro / Copilot Business licences, Anthropic API key (or Bedrock / Vertex / Foundry credentials), Datadog OR Grafana Cloud workspace, Docker Desktop 4.61+, Claude Code installed, Terraform (or OpenTofu) CLI installed |
| **Tools** | **Anthropic Claude Code Action**, **Copilot Agentic Workflows**, **Datadog/Grafana MCP**, **Docker Gordon**, the Phase 6 **subagent + skill templates** |
| **Output** | `@claude` fix-on-mention live on the repo, the project's chosen AI PR reviewer wired (Step 4.2), observability MCP connected for incident workflows, `AGENTS.md` committed, and the two Phase 6 subagents + six bring-up skills installed and discoverable for every developer |
| **Human** | DevOps Lead authorises the org-level MCP servers and commits `AGENTS.md`; each developer authenticates once per project via OAuth |

This step is done **once per project**. The cost of skipping it is paying the "AI tax" by hand on every story for the rest of the phase.

> **There is no IaC MCP server in this phase.** The Terraform/OpenTofu loop runs through the CLI in the Claude Code shell (`validate`, `plan`, `output`, `state list`) plus the `terraform-iac-engineer` subagent. The MCP roster for Phase 6 is GitHub + observability only. What is genuinely lost with it is org-wide resource search across every environment — see the risk register and Step 2.7.

#### 0.1 — Install the Anthropic Claude Code Action in the repo

The official action (`anthropics/claude-code-action@v1`, GA in 2026) runs the Claude Code runtime inside a GitHub Actions runner. It auto-detects mode based on workflow context — `@claude` mention in a PR comment triggers fix-mode; `pull_request: opened` trigger runs review-mode; explicit prompts run automation.

```bash
# From any developer's terminal, in the repo root
claude  # opens Claude Code
> /install-github-app
```

The slash command provisions: a fine-grained PAT or GitHub App, the `ANTHROPIC_API_KEY` repository secret (or `CLAUDE_CODE_OAUTH_TOKEN`, or `AWS_BEDROCK_*` / `GCP_VERTEX_*` / `AZURE_FOUNDRY_*` if using a cloud provider's Claude endpoint), and a starter `.github/workflows/claude.yml`. Commit the workflow; the team gets `@claude` review on every PR going forward.

> **`@claude` fix-on-mention and AI PR review are separate decisions.** Fix-on-mention is what this Step 0.1 setup buys you and is worth having on any repo. Automated PR *review* is a per-project choice between CodeRabbit and the Claude PR review bot — see [Phase 3 → Step 4.3](../03-development/PROCESS.md#step-4-code-review) — and if the project picks the Claude bot, the starter workflow's review-mode is a **bootstrap, not the production reviewer**: it has no size gate, no re-review idempotency, and no fail-open handling, the three things that decide whether it stays useful past week two. Build it from [PR-REVIEW-BOT.md](../03-development/PR-REVIEW-BOT.md) instead and keep `claude.yml` for fix-on-mention only (Step 4.2).

#### 0.2 — Configure GitHub Copilot Agentic Workflows for repository automation

GitHub Agentic Workflows (technical preview, 2026) are Markdown-authored automations that run as standard GitHub Actions with sandboxed permissions and explicit safe-output approvals (create-PR, add-comment). They can use **Copilot CLI, Claude Code, or OpenAI Codex** as the agent engine — pick Claude Code to keep the Phase 3 / Phase 6 toolchain consistent.

Author the agentic workflows under `.github/agentic-workflows/` (Markdown) and pin the agent runtime in `.github/workflows/copilot-setup-steps.yml`. See [PROMPTS.md → agentic-workflow-templates](./PROMPTS.md#agentic-workflow-templates) for the starter set the team should commit (triage stale issues, generate release notes, repair flaky test, fix lint debt).

#### 0.3 — Connect the observability MCP server

```bash
# Grafana Cloud (Grafana Assistant + Sift via MCP)
claude mcp add --transport http --scope project grafana https://<workspace>.grafana.net/api/mcp

# Datadog (via Datadog MCP)
claude mcp add --transport http --scope project datadog https://mcp.datadoghq.com/mcp
```

Add whichever matches the Step 1.4 backend decision — not both. Smoke test: `Via the observability MCP, list the services emitting the most 5xx responses in the last 24h, then pull the traces behind the worst one.`

#### 0.4 — Enable Docker Gordon in developer environments

Docker Desktop 4.61+ ships **Gordon** (the AI agent formerly known as Ask Gordon). Gordon runs as a CLI bundled with Docker Desktop and a chat panel in the docs site. Enable it under **Docker Desktop → Settings → Beta features → Docker AI**. Validate with `docker ai "rate my Dockerfile"` from a project root — Gordon should propose multi-stage refactors, base-image hardening, and a `.dockerignore`.

#### 0.5 — Author an `AGENTS.md` at the repo root

The Phase 6 subagents and skills (and most modern AI coding agents — Claude Code, Copilot agent) read [AGENTS.md](https://agents.md) as the canonical project context file. Capture: resource naming convention, region defaults, mandatory tags (`Project`, `Environment`, `Owner`, `CostCenter`), forbidden resource types, ADR pointers, and the production-deploy approval requirement. Without this, every prompt drifts; with it, the agent fences itself within team standards automatically. Use the [`agents-md-authoring`](./PROMPTS.md#agentsmd-authoring) prompt for the first draft, then have the platform lead review and commit.

> **The mandatory tag set is now a load-bearing control, not a convention.** With no cross-stack resource-search tool in the prescribed stack (Step 2.7), cloud-native inventory queries are the only way to answer "show me every database tagged `Environment=prod`" — and an untagged resource is invisible to them. Treat a missing mandatory tag as a defect, not a style nit.

#### 0.6 — Install the Phase 6 subagents and skills

Phase 6 ships two subagents and six bring-up skills as templates. Copy them into the consuming repo, fill the placeholders, and commit — they are project artefacts, not personal config, and they transfer with the repo at Phase 7. This sub-step is last because both helpers read `AGENTS.md`, so it must exist first.

```bash
# From the consuming repo root
mkdir -p .claude/agents .claude/skills
cp    <ai-dlc>/aidlc-phases/06-cicd-devops/subagent-prompts/terraform-iac-engineer.md   .claude/agents/
cp    <ai-dlc>/aidlc-phases/06-cicd-devops/subagent-prompts/container-image-engineer.md .claude/agents/
cp -r <ai-dlc>/aidlc-phases/06-cicd-devops/skill-prompts/*/                             .claude/skills/
```

Skills are **folders**, not files — `.claude/skills/<skill-name>/SKILL.md`, where the folder name must equal the `name:` field. A mismatch means the skill silently never loads, so copy the directories with `cp -r` rather than globbing the Markdown.

| Helper | Type | Owns | Gate |
|--------|------|------|------|
| `terraform-iac-engineer` | Subagent | Step 3.2 — authors the HCL, loops `validate` + `plan` until clean | Gate 1 |
| `container-image-engineer` | Subagent | Step 5.1 — multi-stage Dockerfile + `.dockerignore` | Gate 3 |
| `iac-state-backend-bringup` | Skill | Steps 3.1, 3.4 — remote backend, versioning, encryption, locking, split state IAM | Gate 1 |
| `ci-identity-and-secrets-bringup` | Skill | Steps 3.5, 3.6 — OIDC trust policy, secret manager, drift workflow | Gate 1 |
| `cost-guardrails-bringup` | Skill | Step 2 — Infracost on PR, budget alerts, unit-economics metric | Gate 1 (cost items) |
| `cicd-pipeline-bringup` | Skill | Step 4 — pipeline stages, `@claude` wiring, branch protection | Gate 2 |
| `observability-bringup` | Skill | Step 6 — dashboards → SLOs → burn-rate alerts → runbook per alert | Gate 4 |
| `deploy-and-rollback-bringup` | Skill | Step 7 — staging auto-deploy, prod approval gate, auto-rollback, the drill | Gate 5 |

Fill every placeholder before committing — an unfilled `{{CLOUD_PROVIDER}}` or `{{TF_CLI}}` makes the agent guess, and guessing about infrastructure is the one mistake in this phase with no cheap undo. Verify subagents with `/agents`; verify skills by confirming each appears in the available-skills list under its slug. Smoke-test one of each: `Use the terraform-iac-engineer to run terraform validate and summarise the current plan — do not apply.`

> **The `terraform-iac-engineer` subagent must not be granted an apply-capable credential.** Its scope covers `init -backend=false`, `validate`, `plan`, `fmt`, `test` and file writes. `apply` exists only as a CI job under an OIDC role that no human and no agent can assume locally. **This credential separation is what replaces the vendor-side "review-mode" org policy the previous IaC stack provided** — and unlike a product setting, it is verifiable by trying to break it.

#### Verification checklist

- [ ] `claude mcp list` shows `grafana` (or `datadog`) and `github` connected for every developer
- [ ] `.github/workflows/claude.yml` committed and `@claude` test mention on a draft PR returns a review within ~3 minutes
- [ ] `.github/agentic-workflows/` contains at least one starter workflow that runs successfully in dry-run
- [ ] `docker ai "rate my Dockerfile"` runs and returns recommendations
- [ ] `AGENTS.md` committed at repo root, with mandatory tags, forbidden resource types, region defaults and the production-approval requirement filled in
- [ ] `/agents` lists `terraform-iac-engineer` and `container-image-engineer`; all six bring-up skills present under `.claude/skills/` as folders; every placeholder filled; one subagent and one skill smoke-tested

> **Permission inheritance.** Every MCP-driven action runs as the connecting developer — the MCP server cannot escalate beyond their GitHub / Datadog scopes. Neither server in the Phase 6 roster can mutate infrastructure; cloud write access lives exclusively in the CI OIDC role. Off-board by revoking the OAuth grant in each provider; the agent loses access automatically.

---

### Step 1: Infrastructure Tech Stack Selection

> Visual: [Step 1 flowchart](./FLOWCHART.md#step-1-tech-stack)

| Attribute | Detail |
|-----------|--------|
| **Input** | Architecture proposal (Phase 2), NFRs, budget, team expertise, existing cloud accounts, compliance scope |
| **Tools** | **Claude Code** (trade-off analysis + ADR drafting), cloud-provider pricing calculators |
| **Output** | Tech stack decisions documented as ADRs, plus the Terraform root-module skeleton and `required_providers` pinning committed to the repo |
| **Human** | Final calls on cloud provider, IaC language, runtime, observability stack |

**Workflow:**

**1.1 — Cloud provider trade-off.** From Claude Code, run the [`cloud-provider-comparison`](./PROMPTS.md#cloud-provider-comparison) prompt. Feed: NFRs, expected scale (launch + 12 months), regions, compliance scope, team's existing AWS/GCP/Azure footprint. Compare AWS / GCP / Azure / Cloudflare / Vercel / Fly.io across feature fit, 12-month cost, ecosystem maturity, lock-in risk. **Don't auto-pick AWS** — for many MVPs a managed PaaS (Vercel, Fly.io, Render, Railway, Cloud Run) is cheaper, simpler, and avoids the Phase 6 ops weight entirely.

**1.2 — IaC tool and licence decision — Terraform (BSL 1.1) vs OpenTofu (MPL 2.0).** This is a real ADR with a real decision in it, not a formality. The two are CLI-compatible: same HCL, same state format, same provider protocol, and a registry mirror for providers — so the decision is **reversible at low cost**, which is exactly why it should be made explicitly rather than by default.

- **Terraform** is the prescribed primary: the largest provider ecosystem, first-party `terraform test` for module unit tests (GA since 1.6), the deepest pool of existing team knowledge and community modules, and the toolchain the Phase 6 subagent and skills are authored against. Its licence is the **Business Source License 1.1** — source-available, free for essentially all end-user infrastructure work, but with a competitive-use restriction and a delayed conversion to MPL 2.0.
- **OpenTofu** is the CLI-compatible fallback under **MPL 2.0** — a conventional OSI-approved open-source licence. Choose it when the BSL is unacceptable: a client contract or internal policy that requires OSI-approved licensing throughout, a legal team that will not sign off on source-available terms, or an organisation that itself sells infrastructure tooling and therefore sits close to the BSL's restricted field of use.

Whichever is chosen, the ADR must state: which licence, who reviewed it, which providers the project depends on and that they are available for the chosen CLI, and the migration trigger that would flip the decision.

> **What the framework gave up by consolidating on HCL — record this in the ADR, do not bury it.** Earlier versions of this phase prescribed Pulumi for three reasons that were, and remain, good ones: (a) general-purpose languages (TypeScript / Python / Go / .NET / Java) removed the HCL learning curve and unlocked genuine unit-testing frameworks (pytest, Jest, Go test) over infrastructure logic; (b) Apache 2.0 licensing sidestepped the Terraform BSL question entirely — a question this ADR now has to answer on its own merits; (c) its autonomous infra agent, conversational assistant and MCP server were the deepest first-party AI integration of any IaC tool. Consolidating on Terraform/OpenTofu buys ecosystem breadth, one HCL toolchain, native Infracost cost-on-PR with no vendor bridge, and a **$0 control plane** running in the project's own cloud account. It costs the language ergonomics, the language-native test story (`terraform test` is first-party and free but strictly less expressive — Step 3.3), and org-wide resource search (Step 2.7). Both sides belong in the ADR; a reader six months from now should be able to see the trade rather than infer it.

**1.3 — Orchestration decision.** Kubernetes only when you genuinely need it (multi-team platform, complex microservices, on-prem portability). Otherwise: managed PaaS (ECS Fargate, Cloud Run, App Runner, Container Apps, Vercel, Fly.io) is simpler, cheaper, and one fewer thing for AI to misconfigure. If K8s, plan for K8sGPT (in-cluster operator) from day one — it pays back in incident response.

**1.4 — Observability platform decision.** Two viable defaults: **Datadog + Bits AI SRE** (best autonomous incident triage, 90% faster restoration in Datadog's published benchmarks; cost scales with hosts/usage — model the bill carefully) or **Grafana Cloud + Grafana Assistant + Sift** (free tier is generous, Grafana Assistant went free in April 2026, OSS-friendly, full data ownership when self-hosted). Either stack carries application errors on the same OpenTelemetry signal as everything else — errors surface as the error-rate SLO and its burn-rate alert (Step 6.4), not through a separate error-tracking product.

**1.5 — Document each decision as an ADR.** Use [`/templates/adr-template.md`](../../templates/adr-template.md). One ADR per pillar (cloud provider, IaC tool & licence, **state backend**, orchestration, observability, secrets, deployment strategy). Cite the Phase 1 NFR ID each decision satisfies. **State backend is an ADR pillar in its own right** — it used to be a managed-control-plane default, and now it is a thing the team builds.

**Escalation:** If the stack requires technology nobody on the team has touched, time-box a one-week spike before committing. Point the `terraform-iac-engineer` subagent at the proposed architecture and let it generate a throwaway root module to review — a spike artefact, never a merge candidate.

---

### Step 2: Cost Analysis & FinOps

> Visual: [Step 2 flowchart](./FLOWCHART.md#step-2-cost)

| Attribute | Detail |
|-----------|--------|
| **Input** | Architecture (Phase 2), expected traffic from NFRs, IaC code (after Step 3) |
| **Tools** | **Infracost** (native Terraform / OpenTofu cost-on-PR + AutoFix), **Claude Code** (estimate generation, optimisation review), **`cost-guardrails-bringup` skill** (wires the CI job, budget alerts and the unit-economics metric), **CAST AI / Kubecost** (K8s only) |
| **Output** | Monthly cost estimate at launch + projected at 12 months; PR comments on every IaC change with delta cost |
| **Human** | Validate against budget, sign off on cost-impacting PRs |

**Workflow:**

**2.1 — Initial cost model.** Run the [`cost-estimation`](./PROMPTS.md#cost-estimation) prompt in Claude Code. Feed architecture overview, expected traffic (p50 + p99 RPS), data volumes, regions. Output: cost per service, three scenarios (low / expected / peak), top three cost drivers, hidden-cost callouts (egress, NAT gateway, cross-AZ traffic, observability ingest).

**2.2 — Cost-on-PR — one path, Infracost.** Run `infracost breakdown` on the base branch and `infracost diff` on the PR, and post the result with the `infracost/actions/comment` step. Turn on **Infracost AutoFix** so Infracost opens AI-generated cost-reduction PRs (10× faster remediation per Infracost's 2026 benchmarks). Use the `cost-guardrails-bringup` skill to wire the job, the breakdown baseline and the comment step in one pass.

**The vendor gap that shaped earlier versions of this phase is closed.** Infracost has never supported Pulumi natively ([infracost/infracost#187](https://github.com/infracost/infracost/issues/187) remains open), which is why this step used to fork into a Pulumi path with a bespoke Claude Code cost-delta action bridging it, and a Terraform path. Terraform and OpenTofu are Infracost's **native, first-class inputs** — so there is one path, no bridge, and no second cost tool to keep calibrated. The bespoke cost-delta prompt is retired rather than ported: a second, LLM-derived number posted next to Infracost's priced one is worse than no second number, because the wrong one will occasionally be the one a reviewer reads. What that prompt did that Infracost cannot — judging whether a cost-bearing change is *justified* — survives as the reviewer checklist in `cost-guardrails-bringup` and the self-check in `terraform-iac-engineer`.

**2.3 — Quarterly optimisation review.** Run the [`cost-optimisation`](./PROMPTS.md#cost-optimisation) prompt with a paste of the top 50 resources by spend (from cloud billing API). Claude returns ranked opportunities by savings/effort ratio across right-sizing, reserved instances/Savings Plans, spot, storage tiering, egress reduction, auto-scaling tuning, and unused-resource cleanup.

**2.4 — Kubernetes-specific cost (if applicable).** Wire **CAST AI** for autonomous bin-packing, spot-orchestration, and right-sizing — savings-based pricing means it pays for itself. **Kubecost** is the OSS fallback for cost allocation and showback when the org isn't ready for an external SaaS.

**2.5 — Budget alerts.** Configure cloud-native budget alerts (AWS Budgets / GCP Budgets / Azure Cost Management) at 50% / 80% / 100% of monthly target. Route to the team's incident channel — surprise bills are incidents.

**2.6 — Track unit economics.** Cost per active user / cost per request / cost per inference (if AI features ship). Wire to Grafana or Datadog as a dashboard (the `cost-guardrails-bringup` skill does this); review monthly. This is the metric that survives investor and board conversations.

**2.7 — Inventory the fleet with cloud-native tooling.** Cross-stack resource search — *"show me every managed database tagged `Environment=prod` across all three environments"* — has **no tool in the prescribed stack.** Use the cloud provider's own inventory (AWS Resource Explorer or Config, Azure Resource Graph, GCP Cloud Asset Inventory), queried by hand or by Claude Code over the cloud CLI. These are unassessed in [`docs/tools-evaluation/`](../../docs/tools-evaluation/6.AIDLC_CICD_DevOps_Phase_Tools.md) — treat them as a documented path, not a prescription, and extend the evaluation before standardising on one. **The mandatory tag set in `AGENTS.md` is what makes these queries answerable at all** (Step 0.5): an untagged resource is invisible to the only inventory capability the phase has left.

---

### Step 3: Infrastructure Provisioning (IaC) with Terraform

> Visual: [Step 3 flowchart](./FLOWCHART.md#step-3-iac)

| Attribute | Detail |
|-----------|--------|
| **Input** | Approved architecture (Phase 2), tech stack ADRs (Step 1, incl. the Terraform-vs-OpenTofu licence ADR and the state-backend ADR), `AGENTS.md`, NFRs |
| **Tools** | **`iac-state-backend-bringup`** + **`ci-identity-and-secrets-bringup`** skills (backend, locking, OIDC, secret manager, drift workflow) + **`terraform-iac-engineer`** subagent (HCL authoring, `validate` + `plan` loop) + **Terraform / OpenTofu CLI** |
| **Output** | Complete IaC repo provisioning all environments reproducibly, `terraform plan` clean and reviewed, remote state encrypted + locked + versioned, secrets resolved from the project's secret manager, CI holding the only apply-capable credential |
| **Human** | Review every generated change and read the full `terraform plan` before merge; an SRE reviews the backend and IAM bring-up before the first apply |

**Workflow:**

**3.1 — Bootstrap the project and its backend.** Run the `iac-state-backend-bringup` skill — this is a one-time, high-blast-radius bring-up, and the skill exists so it is done the same way every project. It produces, in order:

1. The **root module skeleton** with `required_version` and `required_providers` pinned (`~>` constraints), and `.terraform.lock.hcl` committed so provider resolution is reproducible.
2. The **remote backend** in the project's own cloud account — see 3.4.
3. The **environment directory layout** from 3.3, with state out of version control and verified not already tracked.

Then run `ci-identity-and-secrets-bringup` for the CI OIDC trust relationship, the secret-manager wiring, and the drift workflow (3.5, 3.6).

```bash
terraform init          # after the backend block is in place (3.4)
terraform fmt -recursive
terraform validate
```

An SRE reviews the backend and the IAM trust policy before the first `apply` — they are the two artefacts in this phase with no cheap undo.

**3.2 — Generate the initial configuration with the `terraform-iac-engineer` subagent.** Invoke it by name from Claude Code, feeding the architecture, the NFRs, the target cloud and region, and `AGENTS.md`. It writes the HCL into shared modules (`/modules/vpc`, `/modules/database`, `/modules/service`) plus the per-environment root configuration from 3.3, then loops `terraform fmt` → `validate` → `test` → `plan` until the plan is clean, reporting what it changed on each pass. The [`terraform-iac-generation`](./PROMPTS.md#terraform-iac-generation) prompt is the direct-invocation fallback when the subagent is not installed.

**The subagent holds no apply credential.** It can `init -backend=false`, `validate`, `plan`, `fmt`, `test` and write files. It cannot `apply`, and neither can the developer running it against staging or prod (3.5). A clean plan is the subagent's deliverable; a human reading that plan and merging the PR is the only path to a changed resource.

**3.3 — Structure for environments.** Prescribed default: **one directory per environment with a shared module library.**

```
/modules/{vpc,database,service,queue}/     # reusable, versioned, tested
/envs/{dev,staging,prod}/                  # thin root modules: backend block + module calls + tfvars
```

Directory-per-environment is explicit and diffable: the prod configuration is a file you can read, and there is no chance of applying to the wrong workspace because you forgot which one was selected. **Terraform workspaces** are the alternative, and are reasonable only when environments are near-identical and differ solely by variable values — document the choice as an ADR either way.

Every shared module carries at least one **native `terraform test` case** (`.tftest.hcl`, GA since Terraform 1.6) plus `terraform validate` in CI. Be clear-eyed about what this is: `terraform test` is first-party, free, needs no new vendor, and runs plan-level assertions against module inputs with `mock_provider` — and it is materially less expressive than unit-testing infrastructure logic in a general-purpose language with pytest or Jest. No fixtures, no parameterised cases, no coverage measurement. That expressiveness is a real cost of the HCL consolidation, recorded in the Step 1.2 ADR. Terratest would recover some of it and is deliberately **not** prescribed: it requires real provisioned infrastructure and introduces a Go test harness the framework carries nowhere else.

> **Never write a `terraform test` run block with `command = apply`.** It provisions real infrastructure — it is an apply wearing a test's clothing, and with no managed runner to refuse it, nothing outside the rule stops it. Plan-mode run blocks with mocked providers only.

**3.4 — Remote state: encrypted, versioned, locked, in your own account.** There is **no managed control plane** in the prescribed path and no per-user licence — the state backend is a bucket in the project's own cloud account, chosen by the Step 1.1 cloud decision.

| Cloud | State store | Locking | Encryption |
|-------|-------------|---------|------------|
| AWS | S3 bucket, **versioning on**, public access blocked | DynamoDB lock table (or S3 native lockfile on Terraform ≥ 1.10) | SSE-KMS with a customer-managed key |
| Azure | Blob container in a Storage Account, versioning on | Blob lease (native) | Storage Service Encryption, customer-managed key |
| GCP | GCS bucket, object versioning on | Object generation preconditions (native) | CMEK |

Non-negotiables:
- **Never commit state to git.** Never upload it as a CI artefact, never `cat` it into a log.
- **Treat state as a secret.** `terraform.tfstate` contains every resource ID and, for many providers, secret values in plaintext. Encryption at rest and a restrictive bucket policy are what stand between the state file and a credential dump. **Nothing in the prescribed stack scans state for leaked secrets.**
- **Separate read and write.** The CI apply role writes; humans get a read-only role. Nobody gets both.
- **Bootstrap the backend once, then lock it down.** The bucket and lock table are created by a human with elevated rights on day one and never touched by the pipeline that stores state in them. Record in writing whether that bootstrap is a committed script or a documented manual prerequisite — leaving it undocumented is how a team discovers nobody can rebuild the backend.

**Fallback only:** Spacelift Intelligence or HCP Terraform, when a managed control plane is a hard organisational requirement. The cloud-native path above passes every Gate 1 checkbox with a $0 control plane; a managed runner buys away ops burden, not compliance.

**3.5 — Apply only in CI, under OIDC. Nothing outside CI can apply.** A managed control plane could enforce this org-wide as a product setting; Terraform cannot enforce anything — the CLI will happily apply production for anyone holding credentials. So the rule is enforced by **two mechanisms working together**, and both are Gate 1 items:

1. **Branch protection** — `plan` runs on every IaC PR and its output is reviewed; `apply` runs only on `main`, only in the workflow, behind a GitHub Environment (`production`) with required reviewers.
2. **IAM — the load-bearing half.** **No human principal holds an apply-capable credential for staging or prod.** The only principal with write access to the prod state backend and the prod cloud account is a role assumable *exclusively* by the GitHub Actions OIDC identity, scoped by `sub` claim to this repository, branch and environment. Developers hold read-only. A laptop apply then fails on **authorisation, not on discipline** — which is the only version of this control that survives a deadline.

```yaml
# The shape of it — generated by cicd-pipeline-bringup
permissions:
  id-token: write        # OIDC
  contents: read
steps:
  - uses: hashicorp/setup-terraform@v3      # or opentofu/setup-opentofu@v1
  - uses: aws-actions/configure-aws-credentials@v4   # or azure/login, google-github-actions/auth
    with:
      role-to-assume: ${{ vars.TF_APPLY_ROLE_ARN }}
      aws-region: ${{ vars.AWS_REGION }}
  - run: terraform plan -input=false -out=tfplan
  - run: terraform apply -input=false -auto-approve tfplan   # main only, environment-gated
```

Plan once, apply that exact saved plan — never re-plan inside the apply job, or the thing reviewed is not the thing applied.

**Secrets that OIDC cannot cover** (Anthropic API key, Datadog / Grafana tokens, third-party API keys) live in the project's secret manager — AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, HashiCorp Vault, Doppler, or 1Password — and reach the job either through the provider's secrets action *under the OIDC role* or, where no integration exists, as GitHub Environment-scoped encrypted secrets released only after reviewer approval. **These are static, long-lived credentials.** The versioned, immutable, dynamically-brokered secret layer earlier versions of this phase relied on is not reproduced by anything in this stack: rotation is now a scheduled task with a named owner, not a property of the system. That is a real regression and it carries its own risk row.

**3.6 — Drift detection and remediation.** A scheduled workflow runs `terraform plan -detailed-exitcode` against every environment daily (exit `0` = no drift, `2` = drift, `1` = error). On drift it opens or updates a Linear issue and, for production, pages on-call — **drift in production is an incident.** **Don't auto-remediate prod drift without a human**; auto-apply is fine on dev and ephemeral PR environments. `ci-identity-and-secrets-bringup` generates the workflow, and that first scheduled run doubles as the end-to-end proof that the backend, the trust policy and the secret wiring all work. Note the honest difference from a managed-runner path: nothing here is scheduled *for* you, so if the cron workflow is disabled or its credential expires, drift detection silently stops — Gate 6 checks the **last successful run**, not just the file's existence.

**3.7 — Document the runbook.** `/docs/infrastructure.md` covers: how to provision from scratch, how to make changes (PR flow), **how to recover from corrupted state (restore the prior object version from the versioned bucket)**, **how to clear a stuck lock (`terraform force-unlock` — only after confirming in the Actions run list that no apply is actually in flight; forcing a live lock corrupts state)**, how to handle drift, and how to rotate secret-manager secrets and the OIDC trust policy. Generate the first draft with the [`infra-runbook`](./PROMPTS.md#infra-runbook) prompt; an SRE reviews and signs off. These procedures are more manual than the managed-control-plane equivalents — which is exactly why the runbook has to be exact, and why Phase 7's handoff drills the state-recovery path live rather than just documenting it.

> **Gate 1 — Infrastructure foundation** must pass before any workload deploys. See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-infrastructure-foundation).

---

### Step 4: CI/CD Pipeline Setup

> Visual: [Step 4 flowchart](./FLOWCHART.md#step-4-pipeline)

| Attribute | Detail |
|-----------|--------|
| **Input** | Repo, test suite (Phase 4), security tools (Phase 5), IaC (Step 3), `AGENTS.md` |
| **Tools** | **`cicd-pipeline-bringup` skill** + **GitHub Actions** + **GitHub Copilot** (workflow generation) + **Claude Code Action** (`@claude` fix-on-mention) + the project's chosen AI PR reviewer — **CodeRabbit** and/or the **Claude PR review bot** ([Phase 3 Step 4.3](../03-development/PROCESS.md#step-4-code-review)) — + **Copilot Agentic Workflows** (autonomous repository tasks); **GitLab CI + GitLab Duo** as fallback |
| **Output** | Fully automated PR-to-prod pipeline with AI-generated YAML, AI PR review, AI-driven failure RCA, AI-fixable lint/test debt |
| **Human** | Review pipeline for least-privilege, secrets handling, manual approval gates |

**Workflow:**

**4.1 — Generate pipeline configs with the `cicd-pipeline-bringup` skill.** The skill walks the whole bring-up — stage generation, the `@claude` workflow, the agentic workflows, secrets, branch protection — and ends at the Gate 2 checklist. For ad-hoc work, open the repo in VS Code with GitHub Copilot and run the [`pipeline-generation`](./PROMPTS.md#cicd-pipeline-generation) prompt against the tech-stack ADR directly. Copilot scaffolds the YAML; Claude Code (via terminal) refines edge cases (matrix builds, reusable workflows, OIDC). Standard stages, in order:

1. **Lint & format** — fast feedback < 2 min, fail closed.
2. **Build** — compile, build container image with multi-stage Dockerfile from Step 5.
3. **Unit + integration tests** (Phase 4) — the test runner's own coverage threshold fails the job below 80% on new code; the coverage report uploads as a CI artefact.
4. **Security scans** — CodeQL SAST + Dependabot + ggshield + Claude Code `/security-review` (Phase 5).
5. **Terraform plan** — `terraform plan` against the target environment, plan output posted to the PR alongside the Infracost delta comment (Step 2.2).
6. **Deploy to staging** — auto on merge to `main`; the workflow applies the saved plan under the CI OIDC role.
7. **E2E tests against staging** — Playwright suite from Phase 4.
8. **Load test on release candidate** — k6 against staging, only on tagged RC builds.
9. **Deploy to production** — manual approval gate, blue/green or canary.
10. **Post-deploy verification** — smoke tests + Grafana / Datadog deploy annotation.

**4.2 — Wire AI review and `@claude` fix-on-mention into the PR workflow.** Two independent decisions; keep them in separate workflow files.

**a) Automated AI PR review — configure the project's chosen reviewer.** The selection rule belongs to [Phase 3 → Step 4.3](../03-development/PROCESS.md#step-4-code-review), which offers two peers (not a primary and a fallback). This step is where the choice gets wired:

| Choice | What Phase 6 does |
|--------|-------------------|
| **CodeRabbit** | Install the GitHub app on the repo and commit its config file. No workflow to author, no secret to manage, nothing in `.github/workflows/`. |
| **Claude PR review bot** | Commit the workflow, prompt template, and checklist specified in full in **[Phase 3 → PR-REVIEW-BOT.md](../03-development/PR-REVIEW-BOT.md)** — trigger matrix and draft opt-in, the size gate that keeps token spend bounded, the prompt-template / checklist split, idempotency across re-review passes, the `REQUEST_CHANGES`-never-`APPROVE` output contract, prompt-injection hardening for untrusted PR titles, and the fail-open posture. Build it from that document rather than from scratch — review *policy* belongs to Phase 3; the CI wiring here only hosts it. |
| **Both** | Do both of the above. Budget for both cost models. |

Whichever is configured, **the AI review job is never a required status check** (Step 4.6).

**b) `@claude` mention for fixes — independent of the above.** Generate `.github/workflows/claude.yml` with the [`claude-code-action--pr-review-workflow`](./PROMPTS.md#claude-code-action--pr-review-workflow) prompt. A developer types `@claude please fix the failing TypeScript build` on a PR comment; Claude opens a follow-up commit on the same branch with the fix and re-runs CI. This is worth having on any repo regardless of which reviewer was chosen — including a CodeRabbit-only repo. Unlike a reviewer, it **writes to the branch**: scope its token accordingly and never grant it to workflows triggered by `pull_request_target`.

Platform rules for every `claude-code-action` workflow: pin an exact version (`anthropics/claude-code-action@v1.0.158`, not a floating `@v1`) so behaviour changes arrive as a reviewed PR; declare the narrowest `permissions:` block the job needs; use the smallest credential scope possible (a fine-grained PAT or GitHub App with read/write only on the relevant repos); and gate on `if: github.actor != 'dependabot[bot]'` so none of them burn tokens on bot PRs.

**4.3 — Add Copilot Agentic Workflows for repository hygiene.** In `.github/agentic-workflows/` commit Markdown agents for: triaging stale issues, generating release notes from merged PRs, repairing the most-flaky test of the week, fixing top-N lint debt. Each is a Markdown file declaring trigger, agent engine (set to Claude Code), and safe outputs (create PR, add comment) — no write access without an explicit safe-output declaration.

**4.4 — GitLab Duo Root Cause Analysis (fallback platform).** If the org is on GitLab, enable Duo Root Cause Analysis on the Premium tier — it summarises failed CI logs, identifies the likely cause (syntax / compile / Docker build), and proposes a fix patch. Self-Hosted Duo supports Anthropic / Mistral / OpenAI model families — use Anthropic Claude for tool consistency with Phase 3.

**4.5 — Secrets in CI.** **Cloud authentication uses OIDC — there are no long-lived cloud keys anywhere in the pipeline** (Step 3.5). Everything OIDC cannot cover lives in the project's secret manager (AWS Secrets Manager / Azure Key Vault / GCP Secret Manager / Vault / Doppler / 1Password) and is fetched at job runtime under the OIDC role. Where no manager integration exists, use GitHub-encrypted secrets **scoped to a GitHub Environment** — the `production` environment requires reviewer approval before its secrets are released to a job, which is the closest this stack gets to approval-gated secret release. Scope every secret to the narrowest environment that needs it, so a leak blasts one environment rather than all three, and put a named owner and a quarterly rotation date on each one — nothing in the stack rotates them for you.

**4.6 — Branch protection.** `main` requires: PR + passing CI + AI review acknowledged (whichever reviewer the project configured) + 1 human approval + Linear issue identifier in the title (Phase 3 convention). High-blast-radius changes (auth, schema migrations, IAM) require **2** humans. **Do not make the AI review job a required check** — it depends on a third-party LLM provider, and a rate-limit spike would freeze the merge queue. Acknowledgement is a human responsibility at Gate 2, enforced by the reviewer, not by branch protection.

> **Gate 2 — CI/CD pipeline operational** must pass before the team relies on automation for deploys. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-cicd-pipeline-operational).

---

### Step 5: Containerization

> Visual: [Step 5 flowchart](./FLOWCHART.md#step-5-containers)

| Attribute | Detail |
|-----------|--------|
| **Input** | Application code, runtime dependencies, target runtime (ECS / Cloud Run / K8s) |
| **Tools** | **Docker Gordon** (Dockerfile rate + propose, in Docker Desktop 4.61+) + **Claude Code** (multi-file Dockerfile + K8s manifest gen) + **GitHub Copilot** (inline) + **K8sGPT** (cluster diagnostics, in-cluster operator) |
| **Output** | Multi-stage Dockerfiles with digest-pinned bases, non-root runtime and `HEALTHCHECK`; K8s manifests / Helm charts (when needed) |
| **Human** | Review every Dockerfile for base-image safety, non-root user, layer ordering |

**Workflow:**

**5.1 — Generate Dockerfiles.** For multi-service repos invoke the **`container-image-engineer`** subagent — it reads cross-file context (dependency manifest, build scripts, the runtime ADR) and produces the Dockerfile plus `.dockerignore` together, then self-checks against the container standards it can actually verify. The [`dockerfile-generation`](./PROMPTS.md#dockerfile-generation) prompt is the direct-invocation fallback. For single-Dockerfile triage, Gordon is quicker — from the project root:

```bash
docker ai "rate my Dockerfile"   # Gordon: triage existing
docker ai "generate a multi-stage Dockerfile for a Node 22 + Next.js app, distroless runtime, non-root user"
```

For multi-service repos use Claude Code with the [`dockerfile-generation`](./PROMPTS.md#dockerfile-generation) prompt — it understands cross-file context (package.json, build scripts, ADR-2 runtime decisions) better than Gordon for complex projects.

**5.2 — Multi-stage build discipline.**
- **Build stage** — full SDK, test runners, build tools.
- **Runtime stage** — distroless or Alpine, only runtime deps, non-root user, `READONLY_ROOTFS=true`, healthcheck.
- **Pin base images by digest**, not just `:latest` or `:22` — repeatability matters for both reliability and supply-chain.

**5.3 — Cache for CI velocity.** Layer order is dependency manifest → install → source. Use BuildKit cache mounts (`--mount=type=cache`) and the GHA `actions/cache` step. AI-generated Dockerfiles often miss this — review for cache-busting copies.

**5.4 — Kubernetes manifests (only if K8s).** Use Claude Code with the [`k8s-manifests-generation`](./PROMPTS.md#kubernetes-manifests-generation) prompt for Deployment + Service + Ingress + ConfigMap + Secret (via External Secrets Operator) + HPA + PDB + NetworkPolicy. **K8sGPT operator** in-cluster catches misconfigurations the manifests passed but the cluster doesn't tolerate (e.g., missing PDBs on autoscaled deployments, ingress class mismatches). K8sGPT also exposes an MCP server mode — connect it to Claude Code for in-session troubleshooting.

**5.5 — Helm or Kustomize for environment overlays.** Pick one, document the choice as an ADR. Claude Code generates either; consistency matters more than the choice.

> **Gate 3 — Containerization standards** must pass for every service container. See [QUALITY-GATES.md → Gate 3](./QUALITY-GATES.md#gate-3-containerization-standards).

---

### Step 6: Observability — Logging, Monitoring, Alerting

> Visual: [Step 6 flowchart](./FLOWCHART.md#step-6-observability)

| Attribute | Detail |
|-----------|--------|
| **Input** | Running application + infrastructure, NFRs from Phase 1 (latency / availability / error rate targets) |
| **Tools** | **OpenTelemetry SDK** (instrumentation, vendor-neutral); **Datadog + Bits AI SRE** *or* **Grafana Cloud + Grafana Assistant + Sift**; **Claude Code with the observability MCP server** (incident assistant) |
| **Output** | Production dashboards, SLOs with burn-rate alerts, runbooks per alert, AI-driven first-responder triage |
| **Human** | Define SLOs from NFRs; sign off on alert rules; review autonomous remediation suggestions |

**Workflow:**

**6.1 — Instrument with OpenTelemetry.** OTEL is the vendor-neutral floor — auto-instrumentation now covers Java, Node, Python, Go, .NET out of the box. Emit traces + metrics + structured JSON logs with correlation IDs. **Don't lock into a vendor SDK** — OTEL means switching from Datadog to Grafana (or vice versa) is a config change, not a rewrite.

**6.2 — Pick the observability backend.**
- **Datadog + Bits AI SRE** — picks up alerts the moment they fire, walks the telemetry, runbooks, dependency graph, and recent deploys, and posts a hypothesis with a confidence score to Slack/Teams before the on-call has logged in. GA Dec 2025; Q1 2026 update is ~2× faster and supports HIPAA + RBAC. Cost is the catch — model the bill at expected host count + ingest rate before committing.
- **Grafana Cloud + Grafana Assistant + Sift** — Grafana Assistant became free in April 2026, generates dashboards from connected data sources, suggests PromQL/LogQL, runs Sift investigations on incidents (Asserts.ai-powered RCA, surfaced in the Incident workflow). Cheaper and more portable; the trade-off is that autonomous incident response is less mature than Bits AI SRE.

**6.3 — Generate dashboards via AI.** Run this through the **`observability-bringup`** skill, which chains 6.3 → 6.4 → 6.5 in order so each step consumes the previous one's output — the SLO targets populate the SLO dashboard, the alerts derive from those SLOs, and the runbooks derive from those alerts. Running them as three separate sessions with metric names re-pasted between them is exactly where invented metric names enter. Don't hand-author Grafana JSON. From Claude Code, run the [`dashboard-generation`](./PROMPTS.md#observability--dashboard-generation) prompt against the OpenAPI spec + the SLO targets + the architecture; it returns Grafana JSON or Datadog dashboard JSON ready to commit. Store dashboards as code in `/observability/dashboards/`.

**6.4 — Define SLOs from NFRs.** Each Phase-1 NFR becomes one SLO — `p95 API latency < 200ms`, `99.9% availability`, `error rate < 0.1%`. Configure burn-rate alerts (fast burn 2% in 1h → page; slow burn 10% in 6h → ticket). Use [`slo-and-alert-generation`](./PROMPTS.md#slo-and-alert-generation) prompt — Claude Code drafts both Prometheus AlertManager rules and the matching Grafana / Datadog alert config.

**6.5 — Runbook per alert.** Every alert links to `/docs/runbooks/alerts/<alert-name>.md`, where `<alert-name>` is the alert's name exactly as configured in the observability backend, lowercased and kebab-cased — so the runbook path is derivable from the alert rather than hand-maintained. (Service operational runbooks live alongside at `/docs/runbooks/services/<service>.md` and are owned by the development phase; the two never collide.) Use [`runbook-generation`](./PROMPTS.md#runbook-generation-from-alert) — Claude Code drafts diagnosis steps, remediation, escalation. **AI runbooks are starting points, not final** — an SRE reviews each before it's wired to the alert.

**6.6 — Alerting discipline.** If it doesn't require action, it's not an alert. Bits AI SRE / Sift filter noise but they can't fix bad alert design — review fired alerts every retro for "did this require human action? if no, mute or upgrade to a metric."

> **Gate 4 — Observability & alerting** must pass before production traffic reaches the service. See [QUALITY-GATES.md → Gate 4](./QUALITY-GATES.md#gate-4-observability--alerting).

---

### Step 7: Deployment Automation & Rollback

> Visual: [Step 7 flowchart](./FLOWCHART.md#step-7-deploy)

| Attribute | Detail |
|-----------|--------|
| **Input** | Approved release candidate (Phases 4–5), green Gates 1–4 |
| **Tools** | **`deploy-and-rollback-bringup` skill** + **GitHub Actions** (apply under OIDC) + **Datadog / Grafana deploy markers** + **Bits AI SRE / Sift** (post-deploy watch) |
| **Output** | Zero-downtime deployments with automatic rollback on SLO breach |
| **Human** | Approve production deploys; manage incidents |

**Workflow:**

**7.1 — Auto-deploy to staging.** Every merge to `main` deploys to staging with no human in the loop. The same workflow that ran `plan` on the PR now applies the saved plan against staging under the CI OIDC role.

**7.2 — Manual approval gate for production.** GitHub Environment `production` has required reviewers — a human clicks "deploy" or schedules to a release window. The approval is the human checkpoint; everything before and after is automated.

**7.3 — Deployment strategy** (decided in Step 1 ADR):
- **Blue/Green** — provision the new environment, switch traffic, keep old for instant rollback. Best for fast rollback at the cost of double resources during deploy.
- **Canary** — small % of traffic to the new version, watch SLOs, gradually increase. Best for high-traffic services where bad changes only show under real load.
- **Rolling** — instances replaced one at a time. Simplest; no instant rollback.

**7.4 — Health checks and post-deploy smoke tests gate full rollout.** New version must pass `/healthz` (liveness), `/readyz` (readiness), and the Phase-4 Playwright smoke suite before traffic fully routes. Failure here triggers automatic rollback.

**7.5 — Automatic rollback on SLO breach.** Wire Datadog / Grafana to GitHub Actions: if error rate or p95 latency exceeds threshold within 10 minutes of a deploy, the watcher fires a `repository_dispatch` event that triggers the rollback workflow — it re-deploys the **previous image digest** for application changes and, for infrastructure changes, re-applies the **last known-good IaC commit** (`git revert` of the offending commit, then `plan` → `apply` under the same OIDC role).

**Be honest about the caveat.** Terraform has no "previous stack revision" primitive — rollback *is* re-applying an earlier commit, and that only works if the earlier commit is still applyable. A destructive change in between (a dropped column, a deleted resource, a renamed module that moved state addresses) makes the rollback itself a forward migration. So: separate infrastructure rollback from application rollback in the runbook, roll the image digest first because it is always safe, and treat any IaC change that cannot be reverted by re-applying its parent commit as high-blast-radius — 2 approvers, and a written forward-fix plan in the PR before merge.

**7.6 — Deploy markers on dashboards.** Every deploy posts an annotation to Datadog / Grafana — incident correlation collapses from minutes to seconds. Bits AI SRE / Sift use these markers when proposing root-cause hypotheses.

**7.7 — Quarterly rollback drill.** Schedule a deliberate rollback against a staging release candidate every quarter. If rollback takes more than 5 minutes, the runbook is wrong. Treat the drill as a Phase 4 gate.

> **Gate 5 — Deployment & rollback** must pass for every production deploy. See [QUALITY-GATES.md → Gate 5](./QUALITY-GATES.md#gate-5-deployment--rollback).

---

## Phase Handoff

When CI/CD & DevOps is complete, the following artefacts hand off to **Phase 7: Delivery & Handoff**:

| Artefact | Format | Location |
|----------|--------|----------|
| Terraform IaC repository | HCL | Git repo with `/modules`, `/envs`, `.terraform.lock.hcl`, `.tftest.hcl` module tests |
| Remote state backend | S3 + DynamoDB / Azure Blob / GCS | Project's own cloud account; versioned + encrypted + locked |
| Secret-manager secrets + CI OIDC trust policy | Cloud secret manager / Vault / Doppler / 1Password + IAM | Per environment, each with a named owner and rotation date |
| Phase 6 subagents + skills | Markdown | `.claude/agents/` + `.claude/skills/` (transfer with the repo) |
| CI/CD workflows | YAML + Markdown agentic workflows | `.github/workflows/` + `.github/agentic-workflows/` |
| Claude Code Action config (`@claude` fix-on-mention) | YAML | `.github/workflows/claude.yml` |
| AI PR reviewer — *if CodeRabbit* | Vendor config | `.coderabbit.yaml` (app installed on the repo) |
| AI PR reviewer — *if Claude PR review bot* | YAML + Markdown | `.github/workflows/claude-pr-review.yml` + `.github/prompts/pr-review.md` + `.claude/prompts/pr-review-checklist.md` |
| Dockerfiles + .dockerignore | Per service | Service directories |
| K8s manifests / Helm charts (if K8s) | YAML / Helm | `/deploy/` |
| Observability dashboards | Datadog / Grafana JSON | `/observability/dashboards/` |
| SLO + alert definitions | YAML / Datadog config | `/observability/slos/` |
| Runbooks per alert | Markdown | `/docs/runbooks/alerts/` |
| Infrastructure runbook | Markdown | `/docs/infrastructure.md` |
| Deployment runbook | Markdown | `/docs/deployment.md` |
| Cost analysis report | Markdown + Infracost breakdown JSON | `/docs/cost-analysis.md` |
| AGENTS.md | Markdown | Repo root |

**Handoff Checklist:**

- [ ] All five gates (1–5) passed
- [ ] Terraform configuration reproduces all environments from a clean clone in < 30 minutes
- [ ] `terraform plan` is clean against every environment (no unexplained drift)
- [ ] Remote state is versioned, encrypted, locked, and never committed; state bucket denies public access
- [ ] No human principal holds an apply-capable credential for staging or prod — verified by attempting a laptop apply and confirming it fails on **authorisation**
- [ ] CI/CD pipeline auto-deploys PR → staging without manual steps
- [ ] Production deploy works zero-downtime; rollback drill passes < 5 min
- [ ] Observability covers application + infrastructure end-to-end
- [ ] SLOs defined and being measured against NFRs from Phase 1
- [ ] Every alert is actionable and has a linked runbook
- [ ] Cost estimate validated and within budget; cost-on-PR active
- [ ] All secrets resolved through the project's secret manager or CI OIDC; no hardcoded credentials anywhere; every static secret has a named owner and a rotation date
- [ ] Cloud audit logging covers the state backend (CloudTrail data events / Storage logging / GCS audit logs); GitHub audit log enabled
- [ ] The project's chosen AI PR reviewer ([Phase 3 Step 4.3](../03-development/PROCESS.md#step-4-code-review)) posts a review on every PR and is **not** a required check — if the Claude PR review bot, also size-gated and verified not to re-raise findings the author already answered on a second pass ([PR-REVIEW-BOT.md § Adoption checklist](../03-development/PR-REVIEW-BOT.md#9-adoption-checklist)); `@claude` fix workflow tested separately
- [ ] Bits AI SRE / Grafana Sift configured with on-call rotation
- [ ] AGENTS.md committed and referenced by the Phase 6 subagents and skills
- [ ] `/agents` lists `terraform-iac-engineer` and `container-image-engineer`; all six bring-up skills present and smoke-tested

---

## Risks & Guardrails

| Risk | Mitigation |
|------|------------|
| **AI-generated IaC introduces production-breaking config** (over-permissive IAM, public S3, missing encryption) | `terraform plan` runs on every IaC PR and its resource diff is read by a human before merge; `AGENTS.md` fences the agent with mandatory tags and forbidden resource types; the `terraform-iac-engineer` subagent holds no apply credential, so a clean plan is its ceiling. There is no automated policy-as-code gate in the prescribed stack — this risk is carried by human review, so treat IAM, network and encryption diffs as high-blast-radius (2 approvers, per Step 4.6). |
| **State-file blast radius** — `terraform.tfstate` holds every resource ID and, for many providers, secret values in plaintext | Encrypted bucket (SSE-KMS / CMEK / customer-managed key), bucket policy restricted to the CI OIDC write role plus one break-glass human read role, versioning on, public access blocked, never committed to git, never uploaded as a CI artefact. **Nothing in the prescribed stack scans state for leaked secrets** — this is carried by the bucket policy and by not putting state anywhere else. |
| **`terraform apply` from a laptop** — the CLI applies production for anyone holding credentials, and no product setting prevents it | A managed control plane could enforce this org-wide; Terraform cannot enforce anything. The control is credential separation: the only principal with write access to the prod state backend and prod cloud account is a role assumable exclusively by the GitHub Actions OIDC identity, `sub`-scoped to this repo, branch and environment. Humans hold read-only. Verified at Gate 1 by attempting a laptop apply and confirming it fails on **authorisation**, not on discipline. |
| **Static long-lived third-party secrets** — the versioned, dynamically-brokered secret layer has no inheritor | OIDC covers every cloud credential, so nothing is stored for the largest category. For the rest (Anthropic, Datadog, Grafana, third-party APIs): store in the project's secret manager, scope to a single GitHub Environment so a leak blasts one environment, assign a named owner and a quarterly rotation date, and rely on ggshield (Phase 5) to catch a commit. **Accept that a leaked token stays valid until a human rotates it.** |
| **No cross-stack resource search** — org-wide inventory and compliance query have no inheritor | Cloud-native inventory (AWS Resource Explorer / Config, Azure Resource Graph, GCP Cloud Asset Inventory) queried by hand or by Claude Code over the cloud CLI; unassessed in `docs/tools-evaluation/`, so treat as a documented path and extend the evaluation before standardising. The mandatory tag set in `AGENTS.md` is what makes those queries answerable — an untagged resource is invisible to the only inventory capability the phase has left. |
| **Rollback assumes the previous commit is still applyable** | Terraform has no previous-revision primitive; rollback is re-applying an earlier commit. Destructive changes in between turn the rollback into a forward migration. Roll the image digest first (always safe), separate app rollback from IaC rollback in the runbook, and treat any non-revertible IaC change as high-blast-radius (2 approvers + a written forward-fix plan in the PR). |
| **`terraform force-unlock` used casually** — a stuck lock invites forcing it, which corrupts state if the other apply is genuinely running | The runbook requires confirming in the GitHub Actions run list that no apply is in flight **before** unlocking, and recording who unlocked and why. Bucket versioning is the only recovery path if this goes wrong — verify it is on at Gate 1. |
| **Provider version drift** — an unpinned provider changes plan output between two runs of the same code | `required_providers` pinned with `~>` constraints and `.terraform.lock.hcl` committed; Dependabot raises provider-update PRs; the plan reviewed on the PR is the plan applied on merge (one plan, saved and reused). |
| **The bring-up skills touch the highest-blast-radius surface in the phase** — they author the state backend, the IAM trust policy and the secret-manager wiring | An SRE reviews the backend, bucket policy and OIDC trust policy before the first `apply`; the bootstrap runs once and the bucket is then locked down; the skill's output is a PR like any other, never applied directly. A backend or trust-policy mistake is the one error in this phase with no cheap undo. |
| **Claude Code Action token cost runs away** on bot-driven PRs | Gate the action behind `if: github.actor != 'dependabot[bot]'` and similar; cap monthly Anthropic API spend; route to Bedrock / Vertex if cloud commits are already in place. |
| **Bits AI SRE / Sift confidence-score over-trust** — engineers stop validating | First responder still owns the incident. AI suggestion is a hypothesis; the runbook is the source of truth. Track AI-suggested-vs-actual-cause variance in retros. |
| **Drift detection auto-remediates production** | Auto-remediate only on ephemeral / dev environments; prod drift pages on-call. Document the policy in `/docs/infrastructure.md`. And if the cron workflow silently stops — disabled, or its credential expired — drift detection stops with it, so Gate 6 checks the **last successful run**, not the file's existence. |
| **MCP server scope creep** — granting an agent more cloud rights than needed | Each MCP connection inherits the connecting developer's scopes; the Phase 6 roster is GitHub + observability only, neither of which can mutate infrastructure. Cloud write access lives exclusively in the CI OIDC role. |
| **AI-generated Dockerfiles use vulnerable bases** | The prescribed stack has **no image scanner** — this is carried entirely at authoring and review time. Step 5.2 requires digest-pinned, minimal (distroless / Alpine) bases; Gate 3 checks each Dockerfile against that bar; Dependabot raises PRs on base-image and dependency updates it can see from the manifest. Accept that a CVE introduced by a base image is found later here than it would be with a registry scanner. |
| **Rollback drift over time** — the rollback path stops working because nobody tested it | Quarterly rollback drill; if the drill takes > 5 min, treat it as a Sev-2 retrospective trigger. |

---

## Daily DevOps Workflow (steady state)

```
Morning:
  1. Pull latest main
  2. Read the overnight drift workflow result — clean exit 0, or the Linear issue it filed on exit 2
  3. Triage Bits AI SRE / Sift hypotheses from overnight alerts; close noise, file Linear issues for real signal

Pipeline work:
  4. Open PR for IaC or pipeline change
  5. Configured AI reviewer posts review automatically (advisory; never blocks)
  6. terraform plan output + Infracost delta comment on the PR
  7. Address review comments; @claude for routine fixes
  8. Human approves; merge → CI applies the saved plan to staging under the OIDC role

Production deploy:
  9. Tag release candidate; pipeline runs full e2e + load
 10. Manual approval gate; deploy to prod (blue/green or canary)
 11. Bits AI SRE / Sift watches first 30 min; rollback if SLO breach — image digest first, IaC re-apply second
 12. Deploy marker on Grafana / Datadog dashboards

Incident:
 13. Alert fires → Bits AI SRE / Sift posts hypothesis + correlated deploys
 14. On-call opens runbook; Claude Code with observability MCP triages
 15. Mitigate (rollback or fix-forward); post-mortem within 48h
```

---

## Related Documents

- [Prompt Templates →](./PROMPTS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Process Flowchart →](./FLOWCHART.md)
- [CI/CD & DevOps Tools Evaluation →](../../docs/tools-evaluation/6.AIDLC_CICD_DevOps_Phase_Tools.md)
- [Runbook Template →](../../templates/runbook-template.md)
- [Post-Mortem Template →](../../templates/post-mortem-template.md)
- [ADR Template →](../../templates/adr-template.md)
- [Phase 3 Linear MCP setup (carries over) →](../03-development/PROCESS.md#step-0-one-time-setup--connect-claude-code-to-linear-via-mcp)

## External References

- [Terraform](https://developer.hashicorp.com/terraform) — BSL 1.1; `required_providers`, backend configuration, `terraform test`
- [Terraform `test` command](https://developer.hashicorp.com/terraform/language/tests) — native module unit testing, GA since 1.6
- [OpenTofu](https://opentofu.org/) — MPL 2.0, CLI-compatible fork; `opentofu/setup-opentofu`
- [`hashicorp/setup-terraform`](https://github.com/hashicorp/setup-terraform) — the CI action
- [Configuring OpenID Connect in cloud providers](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect) — the mechanism that removes long-lived cloud keys from CI
- [Infracost](https://www.infracost.io/docs/) + [Infracost AutoFix](https://www.infracost.io/docs/infracost_cloud/autofix/) — native Terraform / OpenTofu cost-on-PR
- [Anthropic Claude Code GitHub Action](https://github.com/anthropics/claude-code-action) — official `anthropics/claude-code-action@v1`
- [Claude Code GitHub Actions docs](https://code.claude.com/docs/en/github-actions)
- [GitHub Agentic Workflows](https://github.blog/ai-and-ml/automate-repository-tasks-with-github-agentic-workflows/) — Markdown-authored, Actions-executed
- [GitLab Duo Root Cause Analysis](https://docs.gitlab.com/user/gitlab_duo/use_cases/) — fallback CI failure RCA
- [Docker Gordon (AI agent in Docker Desktop)](https://docs.docker.com/ai/gordon/)
- [K8sGPT operator + MCP mode](https://docs.k8sgpt.ai/reference/operator/overview/)
- [Datadog Bits AI SRE](https://www.datadoghq.com/product/ai/bits-ai-sre/) — autonomous incident triage, GA Dec 2025, 2× faster Q1 2026
- [Grafana Assistant + Sift](https://grafana.com/products/cloud/ai-assistant/) — free as of April 2026
- [Spacelift Intelligence (fallback IaC orchestration)](https://spacelift.io/blog/introducing-spacelift-intelligence)
- [Infracost AutoFix](https://www.infracost.io/docs/infracost_cloud/autofix/) — Terraform/OpenTofu only as of 2026
- [AGENTS.md spec](https://agents.md) — open standard for AI coding agent project context
