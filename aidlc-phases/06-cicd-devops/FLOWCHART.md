# Phase 6: CI/CD & DevOps — Process Flowchart

This flowchart visualises the [Phase 6 PROCESS](./PROCESS.md). Each per-step diagram corresponds to a step section; gates map to [QUALITY-GATES.md](./QUALITY-GATES.md). The 🤖 / 👤 markers show which actions are AI-driven and which require a human decision.

## Abbreviations

| Abbreviation | Meaning |
|--------------|---------|
| ADR | Architecture Decision Record |
| BC | Business Critical (Pulumi licensing tier) |
| CC | Claude Code |
| CI / CD | Continuous Integration / Continuous Delivery (or Deployment) |
| CrossGuard | Pulumi's policy-as-code engine |
| CVE | Common Vulnerabilities and Exposures |
| e2e | End-to-end |
| ESC | Pulumi Environments, Secrets and Configuration |
| FinOps | Cloud Financial Operations |
| GA | General Availability |
| GHA | GitHub Action |
| HCL | HashiCorp Configuration Language |
| IaC | Infrastructure as Code |
| K8s | Kubernetes |
| MCP | Model Context Protocol |
| NFR | Non-Functional Requirement |
| OIDC | OpenID Connect |
| OPA | Open Policy Agent |
| OSS | Open Source Software |
| PR | Pull Request |
| RC | Release Candidate |
| RCA | Root Cause Analysis |
| SLO | Service Level Objective |
| SRE | Site Reliability Engineer |
| YAML | YAML Ain't Markup Language |

---

## Step 0: One-Time Setup

```mermaid
flowchart TD
    SETUP_START([One-time setup<br/>per project]) --> MCP_PULUMI

    MCP_PULUMI[Connect Claude Code → Pulumi MCP<br/>🤖 OAuth via mcp.ai.pulumi.com] --> ACTION_INSTALL
    ACTION_INSTALL[Install Anthropic Claude Code Action<br/>🤖 claude /install-github-app] --> AGENTIC
    AGENTIC[Author Copilot Agentic Workflows<br/>👤 .github/agentic-workflows/*.md] --> ESC
    ESC[Configure Pulumi ESC environments<br/>👤 pulumi env init + secret pulls] --> OBS_MCP
    OBS_MCP[Connect Grafana / Datadog / Sentry MCP<br/>🤖 OAuth per developer] --> DOCKER
    DOCKER[Enable Docker Gordon + Trivy<br/>🤖 Docker Desktop 4.61+] --> AGENTS_MD
    AGENTS_MD[Author AGENTS.md at repo root<br/>👤 stack conventions, mandatory tags, ADR refs] --> SETUP_DONE
    SETUP_DONE([Setup complete<br/>verification checklist passes])

    style SETUP_START fill:#1B3A5C,color:#fff
    style SETUP_DONE fill:#1B3A5C,color:#fff
```

---

## End-to-End Phase Flow Overview

High-level chain across all seven steps. Each step has its own detailed flowchart below.

```mermaid
flowchart LR
    START([Phase 6 Input:<br/>Architecture + Code + Tests + AGENTS.md]) --> S1[Step 1<br/>Tech Stack]
    S1 --> S2[Step 2<br/>Cost & FinOps]
    S2 --> S3[Step 3<br/>IaC with Pulumi AI]
    S3 --> G1{Gate 1}
    G1 --> S4[Step 4<br/>CI/CD Pipeline]
    S4 --> G2{Gate 2}
    G2 --> S5[Step 5<br/>Containerization]
    S5 --> G3{Gate 3}
    G3 --> S6[Step 6<br/>Observability]
    S6 --> G4{Gate 4}
    G4 --> S7[Step 7<br/>Deployment]
    S7 --> G5{Gate 5}
    G5 --> HANDOFF([→ Phase 7:<br/>Delivery & Handoff])

    style START fill:#1B3A5C,color:#fff
    style HANDOFF fill:#1B3A5C,color:#fff
    style S1 fill:#f0f7ff,stroke:#2E75B6
    style S2 fill:#f0f7ff,stroke:#2E75B6
    style S3 fill:#e8f4e8,stroke:#2E8B57
    style S4 fill:#e8f4e8,stroke:#2E8B57
    style S5 fill:#e8f4e8,stroke:#2E8B57
    style S6 fill:#fff3e0,stroke:#E65100
    style S7 fill:#fff3e0,stroke:#E65100
```

---

## Step 1: Tech Stack

Cloud + IaC language + orchestration + observability ADRs that gate everything downstream.

```mermaid
flowchart TD
    S1_IN([Architecture + NFRs<br/>+ team expertise + budget]) --> STACK
    STACK[Cloud + IaC language + orchestration choice<br/>🤖 Pulumi Copilot + Claude Code] --> ADR
    ADR[Document as ADRs<br/>👤 Tech Lead] --> S1_OUT([→ Step 2: Cost Analysis])

    style S1_IN fill:#f0f7ff,stroke:#2E75B6
    style S1_OUT fill:#f0f7ff,stroke:#2E75B6
```

---

## Step 2: Cost

Cost-on-PR via Pulumi Insights (or Infracost for HCL paths) + cloud-native budget alerts.

```mermaid
flowchart TD
    S2_IN([Architecture + IaC code]) --> COST
    COST[Initial cost estimate<br/>🤖 Claude Code + cloud calculators] --> COST_PR
    COST_PR{IaC tool?}
    COST_PR -- Pulumi --> PULUMI_INSIGHTS[Pulumi Insights + pulumi-cost-delta<br/>🤖 Claude Code GHA on PR]
    COST_PR -- Terraform/OpenTofu --> INFRACOST[Infracost diff + AutoFix<br/>🤖 PR comment]
    PULUMI_INSIGHTS --> BUDGET
    INFRACOST --> BUDGET
    BUDGET[Cloud-native budget alerts<br/>👤 50% / 80% / 100% thresholds] --> S2_OUT([→ Step 3: IaC])

    style S2_IN fill:#f0f7ff,stroke:#2E75B6
    style S2_OUT fill:#f0f7ff,stroke:#2E75B6
```

---

## Step 3: IaC

Pulumi Neo (autonomous, Enterprise+) or Claude Code + Pulumi MCP (Team tier) generate the stacks; CrossGuard gates; Pulumi Deployments runs them.

```mermaid
flowchart TD
    S3_IN([Approved architecture + tech-stack ADRs]) --> IAC
    IAC[pulumi new + stack init<br/>👤 DevOps Engineer] --> GEN_PATH
    GEN_PATH{IaC generation path?}
    GEN_PATH -- Enterprise / BC --> NEO[Pulumi Neo autonomous PR<br/>🤖 review-mode default]
    GEN_PATH -- Team tier --> CLAUDE_PULUMI[Claude Code + Pulumi MCP<br/>🤖 pulumi-iac-generation prompt]
    NEO --> MODULES
    CLAUDE_PULUMI --> MODULES
    MODULES[Component resources + tests<br/>👤 Jest / pytest / Go test] --> ESC_LINK
    ESC_LINK[Wire ESC to stacks<br/>🤖 pulumi config env add] --> CROSSGUARD
    CROSSGUARD[CrossGuard policy gate on preview<br/>🤖 OPA + AI-suggested fix] --> DEPLOYMENTS
    DEPLOYMENTS[Pulumi Deployments managed runner<br/>🤖 drift detection + scheduled ops] --> GATE1
    GATE1{Gate 1:<br/>Infrastructure<br/>Foundation}
    GATE1 -- No --> IAC
    GATE1 -- Yes --> S3_OUT([→ Step 4: Pipeline])

    style S3_IN fill:#e8f4e8,stroke:#2E8B57
    style S3_OUT fill:#e8f4e8,stroke:#2E8B57
```

---

## Step 4: Pipeline

GitHub Actions + Copilot for YAML, Anthropic Claude Code Action for PR review and `@claude` fix-on-mention, Copilot Agentic Workflows for repo hygiene.

```mermaid
flowchart TD
    S4_IN([Repo + tests + security tools + IaC]) --> PIPELINE
    PIPELINE[Generate workflows<br/>🤖 Copilot + Claude Code] --> CLAUDE_ACTION
    CLAUDE_ACTION[Wire claude.yml — auto-review + @claude fix<br/>🤖 anthropics/claude-code-action@v1] --> AGENTIC_HYGIENE
    AGENTIC_HYGIENE[Repository hygiene agentic workflows<br/>🤖 stale issues / release notes / flaky-test repair] --> SECRETS
    SECRETS[Secrets via Pulumi ESC<br/>👤 OIDC for cloud auth] --> BRANCH
    BRANCH[Branch protection<br/>👤 PR + CI + CC review + 1 human] --> GATE2
    GATE2{Gate 2:<br/>Pipeline<br/>Operational}
    GATE2 -- No --> PIPELINE
    GATE2 -- Yes --> S4_OUT([→ Step 5: Containers])

    style S4_IN fill:#e8f4e8,stroke:#2E8B57
    style S4_OUT fill:#e8f4e8,stroke:#2E8B57
```

---

## Step 5: Containers

Docker Gordon + Claude Code for Dockerfile generation, Trivy + Docker Scout for scanning, K8sGPT for cluster diagnostics.

```mermaid
flowchart TD
    S5_IN([Application code + runtime deps]) --> CONTAINER
    CONTAINER[Generate Dockerfiles<br/>🤖 Docker Gordon + Claude Code] --> MULTISTAGE
    MULTISTAGE[Multi-stage build + distroless + non-root<br/>👤 review] --> SCAN
    SCAN[Trivy in CI + Docker Scout AI remediation<br/>🤖 fail on Critical CVE] --> K8S_OPT
    K8S_OPT{K8s?}
    K8S_OPT -- Yes --> K8S_MANIFESTS[K8s manifests + K8sGPT in-cluster<br/>🤖 Claude Code + operator]
    K8S_OPT -- No --> CONTAINER_DONE[Done]
    K8S_MANIFESTS --> CONTAINER_DONE
    CONTAINER_DONE --> GATE3
    GATE3{Gate 3:<br/>Container<br/>Standards}
    GATE3 -- No --> CONTAINER
    GATE3 -- Yes --> S5_OUT([→ Step 6: Observability])

    style S5_IN fill:#e8f4e8,stroke:#2E8B57
    style S5_OUT fill:#e8f4e8,stroke:#2E8B57
```

---

## Step 6: Observability

OpenTelemetry instrumentation; Datadog + Bits AI SRE *or* Grafana + Assistant + Sift; Sentry + Seer for errors; SLOs and runbooks per alert.

```mermaid
flowchart TD
    S6_IN([Running app + infra + NFR targets]) --> OBS
    OBS[OpenTelemetry instrumentation<br/>🤖 auto-instr libs] --> BACKEND
    BACKEND{Backend?}
    BACKEND -- Commercial --> DD[Datadog + Bits AI SRE<br/>🤖 autonomous alert triage]
    BACKEND -- OSS-leaning --> GRAFANA[Grafana Cloud + Assistant + Sift<br/>🤖 dashboard gen + RCA]
    DD --> SENTRY
    GRAFANA --> SENTRY
    SENTRY[Sentry + Seer + MCP<br/>🤖 errors + AI patch] --> SLOS
    SLOS[Define SLOs from NFRs<br/>👤 latency / availability / error] --> ALERTS
    ALERTS[Burn-rate alerts<br/>🤖 Claude Code generates rules] --> RUNBOOKS
    RUNBOOKS[Runbook per alert<br/>🤖 generate, 👤 SRE review] --> GATE4
    GATE4{Gate 4:<br/>Observability<br/>Ready}
    GATE4 -- No --> OBS
    GATE4 -- Yes --> S6_OUT([→ Step 7: Deploy])

    style S6_IN fill:#fff3e0,stroke:#E65100
    style S6_OUT fill:#fff3e0,stroke:#E65100
```

---

## Step 7: Deploy

Pulumi Deployments for staging auto-deploy, GitHub Environments for prod manual approval, AI-watched 30-min post-deploy window with auto-rollback on SLO breach.

```mermaid
flowchart TD
    S7_IN([Approved release candidate<br/>+ green Gates 1–4]) --> DEPLOY
    DEPLOY[Deploy strategy ADR<br/>👤 blue/green / canary / rolling] --> AUTO_STAGING
    AUTO_STAGING[Auto-deploy to staging on merge<br/>🤖 Pulumi Deployments] --> APPROVAL
    APPROVAL[Manual approval gate for prod<br/>👤 GitHub Environment reviewers] --> HEALTH
    HEALTH[Health checks + smoke tests<br/>🤖 block traffic until green] --> WATCH
    WATCH[Bits AI SRE / Sift watch 30 min<br/>🤖 auto-rollback on SLO breach] --> MARKER
    MARKER[Deploy marker on dashboards<br/>🤖 incident correlation] --> GATE5
    GATE5{Gate 5:<br/>Deployment<br/>Safe}
    GATE5 -- No --> DEPLOY
    GATE5 -- Yes --> HANDOFF([→ Phase 7:<br/>Delivery & Handoff])

    style S7_IN fill:#fff3e0,stroke:#E65100
    style HANDOFF fill:#1B3A5C,color:#fff
```

---

## Step-by-Step Anchors

The PROCESS.md links into these sections by anchor — keep the headings stable.

### Step 1: Tech Stack
The cloud + IaC language + orchestration + observability ADRs that gate everything downstream. See [PROCESS.md → Step 1](./PROCESS.md#step-1-infrastructure-tech-stack-selection).

### Step 2: Cost
Cost-on-PR via Pulumi Insights (or Infracost for HCL paths) + quarterly optimisation review. See [PROCESS.md → Step 2](./PROCESS.md#step-2-cost-analysis--finops).

### Step 3: IaC
Pulumi Neo (autonomous, Enterprise+) or Claude Code + Pulumi MCP (Team tier) generate the stacks; CrossGuard gates; Pulumi Deployments runs them. See [PROCESS.md → Step 3](./PROCESS.md#step-3-infrastructure-provisioning-iac-with-pulumi-ai).

### Step 4: Pipeline
GitHub Actions + Copilot for YAML, Anthropic Claude Code Action for PR review and `@claude` fix-on-mention, Copilot Agentic Workflows for repo hygiene. See [PROCESS.md → Step 4](./PROCESS.md#step-4-cicd-pipeline-setup).

### Step 5: Containers
Docker Gordon + Claude Code for Dockerfile generation, Trivy + Docker Scout for scanning, K8sGPT for cluster diagnostics. See [PROCESS.md → Step 5](./PROCESS.md#step-5-containerization).

### Step 6: Observability
OpenTelemetry instrumentation; Datadog + Bits AI SRE *or* Grafana + Assistant + Sift; Sentry + Seer for errors. See [PROCESS.md → Step 6](./PROCESS.md#step-6-observability--logging-monitoring-alerting).

### Step 7: Deploy
Pulumi Deployments for staging auto-deploy, GitHub Environments for prod manual approval, AI-watched 30-min post-deploy window with auto-rollback on SLO breach. See [PROCESS.md → Step 7](./PROCESS.md#step-7-deployment-automation--rollback).

---

## Key Decision Points

1. **Pulumi Neo vs Claude Code path?** Neo is gated to Enterprise / Business Critical; Team-tier orgs use Claude Code + Pulumi MCP for the same review-mode workflow without the autonomous-task assignment.
2. **Datadog vs Grafana?** Datadog wins on autonomous incident triage today (Bits AI SRE GA, 2× faster in 2026); Grafana wins on cost and OSS portability (Assistant became free in April 2026).
3. **Pulumi vs OpenTofu?** Pulumi is the default in 2026 — real languages, native testing, Apache 2.0, deepest AI integration. Stay on OpenTofu only when HCL skills dominate the team or a critical provider is HCL-only.
4. **Manual approval before prod?** Always. Automate the path, gate the button.
5. **Auto-rollback trigger?** SLO breach within 10 min of deploy fires a `repository_dispatch` to roll back the last `pulumi up`.

---

## The Developer Experience

```
Developer's day:
  PR opened → CI runs (Pulumi preview + cost + tests + scans) →
  Claude Code Action posts review → CrossGuard gates pass →
  Human approval → Merge → Pulumi Deployments → Staging

Release day:
  Tag RC → Full e2e + load tests → Manual approval → Prod deploy (blue/green) →
  Bits AI SRE / Sift watch first 30 min → Deploy marker on dashboards →
  Either celebrate or auto-rollback fires

Incident day:
  Alert fires → Bits AI SRE / Sift posts hypothesis + correlated deploy →
  On-call opens runbook → Claude Code with observability MCP triages →
  Mitigate → Post-mortem within 48h
```
