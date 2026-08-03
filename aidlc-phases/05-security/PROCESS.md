# Phase 5: Security & Compliance — Process Definition

## Overview

This document defines the AI-assisted workflow for the Security & Compliance phase of the AI-DLC. It is the phase where the team's AI productivity gains are stress-tested against an adversarial reality: AI-generated code now ships at ~1.7× the vulnerability density of human-written code, AI coding tools leak secrets in commits at roughly 2× the historical baseline, and the OWASP Top 10 for Agentic Applications (Dec 2025) has added prompt injection, agent goal hijacking, and tool-misuse risks to the list of things every team must defend against. The process below is built around a **deliberately narrow, GitHub-native stack** drawn from [`5.Security_Compliance_Phase_Tools.md`](../../docs/tools-evaluation/5.Security_Compliance_Phase_Tools.md): **CodeQL + Claude Code `/security-review` + GitHub Copilot Autofix** for static analysis; **Dependabot** for dependency hygiene; **GitGuardian + ggshield AI hook** for secrets at every layer including the AI prompt loop; **human review against `AGENTS.md` and a fixed required-controls list** for containers and IaC; and **Claude Code** for threat models, compliance checklists, evidence dossiers, and the quarterly posture report.

**Phase Duration:** Continuous from project start + focused audit sprint before each release
**Phase Owner:** Security Champion / Tech Lead
**Tools Used:**

- **SAST:** **CodeQL** (GitHub code scanning) as the scanning baseline — per-PR diff analysis **and** a scheduled full-repo scan — with project-specific **custom CodeQL queries** committed to `.github/codeql/`; **Claude Code `/security-review`** slash command and the **`anthropics/claude-code-security-review` GitHub Action** as the semantic layer; **GitHub Copilot Autofix** on CodeQL findings
- **Dependencies:** **Dependabot** (alerts + security updates + version updates), supplemented by package-manager-native audit (`npm audit`, `pip-audit`, `cargo audit`)
- **Container & IaC:** human review — base images digest-pinned; Dockerfiles, K8s manifests, and IaC diffs reviewed against `AGENTS.md` and the required-controls list in Step 4. **No automated enforcement in this phase**
- **Secrets:** GitGuardian platform + **ggshield AI hook** (March 2026) for Claude Code — pre-prompt, pre-tool-use, and post-tool-use scanning
- **AI / agent security:** OWASP LLM Top 10 + **OWASP Top 10 for Agentic Applications 2026** mapped per service; **Anthropic Claude Opus 4.7 prompt-injection defences** as the model-layer baseline; MCP allow-list in `AGENTS.md` pinned by the project-scoped `.mcp.json`
- **Compliance:** Claude Code (checklists + evidence) + CodeQL / Dependabot / GitGuardian / `/security-review` as automated control evidence, with documented manual attestation for every control no tool covers; NIST AI RMF + ISO/IEC 42001 crosswalk for AI products
- **AI authoring & remediation:** **Claude Code** as the orchestrator — the same model the team already uses in Phase 3 and Phase 6

> **Tool Philosophy.** The stack is deliberately narrow: **one detector per capability, GitHub-native where GitHub has one, GitGuardian for secrets, Claude Code for everything that needs judgement.** Narrow is a choice, not a compromise — a finding from a single named detector has a single named owner, and there is no cross-tool triage tax spent reconciling three reports that disagree. **CodeQL is the deterministic floor:** it scans the whole repo, runs on a schedule as well as per-PR, emits SARIF for audit, and accepts committable custom queries, so pattern-level coverage stays under version control. `/security-review` is the semantic layer on top of it, reasoning about the intent of what changed. ggshield's AI hook closes the loop the developer's AI assistant opened — pre-prompt, pre-tool-use, post-tool-use. Where a capability has **no** automated detector — container and IaC policy enforcement — this process says so plainly and assigns it to a named human reviewer rather than implying coverage that does not exist. Every state-changing remediation requires a human approval gate.

---

## Tool Stack

| Layer | Primary | Fallback | Cost |
|-------|---------|----------|------|
| **SAST baseline** | **CodeQL** via GitHub code scanning — per-PR diff analysis + scheduled full-repo scan; SARIF archived as a CI artefact | — | Free on public repos; GHAS Code Security $30/active-committer/mo on private |
| **SAST custom rules** | **Custom CodeQL queries** committed to `.github/codeql/`, authored in Claude Code and verified against two fixtures | — | Free |
| **AI security review** | **Claude Code `/security-review`** slash command + **`anthropics/claude-code-security-review` GitHub Action** | CodeRabbit security rules (Phase 3 carry-over) | Anthropic API usage-based |
| **AI auto-fix** | **GitHub Copilot Autofix** — posts a suggested patch on CodeQL findings | Claude Code [`security-fix-generation`](./PROMPTS.md#security-fix-generation) | Free with Code Security licence; included with Copilot in public OSS |
| **Dependencies** | **Dependabot** — alerts, security updates, version updates; patch-level CVE auto-merge | Package-manager-native audit (`npm audit`, `pip-audit`, `cargo audit`) — ships with the toolchain | Free |
| **Container & IaC** | **Human review** — digest-pinned base images; Dockerfile / K8s / IaC diffs checked against `AGENTS.md` + the Step 4 required-controls list | — | Review time only |
| **Secrets — platform + pre-commit** | **GitGuardian** (500+ detectors, real-time push scan) + **ggshield** pre-commit hook | — | Free for teams < 25 devs; per-dev paid above |
| **Secrets — AI hook** | **ggshield AI hook** (March 2026) — pre-prompt + pre-tool-use + post-tool-use scanning for Claude Code | — | Free with ggshield |
| **AI / agent security** | **OWASP LLM Top 10** + **OWASP Top 10 for Agentic Applications 2026** mapped via the Claude Code threat-model prompt; **Anthropic Claude Opus 4.7** model-layer prompt-injection defences | — | Anthropic defences built-in |
| **Compliance — checklists / evidence** | **Claude Code** + Phase 5 prompt library; CodeQL SARIF, Dependabot alerts, GitGuardian incident log, and `/security-review` comments as the automated evidence base | Documented manual attestation for controls no tool covers | Already paid |

---

## Process Steps

### Step 0: One-Time Setup — Wire AI Security Tools into the Loop

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | GitHub org with Code Security licence (or a public repo), Anthropic API key, GitGuardian workspace + ggshield, Claude Code installed |
| **Tools** | **Claude Code `/security-review`**, **`anthropics/claude-code-security-review` GitHub Action**, **GitHub code scanning (CodeQL)**, **Copilot Autofix**, **Dependabot**, **ggshield AI hook**, GitHub Actions |
| **Output** | Every developer's Claude Code can scan diffs locally and via PR; CodeQL + dependency + secrets scanning all run in CI; AI hook intercepts secrets at prompt and tool-use time |
| **Human** | Security Champion owns the code-scanning configuration; each developer authenticates ggshield once; Tech Lead grants the security-review workflow its write permissions (PR comments, issue creation) via the GitHub Action token scopes and Claude Code's tool-permission settings |

This step is done **once per project**. Skipping it means paying the security tax by hand on every PR for the rest of the phase.

#### 0.1 — Install Claude Code `/security-review` and the GitHub Action

The official slash command (`/security-review`) ships with Claude Code and detects injection (SQL / cmd / LDAP / XPath / NoSQL / XXE), authn/authz issues, hardcoded secrets, and sensitive-data logging. It is **diff-aware** — it scans only what changed — and ships a false-positive filter. The matching GitHub Action runs the same engine on every PR.

```bash
# Local: confirm the slash command is available
claude
> /security-review --help

# (Optional) Customise by copying the command markdown into the repo
mkdir -p .claude/commands
curl -sL https://raw.githubusercontent.com/anthropics/claude-code-security-review/main/security-review.md \
  > .claude/commands/security-review.md

# CI: install the GitHub Action
# Commit .github/workflows/claude-security-review.yml referencing
# anthropics/claude-code-security-review (pin to a tagged release)
```

Grant the security-review workflow its write permissions — **post comments on PRs** and **create issues** — through the GitHub Action's `GITHUB_TOKEN` scopes (in the workflow YAML) and Claude Code's tool-permission settings; keep both withheld by default and enable only when the team is ready. Keep `delete` operations off — humans close findings, the agent does not.

#### 0.2 — Enable GitHub code scanning (CodeQL) and commit the workflow

CodeQL is the **SAST baseline** for the phase. Turn on code scanning and commit the workflow so the configuration is reviewable in Git rather than living only in repo settings.

```bash
# GitHub: enable code scanning
# Settings → Code security → Code scanning → CodeQL analysis → Set up
#   • "Default" gets you running in one click
#   • "Advanced" writes .github/workflows/codeql.yml — use this, because
#     custom queries and SARIF archiving both need the committed workflow

# Verify the languages CodeQL detected match the repo
gh api repos/:owner/:repo/code-scanning/analyses --jq '.[0] | {tool: .tool.name, ref, created_at}'
```

Configure **both cadences** in `.github/workflows/codeql.yml`:

- **`pull_request`** — analysis on every PR; findings surface as PR checks and annotations, and gate the merge.
- **`schedule:` (nightly `cron`) on the default branch** — full-repo analysis. This is what catches drift in code no PR touched, and re-evaluates existing code against newly added queries.

Two settings the rest of the phase depends on:

1. **Fail the build on Critical / High.** Set the code-scanning merge protection (or a `github/codeql-action/upload-sarif` follow-up step) so Critical / High alerts block the PR rather than reporting as advisory.
2. **Archive the SARIF as a CI artefact.** CodeQL uploads results to the code-scanning API by default; add an `actions/upload-artifact` step for the SARIF file as well, so Step 7 evidence compilation and the quarterly posture report have a durable, exportable source that does not depend on API retention.

Finally, turn on **Copilot Autofix** in **Settings → Code security → Code scanning**. As of Q2 2026 it covers C# / C / C++ / Go / Java / Kotlin / Swift / JS / TS / Python / Ruby / Rust, with Shell / Dockerfile / Terraform / PHP rolling out. Autofix does **not** require a Copilot subscription, and pull-request insights cover all protected branches as of March 2026.

#### 0.3 — Install ggshield + the AI hook for Claude Code

ggshield's AI hook (March 2026) is the **third** secrets layer, after pre-commit and platform-side. It intercepts at three points in the AI loop:

1. **Pre-prompt** — scans the developer's input before it goes to the model (catches secrets the developer pasted in).
2. **Pre-tool-use** — scans tool calls the model wants to execute (catches secrets in `bash` / file-read / MCP-call args).
3. **Post-tool-use** — scans tool output (catches secrets pulled from logs / files / MCP responses before they enter the model context).

```bash
# Install ggshield (pre-commit + AI hook share the same CLI)
brew install gitguardian/tap/ggshield   # macOS
# or: pipx install ggshield              # cross-platform

# Auth once per developer
ggshield auth login

# 1. Pre-commit hook (developer-side)
ggshield install --mode local

# 2. AI hook (developer-side, Claude Code)
ggshield ai install              # configures the hook for detected AI tools
ggshield ai status               # verify which clients are wired up

# 3. Platform-side: enable GitGuardian on every repository
# Settings → Integrations → enable for org; turn on historical scan
```

Run a **one-time historical scan** in the GitGuardian dashboard. 64% of secrets leaked in 2022 are still active in 2026 — the historical sweep almost always finds something to rotate.

#### 0.4 — Wire dependency scanning: Dependabot

```bash
# GitHub: enable Dependabot alerts + security updates + version updates
# Settings → Code security → Dependabot → Enable all three

# Commit .github/dependabot.yml with auto-merge for patch-level CVE fixes
# (require PR review for minor/major updates)
```

Set the `.github/dependabot.yml` schedule to **weekly for version updates** and leave **security updates real-time**. Auto-merge patch-level CVE fixes once CI is green; minor and major bumps go through review. Where the toolchain ships its own auditor (`npm audit`, `pip-audit`, `cargo audit`), wire it into the same CI job as a cheap second opinion on the lockfile — it costs nothing and catches advisories before Dependabot's next poll.

#### 0.5 — Add a security conventions section to `AGENTS.md`

The `AGENTS.md` already committed for Phase 6 (and Phase 3) is the canonical project context file — Claude Code reads it on every session. Add a **Security conventions** section: forbidden APIs (e.g., raw SQL string concatenation), required input validators, auth middleware that must wrap every endpoint, secret-handling rules ("never call `console.log` on objects that may contain credentials"), the **required-controls list from Step 4**, the **MCP server allow-list from Step 6.4**, and the Phase 3 / 5 AI-generated-code review escalation path. Without this, every prompt drifts; with it, each agent fences itself within team standards.

#### Verification checklist

- [ ] `claude /security-review` runs locally and posts findings on a synthetic vulnerable diff
- [ ] `.github/workflows/claude-security-review.yml` committed; smoke-tested on a draft PR with an injected SQL-i string — Critical comment appears within ~3 minutes
- [ ] **CodeQL code scanning enabled**; `.github/workflows/codeql.yml` committed with both `pull_request` and scheduled `cron` triggers; SARIF archived as a workflow artefact
- [ ] `ggshield ai status` confirms hook active for Claude Code
- [ ] GitGuardian historical scan complete; any leaks routed through Step 5 incident response
- [ ] Dependabot active on every repo; auto-merge configured for patch-level CVE fixes
- [ ] Copilot Autofix on for the languages in scope
- [ ] `AGENTS.md` updated with **Security conventions** section
- [ ] Anthropic admin policies expanded: `/security-review` may post comments and create issues; `delete` operations remain disabled

> **Permission inheritance.** Every agent-driven action runs as the connecting developer — the agent cannot escalate beyond their GitHub / GitGuardian scopes. Off-board by revoking the OAuth grant in each provider; the agent loses access automatically.

---

### Step 1: Threat Modelling & Security Architecture Review

> Visual: [Step 1 flowchart](./FLOWCHART.md#step-1-threat-modelling)

| Attribute | Detail |
|-----------|--------|
| **Input** | Phase 2 architecture proposal, Phase 1 NFRs, data classification, compliance scope |
| **Tools** | **Claude Code** with [`threat-model-stride`](./PROMPTS.md#threat-model-stride), [`ai-agent-threat-review`](./PROMPTS.md#ai-agent-threat-review), and [`threat-model-mitigation-verification`](./PROMPTS.md#threat-model-mitigation-verification) prompts; OWASP Top 10 + OWASP API Top 10 + **OWASP LLM Top 10** + **OWASP Top 10 for Agentic Applications 2026** as rule sets |
| **Output** | `/docs/security/threat-model.md` with STRIDE issues + AI/agent issues, prioritised mitigations, P0/P1/P2 timeline; a per-release mitigation verification table at `/docs/security/mitigation-verification-<release>.md` |
| **Human** | Security Champion validates and prioritises; Tech Lead signs off on must-fix-before-launch items and on any mitigation left unlanded at a release candidate |

**Workflow:**

**1.1 — STRIDE pass against the architecture.** From Claude Code, run [`threat-model-stride`](./PROMPTS.md#threat-model-stride) against the Phase 2 architecture description and diagrams. Output: STRIDE issues (Spoofing / Tampering / Repudiation / Information Disclosure / DoS / Elevation of Privilege) per component, severity, recommendation, priority. Cross-reference OWASP Top 10 and OWASP API Top 10.

**1.2 — AI / agent threat pass (only if AI features are in the product).** Run [`ai-agent-threat-review`](./PROMPTS.md#ai-agent-threat-review) — covers **OWASP LLM Top 10** (prompt injection, insecure output handling, training-data poisoning, model DoS, supply-chain, sensitive-information disclosure, insecure plugin design, excessive agency, overreliance, model theft) **and** the new **OWASP Top 10 for Agentic Applications 2026** (released Dec 2025, peer-reviewed by 100+ experts). The top agentic risk is **ASI01: Agent Goal Hijacking** — poisoned inputs in emails / docs / web pages hijack the agent's objective. The pass also covers goal misalignment, tool misuse, delegated trust, inter-agent communication, persistent memory, emergent autonomous behaviour.

**1.3 — Document and triage.** Commit the threat model at `/docs/security/threat-model.md`. Each finding has Issue / STRIDE-or-OWASP-Agentic category / Severity / Affected component / Recommendation / Priority (P0 must-fix before launch / P1 within 30 days / P2 within 90 days). Open Linear issues for every P0 and P1.

**1.4 — Wire mitigations into downstream phases.** P0 mitigations become acceptance criteria on Phase 3 stories, custom CodeQL queries (Step 2.3), or Phase 6 pipeline controls. **Don't leave threat-model findings on a wiki** — they have to land in the backlog or they don't ship. The backlog is where the loop opens; **1.5 is what closes it**, by proving the mitigation reached the code and not just the ticket.

**1.5 — Verify the mitigations landed.** Before each release candidate, run [`threat-model-mitigation-verification`](./PROMPTS.md#threat-model-mitigation-verification) against the P0 (and in-scope P1) list from 1.3 plus the diff since the last verified release. For each mitigation Claude either locates the implementing code and cites `file:line`, or reports it **not landed** and says where it looked; a mitigation implemented differently from what the threat model specified comes back **partial**, with the difference stated, so the Security Champion decides whether to accept the variant or reopen the finding. **Only implementing code is evidence** — a comment, a TODO, a test name, or a Linear issue link is not. The check does **not** hunt for new vulnerabilities (that is `/security-review` in Step 2.2) and does not propose fixes. It runs on every project, compliance scope or not, and its table is the producer behind the "P0 mitigations landed" checkbox at [Gate 7](./QUALITY-GATES.md#gate-7-compliance--audit-readiness) and at phase handoff. Commit it at `/docs/security/mitigation-verification-<release>.md`.

> **Gate 1 — Threat model & baseline scanning** must pass before a release candidate is built. See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-threat-model--baseline).

---

### Step 2: SAST — Continuous AI-Assisted Static Analysis

> Visual: [Step 2 flowchart](./FLOWCHART.md#step-2-sast)

| Attribute | Detail |
|-----------|--------|
| **Input** | Every PR + scheduled full-repo scan |
| **Tools** | **CodeQL** (SAST baseline in GitHub code scanning) + **custom CodeQL queries** in `.github/codeql/` + **Claude Code `/security-review`** + **`anthropics/claude-code-security-review` GitHub Action** + **GitHub Copilot Autofix** |
| **Output** | PR-level findings with severity buckets (Critical / High / Medium / Low) + Autofix patch suggestions on CodeQL findings + scheduled full-repo trend report |
| **Human** | Developer addresses findings; Security Champion approves any dismissal of Critical / High findings |

**Workflow:**

**2.1 — Baseline SAST in CI (CodeQL).** **CodeQL is the SAST baseline** — the code-scanning workflow committed in Step 0.2 is the job this step gates on. Confirm three things: (a) the workflow analyses **every language present in the repo**, running the `security-extended` query suite plus the project's own queries in `.github/codeql/`; (b) the job **fails the build on Critical / High findings** rather than reporting them as advisory; (c) **both cadences run** — **per-PR analysis** (gates the merge) and a **scheduled full-repo scan** on the default branch (catches drift in code no PR touched, and re-evaluates existing code against newly added queries). Archive the SARIF output as a CI artefact so Step 7 evidence compilation and the quarterly posture report have a durable source.

**2.2 — `/security-review` on every PR.** The `anthropics/claude-code-security-review` GitHub Action runs on `pull_request: opened` and `synchronize`. It scans the diff — not the full repo, which is the scheduled CodeQL scan's job — for: SQL / cmd / LDAP / XPath / NoSQL / XXE injection, authn/authz, hardcoded secrets, sensitive-data logging. Findings post as PR comments bucketed Critical / High / Medium / Low. **Diff-aware filtering** keeps signal-to-noise high — the FP rate is materially lower than traditional full-repo SAST. It is the *semantic* layer over CodeQL's *dataflow* layer, **not a replacement for it**: CodeQL traces taint from source to sink deterministically across every line of the repo, `/security-review` reasons about the intent of what changed.

**2.3 — Author custom CodeQL queries for project-specific patterns.** When a finding is project-specific ("we never call `eval` on user input", "every endpoint must call `requireAuth`", "no `dangerouslySetInnerHTML` outside `/components/admin/`"), use Claude Code's [`codeql-custom-query-generation`](./PROMPTS.md#codeql-custom-query-generation) prompt. Commit the query under `.github/codeql/` and reference it from the workflow's `queries:` block so every future PR is gated. **Verify against two fixtures before committing** — the query must fire on the anti-example and stay silent on the allowed example. A query that only ever passes is worse than no query, because it manufactures confidence.

**2.4 — AI auto-fix path (Copilot Autofix).** Autofix is GA in 2026 and covers C# / C / C++ / Go / Java / Kotlin / Swift / JS / TS / Python / Ruby / Rust, expanding to Shell / Dockerfile / Terraform / PHP. It posts a suggested patch on every CodeQL finding; review and accept inline. Autofix suggestions are reviewed like any other patch — accepting one without reading it is how a tautological fix lands.

**2.5 — Manual remediation via Claude Code for the rest.** When auto-fix is unavailable or low-confidence, run [`security-fix-generation`](./PROMPTS.md#security-fix-generation) in Claude Code — feed the finding ID + CWE + vulnerable code; output is a patch with explanation, attack scenario, regression test, related-pattern check, and defence-in-depth notes.

**2.6 — Quality gate behaviour.** Critical / High findings **block merge** until resolved or formally dismissed by Security Champion (with a documented justification in the PR). Medium findings ticket to Linear with a 30-day SLA. Low / informational findings track but don't block.

> **Gate 2 — SAST quality gate** must pass on every PR. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-sast-quality-gate).

---

### Step 3: Dependency Review

> Visual: [Step 3 flowchart](./FLOWCHART.md#step-3-dependency-review)

| Attribute | Detail |
|-----------|--------|
| **Input** | Lockfiles (`package-lock.json`, `pnpm-lock.yaml`, `requirements.txt`, `pom.xml`, `go.sum`, `Cargo.lock`, etc.) |
| **Tools** | **Dependabot** (alerts + security updates + version updates); package-manager-native audit (`npm audit`, `pip-audit`, `cargo audit`) as a supplement; **Claude Code** with [`reachability-triage`](./PROMPTS.md#reachability-triage) and [`dependency-upgrade-impact`](./PROMPTS.md#dependency-upgrade-impact); **Infracost** cross-link from Phase 6 for cost-impact of infrastructure-affecting upgrades |
| **Output** | Auto-PRs for CVE fixes with patch-level auto-merge; a triaged High/Critical CVE list; upgrade-impact analysis attached to every major bump |
| **Human** | Developer reviews / merges Dependabot PRs; Tech Lead approves major version bumps and documents risk acceptance for any unfixed High CVE |

**Workflow:**

**3.1 — Dependabot continuously.** Commit `.github/dependabot.yml` with: weekly schedule for non-security updates, real-time for security updates, auto-merge for patch-level CVE fixes once CI is green, manual review for minor / major updates. Dependabot alerts are the single source of truth for "what CVEs are open against us" — Gate 3 reads from that view, not from a spreadsheet.

**3.2 — Package-manager-native audit as a supplement.** Run the toolchain's own auditor (`npm audit --audit-level=high`, `pip-audit`, `cargo audit`, `go list -m -u all` + `govulncheck`) in the same CI job that installs dependencies. These ship with the toolchain — no new vendor, no extra cost — and they surface advisories against the exact resolved tree in the lockfile, which is a useful cross-check on Dependabot's polling cadence.

**3.3 — Reachability triage on the open CVE list.** Dependabot tells you a vulnerable package is in the tree; it does not tell you whether the vulnerable symbol is on a path anything you ship can reach. When the open High / Critical list runs past ~5 items, export the Dependabot alerts (package, vulnerable version range, patched version, CVE ID, CWE, severity, and the affected symbol where the advisory names one) and run [`reachability-triage`](./PROMPTS.md#reachability-triage) in Claude Code against the repo. Output is the list ranked **Definitely reachable → Probably reachable → Not reachable**, each reachable item carrying the entry point that leads there, the severity if reached, and the quickest mitigation — which is the merge order for the week. Two standing rules: reachability is a **triage aid, not a verdict**, and **Critical / High still get upgraded even when classified Not reachable**, unless the Tech Lead documents a specific exception.

**3.4 — Major-version upgrade impact analysis.** For complex upgrades (major version bumps, deprecated APIs), run [`dependency-upgrade-impact`](./PROMPTS.md#dependency-upgrade-impact) — feed the changelog + the project's usage of the package and Claude returns: breaking changes, migration steps, security fixes included, risk level, test strategy, rollback plan.

**3.5 — Monthly + per-release audits.** First Monday of each month: Security Champion reviews the **Dependabot alerts view** (Security → Dependabot alerts, filtered to open) and files Linear tickets for any neglected High / Critical alert. Pre-release: 0 Critical CVEs in production dependencies; High CVEs require documented Tech Lead risk acceptance.

**3.6 — Cost-of-upgrade cross-check (Phase 6).** When a dependency upgrade implies a base-image change (e.g., Node 20 → 22) that triggers a multi-stage Dockerfile rebuild and a `terraform plan`, cross-link to the [Phase 6 cost-on-PR step](../06-cicd-devops/PROCESS.md#step-2-cost-analysis--finops) so the Infracost delta comment makes the cost impact visible in the same PR.

> **Gate 3 — Dependency hygiene** must pass for every release candidate. See [QUALITY-GATES.md → Gate 3](./QUALITY-GATES.md#gate-3-dependency-hygiene).

---

### Step 4: Container & IaC Review

> Visual: [Step 4 flowchart](./FLOWCHART.md#step-4-container--iac-review)

| Attribute | Detail |
|-----------|--------|
| **Input** | Dockerfiles, container images, IaC files (Terraform / OpenTofu / CloudFormation / K8s manifests) |
| **Tools** | Human review against `AGENTS.md` security conventions + the required-controls list below; Claude Code as a reading aid on large diffs |
| **Output** | Base images digest-pinned; a reviewer-confirmed pass over the required controls, recorded in the PR |
| **Human** | Named reviewer per PR; Security Champion signs off on any documented exception |

> **This step is human review. There is no automated enforcement.** No scanner, policy engine, or admission controller gates container and IaC changes in this phase — the control is a named reviewer working through a fixed list. Stating that plainly is the point: a team that believes a tool is watching this surface will review it less carefully than a team that knows nobody is. Phase 6's pipeline controls are the backstop, not a substitute.

**Workflow:**

**4.1 — Container review.** Every Dockerfile or image change gets a reviewer confirming:

- **Base images are pinned by digest**, not by floating tag. A floating tag means the image you reviewed is not the image that ships.
- The base image is a currently-maintained release line, and the PR states which one and why.
- The Dockerfile follows the `AGENTS.md` security conventions: non-root user, no secrets in build args or layers, minimal installed surface, no `curl | sh` in a build step.
- K8s manifests set a non-root security context, resource limits, and no privileged escalation.

**4.2 — IaC review against the required-controls list.** Every PR touching Terraform / OpenTofu / CloudFormation / K8s manifests is read against this list. Each control is either **satisfied**, or **explicitly excepted with a written reason in the PR** — silence is not a pass.

| Required control | What the reviewer confirms |
|------------------|---------------------------|
| **Required tags** | Every provisioned resource carries the project's mandatory tag set (owner, environment, cost-centre) |
| **Encryption at rest** | Storage, databases, and volumes have encryption enabled with a named key |
| **No public storage** | No object store or bucket is publicly readable or writable |
| **IAM least privilege** | No wildcard actions or wildcard resources in a policy that a narrower grant would satisfy |
| **Region allow-list** | Resources are provisioned only into approved regions (data-residency scope from Phase 1) |
| **Instance-size caps** | No instance or node type above the project's agreed ceiling without Tech Lead sign-off |
| **No root containers** | Container workloads run as a non-root user |
| **Resource limits** | CPU / memory requests and limits are set on every workload |
| **Network policies** | Namespaces carry a default-deny network policy plus explicit allows |

Keep the same list in `AGENTS.md` (Step 0.5) so it is the identical checklist every time, and so Claude Code can pre-read a large IaC diff against it before the human reviewer starts.

> **Gate 4 — Container & IaC review** must pass before any deploy. See [QUALITY-GATES.md → Gate 4](./QUALITY-GATES.md#gate-4-container--iac-review).

---

### Step 5: Secrets — Layered Defence with AI Hooks

> Visual: [Step 5 flowchart](./FLOWCHART.md#step-5-secrets)

| Attribute | Detail |
|-----------|--------|
| **Input** | Every commit (local + pushed), every external channel (Slack, email, paste), every AI-tool prompt and tool call |
| **Tools** | **GitGuardian** (platform, real-time push scan, 500+ detectors) + **ggshield** (pre-commit) + **ggshield AI hook** (March 2026; Claude Code pre-prompt + pre-tool-use + post-tool-use) + **a managed secret manager** for storage; runtime secrets from **a managed secret store** (Phase 6 platform default or cloud-native equivalent) |
| **Output** | Zero secrets in code; zero secrets leaked through AI prompts or tool calls; rotation closure within 1 hour for production-grade leaks |
| **Human** | Every developer runs ggshield + AI hook; Security Champion manages incidents |

**Workflow:**

#### Prevention (continuous)

**5.1 — Pre-commit hook (ggshield).** `ggshield install --mode local` on every developer's machine — blocks commits containing secrets before they reach the remote.

**5.2 — Platform-side push scan (GitGuardian).** Real-time on every push; alerts within seconds. 500+ detectors covers a materially wider range of secret types than the generic entropy-plus-regex approach, which matters most for the long tail of provider-specific token formats.

**5.3 — AI hook — the third layer (ggshield AI hook, March 2026).** Three intercept points around the developer's AI loop:
1. **Pre-prompt** — secret in developer's input → block before model sees it.
2. **Pre-tool-use** — secret in a tool call (`bash` command, file read, MCP call args) → block before the tool runs.
3. **Post-tool-use** — secret in tool output (logs, file contents, MCP responses) → block before output enters the model context.

The data backs the design: GitGuardian's State of Secrets Sprawl 2026 found **28.65M new hardcoded secrets** in 2025 (+34% YoY), AI-tool commits leak at ~3.2% (~2× baseline), and **24,008 unique secrets were found in MCP config files alone**. The AI hook closes the loop the AI assistant opens.

**5.4 — Storage discipline.** Personal and shared secrets live in **a managed secret manager**. Runtime secrets come from **a managed secret store** — the Phase 6 platform default, or the cloud-native equivalent. Never in code, email, Slack, paste buffers, or unencrypted notes.

#### Incident Response (when a secret leaks)

**5.5 — Rotate first, scrub later.** Run [`secrets-incident-response`](./PROMPTS.md#secrets-incident-response) — Claude Code generates a service-specific rotation runbook. Order of operations:

1. **Rotate the credential** (15 minutes from detection, 1 hour for production-grade).
2. **Verify the new credential works** in every environment.
3. **Revoke the old credential** completely.
4. **Audit logs** for unauthorised use of the leaked credential while it was live.
5. **Notify** downstream services and team members affected.
6. **Optional: scrub git history** with `git filter-repo` or BFG — only if compliance requires; rotation is the real mitigation.
7. **Post-mortem** within 48 hours; document the leak path and the process change that prevents recurrence.

**5.6 — The numbers that should keep you honest.** 64% of secrets leaked in 2022 are still active in 2026. Detection without rotation is meaningless. The AI hook stops the new ones; rotation closes the old ones.

> **Gate 5 — Secrets hygiene** must pass continuously. See [QUALITY-GATES.md → Gate 5](./QUALITY-GATES.md#gate-5-secrets-hygiene).

---

### Step 6: AI / Agent-Specific Security

> Visual: [Step 6 flowchart](./FLOWCHART.md#step-6-ai-agent-security)

| Attribute | Detail |
|-----------|--------|
| **Input** | Any AI feature in the product (LLM-backed, agentic, MCP-using, RAG, chatbot, autonomous tooling) |
| **Tools** | **OWASP LLM Top 10** + **OWASP Top 10 for Agentic Applications 2026** as rule sets; **Anthropic Claude Opus 4.7** model-layer prompt-injection defences; `AGENTS.md` + project-scoped `.mcp.json` for MCP enforcement |
| **Output** | AI feature surface threat-modelled and instrumented; MCP server allow-list enforced; AI inventory maintained if compliance requires |
| **Human** | Security Champion + Tech Lead sign off; AI Champion (Phase 3 role) maintains the inventory |

**Workflow:**

**6.1 — Identify whether AI is in the product.** If yes, this step is mandatory. If no (AI is only in tooling / IDEs / CI), the team's exposure is in Steps 0.3 (ggshield AI hook) and 5.3 — and you can skip the rest of Step 6.

**6.2 — Run [`ai-agent-threat-review`](./PROMPTS.md#ai-agent-threat-review).** Same prompt as Step 1.2, focused on the AI feature surface. Required coverage:
- **OWASP LLM Top 10:** prompt injection, insecure output handling, training data poisoning, model DoS, supply-chain (provenance of prompts / fine-tunes), sensitive-information disclosure, insecure plugin/tool design, excessive agency, overreliance, model theft.
- **OWASP Top 10 for Agentic Applications 2026:** ASI01 Agent Goal Hijacking (poisoned inputs hijack objectives), goal misalignment, tool misuse, delegated trust, inter-agent communication, persistent memory, emergent autonomous behaviour. Top risk in 2026 is ASI01 — poisoned inputs in emails / docs / web pages turning the agent against the user.

**6.3 — Model-layer defences (Anthropic prompt-injection defences).** Anthropic's Claude Opus 4.5 published a **1.4% attack success rate** for prompt injection vs 10.8% for Sonnet 4.5 with the previous generation of safeguards — **multi-layer defence: model-trained classifiers + production monitoring / red teaming + server-side prompt-injection probe at input layer**. The team standardises on **Claude Opus 4.7** for the toolchain; treat the published 4.5 numbers as the public benchmark for what these defences buy. **Adaptive attacks still succeed >85%** — agentic security needs defence at every level, not just the model.

**6.4 — MCP enforcement (the new-in-2026 control surface).** MCP servers expose tools to your agent. Each tool can read data, mutate state, or call external services on the user's behalf. The threat: an over-permissive MCP grant becomes a delegated-trust escalation path.

- Maintain the **MCP server allow-list** in `AGENTS.md` — which servers are approved, per environment, and what each may do. Pin the approved set in the repo's **project-scoped `.mcp.json`**, committed to Git, so the allow-list is enforced by what the agent can actually connect to and any drift shows up as a reviewable diff.
- Use [`mcp-enforcement-policy`](./PROMPTS.md#mcp-enforcement-policy) to draft the policy from the agent's intended capabilities.
- Audit the MCP config files — GitGuardian found **24,008 unique secrets in MCP config files** in 2025; the `ggshield` AI hook (Step 5.3) catches new ones.

**6.5 — AI inventory (mostly for compliance).** Maintain a plain-Markdown AI inventory at `/docs/security/ai-inventory.md`, owned by the AI Champion and reviewed quarterly, covering six categories:

1. **AI code assistants** — which tools, which developers, which repos.
2. **AI models** — provider, model version, where each is called from.
3. **AI infrastructure** — inference endpoints, vector stores, gateways, caches.
4. **MCP servers** — the allow-list from 6.4, with scope per server.
5. **AI secrets** — which credentials the AI surface holds, where they are stored, rotation cadence.
6. **AI packages** — SDKs, agent frameworks, and prompt libraries in the dependency tree.

For a compliance-driven org this is the equivalent of an SBOM for the AI surface, and it is the artefact an auditor will ask for first.

**6.6 — Defence-in-depth checks at runtime.** Output validation against an allow-list (don't echo the model's raw text into HTML); rate limits per user / per session; cost caps so a prompt-injection attack cannot rack up an unbounded bill; audit logging of every tool call so an agent goal-hijack is forensically traceable.

> **Gate 6 — AI / agent security** applies when AI is in the product. See [QUALITY-GATES.md → Gate 6](./QUALITY-GATES.md#gate-6-ai-agent-security).

---

### Step 7: Compliance — AI-Generated Checklists, Evidence, and Audit

> Visual: [Step 7 flowchart](./FLOWCHART.md#step-7-compliance)

| Attribute | Detail |
|-----------|--------|
| **Input** | Applicable regulations / frameworks (GDPR, HIPAA, SOC 2, PCI-DSS, ISO 27001 / 27701 / 42001, NIST AI RMF, FedRAMP, CCPA), codebase, infrastructure, security artefacts from Steps 1–6 |
| **Tools** | **Claude Code** with [`compliance-checklist-generation`](./PROMPTS.md#compliance-checklist-generation) + [`evidence-compilation`](./PROMPTS.md#evidence-compilation) + [`security-posture-report`](./PROMPTS.md#security-posture-report) + [`pre-release-self-review`](./PROMPTS.md#pre-release-self-review); CodeQL / Dependabot / GitGuardian / `/security-review` as the automated evidence base |
| **Output** | Checklists per framework, evidence dossier per audit period, audit-ready document with an explicit automated-vs-manual split |
| **Human** | Security Champion validates technical compliance; Legal validates regulatory; external auditor (if certifying) |

> **Start the AI-standards workstream early.** If the product itself contains AI features, plan the **NIST AI RMF + ISO/IEC 42001** work at the beginning of the phase, not at audit time. The official NIST → ISO/IEC 42001 crosswalk is the bridging document; **ISO 42001 is certifiable, NIST AI RMF is not**, and a combined implementation runs **8–12 months**. See 7.4.

**Workflow:**

**7.1 — Generate checklists per applicable framework.** For each in-scope framework, run [`compliance-checklist-generation`](./PROMPTS.md#compliance-checklist-generation) — feed the regulation excerpt + project context + tech stack and Claude returns a structured checklist with Technical Controls (each tied to a verifying tool: CodeQL / `/security-review` / Dependabot / ggshield / GitGuardian / manual), Operational Controls, and Documentation Requirements; categorised Automated / Manual / Requires Legal Review.

**7.2 — Split controls honestly: automated vs manual.** Every technical control in the 7.1 checklist gets classified into exactly one of two buckets, and the classification is the deliverable:

| Bucket | What qualifies | Evidence |
|--------|----------------|----------|
| **Automated** | A control a named tool verifies on a schedule: code-level vulnerability classes (**CodeQL**, with the query or suite named), diff-level review (**`/security-review`**), dependency CVE exposure (**Dependabot**), secrets exposure (**GitGuardian + ggshield**) | Tool output with a timestamp — SARIF artefact, alert export, incident log, PR comment thread |
| **Manual** | Everything else, explicitly including **container and IaC controls** (Step 4), access-control reviews, retention and deletion, vendor management, and every operational and documentation control | A dated attestation naming the person who performed the check, what they inspected, and the outcome |

The failure mode this replaces is a checklist that lists a tool against a control the tool does not actually verify. If no tool covers a control, it goes in the Manual bucket with an owner and a review cadence — **an unverifiable "automated" claim is worse than an honest manual one**, because an auditor will test it.

**7.3 — Evidence compilation per audit period.** Run [`evidence-compilation`](./PROMPTS.md#evidence-compilation) — feed paths to the archived CodeQL SARIF, `/security-review` PR comments, Dependabot alert export, GitGuardian incident log, CI logs, access-control audit logs, the Step 4 review records, and the checklist from 7.1. Output is an audit-ready document mapping each control to specific evidence items with timestamps, gap callouts for any control with insufficient evidence, and auditor-friendly explanations of automated controls.

**7.4 — NIST AI RMF + ISO/IEC 42001 if AI is in the product.** April 7, 2026 NIST released the AI RMF Profile concept note for Trustworthy AI in Critical Infrastructure. The official NIST → ISO/IEC 42001 crosswalk is the bridging document. **ISO 42001 is certifiable; NIST RMF is not.** Combined implementation runs **8–12 months** — start the workstream early. The Step 6 threat model and the six-category AI inventory (6.5) are the technical evidence base.

**7.5 — Per-release pre-deploy self-review.** Run [`pre-release-self-review`](./PROMPTS.md#pre-release-self-review) before clicking the production-deploy approval. Checks: high-risk changes flagged (auth / schema / IAM / networking), 0 Critical SAST findings on `main`, 0 Critical CVEs in production deps, 0 secrets in current code, all in-scope compliance checklists complete, no open GitGuardian incidents.

**7.6 — Quarterly security posture report.** Run [`security-posture-report`](./PROMPTS.md#security-posture-report) — feed counts from CodeQL / `/security-review` / Dependabot / GitGuardian and Claude returns an executive summary, findings trend, top 5 risks, dependency health, secrets hygiene, compliance status, next-period focus. Share with leadership.

> **Gate 7 — Compliance & audit readiness** must pass before any release that crosses a compliance boundary. See [QUALITY-GATES.md → Gate 7](./QUALITY-GATES.md#gate-7-compliance--audit-readiness).

---

## Phase Handoff

When Security & Compliance is complete, the following artefacts hand off to **Phase 6: CI/CD & DevOps** (concurrent) and ultimately to **Phase 7: Delivery & Handoff**:

| Artefact | Format | Location |
|----------|--------|----------|
| Threat model (STRIDE + OWASP LLM/Agentic) | Markdown | `/docs/security/threat-model.md` |
| P0 mitigation verification table (per release candidate) | Markdown | `/docs/security/mitigation-verification-<release>.md` |
| SAST baseline + scheduled scan results | CodeQL SARIF + `/security-review` PR comments | CI artefact storage + GitHub code scanning + GitHub PRs |
| Dependency alert history | Dependabot alerts export | GitHub Security tab |
| Custom CodeQL queries | `.ql` / `.qls` | `.github/codeql/` in Git |
| Container & IaC review record | PR review comments + required-controls confirmation | GitHub PRs |
| Secrets incident log | Private Markdown / GitGuardian export | Internal wiki + GitGuardian |
| AI / agent threat review (if AI in product) | Markdown | `/docs/security/ai-threat-review.md` |
| MCP enforcement policy | Markdown + committed config | `AGENTS.md` + project-scoped `.mcp.json` |
| AI inventory (six categories, if AI in product) | Markdown | `/docs/security/ai-inventory.md` |
| Compliance checklists per framework | Markdown | `/docs/compliance/<framework>.md` |
| Evidence dossier per audit period | Markdown + linked artefacts | `/docs/compliance/evidence/<period>/` |
| Security posture report (quarterly) | Markdown | `/docs/security/posture-<quarter>.md` |

**Handoff Checklist:**

- [ ] All seven gates (1–7) passed (Gate 6 only if AI is in the product)
- [ ] Threat model committed; every P0 mitigation verified landed **in code** by [`threat-model-mitigation-verification`](./PROMPTS.md#threat-model-mitigation-verification) with a `file:line` citation — anything still **not landed** carries a dated Tech Lead deferral
- [ ] 0 Critical / 0 High SAST findings on `main` (CodeQL + `/security-review`)
- [ ] 0 Critical CVEs in production dependencies (Dependabot); High CVEs carry documented Tech Lead risk acceptance
- [ ] Container & IaC review recorded per PR: base images digest-pinned; required-controls list confirmed or exceptions written down
- [ ] 0 secrets detected in current code; no open GitGuardian incidents; ggshield + AI hook installed on every developer machine
- [ ] AI / agent threat review complete (if AI in the product); MCP allow-list enforced in `AGENTS.md` + `.mcp.json`
- [ ] Compliance checklists complete for every applicable framework, with the automated-vs-manual split recorded
- [ ] Evidence dossier archived for the audit period
- [ ] Security posture report shared with leadership for the quarter
- [ ] AGENTS.md security conventions section current
- [ ] All Anthropic admin policies for `/security-review` write scopes reviewed; off-boarded developers' OAuth grants revoked

---

## Risks & Guardrails

| Risk | Mitigation |
|------|------------|
| **AI-generated code ships with 1.7× the vulnerability density of human code** (CodeRabbit / academic data, 2026) | CodeQL + `/security-review` on every PR; the code review checklist's AI-extra-scrutiny section is mandatory for AI-noted PRs. |
| **Alert fatigue — 70% of security alerts go un-actioned once volume outruns triage capacity** | One detector per capability, so there are no duplicate findings to reconcile; CodeQL runs its precision-tuned query suites rather than every available query; `/security-review` is diff-aware; a repeating false positive is fixed by tightening a custom CodeQL query (Step 2.3), not by learning to ignore the detector. |
| **No automated container / IaC policy enforcement in this phase** — a misconfiguration reaches deploy unless a human catches it | Step 4 is explicitly framed as human review, not tooling; the required-controls list is fixed and lives in `AGENTS.md` so it is the same checklist every time; digest-pinned base images remove the largest silent-drift source; Phase 6 pipeline controls are the backstop. Do not let a green pipeline imply this surface was scanned. |
| **Prompt injection against AI features in the product (>85% adaptive-attack success rate)** | Anthropic Claude Opus 4.7 model-layer defences (1.4% baseline ASR for Opus 4.5 vs 10.8% prior gen) + output validation + rate limits + cost caps + audit logs; assume model defences are not enough alone. |
| **Agent goal hijacking (OWASP Agentic Top 10 ASI01)** — poisoned inputs in emails / docs / web hijack agent objectives | Treat any model input from outside the trust boundary as adversarial; whitelist tool calls; require human-in-the-loop for state changes; log every tool invocation for forensic replay. |
| **MCP scope creep** — granting an agent more rights than needed; **24,008 secrets found in MCP config files** (GitGuardian 2026) | MCP allow-list in `AGENTS.md`, pinned by the committed project-scoped `.mcp.json` so drift is a reviewable diff; ggshield AI hook scans MCP config + tool args; off-boarding revokes per-developer OAuth. |
| **Secrets leak via the AI loop** — developer pastes a key into a prompt; tool reads a `.env` and the model echoes it back | ggshield AI hook (March 2026) — pre-prompt + pre-tool-use + post-tool-use; AI-tool commits leak secrets at ~2× baseline (~3.2%) per GitGuardian 2026. |
| **64% of secrets leaked in 2022 are still active in 2026** | Mandatory rotation on detection; historical scan at Step 0.3; quarterly secret-rotation drill; treat detection-without-rotation as a failure. |
| **Threat-model mitigations that never reach the code** — the finding is ticketed, the ticket is closed, and nothing in the codebase actually changed | Step 1.5 [`threat-model-mitigation-verification`](./PROMPTS.md#threat-model-mitigation-verification) runs before every release candidate and demands a `file:line` per P0 mitigation; a comment, a test name, or a closed ticket is not evidence. Gate 7 and phase handoff read from that table rather than from an assertion. |
| **AI-generated security fixes are tautological** (fix that masks the symptom) | Mandatory regression test per fix from [`security-fix-generation`](./PROMPTS.md#security-fix-generation); Security Champion review on every Critical / High fix; mutation testing in Phase 4 catches some of the tautology. |
| **Custom CodeQL queries that never fire** — a query committed without fixture verification manufactures false confidence | [`codeql-custom-query-generation`](./PROMPTS.md#codeql-custom-query-generation) requires two fixtures: the query must fire on the anti-example and stay silent on the allowed example. No fixtures, no merge. |
| **Compliance theatre** — checklists generated but evidence is stale | Evidence compiled at audit close from timestamped sources (archived CodeQL SARIF, Dependabot export, GitGuardian incident log); every manual control carries a named attester and a review date; Security Champion does monthly random sampling of evidence freshness. |
| **Over-reliance on "free + AI auto-fix"** — team stops thinking | Track AI-suggested-vs-actual-root-cause variance per quarter; demote sources whose hypotheses are wrong > 30% of the time; humans still own the incident. |

---

## Daily Security Workflow (steady state)

```
Morning:
  1. Pull latest main
  2. Claude Code → /security-review on the day's branch (diff-aware)
  3. Triage overnight Dependabot PRs; auto-merge passing patch-level CVE fixes
  4. Triage overnight GitGuardian alerts (should be zero); rotate any real leaks within 1h

Per PR:
  5. Open PR → CodeQL diff analysis + /security-review GitHub Action run in CI
  6. Container / IaC changes → named reviewer walks AGENTS.md + the required-controls list
  7. ggshield + GitGuardian platform scan on push
  8. Address Critical / High findings: Copilot Autofix patch, or Claude Code [`security-fix-generation`](./PROMPTS.md#security-fix-generation)
  9. Custom CodeQL query for any project-specific pattern: [`codeql-custom-query-generation`](./PROMPTS.md#codeql-custom-query-generation)
 10. Human approval; merge

Per release:
 11. [`threat-model-mitigation-verification`](./PROMPTS.md#threat-model-mitigation-verification) — every P0 mitigation Landed with a file:line, or deferred with sign-off
 12. [`reachability-triage`](./PROMPTS.md#reachability-triage) on the open Dependabot High / Critical list when it runs past 5 items
 13. [`pre-release-self-review`](./PROMPTS.md#pre-release-self-review) before production-deploy approval
 14. 0 Critical SAST / 0 Critical CVE / 0 secrets / Step 4 review recorded / AI threat review current

Quarterly:
 15. [`security-posture-report`](./PROMPTS.md#security-posture-report) to leadership
 16. Rotate quarterly-cadence secrets; verify the rotation worked
 17. Run secret-rotation + incident-response drill
 18. Re-run the GitGuardian historical scan; review the AI inventory (Step 6.5)

Incident (secret / vulnerability disclosed):
 19. Rotate / patch first; scrub or fix-forward second
 20. [`secrets-incident-response`](./PROMPTS.md#secrets-incident-response) for the runbook
 21. Post-mortem within 48h; process change to prevent recurrence
```

---

## Related Documents

- [Prompt Templates →](./PROMPTS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Process Flowchart →](./FLOWCHART.md)
- [Security & Compliance Tools Evaluation →](../../docs/tools-evaluation/5.Security_Compliance_Phase_Tools.md)
- [Phase 3 Linear MCP setup (carries over) →](../03-development/PROCESS.md#step-0-one-time-setup--connect-claude-code-to-linear-via-mcp)

## External References

- [Claude Code `/security-review` + GitHub Action](https://github.com/anthropics/claude-code-security-review) — official; diff-aware; injection / authn-z / secrets / sensitive-logs
- [Anthropic — `/security-review` announcement](https://x.com/AnthropicAI/status/1953135070174134559)
- [Anthropic — Prompt-injection defences](https://www.anthropic.com/research/prompt-injection-defenses) — Opus 4.5 1.4% ASR vs 10.8% Sonnet 4.5
- [Anthropic — Trustworthy agents](https://www.anthropic.com/research/trustworthy-agents)
- [GitHub — CodeQL code scanning](https://docs.github.com/en/code-security/code-scanning) — setup, PR analysis, scheduled scans, SARIF
- [CodeQL — writing custom queries](https://codeql.github.com/docs/writing-codeql-queries/) — for the `.github/codeql/` project query pack
- [GitHub Copilot Autofix for Code Scanning](https://docs.github.com/en/code-security/concepts/code-scanning/copilot-autofix-for-code-scanning) — GA 2026
- [GitHub — CodeQL PR insights cover all protected branches](https://github.blog/changelog/2026-03-31-codeql-pull-requests-insights-on-security-overview-now-cover-all-protected-branches/)
- [GitHub — Dependabot](https://docs.github.com/en/code-security/dependabot) — alerts, security updates, version updates
- [GitGuardian — ggshield AI hook (March 2026)](https://docs.gitguardian.com/ggshield-docs/integrations/ai-coding-tools/secret-scanning-for-ai-coding-tools)
- [GitGuardian — ggshield AI hook (Help Net Security feature, April 2026)](https://www.helpnetsecurity.com/2026/04/15/product-showcase-gitguardian-ggshield-ai-hook/)
- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) — released Dec 2025
- [OWASP Agentic Skills Top 10](https://owasp.org/www-project-agentic-skills-top-10/)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [NIST AI RMF → ISO/IEC 42001 crosswalk](https://airc.nist.gov/docs/NIST_AI_RMF_to_ISO_IEC_42001_Crosswalk.pdf)
- [AGENTS.md spec](https://agents.md) — open standard for AI coding agent project context
