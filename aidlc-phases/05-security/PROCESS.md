# Phase 5: Security & Compliance — Process Definition

## Overview

This document defines the AI-assisted workflow for the Security & Compliance phase of the AI-DLC. It is the phase where the team's AI productivity gains are stress-tested against an adversarial reality: AI-generated code now ships at ~1.7× the vulnerability density of human-written code, AI coding tools leak secrets in commits at roughly 2× the historical baseline, and the OWASP Top 10 for Agentic Applications (Dec 2025) has added prompt injection, agent goal hijacking, and tool-misuse risks to the list of things every team must defend against. The process below is built around a **free-first, layered AI-native stack** drawn from [`5.Security_Compliance_Phase_Tools.md`](../../docs/tools-evaluation/5.Security_Compliance_Phase_Tools.md): **SonarQube CE + Semgrep MCP + Claude Code `/security-review` + GitHub Copilot Autofix** for SAST; **Dependabot/Renovate + Trivy + Snyk Open Source / Endor Labs** for SCA with reachability; **Trivy MCP + Docker Scout + Checkov + OPA Gatekeeper** for containers and IaC; **GitGuardian + ggshield AI hook** for secrets at every layer including the AI prompt loop; **Cycode AI Governance + AIBOM** (optional) for MCP enforcement and AI inventory; and **Claude + Vanta Agents / Drata MCP** for compliance checklists and evidence.

**Phase Duration:** Continuous from project start + focused audit sprint before each release
**Phase Owner:** Security Champion / Tech Lead
**Tools Used:**

- **SAST:** SonarQube CE (security profile) + Semgrep OSS with **Semgrep MCP server** + **Claude Code `/security-review`** slash command and the **`anthropics/claude-code-security-review` GitHub Action** + **GitHub Copilot Autofix**; **Snyk DeepCode AI / Snyk Agent Fix** as paid upgrade; **Aikido Infinite** as alt all-in-one
- **SCA:** Dependabot (GitHub) or Renovate (other) + **Trivy MCP server** for IDE-driven dep + container + IaC scanning; **Snyk Open Source** or **Endor Labs AURI** for reachability-aware noise reduction
- **Container & IaC:** Trivy + Docker Scout + Checkov + **OPA / Gatekeeper** (with AI policy generation via Red Hat's 2026 dynamic policy generator pattern)
- **Secrets:** GitGuardian platform + **ggshield AI hook** (March 2026) for Cursor / Claude Code / GitHub Copilot — pre-prompt, pre-tool-use, and post-tool-use scanning
- **AI / agent security:** OWASP LLM Top 10 + **OWASP Top 10 for Agentic Applications 2026** mapped per service; **Anthropic Claude Opus 4.7 prompt-injection defences** as the model-layer baseline; **Cycode AI Governance + AIBOM** (optional) for MCP enforcement
- **Compliance:** Claude Code (checklists + evidence) + Trivy/Checkov/OPA (technical controls) + **Vanta Agents** or **Drata MCP** (formal SOC 2 / ISO 27001 / HIPAA only when certifying); NIST AI RMF + ISO/IEC 42001 crosswalk for AI products
- **AI authoring & remediation:** **Claude Code** as the orchestrator, connected to Semgrep MCP, Trivy MCP, GitGuardian, and (optionally) Vanta / Drata MCP servers — same model the team already uses in Phase 3 and Phase 6

> **Tool Philosophy.** Security in 2026 is layered, AI-native, and free at the floor. Traditional SAST generates 80–90% false positives; the layered stack here uses Semgrep MCP and Claude Code's `/security-review` for diff-aware scanning, GitHub Copilot Autofix and Snyk DeepCode for AI fix generation (80% accuracy in Snyk's 2026 benchmarks; 97% triage agreement for Semgrep Assistant), and reachability-aware SCA (Endor Labs AURI / Snyk Open Source) for 95–97% noise reduction. ggshield's AI hook closes the loop the developer's AI assistant opened — pre-prompt, pre-tool-use, post-tool-use. The tool overlap between detectors is only 18–76% (NC State 2025), so layering is the safety net, not the redundancy. Every state-changing remediation requires a human approval gate.

---

## Tool Stack

| Layer | Primary | Fallback | Cost |
|-------|---------|----------|------|
| **SAST baseline** | SonarQube CE (security profile) — already in Phase 3 stack | GitHub Advanced Security (CodeQL + Copilot Autofix, GitHub-only) | Free OSS / GHAS $30/active-committer/mo |
| **SAST custom rules + MCP** | Semgrep OSS + **Semgrep MCP server** (`security_check`, `semgrep_scan_with_custom_rule`, `write_custom_semgrep_rule`) | TruffleHog SAST patterns | Free OSS; Semgrep Pro $35/contributor/mo |
| **AI security review** | **Claude Code `/security-review`** slash command + **`anthropics/claude-code-security-review` GitHub Action** | CodeRabbit security rules (Phase 3 carry-over) | Anthropic API usage-based |
| **AI auto-fix** | **GitHub Copilot Autofix** (free with Code Security licence; included with Copilot in public OSS) | **Snyk DeepCode AI / Snyk Agent Fix** (80% accuracy, up to 5 fix suggestions, 19+ languages) | Copilot Autofix free tier; Snyk free tier 200 tests/mo, paid from $25/dev/mo |
| **All-in-one alt** | — | **Aikido Infinite** (continuous AI pen-test, 95% noise reduction) + **Aikido Endpoint** (developer-workstation security against OSS supply chain + IDE/AI tool attacks, April 2026) | Aikido from $314/mo |
| **SCA — auto-update** | Dependabot (GitHub-native) or Renovate | OWASP Dependency-Check (Apache 2.0; for SBOM) | Free |
| **SCA — reachability** | **Endor Labs AURI** (function-level reachability across 40+ langs; 97% noise cut; AURI for agentic dev launched March 2026) | **Snyk Open Source** (47-day faster CVE detection vs NVD) | Endor Labs enterprise; Snyk free tier 200 tests/mo |
| **Container & IaC scan** | **Trivy** + **Trivy MCP server** (`aquasecurity/trivy-mcp`) for IDE-triggered scans on `package.json` / `requirements.txt` / `Dockerfile` changes | Snyk Container; Aqua Security (enterprise) | Free OSS |
| **Container registry AI** | Docker Scout (AI base-image upgrade suggestions, Phase 6 carry-over) | — | Free tier |
| **IaC compliance** | Checkov (CIS / NIST benchmarks, Apache 2.0) | — | Free OSS |
| **Policy as code** | **OPA / Gatekeeper** with AI policy generation (Red Hat 2026 dynamic Kubernetes policy generator pattern via MCP) | Pulumi CrossGuard (Phase 6 carry-over for IaC-time policy) | Free OSS |
| **Secrets — platform** | **GitGuardian** (500+ detectors) | TruffleHog | Free for teams < 25 devs; per-dev paid above |
| **Secrets — pre-commit** | **ggshield** | Gitleaks (MIT); detect-secrets (Yelp) | Free OSS |
| **Secrets — AI hook** | **ggshield AI hook** (March 2026) — pre-prompt + pre-tool-use + post-tool-use scanning for Cursor / Claude Code / GitHub Copilot | — | Free with ggshield |
| **AI / agent security** | **OWASP LLM Top 10** + **OWASP Top 10 for Agentic Applications 2026** mapped via Claude Code threat-model prompt; **Anthropic Claude Opus 4.7** model-layer prompt-injection defences | **Cycode AI Governance + AIBOM** (AI inventory across 6 categories: code assistants, models, infrastructure, MCP servers, AI secrets, AI packages; MCP server enforcement) | Anthropic defences built-in; Cycode enterprise |
| **Compliance — checklists / evidence** | **Claude Code** + Phase 5 prompt library | — | Already paid |
| **Compliance — formal certification** | **Vanta Agents** (Compliance / TPRM / Customer Trust agents, March 2026; 4 hrs/week saved per user) | **Drata MCP** + Drata Vendor Risk Management Agent | Vanta from $7,500/yr; Drata from $10,000/yr |

**Optional upgrades:**
- **Snyk Code SAST** ($25/dev/mo) — when AI auto-fix PRs become valuable beyond Copilot Autofix and the team needs unified vulnerability management across SAST + SCA + container + IaC.
- **Endor Labs AURI** — for AI-native AppSec when reachability noise reduction (97%) outweighs cost; relevant for OpenAI / Cursor / Snowflake-style profiles.
- **Cycode AI Governance + AIBOM** — when formal AI usage visibility is a board / compliance mandate (shadow AI, AI Bill of Materials, MCP server policy enforcement).
- **Vanta or Drata** — only when formal SOC 2 / ISO 27001 / HIPAA certification is in scope. NIST AI RMF + ISO/IEC 42001 crosswalk applies if the product itself contains AI.

---

## Process Steps

### Step 0: One-Time Setup — Wire AI Security Tools into the Loop

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | GitHub org with Code Security licence (or Snyk CLI auth), Anthropic API key, Semgrep account (free tier OK), Trivy installed, GitGuardian workspace + ggshield, Claude Code installed; for compliance-certifying orgs: Vanta or Drata workspace |
| **Tools** | **Claude Code `/security-review`**, **`anthropics/claude-code-security-review` GitHub Action**, **Semgrep MCP**, **Trivy MCP**, **ggshield AI hook**, GitHub Advanced Security or Snyk CLI, OPA / Conftest, Vanta or Drata MCP (optional) |
| **Output** | Every developer's Claude Code can scan diffs locally and via PR; SAST + SCA + container + secrets all run in CI; AI hook intercepts secrets at prompt and tool-use time; compliance-cert orgs have agent-driven evidence pipelines |
| **Human** | Security Champion authorises org-level connectors; each developer authenticates once via OAuth; Tech Lead expands Anthropic admin policies for security-review write scopes |

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

In the Anthropic admin console expand the connector tool permissions for the security-review workflow: **post comments on PRs** and **create issues** (kept disabled by default). Keep `delete` operations off — humans close findings, the agent does not.

#### 0.2 — Connect Claude Code to the Semgrep MCP server

The Semgrep MCP server (`https://github.com/semgrep/mcp`) exposes `security_check`, `semgrep_scan`, `semgrep_scan_with_custom_rule`, `get_abstract_syntax_tree`, `semgrep_findings`, `supported_languages`, `semgrep_rule_schema`, plus the built-in `write_custom_semgrep_rule` prompt. It works in Cursor / VS Code / Windsurf / Claude Desktop and (via Claude Code's MCP support) in Claude Code itself.

```bash
# Add Semgrep MCP at user scope so it is available in every repo
claude mcp add --scope user semgrep -- npx -y @semgrep/mcp-server@latest
# Verify
claude mcp list   # semgrep: connected
# Smoke test
claude
> Via Semgrep MCP, run security_check on the current diff and summarise findings by severity.
```

#### 0.3 — Connect Claude Code to the Trivy MCP server

Trivy's official `aquasecurity/trivy-mcp` plugin enables natural-language scanning and IDE-triggered scans on changes to `package.json` / `requirements.txt` / `Dockerfile`. Coverage: CVEs in OS + language packages, IaC misconfigs (Dockerfile, Terraform), secrets, licenses.

```bash
trivy plugin install mcp                           # install Trivy MCP plugin
claude mcp add --transport stdio --scope user trivy -- trivy mcp
# Smoke test
claude
> Via Trivy MCP, scan the current Dockerfile and the lockfile; summarise Critical and High CVEs and propose digest-pinned base-image upgrades.
```

#### 0.4 — Install ggshield + the AI hook for Cursor / Claude Code / Copilot

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

# 2. AI hook (developer-side, supports Cursor / Claude Code / Copilot)
ggshield ai install              # configures the hook for detected AI tools
ggshield ai status               # verify which clients are wired up

# 3. Platform-side: enable GitGuardian on every repository
# Settings → Integrations → enable for org; turn on historical scan
```

Run a **one-time historical scan** in the GitGuardian dashboard. 64% of secrets leaked in 2022 are still active in 2026 — the historical sweep almost always finds something to rotate.

#### 0.5 — Wire SCA: Dependabot (or Renovate) + Snyk CLI / GHAS auth

```bash
# GitHub: enable Dependabot security updates + version updates
# Settings → Code security → Dependabot security updates → Enable

# Commit .github/dependabot.yml with auto-merge for patch-level CVE fixes
# (require PR review for minor/major updates)

# Optional reachability layer (paid):
snyk auth                          # if using Snyk Open Source
# or:
endorctl init                      # if using Endor Labs AURI
```

If on GitHub with a Code Security licence, also turn on **Copilot Autofix** in **Settings → Code security → Code scanning** — as of Q2 2026 it covers C# / C / C++ / Go / Java / Kotlin / Swift / JS / TS / Python / Ruby / Rust, with Shell / Dockerfile / Terraform / PHP rolling out via the AI scanning beyond CodeQL. Autofix does **not** require a Copilot subscription and pull-request insights cover all protected branches as of March 2026.

#### 0.6 — Install OPA / Conftest locally

```bash
brew install opa conftest          # macOS; equivalents elsewhere
opa version && conftest --version
```

Author starter Rego policies under `/policy-packs/` (no-public-S3, required-tags, encryption-at-rest, region-allowlist). For Kubernetes, use **OPA Gatekeeper** in the cluster — see Step 4. The Red Hat 2026 dynamic Kubernetes policy generator (which uses an LLM + 30+ best practices via MCP to generate context-aware Gatekeeper policies) is the AI assist for the Rego learning curve.

#### 0.7 — Connect compliance MCPs (only if formally certifying)

```bash
# Vanta Agents (March 2026: Compliance / TPRM / Customer Trust agents)
claude mcp add --transport http --scope user vanta https://mcp.vanta.com/mcp

# Drata MCP (2026 launch)
claude mcp add --transport http --scope user drata https://mcp.drata.com/mcp

# Smoke test
claude
> Via Vanta MCP, list the open Compliance Agent findings for SOC 2 Type II this period and group by control family.
```

If the product itself contains AI features, plan the **NIST AI RMF + ISO/IEC 42001** workstream early — see Step 7. The official NIST → ISO/IEC 42001 crosswalk is the bridging document; combined implementation runs 8–12 months.

#### 0.8 — Add a security conventions section to `AGENTS.md`

The `AGENTS.md` already committed for Phase 6 (and Phase 3) is the canonical project context file — Pulumi Neo, Claude Code, Cursor, and Copilot all read it. Add a **Security conventions** section: forbidden APIs (e.g., raw SQL string concatenation), required input validators, auth middleware that must wrap every endpoint, secret-handling rules ("never call `console.log` on objects that may contain credentials"), and the Phase 3 / 5 AI-generated-code review escalation path. Without this, every prompt drifts; with it, each agent fences itself within team standards.

#### Verification checklist

- [ ] `claude /security-review` runs locally and posts findings on a synthetic vulnerable diff
- [ ] `.github/workflows/claude-security-review.yml` committed; smoke-tested on a draft PR with an injected SQL-i string — Critical comment appears within ~3 minutes
- [ ] `claude mcp list` shows `semgrep`, `trivy`, and (if applicable) `vanta` / `drata` connected
- [ ] `ggshield ai status` confirms hook active for Cursor / Claude Code / Copilot
- [ ] GitGuardian historical scan complete; any leaks routed through Step 5 incident response
- [ ] Dependabot active on every repo; auto-merge configured for patch-level CVE fixes
- [ ] Copilot Autofix on (or Snyk DeepCode auth'd) for the languages in scope
- [ ] OPA / Conftest installed locally; starter policy packs committed
- [ ] `AGENTS.md` updated with **Security conventions** section
- [ ] Anthropic admin policies expanded: `/security-review` may post comments and create issues; `delete` operations remain disabled

> **Permission inheritance.** Every MCP-driven action runs as the connecting developer — the MCP server cannot escalate beyond their GitHub / Semgrep / Trivy / GitGuardian / Vanta scopes. Off-board by revoking the OAuth grant in each provider; the agent loses access automatically.

---

### Step 1: Threat Modelling & Security Architecture Review

> Visual: [Step 1 flowchart](./FLOWCHART.md#step-1-threat-modelling)

| Attribute | Detail |
|-----------|--------|
| **Input** | Phase 2 architecture proposal, Phase 1 NFRs, data classification, compliance scope |
| **Tools** | **Claude Code** with [`threat-model-stride`](./PROMPTS.md#threat-model-stride) and [`ai-agent-threat-review`](./PROMPTS.md#ai-agent-threat-review) prompts; OWASP Top 10 + OWASP API Top 10 + **OWASP LLM Top 10** + **OWASP Top 10 for Agentic Applications 2026** as rule sets |
| **Output** | `/docs/security/threat-model.md` with STRIDE issues + AI/agent issues, prioritised mitigations, P0/P1/P2 timeline |
| **Human** | Security Champion validates and prioritises; Tech Lead signs off on must-fix-before-launch items |

**Workflow:**

**1.1 — STRIDE pass against the architecture.** From Claude Code, run [`threat-model-stride`](./PROMPTS.md#threat-model-stride) against the Phase 2 architecture description and diagrams. Output: STRIDE issues (Spoofing / Tampering / Repudiation / Information Disclosure / DoS / Elevation of Privilege) per component, severity, recommendation, priority. Cross-reference OWASP Top 10 and OWASP API Top 10.

**1.2 — AI / agent threat pass (only if AI features are in the product).** Run [`ai-agent-threat-review`](./PROMPTS.md#ai-agent-threat-review) — covers **OWASP LLM Top 10** (prompt injection, insecure output handling, training-data poisoning, model DoS, supply-chain, sensitive-information disclosure, insecure plugin design, excessive agency, overreliance, model theft) **and** the new **OWASP Top 10 for Agentic Applications 2026** (released Dec 2025, peer-reviewed by 100+ experts). The top agentic risk is **ASI01: Agent Goal Hijacking** — poisoned inputs in emails / docs / web pages hijack the agent's objective. The pass also covers goal misalignment, tool misuse, delegated trust, inter-agent communication, persistent memory, emergent autonomous behaviour.

**1.3 — Document and triage.** Commit the threat model at `/docs/security/threat-model.md`. Each finding has Issue / STRIDE-or-OWASP-Agentic category / Severity / Affected component / Recommendation / Priority (P0 must-fix before launch / P1 within 30 days / P2 within 90 days). Open Linear issues for every P0 and P1.

**1.4 — Wire mitigations into downstream phases.** P0 mitigations become acceptance criteria on Phase 3 stories or Phase 6 CrossGuard policies. **Don't leave threat-model findings on a wiki** — they have to land in the backlog or they don't ship.

> **Gate 1 — Threat model & baseline scanning** must pass before a release candidate is built. See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-threat-model--baseline).

---

### Step 2: SAST — Continuous AI-Assisted Static Analysis

> Visual: [Step 2 flowchart](./FLOWCHART.md#step-2-sast)

| Attribute | Detail |
|-----------|--------|
| **Input** | Every PR + nightly full-repo scan |
| **Tools** | **SonarQube CE** (security profile, already in Phase 3) + **Semgrep OSS via Semgrep MCP** + **Claude Code `/security-review`** + **`anthropics/claude-code-security-review` GitHub Action** + **GitHub Copilot Autofix**; **Snyk DeepCode AI / Snyk Agent Fix** (paid upgrade, 80% accuracy, up to 5 fix suggestions); **Aikido Infinite** as alt all-in-one |
| **Output** | PR-level findings with severity buckets (Critical / High / Medium / Low) + auto-fix PRs for AI-fixable issues + nightly trend report |
| **Human** | Developer addresses findings; Security Champion approves any dismissal of Critical / High findings |

**Workflow:**

**2.1 — Baseline SAST in CI (SonarQube + Semgrep).** SonarQube CE with the security rule profile is already running in CI from Phase 3 — confirm the security gate is set to **0 Critical, 0 High** (not just code-quality rules). Add Semgrep as a separate scan job: enable the OWASP Top 10 ruleset + language-specific rules + project-specific custom rules in `.semgrep/rules/`. Schedule **nightly full-repo scans** + **per-PR diff scans**.

**2.2 — `/security-review` on every PR.** The `anthropics/claude-code-security-review` GitHub Action runs on `pull_request: opened` and `synchronize`. It scans the diff (not the full repo — that is SonarQube + Semgrep's job) for: SQL / cmd / LDAP / XPath / NoSQL / XXE injection, authn/authz, hardcoded secrets, sensitive-data logging. Findings post as PR comments bucketed Critical / High / Medium / Low. **Diff-aware filtering** keeps signal-to-noise high — the FP rate is materially lower than traditional SAST.

**2.3 — Author custom Semgrep rules via MCP for project-specific patterns.** When a finding is project-specific ("we never call `eval` on user input", "every endpoint must call `requireAuth`", "no `dangerouslySetInnerHTML` outside `/components/admin/`"), use Claude Code's [`semgrep-custom-rule-generation`](./PROMPTS.md#semgrep-custom-rule-generation) prompt — it leverages the Semgrep MCP server's built-in `write_custom_semgrep_rule` prompt. Commit the rule under `.semgrep/rules/` so every future PR is gated.

**2.4 — AI auto-fix path (Copilot Autofix or Snyk DeepCode).**
- **GitHub Copilot Autofix (default if on GHAS):** GA in 2026; covers C# / C / C++ / Go / Java / Kotlin / Swift / JS / TS / Python / Ruby / Rust; expanding to Shell / Dockerfile / Terraform / PHP via AI scanning beyond CodeQL. Autofix posts a suggested patch on every CodeQL finding; review and accept inline.
- **Snyk DeepCode AI / Snyk Agent Fix (paid upgrade):** AI Security Fabric (Feb 2026), 80% auto-fix accuracy, presents up to 5 fix suggestions per finding, Symbolic AI + Generative AI hybrid, 19+ languages, Eclipse + VS support added. Use when the team needs IDE-time fix suggestions during AI-assisted development.

**2.5 — Manual remediation via Claude Code for the rest.** When auto-fix is unavailable or low-confidence, run [`security-fix-generation`](./PROMPTS.md#security-fix-generation) in Claude Code — feed the finding ID + CWE + vulnerable code; output is a patch with explanation, attack scenario, regression test, related-pattern check, and defence-in-depth notes.

**2.6 — Quality gate behaviour.** Critical / High findings **block merge** until resolved or formally dismissed by Security Champion (with a documented justification in the PR). Medium findings ticket to Linear with a 30-day SLA. Low / informational findings track but don't block.

**2.7 — Aikido Infinite (alt path).** If the team wants a single SaaS for SAST + SCA + container + IaC + continuous AI pen-testing, **Aikido Infinite** (Feb 2026) advertises 95% noise reduction with AutoFix in code, deps, and IaC. **Aikido Endpoint** (April 2026) extends to developer-workstation security against the supply-chain attacks now targeting OSS packages, IDE extensions, and AI dev tools. Cost is from $314/mo — only worth it as a Semgrep + Snyk consolidation.

> **Gate 2 — SAST quality gate** must pass on every PR. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-sast-quality-gate).

---

### Step 3: SCA — Reachability-Aware Dependency Audit

> Visual: [Step 3 flowchart](./FLOWCHART.md#step-3-sca)

| Attribute | Detail |
|-----------|--------|
| **Input** | Lockfiles (`package-lock.json`, `pnpm-lock.yaml`, `requirements.txt`, `pom.xml`, `go.sum`, `Cargo.lock`, etc.); base-image manifests |
| **Tools** | **Dependabot** (GitHub) or **Renovate** (others) + **Trivy via MCP** for IDE-time + CI scans; **Snyk Open Source** or **Endor Labs AURI** for reachability; **Pulumi Insights** cross-link from Phase 6 for cost-impact of major upgrades |
| **Output** | Auto-PRs for CVE fixes with patch-level auto-merge; reachability-triaged High/Critical CVE list (95–97% noise reduction); SBOM if compliance requires |
| **Human** | Developer reviews / merges Dependabot PRs; Tech Lead approves major version bumps |

**Workflow:**

**3.1 — Dependabot / Renovate continuously.** Commit `.github/dependabot.yml` (or `renovate.json`) with: weekly schedule for non-security updates, real-time for security updates, auto-merge for patch-level CVE fixes once CI is green, manual review for minor / major updates.

**3.2 — Trivy in CI on every PR.** Run `trivy fs --severity CRITICAL,HIGH .` on the lockfile diff and `trivy image <built-image>` on the container build. **Fail builds on Critical CVEs** in base images or installed packages. Use the [`container-cve-triage`](./PROMPTS.md#container-cve-triage) prompt when Trivy reports more than 5 Critical / High CVEs — Claude Code suggests the smallest base-image upgrade that closes the most CVEs.

**3.3 — Reachability-aware triage (the noise cut).** Traditional SAST / SCA generates 80–90% false positives; **reachability analysis** (does our code path actually call into the vulnerable function?) reduces alert noise by 70–97%.

- **Endor Labs AURI** (primary if budget allows) — function-level reachability across 40+ languages; AURI for AI-driven coding launched March 2026; the Autonomous Plane acquisition (Feb 2026) gives full-stack reachability code-to-container. Customers include OpenAI, Cursor, Snowflake, Atlassian.
- **Snyk Open Source** (alt) — reachability + auto-fix PRs trained on millions of human-written fixes; 47-day faster CVE detection vs NVD.

When reachability is unavailable, run [`reachability-triage`](./PROMPTS.md#reachability-triage) in Claude Code: feed the CVE list + the project's call graph (or a tree-sitter-derived map) and Claude returns a ranked "definitely reachable / probably reachable / not reachable in current code" classification. **Treat this as a noise-reduction aid, not a final answer** — Critical and High findings still go through the full upgrade flow.

**3.4 — Major-version upgrade impact analysis.** For complex upgrades (major version bumps, deprecated APIs), run [`dependency-upgrade-impact`](./PROMPTS.md#dependency-upgrade-impact) — feed the changelog + the project's usage of the package and Claude returns: breaking changes, migration steps, security fixes included, risk level, test strategy, rollback plan.

**3.5 — Monthly + per-release audits.** First Monday of each month: Security Champion reviews the dashboard of outstanding dependency issues, files Linear tickets for any neglected High / Critical issues. Pre-release: 0 Critical CVEs in production dependencies; High CVEs require documented Tech Lead risk acceptance.

**3.6 — SBOM if compliance requires.** Add **OWASP Dependency-Check** for SBOM in CycloneDX or SPDX format when government / enterprise contracts require it. Free, integrates with Maven / Gradle / npm / pip, produces auditor-ready HTML.

**3.7 — Cost-of-upgrade cross-check (Phase 6).** When a dependency upgrade implies a base-image change (e.g., Node 20 → 22) that triggers a multi-stage Dockerfile rebuild and a Pulumi preview, cross-link to the [Phase 6 `pulumi-cost-delta` prompt](../06-cicd-devops/PROMPTS.md#pulumi-cost-delta) so the cost impact is visible in the same PR.

> **Gate 3 — SCA dependency hygiene** must pass for every release candidate. See [QUALITY-GATES.md → Gate 3](./QUALITY-GATES.md#gate-3-sca-dependency-hygiene).

---

### Step 4: Container & IaC Security

> Visual: [Step 4 flowchart](./FLOWCHART.md#step-4-container-iac)

| Attribute | Detail |
|-----------|--------|
| **Input** | Dockerfiles, container images, IaC files (Terraform / Pulumi / CloudFormation / K8s manifests) |
| **Tools** | **Trivy** (CVEs + IaC misconfigs + secrets + licenses) + **Trivy MCP** (IDE-time + Claude Code-driven scans) + **Docker Scout** (registry-side AI base-image upgrades) + **Checkov** (CIS / NIST) + **OPA / Gatekeeper** with AI policy generation |
| **Output** | 0 Critical CVEs in built images; CIS/NIST-compliant IaC; Gatekeeper policies enforcing required tags, encryption-at-rest, no-public-S3, IAM least-privilege; AI-generated Rego with mandatory dryrun-first review |
| **Human** | Review every AI-generated policy in dryrun before enforcement; Security Champion signs off on policy enforcement scope |

**Workflow:**

**4.1 — Trivy on every container build (CI).** Already wired in Phase 6. Confirm: `trivy image --severity CRITICAL,HIGH --exit-code 1 <image>` fails the build on Critical / High CVEs. Pin base images by digest, not floating tags. Use [`container-cve-triage`](./PROMPTS.md#container-cve-triage) when Trivy reports a long CVE list.

**4.2 — Docker Scout on the registry (AI-suggested base-image upgrades).** Already wired in Phase 6. Review and merge the AI-suggested base-image upgrade PRs that fix the most CVEs with the smallest version bump.

**4.3 — Checkov on every IaC PR.** `checkov -d /path/to/iac --soft-fail-on LOW --hard-fail-on HIGH` on every PR that touches Terraform / CloudFormation / Kubernetes manifests / Dockerfiles / Helm. Wire to the GitHub Actions step or pre-commit hook.

**4.4 — OPA / Gatekeeper for runtime policy.** Author Rego in `/policy-packs/`. Required policies at minimum: required-tags, encryption-at-rest, no-public-S3, IAM least-privilege, region-allowlist, instance-size caps, no-root-containers, resource limits enforced, network-policies present. Run `conftest test` in CI; deploy Gatekeeper to the cluster for runtime enforcement.

**4.5 — AI policy generation (the Rego tax killer).** Authoring Rego is the most-cited friction point with Gatekeeper. Use [`opa-policy-generation`](./PROMPTS.md#opa-policy-generation) — Claude Code generates a Gatekeeper ConstraintTemplate + Constraint pair from a plain-English requirement ("no public S3 buckets in production"). The Red Hat 2026 dynamic Kubernetes policy generator pattern is the template: an LLM analyses live cluster state + 30+ best practices via MCP to produce context-aware policies.

> **Mandatory dryrun-first.** Every AI-generated policy is deployed in `enforcementAction: dryrun` first. Watch the Gatekeeper audit logs for at least 7 days; only flip to `deny` after zero false positives. Skipping dryrun is how a single-line policy locks every deploy across the cluster.

**4.6 — Trivy MCP for IDE-time scans.** Trivy MCP triggers scans on file changes (`package.json`, `requirements.txt`, `Dockerfile`). Developers see CVEs in-editor before commit. Smoke test from Claude Code: `Via Trivy MCP, scan the staged Dockerfile and the lockfile; flag Critical CVEs with the smallest base-image upgrade that closes them.`

**4.7 — Cross-link to Phase 6 CrossGuard.** If the project is on Pulumi (Phase 6 default), CrossGuard policy packs run on every `pulumi preview` and gate `pulumi up`. The Phase-5 OPA Rego policies feed the Phase-6 CrossGuard packs through the `pulumi-policy-opa` bridge — author once, enforce both at IaC-time and at runtime.

> **Gate 4 — Container & IaC compliance** must pass before any deploy. See [QUALITY-GATES.md → Gate 4](./QUALITY-GATES.md#gate-4-container--iac-compliance).

---

### Step 5: Secrets — Layered Defence with AI Hooks

> Visual: [Step 5 flowchart](./FLOWCHART.md#step-5-secrets)

| Attribute | Detail |
|-----------|--------|
| **Input** | Every commit (local + pushed), every external channel (Slack, email, paste), every AI-tool prompt and tool call |
| **Tools** | **GitGuardian** (platform, real-time push scan, 500+ detectors) + **ggshield** (pre-commit) + **ggshield AI hook** (March 2026; Cursor / Claude Code / Copilot pre-prompt + pre-tool-use + post-tool-use) + **Bitwarden / 1Password** (storage); platform-side runtime secrets via Pulumi ESC (Phase 6) or AWS Secrets Manager / Azure Key Vault / GCP Secret Manager |
| **Output** | Zero secrets in code; zero secrets leaked through AI prompts or tool calls; rotation closure within 1 hour for production-grade leaks |
| **Human** | Every developer runs ggshield + AI hook; Security Champion manages incidents |

**Workflow:**

#### Prevention (continuous)

**5.1 — Pre-commit hook (ggshield).** `ggshield install --mode local` on every developer's machine — blocks commits containing secrets before they reach the remote.

**5.2 — Platform-side push scan (GitGuardian).** Real-time on every push; alerts within seconds. 500+ detectors covers more secret types than any OSS alternative (Gitleaks / TruffleHog cover ~200–350).

**5.3 — AI hook — the third layer (ggshield AI hook, March 2026).** Three intercept points around the developer's AI loop:
1. **Pre-prompt** — secret in developer's input → block before model sees it.
2. **Pre-tool-use** — secret in a tool call (`bash` command, file read, MCP call args) → block before the tool runs.
3. **Post-tool-use** — secret in tool output (logs, file contents, MCP responses) → block before output enters the model context.

The data backs the design: GitGuardian's State of Secrets Sprawl 2026 found **28.65M new hardcoded secrets** in 2025 (+34% YoY), AI-tool commits leak at ~3.2% (~2× baseline), and **24,008 unique secrets were found in MCP config files alone**. The AI hook closes the loop the AI assistant opens.

**5.4 — Storage discipline.** Personal: Bitwarden / 1Password. Shared: 1Password Teams / AWS Secrets Manager / Azure Key Vault / GCP Secret Manager. Runtime: Pulumi ESC (Phase 6 default) or platform-native. Never in code, email, Slack, paste buffers, or unencrypted notes.

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
| **Tools** | **OWASP LLM Top 10** + **OWASP Top 10 for Agentic Applications 2026** as rule sets; **Anthropic Claude Opus 4.7** model-layer prompt-injection defences; **Cycode AI Governance + AIBOM** (optional) for AI inventory and MCP enforcement |
| **Output** | AI feature surface threat-modelled and instrumented; MCP server allow-list enforced; AIBOM / AI inventory if compliance requires |
| **Human** | Security Champion + Tech Lead sign off; AI Champion (Phase 3 role) maintains the inventory |

**Workflow:**

**6.1 — Identify whether AI is in the product.** If yes, this step is mandatory. If no (AI is only in tooling / IDEs / CI), the team's exposure is in Steps 0.4 (ggshield AI hook) and 5.3 — and you can skip the rest of Step 6.

**6.2 — Run [`ai-agent-threat-review`](./PROMPTS.md#ai-agent-threat-review).** Same prompt as Step 1.2, focused on the AI feature surface. Required coverage:
- **OWASP LLM Top 10:** prompt injection, insecure output handling, training data poisoning, model DoS, supply-chain (provenance of prompts / fine-tunes), sensitive-information disclosure, insecure plugin/tool design, excessive agency, overreliance, model theft.
- **OWASP Top 10 for Agentic Applications 2026:** ASI01 Agent Goal Hijacking (poisoned inputs hijack objectives), goal misalignment, tool misuse, delegated trust, inter-agent communication, persistent memory, emergent autonomous behaviour. Top risk in 2026 is ASI01 — poisoned inputs in emails / docs / web pages turning the agent against the user.

**6.3 — Model-layer defences (Anthropic prompt-injection defences).** Anthropic's Claude Opus 4.5 published a **1.4% attack success rate** for prompt injection vs 10.8% for Sonnet 4.5 with the previous generation of safeguards — **multi-layer defence: model-trained classifiers + production monitoring / red teaming + server-side prompt-injection probe at input layer**. The team standardises on **Claude Opus 4.7** for the toolchain; treat the published 4.5 numbers as the public benchmark for what these defences buy. **Adaptive attacks still succeed >85%** — agentic security needs defence at every level, not just the model.

**6.4 — MCP enforcement (the new-in-2026 control surface).** MCP servers expose tools to your agent. Each tool can read data, mutate state, or call external services on the user's behalf. The threat: an over-permissive MCP grant becomes a delegated-trust escalation path.

- Maintain an **MCP server allow-list** in `AGENTS.md` and (if Cycode is in use) in **Cycode AI Governance**. Cycode's three-layer governance (see / govern / enforce) controls which MCP servers can be reached, what data they can read, what actions they can take.
- Use [`mcp-enforcement-policy`](./PROMPTS.md#mcp-enforcement-policy) to draft the policy from the agent's intended capabilities.
- Audit the MCP config files — GitGuardian found **24,008 unique secrets in MCP config files** in 2025; the `ggshield` AI hook (Step 5.3) catches new ones.

**6.5 — AI inventory / AIBOM (optional, mostly for compliance).** **Cycode AI Governance + AIBOM** (Feb 2026) tracks AI Inventory across 6 categories: AI code assistants, AI models, AI infrastructure, MCP servers, AI secrets, AI packages. 81% of orgs lack visibility (Cycode State of Product Security 2026). For a compliance-driven org, AIBOM is the equivalent of SBOM for the AI surface.

**6.6 — Defence-in-depth checks at runtime.** Output validation against an allow-list (don't echo the model's raw text into HTML); rate limits per user / per session; cost caps so a prompt-injection attack cannot rack up an unbounded bill; audit logging of every tool call so an agent goal-hijack is forensically traceable.

> **Gate 6 — AI / agent security** applies when AI is in the product. See [QUALITY-GATES.md → Gate 6](./QUALITY-GATES.md#gate-6-ai-agent-security).

---

### Step 7: Compliance — AI-Generated Checklists, Evidence, and Audit

> Visual: [Step 7 flowchart](./FLOWCHART.md#step-7-compliance)

| Attribute | Detail |
|-----------|--------|
| **Input** | Applicable regulations / frameworks (GDPR, HIPAA, SOC 2, PCI-DSS, ISO 27001 / 27701 / 42001, NIST AI RMF, FedRAMP, CCPA), codebase, infrastructure, security artefacts from Steps 1–6 |
| **Tools** | **Claude Code** with [`compliance-checklist-generation`](./PROMPTS.md#compliance-checklist-generation) + [`evidence-compilation`](./PROMPTS.md#evidence-compilation) + [`security-posture-report`](./PROMPTS.md#security-posture-report) + [`pre-release-self-review`](./PROMPTS.md#pre-release-self-review); Trivy + Checkov + OPA for technical controls; **Vanta Agents** or **Drata MCP** when formally certifying |
| **Output** | Checklists per framework, evidence dossier per audit period, audit-ready document; Vanta / Drata 100% control coverage if certifying |
| **Human** | Security Champion validates technical compliance; Legal validates regulatory; external auditor (if certifying) |

**Workflow:**

**7.1 — Generate checklists per applicable framework.** For each in-scope framework, run [`compliance-checklist-generation`](./PROMPTS.md#compliance-checklist-generation) — feed the regulation excerpt + project context + tech stack and Claude returns a structured checklist with Technical Controls (each tied to a verifying tool: SonarQube / Trivy / Checkov / OPA / manual), Operational Controls, and Documentation Requirements; categorised Automated / Manual / Requires Legal Review.

**7.2 — Map technical controls to the running stack.** Trivy and Checkov for IaC compliance against CIS / NIST. OPA / Gatekeeper for runtime policy. SonarQube + Semgrep for code-level controls. The map produced in 7.1 links each control to the specific scanner / policy file; the Security Champion fills any gaps.

**7.3 — Evidence compilation per audit period.** Run [`evidence-compilation`](./PROMPTS.md#evidence-compilation) — feed paths to scan reports, CI logs, GitGuardian incident log, access-control audit logs, and the checklist from 7.1. Output is an audit-ready document mapping each control to specific evidence items with timestamps, gap callouts for any control with insufficient evidence, and auditor-friendly explanations of automated controls.

**7.4 — Vanta Agents or Drata MCP for formal certification.** Only when SOC 2 / ISO 27001 / HIPAA / FedRAMP-adjacent certification is in play.

- **Vanta Agents (March 2026):** three agents — **Compliance Agent** (evidence lifecycle, policy inconsistencies, remediation), **TPRM Agent** (vendor risk; 81% faster security reviews per Vanta), **Customer Trust Agent** (security questionnaires). 4 hrs/week saved per user. 80% accuracy improvement. 400+ integrations. Tests run hourly.
- **Drata 2026:** AI-native Trust Management; **Drata MCP**; Vendor Risk Management Agent. 170+ integrations.

The MCP servers from Step 0.7 let Claude Code drive the platforms in-session: fetch open findings, group by control family, draft remediation tickets back into Linear.

**7.5 — NIST AI RMF + ISO/IEC 42001 if AI is in the product.** April 7, 2026 NIST released the AI RMF Profile concept note for Trustworthy AI in Critical Infrastructure. The official NIST → ISO/IEC 42001 crosswalk is the bridging document. **ISO 42001 is certifiable; NIST RMF is not.** Combined implementation runs **8–12 months** — start the workstream early. The Step 6 threat model and AIBOM (if Cycode is in scope) are the technical evidence base.

**7.6 — Per-release pre-deploy self-review.** Run [`pre-release-self-review`](./PROMPTS.md#pre-release-self-review) before clicking the production-deploy approval. Checks: high-risk changes flagged (auth / schema / IAM / networking), 0 Critical SAST findings on `main`, 0 Critical CVEs in production deps, 0 secrets in current code, all in-scope compliance checklists complete, no open GitGuardian incidents.

**7.7 — Quarterly security posture report.** Run [`security-posture-report`](./PROMPTS.md#security-posture-report) — feed counts from SonarQube / Semgrep / Dependabot / Trivy / GitGuardian and Claude returns an executive summary, findings trend, top 5 risks, dependency health, secrets hygiene, compliance status, next-period focus. Share with leadership.

> **Gate 7 — Compliance & audit readiness** must pass before any release that crosses a compliance boundary. See [QUALITY-GATES.md → Gate 7](./QUALITY-GATES.md#gate-7-compliance--audit-readiness).

---

## Phase Handoff

When Security & Compliance is complete, the following artefacts hand off to **Phase 6: CI/CD & DevOps** (concurrent) and ultimately to **Phase 7: Delivery & Handoff**:

| Artefact | Format | Location |
|----------|--------|----------|
| Threat model (STRIDE + OWASP LLM/Agentic) | Markdown | `/docs/security/threat-model.md` |
| SAST baseline + nightly scan results | SonarQube + Semgrep reports + `/security-review` PR comments | SonarQube + GitHub PRs + CI artefact storage |
| SCA dependency dashboard | Dependabot / Snyk / Endor Labs | GitHub UI / Snyk / Endor Labs UI |
| Container scan history | Trivy SARIF | CI artefact storage |
| IaC compliance reports | Checkov + OPA / Gatekeeper audit | CI artefact storage + cluster audit log |
| Custom Semgrep rules | YAML | `.semgrep/rules/` in Git |
| OPA / Gatekeeper policy packs | Rego + ConstraintTemplates | `/policy-packs/` in Git |
| Secrets incident log | Private Markdown / GitGuardian export | Internal wiki + GitGuardian |
| AI / agent threat review (if AI in product) | Markdown | `/docs/security/ai-threat-review.md` |
| MCP enforcement policy | Markdown + YAML | `AGENTS.md` + `/policy-packs/mcp/` |
| AIBOM (if Cycode in scope) | JSON / CSV export | Cycode dashboard |
| Compliance checklists per framework | Markdown | `/docs/compliance/<framework>.md` |
| Evidence dossier per audit period | Markdown + linked artefacts | `/docs/compliance/evidence/<period>/` |
| Security posture report (quarterly) | Markdown | `/docs/security/posture-<quarter>.md` |
| Vanta / Drata dashboard (if certifying) | SaaS | Vanta / Drata UI |

**Handoff Checklist:**

- [ ] All seven gates (1–7) passed (Gate 6 only if AI is in the product)
- [ ] Threat model committed and P0 mitigations landed in code or backlog
- [ ] 0 Critical / 0 High SAST findings on `main` (SonarQube + Semgrep + `/security-review`)
- [ ] 0 Critical CVEs in production dependencies (Dependabot + Trivy + reachability)
- [ ] All container images scan clean (0 Critical CVEs at registry push)
- [ ] All IaC passes Checkov / OPA / Gatekeeper (dryrun-validated for AI-generated policies)
- [ ] 0 secrets detected in current code; no open GitGuardian incidents; ggshield + AI hook installed on every developer machine
- [ ] AI / agent threat review complete (if AI in the product); MCP allow-list enforced
- [ ] Compliance checklists complete for every applicable framework
- [ ] Evidence dossier archived for the audit period
- [ ] Vanta / Drata dashboard at 100% control coverage (if certifying)
- [ ] Security posture report shared with leadership for the quarter
- [ ] AGENTS.md security conventions section current
- [ ] All Anthropic admin policies for `/security-review` write scopes reviewed; off-boarded developers' OAuth grants revoked

---

## Risks & Guardrails

| Risk | Mitigation |
|------|------------|
| **AI-generated code ships with 1.7× the vulnerability density of human code** (CodeRabbit / academic data, 2026) | SonarQube + Semgrep + `/security-review` on every PR; Code Review Checklist's AI-extra-scrutiny section ([`templates/code-review-checklist.md`](../../templates/code-review-checklist.md)) is mandatory for AI-noted PRs. |
| **Alert fatigue from 80–90% traditional SAST false positives, with 70% of alerts ignored** | Diff-aware `/security-review`; reachability-aware SCA (Endor Labs / Snyk); per-PR scan limits; tool overlap is 18–76% (NC State 2025) so the layered stack is the safety net, not redundancy. |
| **Prompt injection against AI features in the product (>85% adaptive-attack success rate)** | Anthropic Claude Opus 4.7 model-layer defences (1.4% baseline ASR for Opus 4.5 vs 10.8% prior gen) + output validation + rate limits + cost caps + audit logs; assume model defences are not enough alone. |
| **Agent goal hijacking (OWASP Agentic Top 10 ASI01)** — poisoned inputs in emails / docs / web hijack agent objectives | Treat any model input from outside the trust boundary as adversarial; whitelist tool calls; require human-in-the-loop for state changes; log every tool invocation for forensic replay. |
| **MCP scope creep** — granting an agent more rights than needed; **24,008 secrets found in MCP config files** (GitGuardian 2026) | MCP allow-list in `AGENTS.md`; Cycode MCP enforcement (if in scope); ggshield AI hook scans MCP config + tool args; off-boarding revokes per-developer OAuth. |
| **Secrets leak via the AI loop** — developer pastes a key into a prompt; tool reads a `.env` and the model echoes it back | ggshield AI hook (March 2026) — pre-prompt + pre-tool-use + post-tool-use; AI-tool commits leak secrets at ~2× baseline (~3.2%) per GitGuardian 2026. |
| **64% of secrets leaked in 2022 are still active in 2026** | Mandatory rotation on detection; historical scan at Step 0.4; quarterly secret-rotation drill; treat detection-without-rotation as a failure. |
| **AI-generated OPA / Rego policies enforced without dryrun** | `enforcementAction: dryrun` mandatory for at least 7 days; flip to `deny` only after zero-FP audit window; the Red Hat 2026 generator pattern bakes dryrun-first into the workflow. |
| **AI-generated security fixes are tautological** (fix that masks the symptom) | Mandatory regression test per fix from [`security-fix-generation`](./PROMPTS.md#security-fix-generation); Security Champion review on every Critical / High fix; mutation testing in Phase 4 catches some of the tautology. |
| **Compliance theatre** — checklists generated but evidence is stale | Vanta / Drata agents auto-collect (hourly tests); evidence compilation at audit close; Security Champion does monthly random sampling of evidence freshness. |
| **Over-reliance on "free + AI auto-fix"** — team stops thinking | Track AI-suggested-vs-actual-root-cause variance per quarter; demote sources whose hypotheses are wrong > 30% of the time; humans still own the incident. |
| **Premature investment in $10K/yr compliance platforms** | Vanta / Drata only when certification is the goal; for internal projects without certification requirements, Claude-generated checklists + Phase-5 evidence pipeline are sufficient. |

---

## Daily Security Workflow (steady state)

```
Morning:
  1. Pull latest main
  2. Claude Code → /security-review on the day's branch (diff-aware)
  3. Triage overnight Dependabot PRs; auto-merge passing patch-level CVE fixes
  4. Triage overnight GitGuardian alerts (should be zero); rotate any real leaks within 1h

Per PR:
  5. Open PR → SonarQube + Semgrep + /security-review GitHub Action run in CI
  6. Trivy + Checkov on container / IaC changes
  7. ggshield + GitGuardian platform scan on push
  8. Address Critical / High findings: Copilot Autofix or Snyk DeepCode patch, or Claude Code [`security-fix-generation`](./PROMPTS.md#security-fix-generation)
  9. Custom Semgrep rule for any project-specific pattern: [`semgrep-custom-rule-generation`](./PROMPTS.md#semgrep-custom-rule-generation)
 10. Human approval; merge

Per release:
 11. [`pre-release-self-review`](./PROMPTS.md#pre-release-self-review) before production-deploy approval
 12. 0 Critical SAST / 0 Critical CVE / 0 secrets / OPA dryrun-clean / AI threat review current

Quarterly:
 13. [`security-posture-report`](./PROMPTS.md#security-posture-report) to leadership
 14. Rotate quarterly-cadence secrets; verify the rotation worked
 15. Run secret-rotation + incident-response drill
 16. Vanta / Drata 100%-coverage check (if certifying)

Incident (secret / vulnerability disclosed):
 17. Rotate / patch first; scrub or fix-forward second
 18. [`secrets-incident-response`](./PROMPTS.md#secrets-incident-response) for the runbook
 19. Post-mortem within 48h; process change to prevent recurrence
```

---

## Related Documents

- [Prompt Templates →](./PROMPTS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Process Flowchart →](./FLOWCHART.md)
- [Security & Compliance Tools Evaluation →](../../docs/tools-evaluation/5.Security_Compliance_Phase_Tools.md)
- [Code Review Checklist (AI-extra-scrutiny section) →](../../templates/code-review-checklist.md)
- [Phase 3 Linear MCP setup (carries over) →](../03-development/PROCESS.md#step-0-one-time-setup--connect-claude-code-to-linear-via-mcp)
- [Phase 6 CrossGuard / Pulumi-policy-OPA bridge →](../06-cicd-devops/PROCESS.md#step-3-infrastructure-provisioning-iac-with-pulumi-ai)

## External References

- [Claude Code `/security-review` + GitHub Action](https://github.com/anthropics/claude-code-security-review) — official; diff-aware; injection / authn-z / secrets / sensitive-logs
- [Anthropic — `/security-review` announcement](https://x.com/AnthropicAI/status/1953135070174134559)
- [Anthropic — Prompt-injection defences](https://www.anthropic.com/research/prompt-injection-defenses) — Opus 4.5 1.4% ASR vs 10.8% Sonnet 4.5
- [Anthropic — Trustworthy agents](https://www.anthropic.com/research/trustworthy-agents)
- [Semgrep MCP server](https://github.com/semgrep/mcp) — `security_check`, `semgrep_scan_with_custom_rule`, `write_custom_semgrep_rule` prompt
- [Semgrep MCP docs](https://semgrep.dev/docs/mcp)
- [Trivy MCP server](https://github.com/aquasecurity/trivy-mcp) — IDE-triggered scanning + `trivy mcp` plugin
- [Trivy MCP — Aqua announcement](https://www.aquasec.com/blog/security-that-speaks-your-language-trivy-mcp-server/)
- [Snyk DeepCode AI / Snyk Agent Fix](https://snyk.io/platform/deepcode-ai/) — 80% auto-fix accuracy, 19+ langs
- [Snyk — Find / auto-fix / prioritise intelligently](https://snyk.io/blog/find-auto-fix-prioritize-intelligently-snyks-ai-powered-code/)
- [Snyk — AI Security Fabric / Autofix](https://snyk.io/blog/ai-code-security-snyk-autofix-deepcode-ai/)
- [GitHub Copilot Autofix for Code Scanning](https://docs.github.com/en/code-security/concepts/code-scanning/copilot-autofix-for-code-scanning) — GA 2026
- [GitHub — CodeQL PR insights cover all protected branches](https://github.blog/changelog/2026-03-31-codeql-pull-requests-insights-on-security-overview-now-cover-all-protected-branches/)
- [Endor Labs — AURI for AI-driven coding](https://securitybrief.news/story/endor-labs-launches-auri-to-secure-ai-driven-coding) — 97% noise cut, 40+ langs
- [Endor Labs platform](https://www.endorlabs.com/platform)
- [Aikido Security — Infinite (Feb 2026)](https://siliconangle.com/2026/02/24/aikido-security-unveils-infinite-automated-self-securing-software-solution/)
- [Aikido Security — Endpoint (April 2026)](https://siliconangle.com/2026/04/20/aikido-security-debuts-endpoint-ai-native-developer-security/)
- [GitGuardian — ggshield AI hook (March 2026)](https://docs.gitguardian.com/ggshield-docs/integrations/ai-coding-tools/secret-scanning-for-ai-coding-tools)
- [GitGuardian — ggshield AI hook (Help Net Security feature, April 2026)](https://www.helpnetsecurity.com/2026/04/15/product-showcase-gitguardian-ggshield-ai-hook/)
- [Cycode — AI Governance + AIBOM + MCP enforcement (Feb 2026)](https://cycode.com/blog/ai-governance-aibom-mcp-enforcemen/)
- [Cycode — Securing the ADLC](https://cycode.com/blog/securing-adlc/)
- [Vanta Agents (March 2026)](https://www.vanta.com/) — Compliance / TPRM / Customer Trust agents
- [Drata MCP (2026)](https://drata.com/) — AI-native Trust Management
- [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/website/) — runtime policy
- [Red Hat — Eliminating the Rego tax (2026)](https://next.redhat.com/2026/03/20/eliminating-the-rego-tax-how-ai-orchestrators-automate-kubernetes-compliance/)
- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) — released Dec 2025
- [OWASP Agentic Skills Top 10](https://owasp.org/www-project-agentic-skills-top-10/)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [NIST AI RMF → ISO/IEC 42001 crosswalk](https://airc.nist.gov/docs/NIST_AI_RMF_to_ISO_IEC_42001_Crosswalk.pdf)
- [AGENTS.md spec](https://agents.md) — open standard for AI coding agent project context
