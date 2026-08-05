# Phase 7: Delivery & Handoff — Quality Gates

## Gate 1: Release Automation Operational

Before treating releases as "production ready."

- [ ] Conventional Commits adopted; commitlint enforces format in pre-commit hook + CI
- [ ] semantic-release configured in CI to auto-tag and generate notes
- [ ] Claude release notes polish prompt documented in team playbook
- [ ] Release Manager role assigned
- [ ] Publishing targets configured (GitHub Releases + CHANGELOG.md + optional public page)
- [ ] Stakeholder notification workflow defined (Slack / email / etc.)

**Pass:** All items checked. Release cycle is automated.

---

## Gate 2: Documentation Complete

Before declaring the project delivered.

### Required Sections
- [ ] Getting Started / Quick Start
- [ ] User guides for core features
- [ ] API reference (auto-generated from OpenAPI)
- [ ] Architecture overview with diagrams
- [ ] All ADRs accessible
- [ ] Local development setup guide
- [ ] Testing guide
- [ ] Deployment guide
- [ ] Runbook per alert (from Phase 6)
- [ ] Incident response playbook
- [ ] Disaster recovery plan
- [ ] Troubleshooting / FAQ
- [ ] Glossary of domain terms

### Quality Checks
- [ ] Claude doc completeness prompt run; **no Critical gaps and no undecided rows** — a row the run marked `[INSUFFICIENT INPUT: …]` is not a clean row, and this box stays unticked until a human has decided it
- [ ] All auto-generated content reviewed by a human
- [ ] Screenshots and diagrams accurate and current
- [ ] Code examples are working (not pseudocode)
- [ ] Links work (no 404s) — evidenced by a real link-check run, not by inspection. If the repo configures no link checker, this item is **unverified**, not passed
- [ ] Documentation site deployed and searchable

> **Four of these nineteen items cannot be certified by any automated documentation run, and the [`finalise-documentation`](./skill-prompts/finalise-documentation/SKILL.md) skill is required to hand them back by name rather than report them.** *Runbook per alert* needs the live alert list — an alert configured in the observability backend but never committed as code is invisible to a repository read, and belongs to whoever owns the alert inventory. *All auto-generated content reviewed by a human* cannot be ticked by the thing that generated it. *Screenshots and diagrams accurate and current* requires seeing a rendered screen. *Documentation site deployed and searchable* is a property of the deployed site, and the skill stops before publish by design. All four are the Tech Lead's or the publisher's. A run reporting all nineteen has fabricated four of them.

**Pass:** All items checked. Documentation is complete.

---

## Gate 3: Handoff Package Ready

Before knowledge transfer begins (Step 4) — the handoff document itself is complete, fact-checked, and shared with the recipient. KT-, credential-, and infrastructure-preparation checks now live under Gate 4 alongside their execution counterparts so prep + execution are validated together.

### Handoff Document
- [ ] Generated using Claude with all Phase 1–6 inputs — checked against the **artefact manifest** the run prints (read / named but unreadable / not supplied), not against an assertion that everything was read
- [ ] Reviewed by Delivery Lead + Tech Lead
- [ ] All sections complete (Executive Summary through Appendices) — **and zero `[VERIFY: …]` markers left unresolved.** A section that is present but carries open verification markers is present, not complete, and this box must not be ticked over one
- [ ] Every `[REDACT-BEFORE-SHARE]` marker resolved by the Tech Lead **before the draft leaves the delivery team** — the marker exists so the decision is made, and an unresolved one shared with the recipient is the customer-facing/internal leak this phase names as a top risk
- [ ] Known limitations section is honest and complete
- [ ] Recommended next steps are specific and actionable
- [ ] Draft shared with recipient and feedback iterated on

> **This gate is a set of human signatures.** The [`handoff-agent`](./subagent-prompts/handoff-agent.md) can self-certify exactly one of these seven lines — *all sections complete*, evidenced by per-section marker counts — and is required to name the rest by owner: the review signatures and the recipient iteration are human acts; the redaction resolution is the Tech Lead's, and **a zero marker count from the agent is only zero *found***; *honest and complete* and *specific and actionable* are judgements against artefacts the agent never saw; and *all inputs* is judged by the Delivery Lead against the printed manifest. The agent's contribution here is the evidence underneath the signatures, never the pass.

**Pass:** All items checked. Handoff document is complete and recipient-aligned; KT can begin.

---

## Gate 4: Handoff Execution

During and immediately after the handoff meeting(s). Each sub-section pairs a **Preparation** checklist (verified before the meeting) with an **Execution** checklist (verified during/after).

### KT Sessions

**Preparation**
- [ ] KT session plan defined with topics and timings
- [ ] Recording tools ready (Loom / Fathom)
- [ ] Demo environments prepared and tested end-to-end
- [ ] Presentation materials reviewed

**Execution**
- [ ] All planned sessions delivered
- [ ] Recordings archived and shared with recipient
- [ ] Transcripts / summaries generated with Claude
- [ ] Recipient questions addressed
- [ ] Open questions documented with owners for follow-up

### Credential Transfer

**Preparation**
- [ ] Credential inventory complete (every service catalogued)
- [ ] Role-transfer paths identified where available
- [ ] Password manager shared vault created
- [ ] Recipient has a password manager account ready to receive

**Execution**
- [ ] All credentials transferred via password manager (NOT email/Slack)
- [ ] Role-based access granted where possible
- [ ] Recipient verified login to every service
- [ ] Delivery team access removed (or scheduled for removal post-support period)
- [ ] Credentials that were shared (not role-transferred) rotated by recipient

### Infrastructure Handoff

**Preparation**
- [ ] IaC repository ready for transfer
- [ ] Terraform state backend migration plan documented
- [ ] Cloud account ownership transfer steps planned

**Execution**
- [ ] IaC repository transferred to recipient
- [ ] Terraform state migrated successfully and `terraform plan` reports **no changes** against the new backend
- [ ] Pre-migration `terraform state pull` backup taken and stored in the shared vault (contains secrets — treat as a credential)
- [ ] Recipient's state backend has versioning, encryption and locking configured before migration (the Phase 6 Gate 1 bar)
- [ ] Reproducibility demonstrated (create a new dev environment live)
- [ ] Cloud account access granted

### Knowledge Base
- [ ] Initial content seeded (FAQ, glossary, troubleshooting)
- [ ] Ownership assigned per content area
- [ ] Review cadence agreed

**Pass:** All items checked. Handoff complete. Recipient confirms they can operate the system.

---

## Gate 5: Support Period Closure

At the end of the agreed support period (typically 30 days).

### Issue Resolution
- [ ] All S0/S1 issues raised during support period resolved
- [ ] All S2 issues have a plan (resolution or documented deferral)
- [ ] No credential issues (recipient had access to everything)
- [ ] No deployment failures requiring delivery team intervention

### Documentation Updates
- [ ] Every issue raised has been traced to a doc gap
- [ ] Relevant docs updated based on real questions
- [ ] Runbooks updated based on any incidents that occurred

### Retrospective
- [ ] Post-handoff retrospective document generated — **every finding anchored to a named data point** (issue count, satisfaction score, named doc gap, dated incident) and **zero `[VERIFY: …]` markers left unresolved.** An unanchored retrospective is sentiment, and sentiment is what this document is generated to replace
- [ ] Lessons learned documented for future engagements — **kept in a separate section from the engagement-specific recommendations**, because they go to different readers and merging them buries the one the recipient must act on
- [ ] Feedback from recipient captured

**Pass:** All items checked. Support period officially closed. Delivery team transitions to on-demand availability only.

---

## Gate 6: 90-Day Debrief (Final)

90 days after handoff.

- [ ] 90-day debrief meeting held with recipient
- [ ] Final lessons learned documented
- [ ] Any systemic issues escalated to the delivery team's retrospective process
- [ ] Recipient satisfaction survey completed
- [ ] Knowledge base reviewed for decay — any content already outdated?
- [ ] Metrics reported: did AI-DLC deliver the promised benefits?

**Pass:** 90-day debrief complete. Engagement fully concluded.

---

## AI-Specific Delivery Standards

| Standard | Rationale |
|----------|-----------|
| **AI drafts, humans edit** | Release notes, docs, handoff docs — AI produces drafts, never publishes |
| **Review AI content for accuracy** | AI can be confidently wrong about domain specifics |
| **Fact-check across artefacts** | Cross-check AI handoff doc against PRD, ADRs — Claude may hallucinate details |
| **Preserve the human touch** | Marketing-style release notes need human voice adjustments |
| **Track AI-generation time savings** | Document the reductions (e.g., "release notes: 2h → 15min") for future ROI assessment |

---

## Metrics to Track

| Metric | Target | Measurement |
|--------|--------|-------------|
| Release notes generation time | < 30 min per release | Self-report |
| Documentation completeness score | 100% required sections | Claude completeness prompt |
| Handoff document generation time | < 2 hours | Self-report |
| KT session retention | > 80% recipient confidence | Post-session survey |
| Issues raised in support period | Trend downward over engagements | Linear count |
| Support-period extensions needed | Rare | Contract variations |
| Documentation decay at 90 days | < 10% content outdated | Claude comparison to current code |
| Recipient satisfaction | ≥ 4/5 | Post-handoff survey |
