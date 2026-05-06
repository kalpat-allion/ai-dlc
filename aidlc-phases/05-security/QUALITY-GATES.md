# Phase 5: Security & Compliance — Quality Gates

The seven gates below align with the seven [PROCESS.md](./PROCESS.md) steps. Step 0 (one-time setup) is the precondition — without it the gates cannot be measured. Pass criteria are absolute — items unchecked block progression.

---

## Gate 1: Threat Model & Baseline

Before a release candidate is built.

### Threat model (Step 1)
- [ ] STRIDE pass committed at `/docs/security/threat-model.md`
- [ ] OWASP Top 10 + OWASP API Top 10 cross-references applied
- [ ] If AI is in the product: OWASP LLM Top 10 + OWASP Top 10 for Agentic Applications 2026 covered (Step 6 entry point)
- [ ] Every P0 finding has a Linear issue with an owner and a fix-before-launch SLA
- [ ] P1 findings ticketed with 30-day SLA; P2 with 90-day SLA

### Baseline scanning live (Step 0)
- [ ] SonarQube CE security profile in CI; security gate set to 0 Critical / 0 High
- [ ] Semgrep OSS in CI with OWASP Top 10 + language-specific rules
- [ ] **Claude Code `/security-review` GitHub Action committed** at `.github/workflows/claude-security-review.yml`; smoke-test passing
- [ ] **Semgrep MCP server connected** for every developer (`claude mcp list` shows `semgrep: connected`)
- [ ] **Trivy MCP server connected** (`trivy mcp` plugin installed; Claude Code MCP listed)
- [ ] Dependabot or Renovate active on every repo; `.github/dependabot.yml` configured for patch-level auto-merge
- [ ] Trivy in container build pipeline; fails on Critical CVEs
- [ ] Checkov on every IaC PR
- [ ] GitGuardian active org-wide; ggshield pre-commit hook on every developer machine; **ggshield AI hook configured** for Cursor / Claude Code / Copilot
- [ ] One-time GitGuardian historical scan complete; any leaks routed through Step 5 incident response
- [ ] Security Champion designated on the team
- [ ] AGENTS.md security-conventions section committed

**Pass:** All items checked. Threat model and baseline scanning live.

---

## Gate 2: SAST Quality Gate

Runs on every pull request. Blocks merge if failed.

### Automated
- [ ] SonarQube security quality gate passes (0 Critical, 0 High)
- [ ] Semgrep PR-diff scan: no Critical / High new findings
- [ ] **`/security-review` GitHub Action passes** with no Critical / High comments unresolved
- [ ] **Copilot Autofix or Snyk DeepCode** patch suggestions reviewed for any auto-fixable findings
- [ ] No secrets detected by ggshield, GitGuardian, or the ggshield AI hook
- [ ] Custom Semgrep rules in `.semgrep/rules/` pass against the diff (project-specific patterns)

### Human
- [ ] Security-sensitive PRs (auth, crypto, data handling, AI features) reviewed by Security Champion
- [ ] Any dismissed Critical / High finding has documented justification in PR comments
- [ ] AI-noted PRs received the extra-scrutiny review per [`code-review-checklist.md`](../../templates/code-review-checklist.md)

**Pass:** All items green. Merge allowed.

**Escalation:** Any Critical finding believed to be a false positive requires Security Champion approval before dismissal. Pattern of repeating false positives → file a Semgrep custom rule (Step 2.3) or tune the SonarQube rule.

---

## Gate 3: SCA Dependency Hygiene

Per release.

### Code dependencies
- [ ] 0 Critical CVEs in production dependencies
- [ ] High CVEs: documented Tech Lead risk acceptance
- [ ] Reachability-aware triage (Endor Labs / Snyk Open Source / `reachability-triage` prompt) applied for High / Critical lists > 5 items
- [ ] Major version bumps in this release have an `dependency-upgrade-impact` analysis attached

### Container & registry
- [ ] All container base images pinned by digest (no floating tags)
- [ ] Trivy: 0 Critical CVEs at registry push
- [ ] Docker Scout AI-suggested base-image upgrades reviewed (and merged or explicitly declined)

### SBOM (when compliance requires)
- [ ] OWASP Dependency-Check SBOM generated in CycloneDX or SPDX format
- [ ] SBOM attached to the release artefacts

**Pass:** All items checked. Dependency hygiene gates the release.

---

## Gate 4: Container & IaC Compliance

Before any deploy.

- [ ] Trivy clean on every shipped image (0 Critical CVEs)
- [ ] Checkov passes on all IaC (CIS / NIST profile per project)
- [ ] OPA / Conftest tests pass in CI for every `/policy-packs/` rule
- [ ] **OPA Gatekeeper deployed** with the project's policies (if Kubernetes is in scope)
- [ ] **Every AI-generated policy** spent at least 7 days in `enforcementAction: dryrun` with zero false positives before flipping to `deny`
- [ ] Required-tags / encryption-at-rest / no-public-S3 / IAM-least-privilege / region-allowlist policies enforced
- [ ] Phase 6 CrossGuard packs (if Pulumi) reference the same Rego sources via the `pulumi-policy-opa` bridge — author once, enforce both at IaC-time and at runtime
- [ ] Dockerfile / K8s manifests reviewed against AGENTS.md security conventions
- [ ] Trivy MCP smoke-test passes (IDE-time scan triggers on `package.json` / `requirements.txt` / `Dockerfile` changes)

**Pass:** All items checked. Container and IaC posture safe for deploy.

---

## Gate 5: Secrets Hygiene

Continuous.

### Prevention layers
- [ ] ggshield pre-commit hook on every developer machine
- [ ] GitGuardian platform scan active; alerts route to Security Champion + on-call
- [ ] **ggshield AI hook** active for Cursor / Claude Code / GitHub Copilot — pre-prompt + pre-tool-use + post-tool-use intercept points all live
- [ ] Bitwarden / 1Password (or equivalent) is the team's default; no secrets in Slack / email / paste buffers
- [ ] Runtime secrets via Pulumi ESC (Phase 6) or platform-native (AWS Secrets Manager / Azure Key Vault / GCP Secret Manager)

### Incident response
- [ ] 0 open GitGuardian incidents at this gate's evaluation
- [ ] Mean time to rotate a leaked production-grade secret < 1 hour
- [ ] Quarterly secret-rotation drill within last 90 days
- [ ] Historical scan re-run quarterly (the 64% / 2022→2026 stat is a forcing function)

**Pass:** All items checked. Secrets hygiene gating release.

---

## Gate 6: AI Agent Security

Applies when AI is in the product. Skip otherwise.

### Threat surface
- [ ] AI / agent threat review committed at `/docs/security/ai-threat-review.md`
- [ ] OWASP LLM Top 10 covered finding-by-finding
- [ ] OWASP Top 10 for Agentic Applications 2026 covered, with explicit defence for **ASI01 Agent Goal Hijacking**

### Model-layer defences
- [ ] Anthropic Claude Opus 4.7 (or higher) prompt-injection defences enabled where the team's toolchain supports it (the published 4.5 baseline is 1.4% ASR vs 10.8% Sonnet 4.5 with prior gen)
- [ ] Server-side prompt-injection probe enabled at input layer
- [ ] Production monitoring + red-teaming cadence documented

### Application-layer defences
- [ ] Output validation against allow-list (no raw model text rendered as HTML / executed)
- [ ] Per-user / per-session rate limits in place
- [ ] Cost caps per session and per tenant
- [ ] Audit logging of every tool call (user, server, tool, args summary) — forensic replay possible

### MCP enforcement
- [ ] **MCP server allow-list** committed to AGENTS.md and `/policy-packs/mcp/allow-list.yaml`
- [ ] Per-server scope policy documented (read / write / state-changing)
- [ ] Off-boarding revokes OAuth grants per provider; verified
- [ ] Cycode AI Governance + AIBOM (if in scope) — three-layer governance configured (see / govern / enforce)
- [ ] AI Inventory across the 6 Cycode categories (code assistants, models, infrastructure, MCP servers, AI secrets, AI packages) — current

**Pass:** All items checked, or formally marked "AI not in product".

---

## Gate 7: Compliance & Audit Readiness

Only applicable if the project has formal compliance requirements.

- [ ] Compliance checklist generated for every applicable framework (GDPR / HIPAA / SOC 2 / PCI-DSS / ISO 27001 / ISO 27701 / ISO/IEC 42001 / NIST AI RMF / CCPA / FedRAMP)
- [ ] All technical controls implemented and verified by automated tools (Trivy / Checkov / OPA / SonarQube / Semgrep / `/security-review`)
- [ ] All operational controls documented in the security runbook
- [ ] Evidence dossier compiled for the audit period at `/docs/compliance/evidence/<period>/`
- [ ] Audit-ready document generated via [`evidence-compilation`](./PROMPTS.md#evidence-compilation)
- [ ] Legal review completed (for regulatory compliance)
- [ ] If certifying: **Vanta Agents** or **Drata MCP** dashboard at 100% control coverage; agent-collected evidence reviewed for hourly-test freshness
- [ ] If AI is in the product and compliance scope includes AI: NIST AI RMF + ISO/IEC 42001 crosswalk applied; 8–12 month combined implementation plan documented
- [ ] [`pre-release-self-review`](./PROMPTS.md#pre-release-self-review) passes for the release candidate
- [ ] Security posture report ([`security-posture-report`](./PROMPTS.md#security-posture-report)) for the quarter shared with leadership

**Pass:** All items checked. Ready for external audit if needed.

---

## Phase Handoff

Before handing off to **Phase 6: CI/CD & DevOps** (concurrent) and **Phase 7: Delivery & Handoff**.

- [ ] All Gate 1–5 items complete; Gate 6 if AI is in product; Gate 7 if certifying
- [ ] Threat model archived; P0 mitigations landed
- [ ] Security posture report current
- [ ] Compliance artefacts archived (if applicable)
- [ ] Known security issues log updated (open vs closed)
- [ ] Secrets rotation log reviewed; no open incidents
- [ ] Security runbook exists at `/docs/security/runbook.md`
- [ ] AGENTS.md security conventions current and referenced by Claude Code, Cursor, Copilot, Pulumi Neo

---

## Metrics to Track

| Metric | Target | Measurement |
|--------|--------|-------------|
| Time to remediate Critical findings | < 24h from detection | Linear / SonarQube / `/security-review` timestamps |
| Time to remediate High findings | < 7 days | Linear / SonarQube timestamps |
| Dependency Critical CVE remediation time | < 7 days | Dependabot / Snyk timestamps |
| Mean time to rotate a leaked production secret | < 1 hour | GitGuardian incident timestamps |
| Signal-to-noise ratio per tool | > 30% actionable (target 50% with reachability) | Sample 20 findings/month per tool |
| Pre-commit secret block rate | Track baseline | ggshield logs |
| **AI-hook block rate** (pre-prompt + pre-tool + post-tool) | Track baseline; investigate any sudden spike | ggshield AI hook logs |
| **`/security-review` accept rate** (suggested fix accepted) | > 60% | GitHub PR-comment reactions / merge statistics |
| **Copilot Autofix / Snyk DeepCode accept rate** | > 60% | GitHub / Snyk dashboards |
| **Reachability-triage noise reduction** | 70–97% (per Endor Labs / Snyk benchmarks) | Compare raw CVE count vs reachable count |
| **Prompt-injection attempts blocked** (model-layer + app-layer) | > 95% (Anthropic Opus 4.5 baseline 1.4% ASR) | Anthropic safety telemetry + app audit log |
| False-positive rate (per tool) | Trend downward | Sample + tag findings |
| Vanta / Drata control coverage (if certifying) | 100% | Vanta / Drata dashboard |
| AI-suggested-vs-actual root-cause variance | Track quarterly | Retros |

---

## AI-Specific Security Standards

| Standard | Rationale |
|----------|-----------|
| **PRs with AI-generated code get extra scrutiny per `code-review-checklist.md`** | AI code has 1.7× more issues per PR than human code |
| **`/security-review` runs diff-aware on every PR** | Lower false-positive rate than full-repo scanning; covers injection / authn-z / secrets / sensitive-logs |
| **ggshield AI hook covers Cursor / Claude Code / Copilot** | Catches secrets at pre-prompt, pre-tool-use, post-tool-use; AI commits leak at ~2× baseline (~3.2%) |
| **Semgrep + Snyk rules include OWASP LLM Top 10 coverage** | Automatic checks for prompt injection, insecure output handling |
| **AI-generated OPA / Rego policies must spend 7 days in `dryrun`** | Prevent a single-line policy locking the cluster (Red Hat 2026 dynamic-generator pattern) |
| **AI-generated security fixes require a regression test** | Mutation testing in Phase 4 catches some tautological fixes; an explicit regression test catches more |
| **MCP allow-list enforced** | 24,008 secrets found in MCP config files in 2025 (GitGuardian) — allow-listing is the primary control; ggshield is the runtime safety net |
| **Anthropic prompt-injection defences are baseline, not enough alone** | Adaptive attacks still > 85% successful; defence at every level required for agentic systems |
| **Audit log every AI-driven security action** | `/security-review` posting comments, agent rotating secrets, Vanta agent collecting evidence — forensic trail when AI acts unexpectedly |
| **Track AI-suggested-vs-actual-root-cause variance** | Confidence calibration improves over cycles; under-performing sources get demoted |
| **Off-boarding revokes per-provider OAuth** | MCP scopes inherit from the connecting human; off-boarding is unchanged when this discipline holds |

---

## Risk Escalation Matrix

| Risk Level | Example | Action | Owner |
|-----------|---------|--------|-------|
| **Critical** | Secret leaked to public repo; active exploit in dependency; prompt-injection attack succeeds in production | Immediate rotation / hotfix; all-hands; incident channel | Security Champion → Tech Lead |
| **High** | SQL injection in user-facing endpoint; agent goal hijack discovered in lab; AI-generated OPA policy locks production | Fix within 24h; block release; rollback if already deployed | Security Champion |
| **Medium** | Outdated dependency with known CVE; AI-generated code missed input validation; Vanta agent evidence stale | Fix within sprint; for compliance staleness, re-collect within 7 days | Developer owning the code / Security Champion |
| **Low** | Code smell in non-critical path; advisory-level Semgrep finding; cosmetic Checkov warning | Backlog | Developer |
