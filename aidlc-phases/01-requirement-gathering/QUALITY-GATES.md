# Phase 1: Requirement Gathering — Quality Gates

## Gate 1: PRD Completeness & Published in Linear

### PRD content
- [ ] Executive Summary exists (one paragraph)
- [ ] Problem Statement defines problem AND affected users
- [ ] Goals include ≥ 3 SMART metrics with numeric targets
- [ ] ≥ 2 personas with demographics, goals, pain points, usage context
- [ ] Functional requirements grouped by feature area with priorities
- [ ] Non-functional requirements cover: performance, security, scalability, accessibility
- [ ] Out of Scope section explicitly lists exclusions
- [ ] ≥ 3 assumptions documented
- [ ] Dependencies on external systems listed
- [ ] ≥ 3 risks identified with mitigations
- [ ] Every requirement is testable (no vague language)
- [ ] No contradictions between requirements
- [ ] AI-generated content reviewed and refined by PM

### Linear publication & sign-off
- [ ] PRD published as a **Linear Document** attached to a Linear Project (created via `prd-to-linear-document`)
- [ ] Linear Project initially in `Planned` state during stakeholder review
- [ ] All H2/H3 headings have stable anchors (verified by Claude's section anchor map output)
- [ ] Stakeholder review conducted on the Linear Document via inline comments (Guests added for non-engineering reviewers as needed)
- [ ] Document header status updated to **Approved v1.0**
- [ ] Sign-offs (PM, Tech Lead, Sponsor) checked in the Document's `## Sign-offs` section
- [ ] Linear Project moved to `Backlog` (or `Started` when Step 3 has begun)

**Pass:** All items checked. **Escalation:** PM → Tech Lead (technical) or Project Sponsor (scope).

---

## Gate 2: User Story Completeness (incl. Linear scaffolding)

### Story content
- [ ] Every functional requirement maps to ≥ 1 user story
- [ ] Every story uses format: As a [persona], I want [goal], so that [benefit]
- [ ] Every persona appears in ≥ 1 story
- [ ] Every story has ≥ 3 acceptance criteria (happy path + error + edge case)
- [ ] Acceptance criteria in Given/When/Then format
- [ ] No story exceeds XL (decompose if larger)
- [ ] Stories organised into epics with dependencies
- [ ] No duplicates
- [ ] Priorities assigned to every story
- [ ] No story introduces scope not in the PRD

### Linear scaffolding (Stages 3b–3c)
- [ ] Linear Project (already created in Step 2.4) is in `Backlog` or `Started` state before Milestones are added
- [ ] PRD epic count = Linear **Milestone** count (added by `prd-to-linear-scaffold`)
- [ ] Milestone names and order match the PRD epic order; target dates match the PRD timeline
- [ ] Every Linear issue is in `Triage` or `Backlog` state at creation — none promoted automatically
- [ ] Every AI-created Linear issue carries **both** `ai-generated` and `needs-human-review` labels
- [ ] Every issue's description contains a **clickable** `**PRD section:** [§X.Y](document-url#anchor)` deep-link that opens the right heading in the Linear Document
- [ ] Pre-flight duplicate check ran in `stories-to-linear-push`; flagged duplicates resolved before creation
- [ ] **AI Inbox cleared** — the saved view `ai-generated AND needs-human-review` is empty before phase handoff
- [ ] No story without a valid PRD deep-link reached the Backlog

**Pass:** All items checked. **Escalation:** If PRD gaps found, loop back to Gate 1.

---

## Gate 3: Final Review (Phase Handoff)

- [ ] Gap analysis completed (Claude prompt against the Linear PRD Document)
- [ ] **Linear gap-sweep run** — `linear-gap-sweep` posted a consolidated comment on the Linear Project; comment reviewed
- [ ] All Critical gaps resolved: PRD Document edited (version bumped), gap stories pushed to Linear with `gap-analysis` label and accepted
- [ ] All High gaps resolved or documented with plan
- [ ] Medium/Low gaps documented as future considerations inside the PRD Document under "Out-of-scope / future"
- [ ] Scalability requirements documented in the PRD Document with measurable targets
- [ ] Cost estimate within budget constraints
- [ ] PRD Document version is **Approved v1.0** (or higher if gap-analysis triggered a re-version)
- [ ] Every Linear issue's PRD deep-link still resolves (no orphaned anchors after Document edits)
- [ ] MCP write actions captured (Anthropic transcript + Linear audit log) and retained for the project
- [ ] Handoff checklist complete — single Linear Project URL is the canonical entry point for Phase 2

**Pass:** All Critical/High resolved. All artifacts stored. Stakeholder sign-off confirmed.

**Escalation:** Critical gaps requiring scope changes → Project Sponsor approval before proceeding.

---

## Metrics to Track

| Metric | Target | Measurement |
|--------|--------|-------------|
| PRD generation time | < 4 hours (AI-assisted) | Context gathering → first draft |
| AI vs. human gap detection | Track ratio | Count per source |
| Story coverage | 100% of functional requirements | Map stories → requirements |
| Revision rounds | ≤ 2 before approval | Count feedback cycles |
