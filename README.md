# AI Development Life Cycle (AI-DLC)

> A reusable, phase-by-phase methodology for integrating AI tools into every stage of the software development lifecycle.

This is a documented process that defines **how**, **when**, and **which** AI tools to use across the entire SDLC. It is designed so that any development team can adopt it on any project, with clear human responsibilities, AI integration points, quality gates, and escalation rules at every step.

## Covered Phases

| # | Phase | Status | Process Doc |
|---|-------|--------|-------------|
| 1 | **Requirement Gathering** | ✅ Defined | [Process →](./aidlc-phases/01-requirement-gathering/PROCESS.md) |
| 2 | **System Design** | ✅ Defined | [Process →](./aidlc-phases/02-system-design/PROCESS.md) |
| 3 | **Development** | ✅ Defined | [Process →](./aidlc-phases/03-development/PROCESS.md) |
| 4 | **Testing & QA** | ✅ Defined | [Process →](./aidlc-phases/04-testing-and-qa/PROCESS.md) |
| 5 | **Security & Compliance** | ✅ Defined | [Process →](./aidlc-phases/05-security/PROCESS.md) |
| 6 | **CI/CD & DevOps** | ✅ Defined | [Process →](./aidlc-phases/06-cicd-devops/PROCESS.md) |
| 7 | **Delivery & Handoff** | ✅ Defined | [Process →](./aidlc-phases/07-delivery-handoff/PROCESS.md) |

---

## Consolidated Tool Stack

A core design principle of this process is **one tool per job**. We deliberately avoid stacking multiple tools that do the same thing. This reduces cost, lowers the learning curve, and eliminates context-switching between overlapping products.

### Foundation Tools (used across multiple phases)

These are paid once per developer and reused across every phase that needs them. They are explicitly listed under each phase below where they apply, so you never have to cross-reference.

| Tool | Subscription | Cost | Used In |
|------|--------------|------|---------|
| **Claude (Max)** | Claude Max (heavy users) / Pro (light users) | $100/mo Max, $20/mo Pro | Phases 1, 2, 3, 4, 5, 6, 7 |
| **Claude Code** | Included in Claude Max; or API usage | Included in Max | Phases 2, 3, 4, 5, 6, 7 |
| **Cursor** | Cursor Pro | $20/mo | Phases 3, 4 |
| **CodeRabbit** | CodeRabbit Pro | $24/mo | Phases 3, 4, 5 |
| **Linear** | Linear Standard / Plus | $8–14/user/mo | Phases 1, 3, 4, 7 |
| **Fireflies.ai** | Fireflies Pro | $10/mo | Phase 1 |
| **SonarQube** | Community (free) | $0 | Phases 3, 4, 5 |

**Total per developer: ~$165/mo** (Max + Linear Standard) or **~$85/mo** (Pro + Linear Standard) — covering ALL AI reasoning, coding, review, and work tracking across all 7 phases.

> **Linear is the project-management spine of the AI-DLC.** Claude integrates directly with Linear via the official **Linear MCP connector** (`https://mcp.linear.app/mcp`). The PRD itself lives in Linear as a **Linear Document** attached to the Linear Project that owns its work breakdown — Claude publishes the PRD, hosts stakeholder review on the Document, scaffolds Milestones, and pushes stories with clickable deep-links back to the PRD's section anchors. PRD, backlog, gap analysis, bug tracking, and release notes all share one continuous, traceable thread inside Linear instead of scattering across Jira / Notion / Confluence / spreadsheets. See [Phase 1 Step 0 setup →](./aidlc-phases/01-requirement-gathering/PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp).

### Per-Phase Tool Stack

#### Phase 1 — Requirement Gathering

| Tool | Purpose | Subscription / Cost |
|------|---------|---------------------|
| **Claude (Max)** | PRD drafting, user stories, acceptance criteria, gap analysis, scalability/cost assessment — **with Linear connector enabled** for context pull and backlog scaffolding | Foundation (~$100/mo Max or $20/mo Pro) |
| **Linear** | Single home for the PRD (as a Linear Document) **and** the work breakdown (Initiatives → Projects → Milestones → Issues). Claude publishes the PRD, hosts stakeholder review on the Document, then scaffolds milestones and pushes stories — all via MCP, behind human approval gates | Foundation ($8–14/user/mo) |
| **Fireflies.ai** | Stakeholder interview recording, transcription, AI summaries | Foundation ($10/mo) |

#### Phase 2 — System Design

| Tool | Purpose | Subscription / Cost |
|------|---------|---------------------|
| **Claude (Max)** | Architecture proposals, ADRs, trade-off analysis, design review, tech stack decisions, API contract drafting | Foundation |
| **Claude Code** | Schema generation, OpenAPI spec generation, Mermaid C4 / sequence / ER diagrams from existing codebases | Foundation (in Max) |
| **Claude Design** | Primary UI/UX wireframing and prototyping — generates live HTML/CSS/React in a canvas, inherits the project design system from the repo, hands off directly into Claude Code. **Enabled-by-default on Pro/Max/Team; opt-in for Enterprise.** | Included in Claude (Pro / Max / Team / Enterprise) |
| **Eraser.io** | Architecture / sequence / ER / cloud / BPMN diagrams — driven from inside Claude via the **official Eraser MCP** (`https://app.eraser.io/api/mcp`), no separate-tool context-switch. Business tier recommended for client work (SOC 2 Type II + audit log). | Free 5 AI generations; Business $15/user/mo |

> **Phase 2 fallbacks (escalation only).** Cursor / v0 / Figma / Bolt are no longer primary for system design — they remain available only when Claude Design demonstrably cannot cover a flow (e.g., complex pixel-perfect brand work, native-app design tooling). Each fallback use is recorded in the ADR for that flow. See [Phase 2 PROCESS.md → Step 4.7](./aidlc-phases/02-system-design/PROCESS.md).

#### Phase 3 — Development

| Tool | Purpose | Subscription / Cost |
|------|---------|---------------------|
| **Cursor** | Primary IDE: AI autocomplete (Supermaven), Composer for multi-file edits, inline chat | Foundation ($20/mo) |
| **Claude Code** | Terminal agent: complex coding, multi-file refactoring, debugging, doc generation, MCP integrations (Linear, Figma, GitHub) | Foundation (in Max) |
| **Claude Code specialist subagents** (`linear-task-agent`, `frontend-engineer`, `backend-engineer`, `code-reviewer`, `refactor-specialist`) | Scoped, role-aware Claude Code sessions committed under `.claude/agents/` — `linear-task-agent` orchestrates the daily Linear → branch → PR loop; the four implementer/reviewer specialists carry the Phase 3 prompts with repo-specific calibration. Templates in [`aidlc-phases/03-development/subagent-prompts/`](./aidlc-phases/03-development/subagent-prompts/) | Foundation (in Max) |
| **Claude (Max)** | Effort estimation, design clarification, architecture-aware reasoning during coding | Foundation |
| **CodeRabbit** | Automated PR code review across all repos | Foundation ($24/mo) |
| **SonarQube** | Static analysis + quality gates in CI | Foundation (free) |

> **Single-vendor alt path.** Teams that prefer one billing line can drop Cursor and run **Claude Code Max 5x** (~$100/dev/mo, early-2026 pricing — verify before committing) as the sole AI coding tool. Lose inline ghost-text autocomplete (recover with **Supermaven standalone** if needed); Composer-style multi-file work shifts onto the specialist subagents above. Decision rules, gates, and quality bars are unchanged. See [Phase 3 PROCESS.md → Single-vendor alternative](./aidlc-phases/03-development/PROCESS.md).

#### Phase 4 — Testing & QA

| Tool | Purpose | Subscription / Cost |
|------|---------|---------------------|
| **Claude Code** | Unit / integration / E2E / load test generation; debugging fixes | Foundation (in Max) |
| **Cursor** | Inline test authoring + Composer for multi-file test refactors | Foundation |
| **CodeRabbit** | AI review of test code alongside production code | Foundation |
| **SonarQube** | Coverage gate enforcement in CI | Foundation (free) |
| **Claude (Max)** | Test plan generation from PRD + architecture | Foundation |
| **Vitest / Jest / pytest / JUnit 5** | Unit & integration test frameworks | Free OSS |
| **Playwright** | E2E web testing | Free OSS |
| **k6** | Load testing | Free OSS |
| **Sentry + Seer + Sentry Agent for Linear** | Production error tracking with AI root-cause analysis (Seer) that auto-files Linear issues with stack traces and suspected fixes attached. Closes the loop from prod error → triaged Linear bug → fix PR with no human copy-paste | Sentry from $26/mo (Linear already in foundation) |
| **Linear Triage Intelligence** | Duplicate detection, label/priority suggestions on every new bug — runs across both human-filed and Sentry-filed issues | Linear Business+ tier ($10/user/mo) |
| **BrowserStack App Automate** | Mobile real-device testing (only if mobile apps in scope) | From $199/mo |

#### Phase 5 — Security & Compliance

| Tool | Purpose | Subscription / Cost |
|------|---------|---------------------|
| **Claude Code `/security-review` + GitHub Action** | Diff-aware AI security review on every PR — injection, authn/authz, hardcoded secrets, sensitive-data logging; official `anthropics/claude-code-security-review` | Foundation (in Max) + Anthropic API usage |
| **Claude Code (orchestrator)** | Threat modelling (STRIDE + OWASP LLM/Agentic Top 10), security-fix generation, custom Semgrep rule authoring, evidence compilation, posture reporting | Foundation (in Max) |
| **SonarQube CE + Semgrep OSS + Semgrep MCP** | SAST baseline (security profile) + custom-rule scanning; `security_check`, `semgrep_scan_with_custom_rule`, `write_custom_semgrep_rule` via MCP | Free OSS |
| **GitHub Copilot Autofix** | AI auto-fix patches on CodeQL findings; included with Code Security licence (no Copilot subscription required); 12+ languages, expanding to Shell/Dockerfile/Terraform/PHP via AI scanning | Free with GHAS Code Security |
| **Snyk DeepCode AI / Snyk Agent Fix (upgrade)** | 80% auto-fix accuracy, up to 5 suggestions per finding, AI Security Fabric (Feb 2026), 19+ languages, Eclipse + VS support | Free tier 200 tests/mo; paid from $25/dev/mo |
| **Aikido Infinite + Endpoint (alt)** | Continuous AI pen-test (95% noise reduction) + April 2026 developer-workstation security against OSS supply-chain + IDE/AI-tool attacks | From $314/mo |
| **Dependabot / Renovate** | SCA: dependency auto-updates with patch-level CVE auto-merge | Free |
| **Endor Labs AURI / Snyk Open Source (reachability)** | Function-level reachability across 40+ languages; 95–97% noise reduction; AURI for AI-driven coding (March 2026) | Endor enterprise; Snyk free tier |
| **Trivy + Trivy MCP** | Container, IaC, dep, secrets, license scanning; IDE-triggered scans on `package.json` / `requirements.txt` / `Dockerfile` changes via official `aquasecurity/trivy-mcp` | Free OSS |
| **Docker Scout** | AI base-image upgrade suggestions on the registry (Phase 6 carry-over) | Free tier |
| **Checkov** | IaC compliance scanning (CIS / NIST) | Free OSS |
| **OPA / Gatekeeper + AI policy generation** | Runtime policy enforcement; Red Hat 2026 dynamic Kubernetes policy generator pattern via MCP; mandatory `dryrun`-first promotion | Free OSS |
| **GitGuardian + ggshield + ggshield AI hook** | Secrets detection (500+ detectors): pre-commit + platform real-time scan + March 2026 AI hook (pre-prompt + pre-tool-use + post-tool-use) for Cursor / Claude Code / Copilot | Free under 25 devs |
| **Cycode AI Governance + AIBOM (optional)** | AI inventory across 6 categories (code assistants, models, infrastructure, MCP servers, AI secrets, AI packages); MCP server policy enforcement (see / govern / enforce) | Enterprise |
| **Vanta Agents / Drata MCP (only if certifying)** | Vanta Compliance / TPRM / Customer Trust agents (March 2026, 4 hrs/week saved per user, 80% accuracy); Drata MCP + Vendor Risk Management Agent | Vanta from $7,500/yr; Drata from $10,000/yr |

#### Phase 6 — CI/CD & DevOps

| Tool | Purpose | Subscription / Cost |
|------|---------|---------------------|
| **Pulumi (Apache 2.0 OSS) + Pulumi MCP server** | Infrastructure as code in TypeScript / Python / Go / .NET; AI-native via Claude Code over MCP | Free CLI; Pulumi Cloud Team from $75/user/mo |
| **Pulumi Neo + Pulumi Copilot** | Autonomous infra agent (Neo) + conversational IaC assistant (Copilot) — primary IaC AI | Neo gated to Enterprise/Business Critical; Copilot in Pulumi Cloud |
| **Pulumi ESC** | Versioned secrets & configuration; pulls from Vault / AWS Secrets Manager / Azure Key Vault / 1Password | Bundled with Pulumi Cloud Team+ |
| **Pulumi Deployments + CrossGuard** | Managed CI/CD runner with drift detection; OPA-based policy-as-code with AI-suggested fixes | Bundled with Pulumi Cloud |
| **Claude Code** | Multi-file IaC, Dockerfile, K8s manifest, dashboard generation; PR review via official `anthropics/claude-code-action@v1` | Foundation (in Max) |
| **GitHub Actions + GitHub Copilot + Agentic Workflows** | CI/CD platform + AI YAML generation + Markdown-authored autonomous workflows (Claude Code engine) | Free tier + Copilot Pro $10/user/mo |
| **GitLab CI + GitLab Duo (fallback)** | Self-hosted DevSecOps with Duo Root Cause Analysis | Free tier + Premium $29/user/mo |
| **Docker Desktop 4.61+ with Gordon** | AI-assisted Dockerfile + docker-compose generation; CLI bundled with Docker Desktop | Personal free; Pro $9/user/mo |
| **Trivy + Docker Scout** | Image CVE scanning + AI base-image upgrade suggestions | Free OSS / free tier |
| **Kubernetes + K8sGPT** (optional) | Runtime orchestration only when needed; K8sGPT operator + MCP for cluster diagnostics | Self-hosted free |
| **Datadog + Bits AI SRE** OR **Grafana Cloud + Assistant + Sift** | Observability — autonomous incident triage (Bits AI SRE GA, 2× faster Q1 2026) or AI dashboard gen + Sift RCA (Grafana Assistant free as of April 2026) | Datadog from $15/host; Grafana Cloud free tier |
| **Sentry + Seer + Sentry MCP** | Application error tracking with AI fix patches (already in stack from Phase 4) | Already paid (Phase 4) |
| **Pulumi Insights / CAST AI / Infracost** | Cost analysis — Pulumi Insights for Pulumi stacks; CAST AI for K8s; Infracost for Terraform/OpenTofu paths only (Pulumi support not yet native) | Insights bundled; Infracost free OSS |

#### Phase 7 — Delivery & Handoff

| Tool | Purpose | Subscription / Cost |
|------|---------|---------------------|
| **Claude (Max)** | Release-note polish, handoff-document synthesis, KT-session summaries | Foundation |
| **Claude Code** | Multi-source doc generation via `--add-dir` across PRD repo, code repo, IaC repo, and docs-site MCP; completeness checks; handoff-document synthesis | Foundation (in Max) |
| **Anthropic Claude Code Action (`anthropics/claude-code-action@v1`)** | Runs on every git tag — converts the raw semantic-release changelog into customer-facing release notes and posts them to GitHub Releases; reused from Phase 6 | Anthropic API usage |
| **semantic-release + Conventional Commits + commitlint** | Automated semver, tagging, raw changelog generation | Free OSS |
| **Mintlify** (primary docs platform) | Customer-facing docs with hosted MCP server at `/mcp`, AI agent built-in, auto-generated `llms.txt` + `skill.md`. Recipient's Claude Code can query the docs site post-handoff with no extra wiring | Pro $300/mo; Enterprise for SSO/audit |
| **GitBook** *(alternate)* / **Docusaurus** *(internal/dev docs)* | GitBook adds `/~gitbook/mcp`; Docusaurus stays free OSS for repo-resident dev docs paired with Claude Code | GitBook Pro $215/mo for 10 seats; Docusaurus free |
| **API reference auto-gen** | Mintlify OpenAPI integration / `docusaurus-plugin-openapi-docs` / Redoc / Scalar — generated from the Phase 2 OpenAPI spec | Free OSS plugins |
| **Atlassian Remote MCP / Notion MCP** | Seeds the recipient's knowledge base from Claude Code (Confluence GA Feb 2026, 72+ tools; Notion `mcp.notion.com/mcp`). Glean is the cross-platform discovery overlay if the org already runs it | Confluence Standard $6.05/user/mo; Notion Business $15/user/mo |
| **Fathom** *(meeting KT)* / **Loom AI** *(screen-recorded KT)* | Fathom for action-item extraction quality (G2 9.4 vs Loom 7.2 in 2026); Loom AI for screen-recorded walkthroughs with auto chapters + summaries | Fathom Free / Premium $19/mo; Loom Business $15/user/mo |
| **NotebookLM Enterprise** *(optional, post-processing)* | Multi-source ingest (300 sources / 500K words each) when KT corpora exceed Claude Code's context comfortably | Via Gemini Enterprise |
| **1Password Business + 1Password CLI** | Secure credential vault and `op run -- claude` injection pattern (no plaintext secrets in `mcp.json`) for credential and MCP-token handoff | $7.99/user/mo |
| **Pulumi Cloud workspace transfer + Pulumi ESC** | IaC, state, and ESC environments transferred to the recipient org with the Phase 6 secret-store posture intact | Already in stack |
| **Sentry MCP + incident.io / PagerDuty AIOps** | Recipient inherits the live error backlog and on-call rotation with AI triage already wired | Sentry already in stack; incident.io from $20/responder/mo |
| **AGENTS.md + `.claude/agents/` transfer** | The handoff is "complete" when the recipient's Claude Code can run the same subagents (`linear-task-agent`, specialists, etc.) and `@claude` PR review the delivery team used — i.e., the recipient inherits the *agentic operating model*, not just the codebase | Transferred with the repo |

### Optional Upgrades (add only when justified)

| Tool | When to Add | Cost |
|------|-------------|------|
| **Snyk Code + Open Source** | When free SAST/SCA tiers are insufficient or AI auto-fix beyond Copilot Autofix becomes valuable | $25/dev/mo |
| **Endor Labs AURI** | When reachability-based noise reduction (97%) outweighs the enterprise spend | Enterprise |
| **Cycode AI Governance + AIBOM** | When formal AI usage visibility (shadow AI, AI BoM, MCP server policy enforcement) is a board / compliance mandate | Enterprise |
| **Vanta / Drata** | When formal SOC 2 / ISO 27001 / HIPAA certification is required | $7,500–10,000/yr |
| **Datadog + Bits AI SRE** | When Grafana Cloud free tier is insufficient or autonomous incident triage is a hard requirement | From $15/host/mo |
| **Mintlify Enterprise** | When SSO + audit logs + custom domain at scale matter and the docs are a product surface | ~$5K+/yr |
| **NotebookLM Enterprise** | When KT corpora (multi-hour recordings + repos + decks) exceed Claude Code's context comfortably | Via Gemini Enterprise |
| **Glean** | When the recipient already runs a multi-tool stack and wants permissions-aware AI search across Slack / Jira / Drive / Confluence / Linear | Enterprise |

**Total project cost at launch: same as Phases 1–3** (~$165/mo per dev). Additional costs only appear when the project justifies specific upgrades.

### Why Not More Tools?

| Eliminated Tool | Reason | What Covers It Instead |
|----------------|--------|----------------------|
| GitHub Copilot | Overlaps with Cursor's AI autocomplete + Claude Code's agent capabilities | **Cursor** (autocomplete) + **Claude Code** (agent) |
| ChatPRD | Overlaps with Claude + our standardised prompt templates | **Claude** + prompt templates in this repo |
| Notion / Confluence (for the PRD) | Linear Documents now hold the PRD natively, attached to the same Project that owns the Milestones and Issues — no separate doc tool needed for the PRD-to-backlog flow | **Linear Documents** (Project-scoped, stable section anchors, comment-able by Guests). Notion/Confluence remain optional for company-wide knowledge bases unrelated to a specific PRD. |
| v0 / Bolt.new / Cursor (for UI design) | Overlaps with Claude Design, which inherits the project's design system and hands off directly to Claude Code | **Claude Design** for wireframes and production-grade components; Cursor stays primary for general dev (Phase 3) |
| Stoplight | Overlaps with Claude Code for OpenAPI spec generation | **Claude Code** generates specs → view in free **Swagger UI** |
| Perplexity AI | Overlaps with Claude's knowledge; web search covers the rest | **Claude** + manual web research when needed |
| Swimm | Overlaps with Claude Code for documentation generation; no MCP server, so Claude can't query the published site | **Claude Code** generates docs → publish with **Mintlify** (primary, hosted MCP) or **Docusaurus** (free, repo-resident dev docs) |

### Alternative Stack (if primary is unavailable)

If Claude/Cursor is not an option (e.g., client restrictions, licensing), use this fallback:

| Primary | Alternative | Notes |
|---------|-------------|-------|
| Claude (Max) | ChatGPT Plus ($20/mo) or Gemini Advanced ($20/mo) | Adjust prompts for the model; core workflow stays the same |
| Claude Code | Aider (free, BYOK) or Cline (free VS Code extension, BYOK) | Open-source agents; bring your own API key |
| Cursor | VS Code + GitHub Copilot ($10–19/mo) | Different UX but covers autocomplete + agent mode |
| CodeRabbit | GitHub Copilot Code Review (included in Copilot plan) | Less comprehensive but zero additional cost |
| Fireflies.ai | Fathom (free unlimited) or Otter.ai ($17/mo) | Fathom is best budget option |
| SonarQube | Codacy (free tier) or ESLint/Prettier only | Codacy bundles SAST + review |

---

## Repository Structure

```
ai-dlc-resources/
├── README.md                                       ← You are here
├── aidlc-phases/                                   ← Process definitions for all 7 SDLC phases
│   ├── 01-requirement-gathering/
│   │   ├── PROCESS.md                              ← Workflow, inputs/outputs, responsibilities
│   │   ├── PROMPTS.md                              ← Prompt templates for every AI task
│   │   ├── QUALITY-GATES.md                        ← Gate criteria, checklists, escalation rules
│   │   └── FLOWCHART.md                            ← Visual process map (Mermaid)
│   ├── 02-system-design/
│   │   ├── PROCESS.md
│   │   ├── PROMPTS.md
│   │   ├── QUALITY-GATES.md
│   │   └── FLOWCHART.md
│   ├── 03-development/
│   │   ├── PROCESS.md
│   │   ├── PROMPTS.md
│   │   ├── QUALITY-GATES.md
│   │   ├── FLOWCHART.md
│   │   └── subagent-prompts/                       ← Specialist Claude Code subagent templates
│   │       ├── README.md                           ← How to instantiate per repo
│   │       ├── frontend-engineer.md                ← UI implementation (Steps 3.1–3.3)
│   │       ├── backend-engineer.md                 ← Server-side implementation (Steps 3.1–3.3)
│   │       ├── code-reviewer.md                    ← Pre-PR self-review (Step 3.5, read-only)
│   │       └── refactor-specialist.md              ← Step 5 refactoring against tech-debt issues
│   ├── 04-testing-and-qa/
│   │   ├── PROCESS.md
│   │   ├── PROMPTS.md
│   │   ├── QUALITY-GATES.md
│   │   └── FLOWCHART.md
│   ├── 05-security/
│   │   ├── PROCESS.md
│   │   ├── PROMPTS.md
│   │   ├── QUALITY-GATES.md
│   │   └── FLOWCHART.md
│   ├── 06-cicd-devops/
│   │   ├── PROCESS.md
│   │   ├── PROMPTS.md
│   │   ├── QUALITY-GATES.md
│   │   └── FLOWCHART.md
│   └── 07-delivery-handoff/
│       ├── PROCESS.md
│       ├── PROMPTS.md
│       ├── QUALITY-GATES.md
│       └── FLOWCHART.md
├── docs/
│   ├── planning/                                   ← Initiative planning artefacts
│   │   ├── AI-DLC_Project_Plan.md                  ← 8-week execution plan, objectives, deliverables
│   │   └── AI-DLC_Scope_Definition.md              ← In-scope / out-of-scope boundaries
│   └── tools-evaluation/                           ← Per-phase AI tool evaluation reports
│       ├── 1.RequirementGathering_Phase_Tools.md
│       ├── 2.SystemDesign_Phase_Tools.md
│       ├── 3.Development_Phase_Tools.md
│       ├── 4.TestingQA_Phase_Tools.md
│       ├── 5.Security_Compliance_Phase_Tools.md
│       ├── 6.AIDLC_CICD_DevOps_Phase_Tools.md
│       └── 7.Delivery_Handoff_Phase_Tools.md
└── templates/                                      ← Reusable artefact skeletons
    ├── prd-template.md                             ← Reusable PRD skeleton (Phase 1)
    ├── user-story-template.md                      ← Story + acceptance criteria format (Phase 1)
    └── code-review-checklist.md                    ← AI + human review checklist (Phase 3)
```

## How to Use This Repository

### For a new project:
1. **Set up the tool stack** — Ensure every team member has: Claude (Max or Pro, with **Claude Design** enabled), Cursor Pro, CodeRabbit on repos, **Linear workspace access**, an **Eraser.io** workspace (Business tier for client work), Fireflies.ai for meetings, and SonarQube in CI.
2. **Connect Claude to Linear via MCP** — follow [Phase 1 Step 0](./aidlc-phases/01-requirement-gathering/PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp) once per workspace and once per developer. This is the integration that powers backlog automation across phases 1, 3, 4, and 7.
3. **Connect Claude to Eraser via MCP** — follow [Phase 2 Step 0](./aidlc-phases/02-system-design/PROCESS.md#step-0-one-time-setup--connect-claude-to-eraser-via-mcp) once per workspace and once per architect. This drives all system-design diagramming from inside Claude. Enterprise admins must also enable **Claude Design** under Admin → Features before Phase 2 Step 4.
4. **Install the Phase 3 specialist subagents** — copy the templates from [`aidlc-phases/03-development/subagent-prompts/`](./aidlc-phases/03-development/subagent-prompts/) into the consuming repo's `.claude/agents/` directory, fill the placeholders (`{{TEAM_STACK}}`, `{{COMPONENT_LIBRARY_PATH}}`, `{{TEST_RUNNER}}`, etc.), and commit. Verify with `/agents` in a Claude Code session. See the [subagent-prompts README](./aidlc-phases/03-development/subagent-prompts/README.md) for the full instantiation checklist.
5. **Read the PROCESS.md** for each phase sequentially as you progress through the SDLC.
6. **Copy prompt templates** from PROMPTS.md into your project's prompt library. Customise placeholders for your project.
7. **Configure quality gates** from QUALITY-GATES.md in your CI/CD pipeline and review process.

### For adopting on an existing project:
1. Identify which phase you are currently in.
2. Read that phase's PROCESS.md and adopt the workflows that apply.
3. Gradually introduce prompt templates and quality gates.

## Key Principles

1. **One tool per job.** Don't stack overlapping tools. Reduce cost and complexity by using each tool for what it does best.
2. **AI assists, humans decide.** Every AI-generated artifact passes through human review before it is considered complete.
3. **Prompts are standardised, not improvised.** The prompt library ensures consistent output regardless of which team member runs the prompt.
4. **Quality gates are non-negotiable.** No phase output advances without passing its defined checklist.
5. **Claude is the backbone.** One AI platform handles reasoning across all phases — requirements, design, development, review. This means one learning curve, one subscription, one set of conventions.

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | April 2026 | Initial process definition for Phases 1–3 |
| 1.1 | April 2026 | Consolidated tool stack — eliminated redundant tools |
| 1.2 | April 2026 | **Week 4 complete** — Process definitions added for Phases 4–7 (Testing & QA, Security & Compliance, CI/CD & DevOps, Delivery & Handoff). All 7 SDLC phases now fully defined. Additional free/OSS tools added for Phases 4–7 with no increase to baseline per-developer cost. |
| 1.3 | April 2026 | Repository restructured — phase definitions moved under `aidlc-phases/`. New `docs/planning/` folder added with the 8-week project plan and scope definition. New `docs/tools-evaluation/` folder added with per-phase tool evaluation reports backing the consolidated stack decisions. Phase folder names normalised (`04-testing-and-qa`, `05-security`). |
| 1.4 | April 2026 | **Linear adopted as foundation PM tool.** Promoted from Phase-4-only to a foundation tool used in Phases 1, 3, 4, and 7. Phase 1 PROCESS.md adds Step 0 (MCP connector setup), three human approval gates, and a PRD-to-Linear loop. PROMPTS.md adds four Linear-aware prompts (`linear-context-pull`, `prd-to-linear-scaffold`, `stories-to-linear-push`, `linear-gap-sweep`). QUALITY-GATES.md and FLOWCHART.md updated to enforce traceability (PRD section citation per issue) and the AI Inbox review queue. |
| 1.5 | April 2026 | **PRD now lives in Linear as a Linear Document.** Eliminated the Notion / Confluence / repo-`/docs/` storage step — the PRD is published to a Linear Document attached to the Linear Project that owns the work breakdown. Stakeholder review happens on the Document via inline comments (Guests for non-engineering reviewers). New `prd-to-linear-document` prompt added; `prd-to-linear-scaffold` simplified to milestone-only since the Project is created in Step 2.4. Issue PRD-citations are now clickable Markdown deep-links into the Document (`[§X.Y](document-url#anchor)`), making per-story acceptance click-to-verify. |
| 1.6 | April 2026 | **Phase 2 (System Design) deepened end-to-end.** PROCESS.md rewritten with numbered sub-stages, four inline gates, a Risks & Guardrails table, and a System-Design-to-Phase-3 loop diagram — matching Phase 1's depth. **Eraser.io is now driven from inside Claude via the official Eraser MCP** (`https://app.eraser.io/api/mcp`); new Step 0 walks through OAuth setup for both claude.ai and Claude Code. **Claude Design replaces Cursor / v0 / Bolt as the primary UI/UX tool** — it inherits the project's design system and hands off directly into Claude Code; Cursor / v0 / Figma / Bolt remain only as fallback escalation paths recorded per-flow in the ADR. Five new prompts added to PROMPTS.md: `eraser-architecture-diagram`, `eraser-er-diagram`, `claude-design-system-bootstrap`, `claude-design-wireframe`, `claude-design-production-component`. **Phase 1 + Phase 2 FLOWCHART.md split** from one monolithic Mermaid diagram each into six per-step flowcharts (Step 0 setup + five process steps), each standalone with explicit entry/exit chaining. Phase 1 wording aligned across PROCESS.md and QUALITY-GATES.md (Linear Project state at Gate 1: now consistently "moved to Backlog or Started", not the looser "out of Planned"). |
| 1.7 | April 2026 | **Phases 3–7 deepened; Claude Code subagent system introduced.** Phase 3 PROCESS.md rewritten with a numbered sub-stage flow, a `linear-task-agent` workflow orchestrator, and **four committed specialist subagent templates** under [`aidlc-phases/03-development/subagent-prompts/`](./aidlc-phases/03-development/subagent-prompts/) — `frontend-engineer`, `backend-engineer`, `code-reviewer`, `refactor-specialist` — each scoped with operating-boundary rules (no Linear writes, no PR open, no push). A **single-vendor alternative path** (Claude Code Max 5x replacing Cursor) is documented with three explicit substitutions for the per-step prose. Phase 3/4 FLOWCHART.md split into per-step Mermaid diagrams matching Phases 1–2. **Phase 4 (Testing & QA)**: Sentry + Seer + Sentry Agent for Linear closes the prod-error → triaged-Linear-bug loop end-to-end; Linear Triage Intelligence added for duplicate detection and label/priority suggestions. **Phase 5 (Security & Compliance)** rewritten around a **free-first layered AI-native stack**: SonarQube CE + Semgrep MCP + Claude Code `/security-review` + Copilot Autofix for SAST; Trivy MCP for IDE-driven dep/container/IaC; ggshield AI hook (March 2026) for the Cursor / Claude Code / Copilot prompt loop; OWASP Top 10 for Agentic Apps 2026 mapped per service; Cycode AI Governance + AIBOM and Vanta Agents / Drata MCP added as optional upgrades. **Phase 6 (CI/CD & DevOps)** rewritten around **Pulumi AI** (Pulumi Neo + Copilot + MCP + ESC + CrossGuard) as the primary IaC stack; **Anthropic Claude Code Action** (`anthropics/claude-code-action@v1`) as the CI/CD spine; Datadog Bits AI SRE / Grafana Assistant + Sift for observability; Docker Gordon for containerization. **Phase 7 (Delivery & Handoff)** rewritten as an **agentic-operating-model transfer**, not just docs: semantic-release + Anthropic Claude Code Action for automated release notes; Mintlify (with hosted MCP at `/mcp`, auto `llms.txt` + `skill.md`) as the primary docs platform; Atlassian Remote MCP / Notion MCP for KB seeding; Fathom / Loom AI / NotebookLM Enterprise for KT; 1Password Business + `op run -- claude` for credential and MCP-token handoff with no plaintext in `mcp.json`; Pulumi Cloud workspace + ESC environment transfer carries the Phase 6 secrets posture intact; Sentry MCP + incident.io / PagerDuty AIOps so the recipient inherits the live error backlog and on-call rotation; **`AGENTS.md` + `.claude/agents/` transfer** so the recipient's first day looks identical to the delivery team's last day. New **30-day support window + 90-day debrief** built into the phase. |
| 1.8 | May 2026 | **Prompts library revamp across all 7 phases.** Each phase's PROMPTS.md reworked so previously inline / ad-hoc prompts become first-class, named entries that PROCESS.md links to directly. New / promoted prompts include `trade-off-interrogation`, `entity-extraction`, `migration-generation` (Phase 2); a substantially expanded Phase 3 prompt set; an expanded Phase 4 testing prompt set; `agents-md-authoring` and `claude-code-action--pr-review-workflow` (Phase 6); and `KT Session Summary`, `Inherited Error Triage`, `Quarterly KB Completeness Check` (Phase 7 — these replace the deferred-prompts appendix that previously sat at the bottom of Phase 7 PROCESS.md). Phase 2/3/6/7 PROCESS.md updated to call the new prompts by name instead of inlining the instructions. **`linear-task-agent` subagent template committed** at [`aidlc-phases/03-development/subagent-prompts/linear-task-agent.md`](./aidlc-phases/03-development/subagent-prompts/linear-task-agent.md) — previously inline-only inside Phase 3 PROCESS.md, now a fifth reference template alongside the four specialist roles, with documented placeholders (`{{TEAM_PREFIX}}`, `{{LINEAR_PROJECT}}`, `{{PR_TITLE_FORMAT}}`, `{{LABEL_CONVENTIONS}}`, `{{TECH_DEBT_LABEL}}`, `{{BUG_LABEL}}`) and `model: sonnet` (down from `opus` — Sonnet handles Linear MCP work at the throughput the dev loop needs). The subagent-prompts README table and the Phase 3 PROCESS.md Step 0 path updated to point at the new template. |
