# Phase 6: CI/CD & DevOps — Process Definition

## Overview

This document defines the AI-assisted workflow for the CI/CD & DevOps phase of the AI-DLC. It is the phase where infrastructure decisions become running production systems and where the productivity gains from AI tooling are largest — modern AI agents now author IaC, generate pipelines, scan and fix Dockerfiles, draft dashboards, and triage incidents end-to-end. The process below is built around **Pulumi AI (Pulumi Neo + Pulumi Copilot + Pulumi MCP)** as the primary IaC stack, with **GitHub Actions + Copilot agentic workflows + the official Anthropic Claude Code Action** as the CI/CD spine, **Docker Gordon + Claude Code** for containerization, **Datadog Bits AI SRE** (or **Grafana Assistant**) for observability, and **Pulumi Insights / Infracost** for cost.

**Phase Duration:** Initial setup 1–2 weeks + continuous refinement alongside Development
**Phase Owner:** DevOps Engineer / Platform Engineering Lead
**Tools Used:**

- **IaC:** Pulumi AI (Pulumi Neo, Pulumi Copilot, Pulumi MCP server, Pulumi ESC, Pulumi CrossGuard)
- **CI/CD platform:** GitHub Actions + GitHub Copilot Agentic Workflows + Anthropic Claude Code Action (or GitLab CI + GitLab Duo as fallback)
- **IaC orchestration:** Pulumi Deployments (primary) or Spacelift Intelligence (fallback)
- **Containerization:** Docker Desktop with Gordon AI + Claude Code; Trivy + Docker Scout for image scanning
- **Cost:** Pulumi Insights + Infracost (Terraform/OpenTofu fallback only — Infracost still does not support Pulumi natively as of 2026)
- **Observability:** Datadog + Bits AI SRE (primary commercial) **or** Grafana Cloud + Grafana Assistant + Sift (primary OSS-friendly); Sentry + Seer for application errors
- **AI authoring:** Claude Code (terminal agent) + Cursor (IDE) — both connected to Pulumi MCP, GitHub MCP, and observability MCP servers

> **Tool Philosophy.** DevOps is now AI-native. We treat Pulumi Neo as a junior platform engineer that reads tickets, plans changes, opens PRs, runs `pulumi preview`, and waits for human review. We treat Bits AI SRE / Grafana Sift as the first responder on every alert. Claude Code remains the orchestrator that ties Pulumi MCP, GitHub MCP, and observability MCP servers into one session — same model the team already uses in Phase 3. Five tool families, zero overlap, every state-changing action requires a human approval gate.

---

## Tool Stack

| Layer | Primary | Fallback | Cost |
|-------|---------|----------|------|
| **IaC engine** | Pulumi (Apache 2.0 OSS) | OpenTofu (MPL 2.0) — for HCL-mandated environments | Free CLI |
| **AI IaC agent** | Pulumi Neo (autonomous infra agent) + Pulumi Copilot (conversational, in Pulumi Cloud) | Claude Code with Pulumi MCP | Neo gated to Enterprise/Business Critical; MCP server is free |
| **IaC orchestration / state** | Pulumi Cloud + Pulumi Deployments | Spacelift Intelligence (multi-IaC) or self-managed S3/Azure Blob/GCS | Pulumi Team from $75/user/mo; Enterprise ~$32.8K/yr |
| **Secrets in CI/CD** | Pulumi ESC (versioned, pulls from Vault/Secrets Manager/Key Vault/1Password) | Platform-native (GitHub/GitLab encrypted secrets) + Doppler | ESC included in Pulumi Cloud Team+ |
| **CI/CD platform** | GitHub Actions + Copilot Agentic Workflows | GitLab CI + GitLab Duo | GH Free tier + Copilot Pro $10/user/mo + Copilot CI seats $4/seat |
| **AI in CI** | Anthropic `anthropics/claude-code-action@v1` (PR review, fix-on-mention, automation) | GitLab Duo Root Cause Analysis | API usage-based (Anthropic / Bedrock / Vertex / Foundry) |
| **Containerization** | Docker Desktop 4.61+ with Gordon AI + Claude Code (multi-file Dockerfile/K8s manifest gen) | GitHub Copilot inline + K8sGPT (in-cluster operator + MCP) | Docker Personal free; Pro $9/user/mo |
| **Image security** | Trivy + Docker Scout (with AI remediation) | Snyk Container | Trivy free; Scout free tier |
| **Cost analysis** | Pulumi Insights (resource search, compliance, spend insights) + cloud-native calculators | Infracost CLI/Cloud (Terraform/OpenTofu only — Pulumi unsupported natively in 2026) | Insights bundled with Pulumi Cloud |
| **K8s cost (if used)** | CAST AI (autonomous bin-packing, savings-based) | Kubecost (OSS) | CAST AI savings-based; Kubecost OSS free |
| **Observability (commercial)** | Datadog + Bits AI SRE (autonomous alert triage, GA Dec 2025, 2× faster Q1 2026) | New Relic AI | Datadog from $15/host + usage |
| **Observability (OSS-leaning)** | Grafana Cloud + Grafana Assistant (free tier, April 2026) + Sift (Asserts.ai-powered RCA) | Self-hosted Prometheus + Loki + Tempo | Grafana Cloud Free tier; Pro $8/user/mo |
| **Error tracking + AI fix** | Sentry + Seer + Sentry MCP server | Bugsnag | Sentry Team from $26/mo |
| **Incident management + AI** | incident.io (AI summary + PagerDuty-compatible) | PagerDuty AIOps | from $20/responder/mo |

**Optional upgrades:**
- **Pulumi Business Critical** — when SOC 2 / HIPAA / FedRAMP-adjacent compliance is in play (~$50K–$500K/yr; AWS Marketplace listing starts at $50K).
- **Datadog Bits AI for Dev / Security** — for autonomous triage on application performance and AppSec, on top of Bits AI SRE.

---

## Process Steps

### Step 0: One-Time Setup — Wire AI Tools into the DevOps Loop

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | Pulumi Cloud account (Team or above), GitHub org with Copilot Pro / Copilot Business licences, Anthropic API key (or Bedrock / Vertex / Foundry credentials), Datadog OR Grafana Cloud workspace, Docker Desktop 4.61+, Claude Code installed |
| **Tools** | **Pulumi MCP server**, **Anthropic Claude Code Action**, **Pulumi ESC**, **Datadog/Grafana MCP**, **Sentry MCP**, **Docker Gordon** |
| **Output** | Every developer's Claude Code can read Pulumi state, every PR is reviewed by Claude Code Action, ESC manages all CI secrets, observability MCP servers connected for incident workflows |
| **Human** | DevOps Lead authorises org-level connectors; each developer authenticates once via OAuth |

This step is done **once per project**. The cost of skipping it is paying the "AI tax" by hand on every story for the rest of the phase.

#### 0.1 — Connect Claude Code to Pulumi MCP

The Pulumi MCP server is the bridge between Claude Code (or Cursor) and Pulumi Cloud — it exposes `get-stacks`, `resource-search` (Lucene), `pulumi preview`, `pulumi up`, `pulumi stack output`, and Pulumi Insights queries to the LLM session.

Two installation paths — pick the **remote MCP** path for organisations standardising on a single Pulumi Cloud workspace; pick the **local MCP** path for developer machines that need to work offline or against self-managed state.

```bash
# Path A — remote (recommended): hosted MCP, OAuth, no Node.js required
claude mcp add --transport http --scope user pulumi https://mcp.ai.pulumi.com/mcp
# Then: claude → /mcp → pulumi → approve OAuth → enter Pulumi Access Token + select org

# Path B — local: npm package, stdio transport
claude mcp add --scope user pulumi -- npx -y @pulumi/mcp-server@latest
# Auth uses the local PULUMI_ACCESS_TOKEN environment variable
```

Verify with `claude mcp list` — `pulumi: connected` must appear. Smoke test: `Via Pulumi MCP, list_stacks for org acme and run resource-search for type:aws:s3/bucket where tag:Environment=prod.` Claude Code should return real stacks and matching resources.

#### 0.2 — Install the Anthropic Claude Code Action in the repo

The official action (`anthropics/claude-code-action@v1`, GA in 2026) runs the Claude Code runtime inside a GitHub Actions runner. It auto-detects mode based on workflow context — `@claude` mention in a PR comment triggers fix-mode; `pull_request: opened` trigger runs review-mode; explicit prompts run automation.

```bash
# From any developer's terminal, in the repo root
claude  # opens Claude Code
> /install-github-app
```

The slash command provisions: a fine-grained PAT or GitHub App, the `ANTHROPIC_API_KEY` repository secret (or `AWS_BEDROCK_*` / `GCP_VERTEX_*` / `AZURE_FOUNDRY_*` if using a cloud provider's Claude endpoint), and a starter `.github/workflows/claude.yml`. Commit the workflow; the team gets `@claude` review on every PR going forward.

#### 0.3 — Configure GitHub Copilot Agentic Workflows for repository automation

GitHub Agentic Workflows (technical preview, 2026) are Markdown-authored automations that run as standard GitHub Actions with sandboxed permissions and explicit safe-output approvals (create-PR, add-comment). They can use **Copilot CLI, Claude Code, or OpenAI Codex** as the agent engine — pick Claude Code to keep the Phase 3 / Phase 6 toolchain consistent.

Author the agentic workflows under `.github/agentic-workflows/` (Markdown) and pin the agent runtime in `.github/workflows/copilot-setup-steps.yml`. See [PROMPTS.md → agentic-workflow-templates](./PROMPTS.md#agentic-workflow-templates) for the starter set the team should commit (triage stale issues, generate release notes, repair flaky test, fix lint debt).

#### 0.4 — Set up Pulumi ESC for CI/CD secrets

Pulumi ESC (Environments, Secrets, Configuration) is the canonical place for every secret that touches the pipeline — cloud credentials, Anthropic API keys, third-party API tokens, database URLs. ESC pulls from upstream secret stores (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, 1Password) and exposes them with versioned, immutable revisions and full audit logs.

```bash
pulumi env init acme/prod-ci
pulumi env set    acme/prod-ci aws.AWS_ACCESS_KEY_ID --secret <value>
pulumi env open   acme/prod-ci         # verify resolved values
```

Wire ESC into GitHub Actions via the `pulumi/esc-action@v1` step or the OIDC-based `pulumi/auth-action`. **Never commit raw secrets to GitHub Actions secrets directly** — ESC is the source of truth so rotation, scoping, and audit happen in one place.

#### 0.5 — Connect observability and error-tracking MCP servers

```bash
# Grafana Cloud (Grafana Assistant + Sift via MCP)
claude mcp add --transport http --scope user grafana https://<workspace>.grafana.net/api/mcp

# Sentry MCP server (for Seer-driven local debugging + PR review)
claude mcp add --transport http --scope user sentry https://mcp.sentry.dev/mcp

# Datadog (via Datadog MCP)
claude mcp add --transport http --scope user datadog https://mcp.datadoghq.com/mcp
```

Smoke test each: `Via Sentry MCP, list issues in project web-frontend ranked by event-count last 24h, then ask Seer for a root-cause analysis on the top one.`

#### 0.6 — Enable Docker Gordon and Trivy in developer environments

Docker Desktop 4.61+ ships **Gordon** (the AI agent formerly known as Ask Gordon). Gordon runs as a CLI bundled with Docker Desktop and a chat panel in the docs site. Enable it under **Docker Desktop → Settings → Beta features → Docker AI**. Validate with `docker ai "rate my Dockerfile"` from a project root — Gordon should propose multi-stage refactors, base-image hardening, and a `.dockerignore`.

Add Trivy to every container build step in CI (Phase 5 already covers this) and turn on Docker Scout in Docker Hub / GHCR for AI-assisted CVE remediation suggestions on the registry side.

#### 0.7 — Author an `AGENTS.md` at the repo root

Pulumi Neo (and most modern AI coding agents — Claude Code, Cursor, Copilot agent) read [AGENTS.md](https://agents.md) as the canonical project context file. Capture: stack naming convention, region defaults, mandatory tags (`Project`, `Environment`, `Owner`, `CostCenter`), forbidden resource types, ADR pointers, and the production-deploy approval requirement. Without this, every prompt drifts; with it, the agent fences itself within team standards automatically. Use the [`agents-md-authoring`](./PROMPTS.md#agentsmd-authoring) prompt for the first draft, then have the platform lead review and commit.

#### Verification checklist

- [ ] `claude mcp list` shows `pulumi`, `grafana` (or `datadog`), `sentry`, `github` all connected for every developer
- [ ] `.github/workflows/claude.yml` committed and `@claude` test mention on a draft PR returns a review within ~3 minutes
- [ ] `.github/agentic-workflows/` contains at least one starter workflow that runs successfully in dry-run
- [ ] `pulumi env open acme/<env>-ci` resolves all CI secrets without manual values in GitHub Actions
- [ ] `docker ai "rate my Dockerfile"` runs and returns recommendations
- [ ] `AGENTS.md` committed at repo root and referenced in `pulumi.yaml` runtime options
- [ ] Pulumi Cloud audit log enabled (Settings → Security → Audit log)

> **Permission inheritance.** Every MCP-driven action runs as the connecting developer — the MCP server cannot escalate beyond their Pulumi / GitHub / Datadog scopes. Off-board by revoking the OAuth grant in each provider; the agent loses access automatically.

---

### Step 1: Infrastructure Tech Stack Selection

> Visual: [Step 1 flowchart](./FLOWCHART.md#step-1-tech-stack)

| Attribute | Detail |
|-----------|--------|
| **Input** | Architecture proposal (Phase 2), NFRs, budget, team expertise, existing cloud accounts, compliance scope |
| **Tools** | **Pulumi Copilot** (multi-cloud comparison, ESC-aware), **Claude Code** (trade-off analysis + ADR drafting) |
| **Output** | Tech stack decisions documented as ADRs, plus a `pulumi.yaml` skeleton committed to the repo |
| **Human** | Final calls on cloud provider, IaC language, runtime, observability stack |

**Workflow:**

**1.1 — Cloud provider trade-off.** From Claude Code, run the [`cloud-provider-comparison`](./PROMPTS.md#cloud-provider-comparison) prompt. Feed: NFRs, expected scale (launch + 12 months), regions, compliance scope, team's existing AWS/GCP/Azure footprint. Compare AWS / GCP / Azure / Cloudflare / Vercel / Fly.io across feature fit, 12-month cost, ecosystem maturity, lock-in risk. **Don't auto-pick AWS** — for many MVPs a managed PaaS (Vercel, Fly.io, Render, Railway, Cloud Run) is cheaper, simpler, and avoids the Phase 6 ops weight entirely.

**1.2 — IaC language decision.** Pulumi is the recommended default in 2026 for three reasons: (a) general-purpose languages (TypeScript / Python / Go / .NET / Java) eliminate HCL learning curve and unlock real testing frameworks (pytest, Jest, Go test), (b) Apache 2.0 licensing avoids the Terraform BSL question entirely, (c) Pulumi Neo + Copilot + the MCP server give the deepest AI integration of any IaC tool today.

Pick **TypeScript** for full-stack teams (best ecosystem, best AI completion in 2026), **Python** for data/ML teams, **Go** when the rest of the platform is Go. **Stay on Terraform/OpenTofu only when:** (i) HCL skills dominate the team and the time-to-productivity matters more than language ergonomics, (ii) a critical provider is Terraform-only and not yet on Pulumi's 150+ provider list, or (iii) compliance dictates HCL/OpenTofu specifically. Document the call as an ADR.

**1.3 — Orchestration decision.** Kubernetes only when you genuinely need it (multi-team platform, complex microservices, on-prem portability). Otherwise: managed PaaS (ECS Fargate, Cloud Run, App Runner, Container Apps, Vercel, Fly.io) is simpler, cheaper, and one fewer thing for AI to misconfigure. If K8s, plan for K8sGPT (in-cluster operator) from day one — it pays back in incident response.

**1.4 — Observability platform decision.** Two viable defaults: **Datadog + Bits AI SRE** (best autonomous incident triage, 90% faster restoration in Datadog's published benchmarks; cost scales with hosts/usage — model the bill carefully) or **Grafana Cloud + Grafana Assistant + Sift** (free tier is generous, Grafana Assistant went free in April 2026, OSS-friendly, full data ownership when self-hosted). Sentry + Seer handles application errors in either stack.

**1.5 — Document each decision as an ADR.** Use [`/templates/adr-template.md`](../../templates/adr-template.md). One ADR per pillar (cloud provider, IaC tool & language, orchestration, observability, secrets, deployment strategy). Cite the Phase 1 NFR ID each decision satisfies.

**Escalation:** If the stack requires technology nobody on the team has touched, time-box a one-week spike before committing. Pulumi Neo's autonomous mode shortens spikes — point it at the proposed architecture and let it generate a working stack to review.

---

### Step 2: Cost Analysis & FinOps

> Visual: [Step 2 flowchart](./FLOWCHART.md#step-2-cost)

| Attribute | Detail |
|-----------|--------|
| **Input** | Architecture (Phase 2), expected traffic from NFRs, IaC code (after Step 3) |
| **Tools** | **Pulumi Insights** (cross-stack resource search + spend), **Claude Code** (estimate generation, optimisation review), **Infracost** (Terraform/OpenTofu fallback only), **CAST AI / Kubecost** (K8s only) |
| **Output** | Monthly cost estimate at launch + projected at 12 months; PR comments on every IaC change with delta cost |
| **Human** | Validate against budget, sign off on cost-impacting PRs |

**Workflow:**

**2.1 — Initial cost model.** Run the [`cost-estimation`](./PROMPTS.md#cost-estimation) prompt in Claude Code. Feed architecture overview, expected traffic (p50 + p99 RPS), data volumes, regions. Output: cost per service, three scenarios (low / expected / peak), top three cost drivers, hidden-cost callouts (egress, NAT gateway, cross-AZ traffic, observability ingest).

**2.2 — Cost-on-PR — Pulumi vs Terraform paths.**
- **If the IaC is Pulumi:** Use **Pulumi Insights** + a Claude Code GitHub Action that runs the [`pulumi-cost-delta`](./PROMPTS.md#pulumi-cost-delta) prompt against `pulumi preview --json`. Claude reads the diff, estimates the monthly delta against the Pulumi Insights resource catalogue, and posts it as a PR comment. **Infracost does NOT support Pulumi natively as of April 2026** ([infracost/infracost#187](https://github.com/infracost/infracost/issues/187) is still open) — don't try to wedge it in.
- **If the IaC is Terraform/OpenTofu:** Run `infracost diff` in CI with the `infracost/actions/comment` step. Turn on **Infracost AutoFix** so Infracost opens AI-generated cost-reduction PRs (10× faster remediation per Infracost's 2026 benchmarks).

**2.3 — Quarterly optimisation review.** Run the [`cost-optimisation`](./PROMPTS.md#cost-optimisation) prompt with a paste of the top 50 resources by spend (from cloud billing API). Claude returns ranked opportunities by savings/effort ratio across right-sizing, reserved instances/Savings Plans, spot, storage tiering, egress reduction, auto-scaling tuning, and unused-resource cleanup.

**2.4 — Kubernetes-specific cost (if applicable).** Wire **CAST AI** for autonomous bin-packing, spot-orchestration, and right-sizing — savings-based pricing means it pays for itself. **Kubecost** is the OSS fallback for cost allocation and showback when the org isn't ready for an external SaaS.

**2.5 — Budget alerts.** Configure cloud-native budget alerts (AWS Budgets / GCP Budgets / Azure Cost Management) at 50% / 80% / 100% of monthly target. Route to the team's incident channel — surprise bills are incidents.

**2.6 — Track unit economics.** Cost per active user / cost per request / cost per inference (if AI features ship). Wire to Grafana or Datadog as a dashboard; review monthly. This is the metric that survives investor and board conversations.

---

### Step 3: Infrastructure Provisioning (IaC) with Pulumi AI

> Visual: [Step 3 flowchart](./FLOWCHART.md#step-3-iac)

| Attribute | Detail |
|-----------|--------|
| **Input** | Approved architecture (Phase 2), tech stack ADRs (Step 1), `AGENTS.md`, NFRs |
| **Tools** | **Pulumi Neo** (autonomous infra agent, primary) + **Pulumi Copilot** (in Pulumi Cloud) + **Claude Code with Pulumi MCP** (terminal authoring) + **Pulumi ESC** (secrets) + **Pulumi CrossGuard** (policy-as-code, OPA + AI-fix) |
| **Output** | Complete IaC repo provisioning all environments reproducibly, policy gates green, secrets resolved through ESC |
| **Human** | Review every Neo / Claude-generated change — "review-and-merge" is the Pulumi Neo operating mode for production stacks |

**Workflow:**

**3.1 — Bootstrap the Pulumi project.** From the repo root:

```bash
pulumi new <typescript|python|go|csharp> --name <project> --description "<desc>"
pulumi stack init dev   # repeat for staging, prod
pulumi config env add prod acme/prod-ci   # link the ESC environment from Step 0.4
```

Commit the generated `pulumi.yaml`, `Pulumi.<stack>.yaml`, and the language scaffolding. Add `AGENTS.md` reference to the project description so Neo and Claude Code pick it up automatically.

**3.2 — Generate the initial stack with Pulumi Neo (autonomous mode) OR Claude Code + Pulumi MCP (review mode).**

- **Pulumi Neo path (Enterprise / Business Critical):** From Pulumi Cloud, assign a Pulumi issue (or Linear issue if connected) directly to Neo with `Generate the initial VPC + RDS + ECS stack for the architecture described in /docs/architecture.md, targeting AWS us-east-1, with all resources tagged per AGENTS.md.` Neo plans, generates the code, runs `pulumi preview`, and opens a PR with the diff and the preview output. **Pulumi Neo defaults to review-mode** — no apply happens until a human approves the PR.
- **Claude Code path (everyone, primary path for non-Neo tiers):** From Claude Code, run the [`pulumi-iac-generation`](./PROMPTS.md#pulumi-iac-generation) prompt. Feed architecture, NFRs, target cloud, and `AGENTS.md`. Claude generates the language code, structures it into modules (`/components/vpc`, `/components/database`, `/components/service`), and runs `pulumi preview` via the MCP server to validate.

**3.3 — Structure for environments.** One Pulumi project, one stack per environment (`dev`, `staging`, `prod`), shared component resources, environment-specific config in `Pulumi.<stack>.yaml` (referencing ESC environments). Component resources are reusable patterns — VPC, database, service, queue — tested with the language-native test framework (Jest / pytest / Go test). **This is where Pulumi pulls ahead of Terraform** — proper unit tests on infra, no Terratest gymnastics.

**3.4 — Remote state (Pulumi Cloud vs self-managed).** Default is **Pulumi Cloud** state — encrypted, versioned, with built-in locking, concurrent-edit detection, and Pulumi Insights search. For data-residency-constrained deployments use the self-managed backend (S3 / Azure Blob / GCS). **Never commit state to git.**

**3.5 — Policy as code (Pulumi CrossGuard + AI fixes).** Author CrossGuard policy packs in TypeScript/Python (or OPA Rego via the `pulumi-policy-opa` bridge) for: required tags, encryption-at-rest, no-public-S3, IAM least-privilege, region restrictions, instance-size caps. CrossGuard runs on every `pulumi preview` and gates `pulumi up`. **When a policy fails in Pulumi Cloud, the 2026 CrossGuard surfaces an AI-generated patch** — review the diff and merge.

**3.6 — Apply only via CI / Pulumi Deployments.** No human runs `pulumi up` from a laptop against staging or prod. Two patterns:

- **Pulumi Deployments (recommended):** Pulumi Cloud-managed runner. Drift detection runs on a schedule, scheduled deploys handle TTL stacks, the CI/CD integration assistant generates the GitHub Actions workflow for you. This is the lowest-ops path.
- **Self-hosted runner in GitHub Actions:** Use the `pulumi/actions@v6` step with OIDC-based cloud auth (no long-lived keys) and ESC for everything else. Suitable when there's a hard requirement that builds run inside the org's network.

**3.7 — Drift detection and remediation.** Turn on Pulumi Cloud's drift detection (or run `pulumi refresh --diff` on a daily schedule). Drift in production is an incident — page on it; **don't auto-remediate prod drift without a human**. Auto-remediation is fine on dev and ephemeral PR stacks.

**3.8 — Document the runbook.** `/docs/infrastructure.md` covers: how to provision from scratch, how to make changes (PR flow), how to recover from corrupted state, how to handle drift, how to rotate ESC secrets. Generate the first draft with [`infra-runbook`](./PROMPTS.md#infra-runbook) prompt; an SRE reviews and signs off.

> **Gate 1 — Infrastructure foundation** must pass before any workload deploys. See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-infrastructure-foundation).

---

### Step 4: CI/CD Pipeline Setup

> Visual: [Step 4 flowchart](./FLOWCHART.md#step-4-pipeline)

| Attribute | Detail |
|-----------|--------|
| **Input** | Repo, test suite (Phase 4), security tools (Phase 5), IaC (Step 3), `AGENTS.md` |
| **Tools** | **GitHub Actions** + **GitHub Copilot** (workflow generation) + **Claude Code Action** (PR review, fix-on-mention) + **Copilot Agentic Workflows** (autonomous repository tasks); **GitLab CI + GitLab Duo** as fallback |
| **Output** | Fully automated PR-to-prod pipeline with AI-generated YAML, AI PR review, AI-driven failure RCA, AI-fixable lint/test debt |
| **Human** | Review pipeline for least-privilege, secrets handling, manual approval gates |

**Workflow:**

**4.1 — Generate pipeline configs with Copilot + Claude Code.** Open the repo in Cursor or VS Code with GitHub Copilot, run the [`pipeline-generation`](./PROMPTS.md#cicd-pipeline-generation) prompt against the tech-stack ADR. Copilot scaffolds the YAML; Claude Code (via terminal) refines edge cases (matrix builds, reusable workflows, OIDC). Standard stages, in order:

1. **Lint & format** — fast feedback < 2 min, fail closed.
2. **Build** — compile, build container image with multi-stage Dockerfile from Step 5.
3. **Unit + integration tests** (Phase 4) with coverage gate.
4. **Security scans** — SonarQube + Semgrep + Trivy + ggshield (Phase 5) + CrossGuard for IaC.
5. **Pulumi preview** — `pulumi preview` against the target stack with cost-delta comment (Step 2.2).
6. **Deploy to staging** — auto on merge to `main` via Pulumi Deployments or `pulumi/actions@v6`.
7. **E2E tests against staging** — Playwright suite from Phase 4.
8. **Load test on release candidate** — k6 against staging, only on tagged RC builds.
9. **Deploy to production** — manual approval gate, blue/green or canary.
10. **Post-deploy verification** — smoke tests + Grafana / Datadog deploy annotation.

**4.2 — Wire the Anthropic Claude Code Action into the PR workflow.** Generate `.github/workflows/claude.yml` with the [`claude-code-action--pr-review-workflow`](./PROMPTS.md#claude-code-action--pr-review-workflow) prompt. Two trigger patterns committed in that file:

- **Auto-review on PR open** — Claude Code reads the diff + linked Linear issue + ADRs + relevant tests, and posts a review with the same severity buckets (Critical / High / Medium / Low) as the Phase 3 self-review prompt.
- **`@claude` mention for fixes** — a developer types `@claude please fix the failing TypeScript build` on a PR comment; Claude opens a follow-up commit on the same branch with the fix and re-runs CI.

Pin the version (`anthropics/claude-code-action@v1`), use the smallest credential scope possible (a fine-grained PAT or GitHub App with read/write only on the relevant repos), and gate the action behind `if: github.actor != 'dependabot[bot]'` so it doesn't burn API tokens on bot PRs.

**4.3 — Add Copilot Agentic Workflows for repository hygiene.** In `.github/agentic-workflows/` commit Markdown agents for: triaging stale issues, generating release notes from merged PRs, repairing the most-flaky test of the week, fixing top-N lint debt. Each is a Markdown file declaring trigger, agent engine (set to Claude Code), and safe outputs (create PR, add comment) — no write access without an explicit safe-output declaration.

**4.4 — GitLab Duo Root Cause Analysis (fallback platform).** If the org is on GitLab, enable Duo Root Cause Analysis on the Premium tier — it summarises failed CI logs, identifies the likely cause (syntax / compile / Docker build), and proposes a fix patch. Self-Hosted Duo supports Anthropic / Mistral / OpenAI model families — use Anthropic Claude for tool consistency with Phase 3.

**4.5 — Secrets in CI.** Pulumi ESC is the source of truth (see Step 0.4). For platform-native secrets that *must* live in GitHub (e.g., the Pulumi access token bootstrap) use GitHub-encrypted secrets with environment scoping (`production` environment requires reviewer approval before secrets are released to a job).

**4.6 — Branch protection.** `main` requires: PR + passing CI + Claude Code Action review acknowledged + 1 human approval + Linear issue identifier in the title (Phase 3 convention). High-blast-radius changes (auth, schema migrations, IAM) require **2** humans.

> **Gate 2 — CI/CD pipeline operational** must pass before the team relies on automation for deploys. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-cicd-pipeline-operational).

---

### Step 5: Containerization

> Visual: [Step 5 flowchart](./FLOWCHART.md#step-5-containers)

| Attribute | Detail |
|-----------|--------|
| **Input** | Application code, runtime dependencies, target runtime (ECS / Cloud Run / K8s) |
| **Tools** | **Docker Gordon** (Dockerfile rate + propose, in Docker Desktop 4.61+) + **Claude Code** (multi-file Dockerfile + K8s manifest gen) + **GitHub Copilot** (inline) + **K8sGPT** (cluster diagnostics, in-cluster operator) + **Trivy** + **Docker Scout** |
| **Output** | Multi-stage Dockerfiles, K8s manifests / Helm charts (when needed), 0 Critical CVEs at registry push |
| **Human** | Review every Dockerfile for base-image safety, non-root user, layer ordering |

**Workflow:**

**5.1 — Generate Dockerfiles.** From the project root:

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

**5.4 — Image scanning is non-negotiable.**
- **Trivy** in CI (from Phase 5): fails on Critical CVEs.
- **Docker Scout** on the registry: AI-suggested base-image upgrades that fix the most CVEs with the smallest upgrade — review and merge.

**5.5 — Kubernetes manifests (only if K8s).** Use Claude Code with the [`k8s-manifests-generation`](./PROMPTS.md#kubernetes-manifests-generation) prompt for Deployment + Service + Ingress + ConfigMap + Secret (via External Secrets Operator) + HPA + PDB + NetworkPolicy. **K8sGPT operator** in-cluster catches misconfigurations the manifests passed but the cluster doesn't tolerate (e.g., missing PDBs on autoscaled deployments, ingress class mismatches). K8sGPT also exposes an MCP server mode — connect it to Claude Code for in-session troubleshooting.

**5.6 — Helm or Kustomize for environment overlays.** Pick one, document the choice as an ADR. Claude Code generates either; consistency matters more than the choice.

> **Gate 3 — Containerization standards** must pass for every service container. See [QUALITY-GATES.md → Gate 3](./QUALITY-GATES.md#gate-3-containerization-standards).

---

### Step 6: Observability — Logging, Monitoring, Alerting

> Visual: [Step 6 flowchart](./FLOWCHART.md#step-6-observability)

| Attribute | Detail |
|-----------|--------|
| **Input** | Running application + infrastructure, NFRs from Phase 1 (latency / availability / error rate targets) |
| **Tools** | **OpenTelemetry SDK** (instrumentation, vendor-neutral); **Datadog + Bits AI SRE** *or* **Grafana Cloud + Grafana Assistant + Sift**; **Sentry + Seer** (errors); **Claude Code with observability MCP servers** (incident assistant) |
| **Output** | Production dashboards, SLOs with burn-rate alerts, runbooks per alert, AI-driven first-responder triage |
| **Human** | Define SLOs from NFRs; sign off on alert rules; review autonomous remediation suggestions |

**Workflow:**

**6.1 — Instrument with OpenTelemetry.** OTEL is the vendor-neutral floor — auto-instrumentation now covers Java, Node, Python, Go, .NET out of the box. Emit traces + metrics + structured JSON logs with correlation IDs. **Don't lock into a vendor SDK** — OTEL means switching from Datadog to Grafana (or vice versa) is a config change, not a rewrite.

**6.2 — Pick the observability backend.**
- **Datadog + Bits AI SRE** — picks up alerts the moment they fire, walks the telemetry, runbooks, dependency graph, and recent deploys, and posts a hypothesis with a confidence score to Slack/Teams before the on-call has logged in. GA Dec 2025; Q1 2026 update is ~2× faster and supports HIPAA + RBAC. Cost is the catch — model the bill at expected host count + ingest rate before committing.
- **Grafana Cloud + Grafana Assistant + Sift** — Grafana Assistant became free in April 2026, generates dashboards from connected data sources, suggests PromQL/LogQL, runs Sift investigations on incidents (Asserts.ai-powered RCA, surfaced in the Incident workflow). Cheaper and more portable; the trade-off is that autonomous incident response is less mature than Bits AI SRE.

**6.3 — Generate dashboards via AI.** Don't hand-author Grafana JSON. From Claude Code, run the [`dashboard-generation`](./PROMPTS.md#observability--dashboard-generation) prompt against the OpenAPI spec + the SLO targets + the architecture; it returns Grafana JSON or Datadog dashboard JSON ready to commit. Store dashboards as code in `/observability/dashboards/`.

**6.4 — Define SLOs from NFRs.** Each Phase-1 NFR becomes one SLO — `p95 API latency < 200ms`, `99.9% availability`, `error rate < 0.1%`. Configure burn-rate alerts (fast burn 2% in 1h → page; slow burn 10% in 6h → ticket). Use [`slo-and-alert-generation`](./PROMPTS.md#slo-and-alert-generation) prompt — Claude Code drafts both Prometheus AlertManager rules and the matching Grafana / Datadog alert config.

**6.5 — Connect Sentry + Seer for application errors.** Sentry captures unhandled exceptions and slow transactions; Seer correlates events with deployed code changes and proposes a patch. Wire the **Sentry MCP server** into Claude Code so developers can ask `Why did SignupForm error spike at 10:42 UTC?` and get an answer + suggested fix in the same session.

**6.6 — Runbook per alert.** Every alert links to `/docs/runbooks/<alert-name>.md`. Use [`runbook-generation`](./PROMPTS.md#runbook-generation-from-alert) — Claude Code drafts diagnosis steps, remediation, escalation. **AI runbooks are starting points, not final** — an SRE reviews each before it's wired to the alert.

**6.7 — Alerting discipline.** If it doesn't require action, it's not an alert. Bits AI SRE / Sift filter noise but they can't fix bad alert design — review fired alerts every retro for "did this require human action? if no, mute or upgrade to a metric."

> **Gate 4 — Observability & alerting** must pass before production traffic reaches the service. See [QUALITY-GATES.md → Gate 4](./QUALITY-GATES.md#gate-4-observability--alerting).

---

### Step 7: Deployment Automation & Rollback

> Visual: [Step 7 flowchart](./FLOWCHART.md#step-7-deploy)

| Attribute | Detail |
|-----------|--------|
| **Input** | Approved release candidate (Phases 4–5), green Gates 1–4 |
| **Tools** | **Pulumi Deployments** (managed runner) + **GitHub Actions** (CI orchestration) + **Datadog / Grafana deploy markers** + **Bits AI SRE / Sift** (post-deploy watch) |
| **Output** | Zero-downtime deployments with automatic rollback on SLO breach |
| **Human** | Approve production deploys; manage incidents |

**Workflow:**

**7.1 — Auto-deploy to staging.** Every merge to `main` deploys to staging with no human in the loop. Pulumi Deployments handles the run; the same workflow that ran preview now runs `up` against the staging stack.

**7.2 — Manual approval gate for production.** GitHub Environment `production` has required reviewers — a human clicks "deploy" or schedules to a release window. The approval is the human checkpoint; everything before and after is automated.

**7.3 — Deployment strategy** (decided in Step 1 ADR):
- **Blue/Green** — provision the new environment, switch traffic, keep old for instant rollback. Best for fast rollback at the cost of double resources during deploy.
- **Canary** — small % of traffic to the new version, watch SLOs, gradually increase. Best for high-traffic services where bad changes only show under real load.
- **Rolling** — instances replaced one at a time. Simplest; no instant rollback.

**7.4 — Health checks and post-deploy smoke tests gate full rollout.** New version must pass `/healthz` (liveness), `/readyz` (readiness), and the Phase-4 Playwright smoke suite before traffic fully routes. Failure here triggers automatic rollback.

**7.5 — Automatic rollback on SLO breach.** Wire Datadog / Grafana to GitHub Actions: if error rate or p95 latency exceeds threshold within 10 minutes of a deploy, the watcher fires a `repository_dispatch` event that triggers `pulumi/actions@v6` to roll back to the previous stack revision.

**7.6 — Deploy markers on dashboards.** Every deploy posts an annotation to Datadog / Grafana — incident correlation collapses from minutes to seconds. Bits AI SRE / Sift use these markers when proposing root-cause hypotheses.

**7.7 — Quarterly rollback drill.** Schedule a deliberate rollback against a staging release candidate every quarter. If rollback takes more than 5 minutes, the runbook is wrong. Treat the drill as a Phase 4 gate.

> **Gate 5 — Deployment & rollback** must pass for every production deploy. See [QUALITY-GATES.md → Gate 5](./QUALITY-GATES.md#gate-5-deployment--rollback).

---

## Phase Handoff

When CI/CD & DevOps is complete, the following artefacts hand off to **Phase 7: Delivery & Handoff**:

| Artefact | Format | Location |
|----------|--------|----------|
| Pulumi IaC repository | TS/Python/Go/.NET project | Git repo with `pulumi.yaml`, component modules, tests |
| Pulumi ESC environments | YAML in Pulumi Cloud | `acme/<env>-ci` per environment |
| CrossGuard policy packs | TS/Python or Rego | `/policy-packs/` in Git |
| CI/CD workflows | YAML + Markdown agentic workflows | `.github/workflows/` + `.github/agentic-workflows/` |
| Claude Code Action config | YAML | `.github/workflows/claude.yml` |
| Dockerfiles + .dockerignore | Per service | Service directories |
| K8s manifests / Helm charts (if K8s) | YAML / Helm | `/deploy/` |
| Observability dashboards | Datadog / Grafana JSON | `/observability/dashboards/` |
| SLO + alert definitions | YAML / Datadog config | `/observability/slos/` |
| Runbooks per alert | Markdown | `/docs/runbooks/` |
| Infrastructure runbook | Markdown | `/docs/infrastructure.md` |
| Deployment runbook | Markdown | `/docs/deployment.md` |
| Cost analysis report | Markdown + Pulumi Insights export | `/docs/cost-analysis.md` |
| AGENTS.md | Markdown | Repo root |

**Handoff Checklist:**

- [ ] All five gates (1–5) passed
- [ ] Pulumi stacks reproduce all environments from a clean clone in < 30 minutes
- [ ] CI/CD pipeline auto-deploys PR → staging without manual steps
- [ ] Production deploy works zero-downtime; rollback drill passes < 5 min
- [ ] All container images scan clean (0 Critical CVEs at registry push)
- [ ] Observability covers application + infrastructure end-to-end
- [ ] SLOs defined and being measured against NFRs from Phase 1
- [ ] Every alert is actionable and has a linked runbook
- [ ] Cost estimate validated and within budget; cost-on-PR active
- [ ] All secrets resolved through Pulumi ESC; no hardcoded credentials anywhere
- [ ] Pulumi Cloud audit log enabled; GitHub audit log enabled
- [ ] Claude Code Action posts a review on every PR; `@claude` fix workflow tested
- [ ] Bits AI SRE / Grafana Sift configured with on-call rotation
- [ ] AGENTS.md committed and referenced by Pulumi Neo, Claude Code, and Cursor

---

## Risks & Guardrails

| Risk | Mitigation |
|------|------------|
| **AI-generated IaC introduces production-breaking config** (over-permissive IAM, public S3, missing encryption) | CrossGuard policy gates on every preview; AI-generated CrossGuard fixes still require human approval; Pulumi Neo runs in review-mode by default for prod stacks. |
| **Pulumi access token leak** in a shared MCP session | ESC stores the token; OAuth-based remote MCP server has token-less per-developer auth. Rotate tokens via ESC; revoke OAuth grants for off-boarded staff. |
| **Infracost gap on Pulumi cost-on-PR** ([infracost/infracost#187](https://github.com/infracost/infracost/issues/187) open as of 2026) | Pulumi Insights + the `pulumi-cost-delta` Claude Code action covers the gap. Watch the Infracost roadmap; switch when native Pulumi support ships. |
| **Claude Code Action token cost runs away** on bot-driven PRs | Gate the action behind `if: github.actor != 'dependabot[bot]'` and similar; cap monthly Anthropic API spend; route to Bedrock / Vertex if cloud commits are already in place. |
| **Bits AI SRE / Sift confidence-score over-trust** — engineers stop validating | First responder still owns the incident. AI suggestion is a hypothesis; the runbook is the source of truth. Track AI-suggested-vs-actual-cause variance in retros. |
| **Drift detection auto-remediates production** | Auto-remediate only on ephemeral / dev stacks; prod drift pages on-call. Document the policy in `/docs/infrastructure.md`. |
| **MCP server scope creep** — granting an agent more cloud rights than needed | Each MCP connection inherits the connecting developer's scopes — never widen org-level Pulumi RBAC just because Neo asks; gate state-changing scopes (`pulumi up`) behind review-mode. |
| **Pulumi Neo "autonomous mode" applied to production by mistake** | Default operating mode for prod stacks is review-only; configure org-wide policy to require approval for any `pulumi up` affecting a stack tagged `Environment=prod`. |
| **AI-generated Dockerfiles use vulnerable bases** | Trivy + Docker Scout on every build; CrossGuard-equivalent policy on container images (digest-pinned bases only); fail closed on Critical CVEs. |
| **Rollback drift over time** — the rollback path stops working because nobody tested it | Quarterly rollback drill; if the drill takes > 5 min, treat it as a Sev-2 retrospective trigger. |

---

## Daily DevOps Workflow (steady state)

```
Morning:
  1. Pull latest main
  2. Claude Code → Pulumi MCP → list_stacks → check drift status on prod / staging
  3. Triage Bits AI SRE / Sift hypotheses from overnight alerts; close noise, file Linear issues for real signal

Pipeline work:
  4. Open PR for IaC or pipeline change
  5. Anthropic Claude Code Action posts review automatically
  6. CrossGuard + Pulumi preview comment on PR (cost delta + policy result)
  7. Address review comments; @claude for routine fixes
  8. Human approves; merge → staging auto-deploys via Pulumi Deployments

Production deploy:
  9. Tag release candidate; pipeline runs full e2e + load
 10. Manual approval gate; deploy to prod (blue/green or canary)
 11. Bits AI SRE / Sift watches first 30 min; auto-rollback if SLO breach
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

- [Pulumi Neo — AI Infrastructure Agent](https://www.pulumi.com/product/neo/) — autonomous infra agent, AGENTS.md support, Custom Instructions, Operating Modes
- [Pulumi Copilot](https://www.pulumi.com/docs/pulumi-cloud/copilot/) — conversational AI in Pulumi Cloud
- [Pulumi MCP Server](https://www.pulumi.com/docs/iac/guides/ai-integration/mcp-server/) — official MCP server, remote (`https://mcp.ai.pulumi.com/mcp`) and npm (`@pulumi/mcp-server`)
- [Pulumi ESC](https://www.pulumi.com/product/esc/) — versioned secrets and configuration management
- [Pulumi Insights](https://www.pulumi.com/product/pulumi-insights/) — cross-stack resource search, compliance, and spend
- [Pulumi Deployments](https://www.pulumi.com/product/pulumi-deployments/) — managed CI/CD runner with drift detection
- [Pulumi CrossGuard](https://www.pulumi.com/crossguard/) — policy-as-code with OPA/Rego support and AI-fix
- [Pulumi vs Terraform / OpenTofu (2026)](https://www.pulumi.com/docs/iac/comparisons/terraform/opentofu/)
- [Anthropic Claude Code GitHub Action](https://github.com/anthropics/claude-code-action) — official `anthropics/claude-code-action@v1`
- [Claude Code GitHub Actions docs](https://code.claude.com/docs/en/github-actions)
- [GitHub Agentic Workflows](https://github.blog/ai-and-ml/automate-repository-tasks-with-github-agentic-workflows/) — Markdown-authored, Actions-executed
- [GitLab Duo Root Cause Analysis](https://docs.gitlab.com/user/gitlab_duo/use_cases/) — fallback CI failure RCA
- [Docker Gordon (AI agent in Docker Desktop)](https://docs.docker.com/ai/gordon/)
- [K8sGPT operator + MCP mode](https://docs.k8sgpt.ai/reference/operator/overview/)
- [Datadog Bits AI SRE](https://www.datadoghq.com/product/ai/bits-ai-sre/) — autonomous incident triage, GA Dec 2025, 2× faster Q1 2026
- [Grafana Assistant + Sift](https://grafana.com/products/cloud/ai-assistant/) — free as of April 2026
- [Sentry Seer + MCP server](https://docs.sentry.io/product/ai-in-sentry/seer/)
- [Spacelift Intelligence (fallback IaC orchestration)](https://spacelift.io/blog/introducing-spacelift-intelligence)
- [Infracost AutoFix](https://www.infracost.io/docs/infracost_cloud/autofix/) — Terraform/OpenTofu only as of 2026
- [AGENTS.md spec](https://agents.md) — open standard for AI coding agent project context
