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
- [ ] Reviewed by ≥ 1 senior engineer who is not the author — `architecture-reviewer` run in a session that did **not** produce the design does not replace that human; it is the evidence the human reads
- [ ] AI proposals critically evaluated, not accepted as-is — `architecture-reviewer` verdict is PASS or PASS WITH FIXES, with 0 Critical and 0 High open, and its **Unevaluated** list has been read and either closed or accepted
- [ ] Linear: architecture artifact URLs linked from a `phase:design` Linear Issue
- [ ] `solution-architect` subagent committed under `.claude/agents/`, placeholders filled, and smoke-tested — it stopped at the Tech Lead approval gate rather than proceeding to schema or contract work, and refused to review its own proposal
- [ ] `architecture-reviewer` subagent committed under `.claude/agents/` and smoke-tested — asked to review a design produced in the same session, it **stopped and said the review is not independent**; asked to soften a High because the gate was due, it restated the counts
- [ ] `render-design-diagrams` skill committed under `.claude/skills/` and smoke-tested — with the Eraser MCP server unreachable it **said so and stopped**, rather than producing the Mermaid mirror alone and presenting it as the diagram set
- [ ] `/adr` command committed at `.claude/commands/adr.md` and smoke-tested — it reported the number it allocated and what it allocated from, and **refused to write a record whose only rejected alternative was a straw man**

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
- [ ] Auth defined in spec on every endpoint absent from the **declared public-endpoint list**
- [ ] Idempotency-Key header on every POST on the **declared non-idempotent-POST list**
- [ ] `x-prd-section` extension links each operation to its PRD anchor
- [ ] Mock servers (Prism) accessible — URL in project README
- [ ] Naming conventions consistent throughout
- [ ] Migration scripts generated and run cleanly against a local DB
- [ ] Seed data present for every entity (5-10 rows)
- [ ] Reviewed by Tech Lead + ≥ 1 backend developer
- [ ] Spec marked "frozen" — change-request process documented
- [ ] `data-model-design` skill committed under `.claude/skills/` and smoke-tested — its backend-developer review stop proved **non-skippable**, including when told the reviewer was unavailable, and it refused to denormalise without a measured query and a recorded decision
- [ ] `api-contract-freeze` skill committed under `.claude/skills/` and smoke-tested — with no public-endpoint list and no non-idempotent-POST list supplied it **refused to audit**, and it left the freeze note unsigned rather than marking the spec frozen itself

> **The two declared lists are what make the two lines above falsifiable.** "Protected" and "requires idempotency" are product judgements, not properties of the spec. Without the lists, an agent infers them and then audits its own inference — which passes every time and tells you nothing. The lists are the human input the audit is checked against; the rest of the audit is mechanical.

**Pass:** All checked. Schema + API contracts approved.

---

## Gate 3: Wireframes & Tech Stack

- [ ] Wireframe screens committed to the repo (React + Tailwind + shadcn); deploy-preview or Storybook link recorded in README
- [ ] Design system established in the repo (inferred from code, or bootstrapped) — every wireframe inherits its tokens
- [ ] frontend-design Skill enabled in Claude Code
- [ ] Wireframes exist for ALL key user flows from the PRD
- [ ] All states shown per primary screen: loading, empty, error, success
- [ ] Responsive: desktop (1440) + mobile (375) variants
- [ ] Accessibility review passes WCAG 2.1 AA (0 Critical, 0 High) — an `accessibility-auditor` verdict of PASS or PASS WITH FIXES. **`PASS WITHHELD` does not pass this gate**: it means the rendered-page scan or the keyboard traversal was never run, so the criteria that can only be established on a rendered page — contrast ratio, reflow at 320 CSS px, resize to 200%, computed tap-target size, focus order against visual reading order — are unmeasured rather than met
- [ ] For any component promoted to the shared library: props/variants tightened, render + a11y tests added, PR opened
- [ ] Stakeholder sign-off captured on the previewed screens (deploy preview / Storybook / Figma)
- [ ] If Figma was used: the Figma file link is recorded and its tokens are reconciled with the repo
- [ ] Tech stack ADRs complete for every major decision (language, framework, datastore, cache, broker, IaC, observability, auth)
- [ ] Each unfamiliar tech choice has a training plan attached to its ADR
- [ ] Team reviewed and agreed (no unresolved objections)
- [ ] Any fallback tool used (Figma / v0 / Bolt) recorded with the reason in the ADR for that flow
- [ ] `accessibility-auditor` subagent committed under `.claude/agents/`, placeholders filled, and smoke-tested — asked to sign off without the rendered-page scan it returned **`PASS WITHHELD`** rather than PASS, and asked to state a contrast ratio it could not measure it reported the pair unmeasured instead of producing a number

**Pass:** All checked. Wireframes approved by PM. Stack approved by team.

---

## Gate 4: Phase Handoff

- [ ] Architecture proposal (produced by `solution-architect`) + design review report (produced by `architecture-reviewer`, in a session that did not author the design) in `/docs/architecture/`
- [ ] ADRs in `/docs/adrs/` (architecture + stack + any denormalisation)
- [ ] Eraser DSL + exports in `/docs/diagrams/eraser/`
- [ ] Mermaid `.mmd` files in `/docs/diagrams/mermaid/` (C4 Context + Container, ER, key sequences)
- [ ] Database schema in source directory (`/src/db/` or `/prisma/`); migrations + seeds present
- [ ] OpenAPI 3.1 spec at `/docs/api/openapi.yaml`; mock server URL in README
- [ ] Wireframe screens in the repo + deploy-preview / Storybook link (and Figma file link if used) referenced from README
- [ ] Accessibility report at `/docs/accessibility/wcag-aa-report.md`
- [ ] Tech stack ADRs + training plans complete
- [ ] Linear: every Phase 2 artifact URL linked from a `phase:design` Issue, and the Phase 1 PRD Document is still cited from each artifact
- [ ] Dev team confirms enough information to start Phase 3 coding (no open "what does this even mean" questions)
- [ ] All seven Phase 2 templates are installed and committed — three subagents listed by `/agents`, three skills discoverable under their slugs, `/adr` in the `/` menu — with every placeholder filled and no `{{...}}` left in any of them

**Pass:** All artifacts stored and linked. Dev team confirms readiness.

---

## Metrics

| Metric | Target |
|--------|--------|
| Architecture proposal time (1.1 → 1.8) | < 2 days |
| Eraser diagram first-pass time (per diagram) | < 15 min |
| Mermaid in-repo C4 generation | < 30 min total |
| OpenAPI spec generation + Swagger walkthrough | < 1 day |
| frontend-design Skill wireframe per user flow (incl. iteration) | < 4 hours |
| Accessibility review issues at Gate 3 | 0 Critical, 0 High |
| Design review issues at Gate 1 | 0 Critical, ≤ 3 High (resolved before pass) |
| API contract change-requests after frontend start | 0 (any change requires PM approval + impact analysis) |
| Fallback-tool use in Step 4 | ≤ 1 per sprint (more than that triggers a frontend-design-coverage retro) |
