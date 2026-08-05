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

### Baseline controls live (Step 0)
- [ ] **GitHub code scanning (CodeQL) enabled** on every repo in scope; `.github/workflows/codeql.yml` committed
- [ ] CodeQL runs on every PR **and nightly** on the default branch; gate set to 0 Critical / 0 High and the job fails the build
- [ ] CodeQL is the configured scanner: the `security-extended` suite for every language present in the repo, plus the project's own queries in `.github/codeql/`
- [ ] CodeQL SARIF archived as a CI artefact (Step 7 evidence compilation reads from it)
- [ ] **Claude Code `/security-review` GitHub Action committed** at `.github/workflows/claude-security-review.yml`; smoke-test passing
- [ ] **Copilot Autofix enabled** for the languages in scope
- [ ] Dependabot active on every repo; `.github/dependabot.yml` configured for patch-level auto-merge
- [ ] GitGuardian active org-wide; ggshield pre-commit hook on every developer machine; **ggshield AI hook configured** for Claude Code
- [ ] One-time GitGuardian historical scan complete; any leaks routed through Step 5 incident response
- [ ] Security Champion designated on the team
- [ ] AGENTS.md security-conventions section committed, including the Step 4 required-controls list and the Step 6.4 MCP allow-list

**Pass:** All items checked. Threat model and baseline scanning live.

---

## Gate 2: SAST Quality Gate

Runs on every pull request. Blocks merge if failed.

### Automated
- [ ] SAST scan passes on the branch: 0 Critical / 0 High
- [ ] CodeQL PR-diff analysis: no Critical / High new findings
- [ ] **`/security-review` GitHub Action passes** with no Critical / High comments unresolved
- [ ] **Copilot Autofix** patch suggestions reviewed for any auto-fixable findings
- [ ] No secrets detected by ggshield, GitGuardian, or the ggshield AI hook
- [ ] Custom CodeQL queries in `.github/codeql/` pass against the diff (project-specific patterns)

### Human
- [ ] Security-sensitive PRs (auth, crypto, data handling, AI features) reviewed by Security Champion
- [ ] Any dismissed Critical / High finding has documented justification in PR comments
- [ ] AI-noted PRs received the extra-scrutiny review per the team's code review checklist

**Pass:** All items green. Merge allowed.

**Escalation:** Any Critical finding believed to be a false positive requires Security Champion approval before dismissal. A pattern of repeating false positives is fixed at the query, not by habitually dismissing: tighten the triggering custom CodeQL query (Step 2.3) with a narrower predicate or an explicit exclusion, re-run both fixtures to prove it still fires on the anti-example, and commit the change to `.github/codeql/`.

---

## Gate 3: Dependency Hygiene

Per release.

- [ ] 0 Critical CVEs in production dependencies (Dependabot alerts view, filtered to open)
- [ ] High CVEs: documented Tech Lead risk acceptance
- [ ] Dependabot active with patch-level CVE auto-merge; no open security alert older than the monthly review
- [ ] Package-manager-native audit (`npm audit` / `pip-audit` / `cargo audit`) runs in CI and is clean at the configured severity floor
- [ ] [`reachability-triage`](./PROMPTS.md#reachability-triage) applied to the open Dependabot High / Critical list whenever it exceeds 5 items; the ranked output is attached to the release and drives merge order — Critical / High still upgrade even when classified Not reachable, unless the Tech Lead documented an exception
- [ ] Major version bumps in this release have a [`dependency-upgrade-impact`](./PROMPTS.md#dependency-upgrade-impact) analysis attached
- [ ] All container base images pinned by digest, no floating tags (confirmed by the reviewer in the Step 4 container review)

**Pass:** All items checked. Dependency hygiene gates the release.

---

## Gate 4: Container & IaC Review

Before any deploy.

> **This gate is verified by human review. No scanner, policy engine, or admission controller enforces it.** Both items below are confirmed by a named reviewer in the PR. If nobody signed off, the gate has not passed — a green pipeline says nothing about this surface.
>
> **An agent's self-check is not the named reviewer.** Phase 6's `container-image-engineer` self-certifies seven of nine container standards and `terraform-iac-engineer` self-checks its own conventions — both against their *own* output, both by construction unable to state a scan verdict. Neither run satisfies a line below. The reviewer named here reads the diff having not written it.

- [ ] Dockerfile / K8s manifests reviewed against AGENTS.md security conventions by a named reviewer (non-root user, no secrets in build args or layers, minimal installed surface, security context and resource limits set)
- [ ] Required-controls list confirmed for every IaC change in the release — **required tags**, **encryption at rest**, **no public storage**, **IAM least privilege**, **region allow-list**, **instance-size caps**, **no root containers**, **resource limits**, **network policies** — each one either satisfied or explicitly excepted with a written reason in the PR

**Pass:** Both items checked, with the reviewer named. Container and IaC posture reviewed for deploy.

---

## Gate 5: Secrets Hygiene

Continuous.

### Prevention layers
- [ ] ggshield pre-commit hook on every developer machine
- [ ] GitGuardian platform scan active; alerts route to Security Champion + on-call
- [ ] **ggshield AI hook** active for Claude Code — pre-prompt + pre-tool-use + post-tool-use intercept points all live
- [ ] A managed secret manager is the team's default for personal and shared secrets; no secrets in Slack / email / paste buffers
- [ ] Runtime secrets come from a managed secret store (Phase 6 platform default or cloud-native equivalent)

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
- [ ] **MCP server allow-list** committed to AGENTS.md and pinned by the project-scoped `.mcp.json` in the repo
- [ ] Per-server scope policy documented (read / write / state-changing)
- [ ] Off-boarding revokes OAuth grants per provider; verified
- [ ] AI inventory current at `/docs/security/ai-inventory.md` across the six categories: AI code assistants, AI models, AI infrastructure, MCP servers, AI secrets, AI packages

**Pass:** All items checked, or formally marked "AI not in product".

---

## Gate 7: Compliance & Audit Readiness

Only applicable if the project has formal compliance requirements.

- [ ] Compliance checklist generated for every applicable framework (GDPR / HIPAA / SOC 2 / PCI-DSS / ISO 27001 / ISO 27701 / ISO/IEC 42001 / NIST AI RMF / CCPA / FedRAMP)
- [ ] Every technical control classified **automated** or **manual**: automated controls name the verifying tool (CodeQL / `/security-review` / Dependabot / GitGuardian) and the evidence artefact; manual controls — including all container and IaC controls — carry a dated attestation naming the person who performed the check
- [ ] All operational controls documented in the security runbook
- [ ] Evidence dossier compiled for the audit period at `/docs/compliance/evidence/<period>/`
- [ ] Audit-ready document generated via [`evidence-compilation`](./PROMPTS.md#evidence-compilation)
- [ ] Legal review completed (for regulatory compliance)
- [ ] If AI is in the product and compliance scope includes AI: NIST AI RMF + ISO/IEC 42001 crosswalk applied; 8–12 month combined implementation plan documented
- [ ] Threat-model P0 mitigations verified **in code** by [`threat-model-mitigation-verification`](./PROMPTS.md#threat-model-mitigation-verification) (Step 1.5) — the per-mitigation table with `file:line` evidence is archived with the release; **0 not landed and 0 unevaluated**, or each remaining item carries a dated Tech Lead deferral. A verdict of `PASS WITHHELD — N unevaluated` **is not a pass**: it means part of the affected surface was never read, and the modules named as unreachable must be supplied and the check re-run
- [ ] [`pre-release-self-review`](./PROMPTS.md#pre-release-self-review) passes for the release candidate
- [ ] Security posture report ([`security-posture-report`](./PROMPTS.md#security-posture-report)) for the quarter shared with leadership

**Pass:** All items checked. Ready for external audit if needed.

---

## Phase Handoff

Before handing off to **Phase 6: CI/CD & DevOps** (concurrent) and **Phase 7: Delivery & Handoff**.

- [ ] All Gate 1–5 items complete; Gate 6 if AI is in product; Gate 7 if certifying
- [ ] Threat model archived; P0 mitigations verified landed in code by [`threat-model-mitigation-verification`](./PROMPTS.md#threat-model-mitigation-verification) — `file:line` per mitigation, **0 not landed and 0 unevaluated**. `PASS WITHHELD` is a withheld verdict, not a pass
- [ ] Security posture report current
- [ ] Compliance artefacts archived (if applicable)
- [ ] Known security issues log updated (open vs closed)
- [ ] Secrets rotation log reviewed; no open incidents
- [ ] Security runbook exists at `/docs/security/runbook.md`
- [ ] AGENTS.md security conventions current and referenced by Claude Code

---

## Metrics to Track

| Metric | Target | Measurement |
|--------|--------|-------------|
| Time to remediate Critical findings | < 24h from detection | Linear / `/security-review` timestamps |
| Time to remediate High findings | < 7 days | Linear timestamps |
| Dependency Critical CVE remediation time | < 7 days | Dependabot alert open→close timestamps |
| Mean time to rotate a leaked production secret | < 1 hour | GitGuardian incident timestamps |
| Signal-to-noise ratio per detector | > 30% actionable | Sample 20 findings/month per detector |
| Pre-commit secret block rate | Track baseline | ggshield logs |
| **AI-hook block rate** (pre-prompt + pre-tool + post-tool) | Track baseline; investigate any sudden spike | ggshield AI hook logs |
| **`/security-review` accept rate** (suggested fix accepted) | > 60% | GitHub PR-comment reactions / merge statistics |
| **Copilot Autofix accept rate** | > 60% | GitHub code-scanning dashboard |
| **Container & IaC review coverage** | 100% of PRs touching Dockerfiles / manifests / IaC | PR review records; count of written exceptions |
| **Prompt-injection attempts blocked** (model-layer + app-layer) | > 95% (Anthropic Opus 4.5 baseline 1.4% ASR) | Anthropic safety telemetry + app audit log |
| False-positive rate (per detector) | Trend downward | Sample + tag findings |
| AI-suggested-vs-actual root-cause variance | Track quarterly | Retros |

---

## AI-Specific Security Standards

| Standard | Rationale |
|----------|-----------|
| **PRs with AI-generated code get extra scrutiny per `code-review-checklist.md`** | AI code has 1.7× more issues per PR than human code |
| **`/security-review` runs diff-aware on every PR** | Lower false-positive rate than full-repo scanning; covers injection / authn-z / secrets / sensitive-logs |
| **ggshield AI hook covers Claude Code** | Catches secrets at pre-prompt, pre-tool-use, post-tool-use; AI commits leak at ~2× baseline (~3.2%) |
| **Every custom CodeQL query ships with two fixtures** | A query that never fires manufactures confidence; the anti-example proves it fires, the allowed-example proves it is not noise |
| **Threat-model mitigations are verified in code, never asserted** | A checkbox nobody can falsify is not a control; [`threat-model-mitigation-verification`](./PROMPTS.md#threat-model-mitigation-verification) must cite implementing code per P0 item — a comment, a test name, or a closed ticket is explicitly not evidence |
| **AI-generated security fixes require a regression test** | Mutation testing in Phase 4 catches some tautological fixes; an explicit regression test catches more |
| **MCP allow-list enforced** | 24,008 secrets found in MCP config files in 2025 (GitGuardian) — allow-listing is the primary control; ggshield is the runtime safety net |
| **Anthropic prompt-injection defences are baseline, not enough alone** | Adaptive attacks still > 85% successful; defence at every level required for agentic systems |
| **Audit log every AI-driven security action** | `/security-review` posting comments, an agent rotating a secret, Claude drafting a control narrative — forensic trail when AI acts unexpectedly |
| **Track AI-suggested-vs-actual-root-cause variance** | Confidence calibration improves over cycles; under-performing sources get demoted |
| **Off-boarding revokes per-provider OAuth** | MCP scopes inherit from the connecting human; off-boarding is unchanged when this discipline holds |

---

## Risk Escalation Matrix

| Risk Level | Example | Action | Owner |
|-----------|---------|--------|-------|
| **Critical** | Secret leaked to public repo; active exploit in dependency; prompt-injection attack succeeds in production | Immediate rotation / hotfix; all-hands; incident channel | Security Champion → Tech Lead |
| **High** | SQL injection in user-facing endpoint; agent goal hijack discovered in lab; a required control shipped unreviewed to production | Fix within 24h; block release; rollback if already deployed | Security Champion |
| **Medium** | Outdated dependency with known CVE; AI-generated code missed input validation; compliance evidence stale | Fix within sprint; for compliance staleness, re-collect within 7 days | Developer owning the code / Security Champion |
| **Low** | Code smell in non-critical path; advisory-level `/security-review` finding; low-severity IaC review note | Backlog | Developer |
