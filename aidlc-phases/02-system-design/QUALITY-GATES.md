# Phase 2: System Design — Quality Gates

Four gates align with the four blocks in [PROCESS.md](./PROCESS.md) and the [FLOWCHART](./FLOWCHART.md). No artifact passes to the next gate without sign-off.

---

## Gate 1: Architecture

- [ ] ≥ 2 architecture options evaluated with trade-off analysis
- [ ] Selected architecture addresses every NFR from the PRD
- [ ] Trade-off interrogation (10x stress-test) completed for the chosen option
- [ ] No unmitigated single point of failure
- [ ] Scalability path defined for 10x load
- [ ] Security boundaries explicit between components
- [ ] Cost estimate within budget (launch + 10x)
- [ ] Eraser diagrams: cloud architecture + system overview + ≥ 2 key sequences (DSL committed, editor URLs noted, PNG/SVG exported)
- [ ] Mermaid: C4 Context + Container in `/docs/diagrams/mermaid/` and rendering in repo
- [ ] ADR for every significant decision (minimum 3) — each cites the PRD section and lists rejected alternatives with specific reasons
- [ ] Reviewed by ≥ 1 senior engineer who is not the author
- [ ] AI proposals critically evaluated, not accepted as-is — design-review prompt outputs resolved (0 Critical, 0 High open)
- [ ] Linear: architecture artifact URLs linked from a `phase:design` Linear Issue

**Pass:** All checked. Architecture approved by Tech Lead.

---

## Gate 2: Data Model & API Contracts

- [ ] All PRD entities represented in the schema
- [ ] Relationships have correct cardinality
- [ ] Indexes cover the top 10 query patterns identified at Step 2.1
- [ ] Eraser ER diagram exists, matches schema, DSL committed
- [ ] Mermaid `erDiagram` in `/docs/diagrams/mermaid/erd.mmd` renders in repo
- [ ] OpenAPI 3.1 spec covers every PRD user flow
- [ ] Every endpoint has: request schema + success response + error responses (400, 401, 403, 404, 409, 422, 500)
- [ ] Consistent error envelope across all endpoints
- [ ] Pagination defined on every list endpoint
- [ ] Auth defined in spec on every protected endpoint
- [ ] Idempotency-Key header on POSTs that require it
- [ ] `x-prd-section` extension links each operation to its PRD anchor
- [ ] Mock servers (Prism) accessible — URL in project README
- [ ] Naming conventions consistent throughout
- [ ] Migration scripts generated and run cleanly against a local DB
- [ ] Seed data present for every entity (5-10 rows)
- [ ] Reviewed by Tech Lead + ≥ 1 backend developer
- [ ] Spec marked "frozen" — change-request process documented

**Pass:** All checked. Schema + API contracts approved.

---

## Gate 3: Wireframes & Tech Stack

- [ ] Claude Design project link recorded in project README
- [ ] Design system onboarded (from repo + brand) — every wireframe inherits it
- [ ] frontend-design Skill enabled on the architect's Claude account
- [ ] Wireframes exist for ALL key user flows from the PRD
- [ ] All states shown per primary screen: loading, empty, error, success
- [ ] Responsive: desktop (1440) + mobile (375) variants
- [ ] Accessibility review passes WCAG 2.1 AA (0 Critical, 0 High)
- [ ] For any flow handed off to code: branch created via Claude Design → Claude Code, PR opened, wireframe URL in PR description
- [ ] Stakeholder sign-off captured as inline comment on the Claude Design canvas
- [ ] Tech stack ADRs complete for every major decision (language, framework, datastore, cache, broker, IaC, observability, auth)
- [ ] Each unfamiliar tech choice has a training plan attached to its ADR
- [ ] Team reviewed and agreed (no unresolved objections)
- [ ] Any fallback tool used (Figma / v0 / Cursor / Bolt) recorded with the reason in the ADR for that flow

**Pass:** All checked. Wireframes approved by PM. Stack approved by team.

---

## Gate 4: Phase Handoff

- [ ] Architecture proposal + design review report in `/docs/architecture/`
- [ ] ADRs in `/docs/adrs/` (architecture + stack + any denormalisation)
- [ ] Eraser DSL + exports in `/docs/diagrams/eraser/`
- [ ] Mermaid `.mmd` files in `/docs/diagrams/mermaid/` (C4 Context + Container, ER, key sequences)
- [ ] Database schema in source directory (`/src/db/` or `/prisma/`); migrations + seeds present
- [ ] OpenAPI 3.1 spec at `/docs/api/openapi.yaml`; mock server URL in README
- [ ] Claude Design project link + handoff branch(es) referenced from README
- [ ] Accessibility report at `/docs/accessibility/wcag-aa-report.md`
- [ ] Tech stack ADRs + training plans complete
- [ ] Linear: every Phase 2 artifact URL linked from a `phase:design` Issue, and the Phase 1 PRD Document is still cited from each artifact
- [ ] Dev team confirms enough information to start Phase 3 coding (no open "what does this even mean" questions)

**Pass:** All artifacts stored and linked. Dev team confirms readiness.

---

## Metrics

| Metric | Target |
|--------|--------|
| Architecture proposal time (1.1 → 1.8) | < 2 days |
| Eraser diagram first-pass time (per diagram) | < 15 min |
| Mermaid in-repo C4 generation | < 30 min total |
| OpenAPI spec generation + Swagger walkthrough | < 1 day |
| Claude Design wireframe per user flow (incl. iteration) | < 4 hours |
| Accessibility review issues at Gate 3 | 0 Critical, 0 High |
| Design review issues at Gate 1 | 0 Critical, ≤ 3 High (resolved before pass) |
| API contract change-requests after frontend start | 0 (any change requires PM approval + impact analysis) |
| Fallback-tool use in Step 4 | ≤ 1 per sprint (more than that triggers Claude-Design-coverage retro) |
