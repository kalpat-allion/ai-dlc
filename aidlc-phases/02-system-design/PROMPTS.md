# Phase 2: System Design — Prompt Templates

> **All prompts target Claude (chat or Code) or Claude Design.** Eraser.io is driven from Claude via MCP — the architect stays in Claude and Eraser receives the call. Mermaid is generated in chat and committed to the repo by hand. Cursor / v0 / Figma are escalation-only and have no prompts here.

---

## Architecture Proposal

```
You are a senior solutions architect designing a system.

## Project Context
- **Product:** [Name and one-line description]
- **PRD reference:** [Linear Document URL + version, e.g., v1.0]
- **Key Features:** [5-8 most important features]
- **NFRs:** Concurrent users: [n], Response time: [target], Availability: [target], Data sensitivity: [type], Compliance: [requirements]
- **Team:** [Size, expertise, tech strengths]
- **Budget:** [Monthly infra budget]
- **Timeline:** [MVP deadline]

## PRD Summary
[Paste PRD executive summary + functional requirements, or reference the Linear Document via the Linear connector]

Generate **2-3 architecture options**, each with:
1. **Name** — Descriptive label
2. **Overview** — 3-4 sentence description
3. **Components** — Every major component with its responsibility
4. **Communication** — How components talk (REST, gRPC, events, queues)
5. **Data Architecture** — Database(s), caching, data flow
6. **Infrastructure** — Cloud services, hosting approach
7. **Trade-offs:** Scalability, Complexity, Cost (at launch + 10x), Team Fit, Time to MVP
8. **Top 3 Risks**

For each option, also list the bounded contexts you would carve and how they map to the components.

Provide a **recommendation** with clear justification. Cite the PRD section for any feature-driven choice.

If the PRD reference, NFRs, or budget are missing or read as placeholders (e.g., "[n]", "TBD"), stop and list exactly what input you need before generating options. Do not invent NFR targets, team profiles, or compliance requirements.

## Worked example (trimmed)
Input: B2C mobile-first marketplace, ~50k MAU, p95 < 400ms, $3k/month infra, 4-engineer team strong in TypeScript.
Trimmed output shape:
- **Option A — Modular monolith on Fly.io + Postgres + Redis**
  - Components: Next.js web, Fastify API, Postgres (RDS-equivalent), Redis cache, S3-compatible blobs
  - Trade-offs: low ops burden, fits team, scales vertically to ~10x; horizontal split deferred until clear seams
  - Top risks: Postgres connection storm at peak, Fly region failover latency, search relevance
  - Bounded contexts: Catalog, Orders, Identity → all in monolith, separate modules
- **Option B — Service-split on AWS ECS Fargate** (sketch only)
- **Recommendation:** Option A. Justification cites PRD §2.1 (team velocity) and §4.3 (cost ceiling). Option B rejected: ops surface exceeds team capacity at this scale.
```

---

## Trade-off Interrogation

> Run after the architecture-proposal output, once per option you are seriously considering. Forces failure modes into the open before they are baked in.

```
You are a senior reliability engineer stress-testing a proposed architecture. Your job is to find where it breaks first, not to validate it.

## Architecture option to interrogate
[Paste a single architecture option from the architecture-proposal output — name, components, communication, data architecture]

## NFRs to honour
- Concurrent users today: [n], at 10x: [10n]
- p95 latency target: [target]
- Availability target: [target]
- Monthly infra budget: [budget]

Answer each of the following directly. If an input is missing, say so and stop — do not assume traffic shape, data sizes, or team capacity.

1. **Where does this break first at 10x current load?** Identify the single component or interaction that fails earliest. State the failure mode (CPU, IO, connection pool, lock contention, queue backlog, etc.) and the approximate threshold.
2. **Single points of failure** — list every component whose loss takes down a user-visible flow. For each, state the blast radius and the cheapest mitigation.
3. **Cost cliffs** — name any component whose cost grows non-linearly with load (egress, cross-AZ, per-request services, write-heavy DBs). Estimate the cost at 1x and 10x.
4. **State and contention** — where does shared state live, and what happens when two writers race for it?
5. **Failure isolation** — when component X dies, what stays up? Draw the dependency graph implied by the architecture.
6. **Mitigation cost** — for the top 2 failure modes above, give the rough engineering cost (hours/days) to mitigate.

Output as a numbered list matching the questions above. Be specific — "Postgres connection pool exhausts at ~3000 RPS with current pgbouncer settings" beats "database might struggle".
```

---

## Eraser Architecture Diagram (via MCP)

> Use with the Eraser MCP connector enabled.

```
Via Eraser MCP, generate a [cloud architecture / system overview / sequence] diagram for the architecture below.

## Architecture
[Paste the chosen architecture description from the architecture-proposal output]

## Required elements
- All major components from the proposal, with labels
- Communication arrows labelled with protocol (REST, gRPC, event, queue)
- Trust boundaries (VPC, public/private subnet, internet edge) drawn explicitly
- Data stores rendered as cylinders, queues as queue shapes, edge layer separated from application layer

## Style
- Use Eraser's [cloud architecture / sequence] diagram syntax
- Group components by tier
- Title: "[Product] - [Diagram type] - [PRD version]"

After generating, return:
1. The Eraser editor URL so I can refine
2. The full DSL so I can commit it to /docs/diagrams/eraser/
3. A PNG export

If anything in the description is ambiguous, list the assumption you made before generating, do not silently invent.
```

---

## Eraser ER Diagram (via MCP)

```
Via Eraser MCP, generate an entity-relationship diagram from the schema below.

## Schema
[Paste the ORM schema or DDL from schema-generation output]

## Required
- Every entity rendered with all fields, types, and PK/FK markers
- Relationships drawn with correct cardinality (1:1, 1:N, N:M)
- Junction tables shown explicitly for N:M
- Group entities by bounded context (use Eraser's grouping syntax)

Return: editor URL + DSL + PNG export.
Flag any relationship in the schema where cardinality is ambiguous from the field names alone.
```

---

## Architecture Decision Record Generation

```
You are a tech lead writing an ADR for the engineering team's permanent record. Be specific and durable — this document outlives the people who wrote it.

Document this architecture decision.

- **Decision:** [What we're deciding]
- **PRD section driving this decision:** [Linear Document anchor, e.g., §3.2]
- **Options considered:** [List options]
- **Constraints:** [Budget, expertise, compliance, performance]

Generate an ADR:

# ADR-[NNN]: [Title]
## Status: Proposed
## Context
[Situation requiring this decision; cite the PRD section by anchor]
## Decision
[What we decided and why]
## Consequences
### Positive
[What becomes easier]
### Negative
[What becomes harder / trade-offs accepted]
### Neutral
[What changes but is neither better nor worse]
## Alternatives Considered
### [Alt 1]
- Pros:
- Cons:
- Reason rejected: [specific, not "less suitable"]
### [Alt 2]
- Pros:
- Cons:
- Reason rejected:
## Validation
[How we will know this decision was right or wrong - measurable signal]
```

---

## Entity Extraction

> Run before schema-generation. Produces the structured entity list and bounded-context map that feeds it.

```
You are a domain modeller extracting entities from a product spec. Your output drives the database schema, so be precise about identity, ownership, and cardinality.

## PRD source
[Paste the PRD functional-requirements + user-flow sections, or reference the Linear Document via the Linear connector]

## Architecture context
- **Bounded contexts from Step 1:** [List the contexts the architecture proposal carved, e.g., Identity, Catalog, Orders]
- **Communication pattern between contexts:** [REST / events / shared DB — from the chosen architecture]

Produce:

1. **Entity list** — every domain entity with:
   - Name (singular, PascalCase)
   - Owning bounded context
   - Purpose (one sentence — what real-world thing it represents)
   - Identity (natural key, surrogate, or composite)
   - Attributes with type intent (string, int, decimal(10,2), enum, json, timestamp) and nullability
   - Lifecycle (created when, deleted/archived when)

2. **Relationships** — for every link between entities:
   - Source → target
   - Cardinality (1:1, 1:N, N:M)
   - Whether it crosses a bounded-context boundary (if yes, flag it — these likely become events or duplicated read models, not foreign keys)

3. **Cross-context concerns** — entities that span contexts. For each, recommend: shared FK, duplicated read model, or event-based propagation, with a one-line reason.

4. **Open questions** — anything in the PRD that left identity, ownership, or cardinality ambiguous. Do not guess; list it.

Output as Markdown with the four sections above. If the PRD reference is missing or unreadable, stop and ask for it — do not invent entities.
```

---

## Schema Generation

```
You are a backend engineer designing a normalised, query-aware database schema. Match existing repo conventions exactly when running in Claude Code.

Generate a database schema from these requirements.

## Data Requirements
[Paste PRD entities, relationships, data-related stories or reference the Linear Document]

## Technical Context
- **Database:** [PostgreSQL / MongoDB / etc.]
- **ORM:** [Prisma / TypeORM / Drizzle / raw SQL]
- **Naming:** [snake_case / camelCase]
- **Existing repo conventions:** [if running Claude Code, infer from repo; otherwise paste relevant existing schema fragments]

## Top query patterns
[List the 5-10 most common queries the application will run - drives index design]

Generate:
1. All entities with fields, types, constraints, defaults
2. Relationships with foreign keys and cardinality
3. Indexes covering the top query patterns above
4. Enums where applicable
5. Timestamps (created_at, updated_at) on all entities
6. Soft delete (deleted_at) where appropriate
7. A Mermaid ER diagram (erDiagram syntax) for /docs/diagrams/mermaid/
8. Initial migration script
9. Seed data (5-10 records per entity)

Output in [Prisma/TypeORM/SQL DDL] format. Flag any field where the PRD is ambiguous about type or nullability rather than guessing.
```

---

## Migration Generation

> Run from Claude Code at Step 2.6 once the schema is reviewed. Produces the initial migration plus seed data so the schema is verified end-to-end before Gate 2.

```
You are a backend engineer producing the first migration for a new schema. Output must run cleanly against an empty database and against a database where the previous migration already ran.

## Inputs
- **ORM / migration tool:** [Prisma migrate / TypeORM / Drizzle / Flyway / Alembic / raw SQL]
- **Database:** [PostgreSQL 16 / MySQL 8 / etc. — include version, it changes available syntax]
- **Schema source:** [Paste the schema, or point at the path the schema lives in if running in Claude Code]
- **Existing migrations directory:** [path, or "none — this is the first migration"]
- **Seed-data scope:** [dev only / dev + test / dev + test + staging]

Produce:

1. **Migration up script** — creates every table, index, enum, constraint, and trigger from the schema. Order DDL so foreign keys resolve.
2. **Migration down script** — reverses the up script. If a column drop would lose data irrecoverably, add a comment flagging it; do not silently destroy production data.
3. **Seed-data script** — 5-10 rows per entity, realistic values (not "foo"/"bar"), referential integrity preserved. Use deterministic IDs/UUIDs so tests are stable.
4. **Run instructions** — exact commands to apply, roll back, and re-seed locally.

Constraints:
- No data-destructive operation (DROP COLUMN, ALTER TYPE that loses precision, table rename without copy) without an explicit comment justifying it.
- All timestamps default to UTC.
- Every script is idempotent where the tool supports it (e.g., `CREATE TABLE IF NOT EXISTS` for raw SQL).

If the schema references an entity, enum, or extension you cannot see, stop and list what is missing — do not stub it.
```

---

## API Contract

```
You are an API designer producing a contract that backend, frontend, and external integrators will all hold to.

Generate a complete OpenAPI 3.1 YAML specification.

- **API Name:** [name]
- **Base URL:** /api/v1
- **Auth:** JWT Bearer tokens
- **Data Model:** [Paste schema or reference /src/db/schema.prisma]
- **User Flows:** [Reference Linear PRD Document sections]

Include:
1. info, servers, security schemes
2. All CRUD endpoints per resource with proper HTTP methods
3. Request bodies with JSON Schema (required fields, types, validation, examples)
4. Responses: 200, 201, 400, 401, 403, 404, 409, 422, 500 with consistent error envelope
5. Pagination on every list endpoint (cursor-based by default)
6. Filtering and sorting query params
7. Idempotency-Key header on POSTs that need it
8. Reusable components (schemas, parameters, responses, errors)
9. Tags grouped by resource, operationId on every operation

Conventions:
- plural lowercase resources, kebab-case for multi-word
- consistent error envelope: { "error": { "code": "", "message": "", "details": [] } }
- include `x-prd-section` extension on each operation linking to the PRD anchor

Validate the output is parseable OpenAPI 3.1. Flag any user flow you could not express in REST.
```

---

## Tech Stack Comparison

```
You are a pragmatic tech lead evaluating options against the team that will actually operate them. Bias toward what the team can ship and run, not what scores best on paper.

Evaluate technology options for this project.

## Decision
Select a [frontend framework / database / queue / observability stack / etc.]

## Options
[Option A], [Option B], [Option C]

## Criteria (with weights)
- Team expertise (20%): [current skills]
- Performance (20%): [targets]
- Scalability (15%): [growth expectations]
- Operational maturity in our environment (15%): [have we run it in prod before]
- Budget (15%): [monthly limit]
- Ecosystem fit (10%): [must integrate with...]
- Client suitability (5%): [data residency, IP, compliance]

For each option evaluate every criterion (1-5 with reason), then produce:
1. A scored comparison matrix
2. A clear recommendation
3. The single biggest risk of the recommendation
4. The training plan required if the team has not shipped this before

Be honest about gaps — if option scoring is close enough to be a coin-toss, say so.
```

---

## Design Review

```
You are a staff engineer doing a pre-build production-readiness review. Be specific, prioritise blockers, do not pad with generic advice.

Review this system architecture for production readiness.

## Architecture
[Paste architecture description + Eraser diagram URLs + Mermaid C4]

## NFRs
[Paste non-functional requirements from PRD]

## Trade-off interrogation answers
[Paste the answers from the architecture-proposal stress-test follow-up]

Review across:
1. **Scalability** — Bottlenecks at 10x and 100x load? Single points of failure? Independent scaling per component? Where does state become contention?
2. **Security** — Auth boundary clear? Every API authenticated? Data encrypted at rest + in transit? Secrets managed? Input validation? Threat model for the highest-risk flow?
3. **Reliability** — Component failure handling? Retries with backoff + jitter? Circuit breaking? Idempotency on writes? DR strategy + RPO/RTO?
4. **Operability** — Zero-downtime deploy? Observability (logs, metrics, traces)? Health checks? Rollback plan? Runbook for the top 3 failure modes?
5. **Cost** — Over-provisioning? Cheaper alternatives for non-critical components? Cost drivers at 10x scale? Egress / cross-AZ traffic considered?

For each issue:
- **Issue** (specific, not "consider observability")
- **Severity** (Critical / High / Medium / Low)
- **Recommendation** (concrete change, not "investigate further")
- **PRD section affected** (if any)

Flag anything in the proposal that you cannot evaluate from the inputs given - do not assume.

End with a one-line verdict: PASS / PASS WITH FIXES / FAIL, plus the count of Critical and High issues.
```

---

## Claude Design — System Bootstrap

> Run once per project at Step 4.1, especially for greenfield work without an existing design system.

```
We are starting a new project that has no existing design system. Build a starter design system for it.

## Product context
- **Product:** [Name + one-line description]
- **Audience / persona summary:** [Paste from PRD]
- **Brand cues we have:** [logo URL or description, brand colours if any, voice/tone notes]
- **Reference products we admire (and what we admire):** [3-5 examples]
- **Reference products we explicitly want to avoid:** [3-5 examples + why]

## What I need
1. A primary + secondary + accent colour palette with hex values, justified against the audience
2. A typography pairing (display + body + mono) that is NOT Inter/Roboto/Open Sans/Lato (default-tier)
3. Core component tokens: button (primary/secondary/ghost/destructive), input, card, modal, toast, table - with hover/focus/disabled states
4. Spacing scale (4-8 stops) and radius scale
5. One reference page (a representative dashboard or detail screen) that uses everything above

Render this as a live HTML/CSS preview in this Claude Design project. After I approve, this becomes the default design system every subsequent wireframe inherits.
```

---

## Claude Design — Wireframe per Flow

```
Generate a wireframe for the user flow below. Use this project's design system - do not introduce new fonts, colours, or components.

## Flow
- **Name:** [e.g., "Therapist accepts a session request"]
- **Persona:** [from PRD]
- **PRD section:** [Linear Document anchor]
- **Trigger:** [What starts this flow]
- **Happy path steps:** [Step 1 → Step 2 → Step 3]
- **Error / edge cases:** [list]
- **Data on screen:** [What information must be visible at each step]

## Required deliverables
1. Desktop layout (1440 width) for every step in the happy path
2. Mobile layout (375 width) for every step
3. All states for the primary screen: loading, empty, error, success
4. Inline annotations on the canvas pointing to non-obvious interactions

## Constraints
- Tap targets ≥ 44px on mobile
- Keyboard-reachable order matches visual reading order
- All copy is real placeholder copy, not Lorem Ipsum
- Use shadcn-aligned components where possible

Render in this Claude Design project. I will refine via inline comments.
```

---

## Claude Design — Production-Grade Component Generation

> Use when a wireframe component is mature enough to ship as code. Triggers Handoff to Claude Code.

```
Promote this component from wireframe to production-grade code.

## Component
[Click the component on the canvas, or describe it]

## Production requirements
- **Stack:** React + TypeScript + Tailwind + shadcn/ui (match repo conventions)
- **Accessibility:** WCAG 2.1 AA - keyboard nav, focus-visible, ARIA only where semantic HTML is insufficient, prefers-reduced-motion respected
- **States:** loading (skeleton), empty, error (with retry), success
- **Variants:** [size: sm/md/lg, intent: primary/secondary/destructive, etc.]
- **Props API:** typed, no anys, controlled + uncontrolled where it makes sense
- **Tests:** at least one happy-path render test and one a11y assertion (axe or testing-library variant)

## Repo target
- Branch name: feature/[component-name]-handoff
- Path: [src/components/ui/[component-name].tsx]
- Storybook story: [optional - include if repo uses Storybook]

Hand off to Claude Code now. Open the PR with the wireframe URL in the description for review traceability.
```

---

## Accessibility Review (Claude)

```
Review the component / page below against WCAG 2.1 Level AA.

## Component / page
[Paste the rendered HTML, the React source, or the Claude Design canvas link]

## Context
- **Stack:** [React + Tailwind + shadcn]
- **Brief:** [What this UI is for, who uses it]

Audit each of the following and produce findings as Issue / Severity / Recommendation / WCAG ref:

1. **Perceivable** — text alternatives, captions, contrast (≥ 4.5:1 normal, 3:1 large), text resize, content reflow
2. **Operable** — keyboard accessible (no traps), focus-visible, focus order matches DOM, skip links on long pages, no flashing > 3Hz, motion respects prefers-reduced-motion
3. **Understandable** — language declared, predictable navigation, form labels and error messages clear, input purpose identified
4. **Robust** — valid semantic HTML, ARIA only where needed and correct, name/role/value exposed to AT, status messages announced
5. **Tap targets** — ≥ 44x44px on mobile

Report:
- Critical: blocking issues (e.g., interactive control unreachable by keyboard)
- High: clear violations (e.g., contrast 3.8:1 on body text)
- Medium: best-practice misses
- Low: polish

End with a one-line verdict: PASS / PASS WITH FIXES / FAIL.
```
