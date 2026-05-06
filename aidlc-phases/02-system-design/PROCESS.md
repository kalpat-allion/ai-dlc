# Phase 2: System Design — Process Definition

## Overview

This document defines the AI-assisted workflow for the System Design phase, using the consolidated AI-DLC tool stack.

**Phase Duration:** 1–2 weeks (varies with system complexity)
**Phase Owner:** Tech Lead / Solutions Architect
**Tools Used:** Claude (chat + Code), **Claude Design** (UI/UX), **Eraser.io** (diagrams, integrated with Claude via MCP), Mermaid (in-repo diagram-as-code), free Swagger UI + Prism (OpenAPI viewing & mocking)

> **Tool Philosophy:** Claude does **all** of the design reasoning — architecture proposals, ADRs, schema generation, OpenAPI specs, tech-stack trade-offs, and design review. **Eraser.io is the single specialist visual tool**, and it is driven from inside Claude through the official Eraser MCP server, so the architect never context-switches into a separate diagram app to do the first draft. **Claude Design** ([anthropic.com/news/claude-design-anthropic-labs](https://www.anthropic.com/news/claude-design-anthropic-labs)) is the primary surface for UI/UX wireframing and prototyping — it generates live HTML/CSS/React (not pixel mockups), inherits the project's design system, and hands off directly into Claude Code. Cursor, v0, and Figma remain only as escalation paths when Claude Design genuinely cannot meet a requirement (see Step 4.7). Mermaid stays in scope for diagrams that must live in git.

---

## Process Steps

### Step 0: One-Time Setup — Connect Claude to Eraser via MCP

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | Eraser.io workspace admin access (Business tier recommended for SOC 2 Type II + audit logs), Claude account (Pro / Max / Team / Enterprise) |
| **Tool** | **Eraser MCP server** (`https://app.eraser.io/api/mcp`) — official, hosted by Eraser, OAuth 2.1; API-key fallback for CLI/CI |
| **Output** | Claude (web/desktop) and Claude Code can `generate`, `generateEdit`, `export`, `search`, `create`, `update`, and `delete` Eraser diagrams, files, folders, and presets |
| **Human** | Workspace admin enables the connector; each user authorises once via OAuth |

The Eraser MCP is the integration mechanism — do **not** wrap the Eraser REST API by hand. It is centrally hosted, OAuth-authenticated, and supported across every Claude surface ([docs.eraser.io/docs/mcp](https://docs.eraser.io/docs/mcp)). Set this up once per workspace, then once per user.

#### Path A — claude.ai (web) and Claude Desktop

1. **Workspace admin (Eraser):** confirm the workspace is on a plan that exposes the MCP endpoint and that no IP allow-list blocks `app.eraser.io`. Enable the audit log under **Settings → Security**.
2. **Workspace admin (Anthropic, Team/Enterprise only):** in the Anthropic admin console, **enable Eraser** under organisation Connectors. Set tool permissions — for Phase 2 we recommend **`generate` + `generateEdit` + `export` + `search`** with **`delete` disabled** until the team is confident (see [Risks & Guardrails](#risks--guardrails)).
3. **Each user (Claude.ai):** open **Settings → Connectors** → find **Eraser** in the directory → **Connect**. OAuth opens in a new tab; sign in to Eraser, review the requested scopes, approve.
4. **Per-conversation toggle:** open a chat, click **+** → **Connectors**, toggle **Eraser** on. (Off by default — explicit opt-in per conversation.)
5. **Smoke test:** prompt Claude with `Via Eraser MCP, list my workspaces and create a new sequence diagram titled "MCP smoke test" with two participants A and B.` Confirm the diagram appears in your Eraser workspace.

#### Path B — Claude Code (developer / CLI use)

Use Claude Code when generating diagrams from inside a repo (e.g., a Mermaid handoff or a CI-driven architecture refresh).

1. **Add the server** (run from repo root for project scope, or anywhere for user scope):
   ```bash
   claude mcp add --transport http --scope user eraser https://app.eraser.io/api/mcp
   ```
   Use `--scope project` if you want the server committed to the repo's `.mcp.json` so the team shares it. For agentic CI pipelines that cannot do interactive OAuth, use the API-key path: `claude mcp add eraser -- npx -y @eraserlabs/eraser-mcp` with `ERASER_API_KEY` in the environment.
2. **Authenticate:** open Claude Code (`claude`) → `/mcp` → select **eraser** → approve OAuth.
3. **Verify:** `claude mcp list` should show `eraser: connected`. In a session: `Via Eraser MCP, generate a cloud architecture diagram for an AWS-based three-tier web app and export it as PNG.`
4. **Revoke when done:** `claude mcp remove eraser`, and revoke the OAuth grant in Eraser under **Settings → Security → Connected applications**.

#### Verification checklist

- [ ] Eraser connector shows **Connected** in Claude settings
- [ ] Smoke-test diagram appears in your Eraser workspace
- [ ] Claude can `search` your workspace and `export` to PNG/SVG
- [ ] Audit logging is on in Eraser
- [ ] Anthropic admin tool-permission policy reviewed (Team/Enterprise) with `delete` scope disabled

> **Permission inheritance:** Claude inherits the connecting user's Eraser permissions. There is no privilege escalation. Off-board by revoking the OAuth grant.

> **Claude Design enablement:** Claude Design ships enabled-by-default for Pro/Max/Team and disabled-by-default for Enterprise ([support.claude.com/en/articles/14604416](https://support.claude.com/en/articles/14604416-get-started-with-claude-design)). For Enterprise, the admin enables it under **Admin → Features → Claude Design** before Step 4. There is no MCP setup for Claude Design — it is a first-party Claude surface.

---

### Step 1: Architecture Design

> Visual: [Step 1 flowchart](./FLOWCHART.md#step-1-architecture-design)

| Attribute | Detail |
|-----------|--------|
| **Input** | Approved PRD (Linear Document attached to the Phase 1 Linear Project — the Project URL is the canonical handoff entry point), non-functional requirements, team expertise profile, Linear context bundle |
| **Tool** | **Claude** (proposals, ADRs, review) + **Eraser.io via MCP** (diagrams) + **Mermaid** (in-repo diagrams) |
| **Output** | Architecture proposal, system + sequence + cloud diagrams, ADRs |
| **Human** | Evaluate proposals, validate against constraints, make final decisions, sign off on diagrams |

**Workflow:**

**1.1 — Pull inputs.** In a Claude conversation with the **Linear connector enabled**, fetch the approved PRD Document and any related initiatives/issues from the Phase 1 Project. Do not paste — let Claude read via MCP so the trace stays clean. Record the PRD Document URL and PRD version (`v1.0`, etc.) — every ADR and diagram must cite both.

**1.2 — Generate architecture options with Claude.** Use the [`architecture-proposal`](./PROMPTS.md#architecture-proposal) prompt. Feed in the PRD reference, NFRs, team profile, budget, and timeline. Claude produces 2–3 candidate architectures with components, communication patterns, data architecture, and trade-off tables. The output stays in chat — no diagrams or ADRs yet.

**1.3 — Trade-off interrogation.** For each option, run the [`trade-off-interrogation`](./PROMPTS.md#trade-off-interrogation) prompt (or follow up in chat with `Stress-test option 2 at 10x current load — where does it break first, and what is the mitigation?`). Force Claude to surface failure modes, single points of failure, and cost cliffs. Capture the answers; they become inputs to the design review prompt at 1.6.

**1.4 — Render diagrams in Eraser.io (via MCP).** With the Eraser connector on, run the [`eraser-architecture-diagram`](./PROMPTS.md#eraser-architecture-diagram) prompt. Claude calls Eraser MCP `generate` to produce a cloud architecture diagram and a system overview diagram in Eraser's diagram-as-code DSL. Eraser supports flowcharts, sequence diagrams, ER diagrams, cloud architecture, and BPMN/swimlane natively ([eraser.io/diagramgpt](https://www.eraser.io/diagramgpt)). For each generated diagram, Claude returns the editor URL and the DSL — open the URL, refine in the visual editor or via inline AI requests, then export PNG/SVG. The DSL is checked into `/docs/diagrams/eraser/` alongside the export so the diagram round-trips.

**1.5 — Render in-repo diagrams in Mermaid.** For diagrams that must live in version control and render in GitHub/GitLab markdown (sequence diagrams in ADRs, C4 Context/Container diagrams), use Claude to produce Mermaid directly. Mermaid in 2026 supports the full C4 model — Context, Container, Component, Dynamic ([mermaid.js.org/syntax/c4](https://mermaid.js.org/syntax/c4.html)). Commit these to `/docs/diagrams/mermaid/` and reference them from the ADR.

**1.6 — Design review with Claude.** Run the [`design-review`](./PROMPTS.md#design-review) prompt against the selected option, attaching the architecture text and the diagram links. Claude returns a severity-ranked issue list across scalability, security, reliability, operability, and cost. Resolve every Critical and High before proceeding.

**1.7 — Create ADRs.** For each significant decision (chosen pattern, datastore selection, sync vs async boundaries, deployment topology) run the [`adr-generation`](./PROMPTS.md#adr-generation) prompt. Store ADRs in `/docs/adrs/` using the [ADR template](../templates/adr-template.md). Each ADR cites the PRD section that drove the decision and embeds the relevant Mermaid diagram inline.

**1.8 — Team review.** Present the proposal, diagrams, and ADRs to the dev team. Use the design-review output as the agenda. Capture objections in the ADR's "Consequences" section before sign-off.

**▼ GATE 1 — Architecture approved.** Tech Lead signs off in the architecture proposal document. See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-architecture).

**Escalation:** If the architecture requires unfamiliar technology (new datastore, new event broker, new runtime), schedule a time-boxed spike (max 3 days) before committing. If the spike fails or the team has no realistic path to operate the technology, drop to a familiar fallback recorded in the ADR's "Alternatives Considered" section. If the architecture cannot satisfy the PRD's NFRs at any reasonable cost, **loop back to Phase 1** — the NFRs themselves need renegotiation, not the design.

---

### Step 2: Data Modelling

> Visual: [Step 2 flowchart](./FLOWCHART.md#step-2-data-modelling)

| Attribute | Detail |
|-----------|--------|
| **Input** | Architecture proposal (Step 1), PRD entities, query patterns from PRD user flows |
| **Tool** | **Claude / Claude Code** (schema generation, migrations) + **Eraser.io via MCP** (ER diagrams) + **Mermaid** (ER in repo) |
| **Output** | Database schema (in ORM format), ER diagram (Eraser + Mermaid), migration scripts, seed data |
| **Human** | Validate relationships, check normalisation, review indexes, approve denormalisation |

**Workflow:**

**2.1 — Extract entities.** Run the [`entity-extraction`](./PROMPTS.md#entity-extraction) prompt with the Linear connector on. Claude reads the PRD Document and produces a structured entity list with attributes, relationships, cardinalities, and a cross-context map. Entities that span bounded contexts may need duplication or an event contract, not a foreign key — the prompt flags these explicitly.

**2.2 — Generate schema with Claude Code.** From inside the repo, run the [`schema-generation`](./PROMPTS.md#schema-generation) prompt. Specify the ORM (Prisma / TypeORM / Drizzle / raw SQL) and naming convention. Claude Code reads existing repo conventions and produces a schema that matches them — this is the key reason to drive schema generation from Claude Code rather than chat. Output includes constraints, indexes covering the top query patterns, soft-delete columns where appropriate, and timestamps on every entity.

**2.3 — Render ER diagram in Eraser.io.** Run the [`eraser-er-diagram`](./PROMPTS.md#eraser-er-diagram) prompt. Claude calls Eraser MCP `generate` with the schema as input — Eraser's ER syntax is purpose-built for this and accepts paste of either DDL or natural language ([eraser.io/use-case/api-diagrams](https://www.eraser.io/use-case/api-diagrams)). Refine in the Eraser editor, export PNG/SVG, commit DSL to `/docs/diagrams/eraser/`.

**2.4 — Render ER diagram in Mermaid for in-repo docs.** Ask Claude for the Mermaid `erDiagram` equivalent. Commit to `/docs/diagrams/mermaid/erd.mmd` so it renders in the README and ADRs.

**2.5 — Review.** Walk the schema with a backend developer. Check: third-normal form unless explicitly denormalised, foreign-key cardinalities match PRD, every list-page query has a covering index, every search field has a strategy (btree, gin, full-text). Flag any AI-suggested denormalisation — it must be backed by a measured query, not a vibe.

**2.6 — Generate migrations and seed data.** Run the [`migration-generation`](./PROMPTS.md#migration-generation) prompt in Claude Code to produce the initial migration script in the chosen ORM's format, plus 5–10 seed records per entity for local dev and tests. Run the migration against a local DB; the schema is not "done" until it migrates cleanly.

**Escalation:** Denormalisation for performance must be documented in an ADR with the measured query that justifies it. Do not denormalise on AI suggestion alone. If query patterns reveal an entity model the PRD did not anticipate, **loop back to Phase 1** to add it before continuing.

---

### Step 3: API Contract Design

> Visual: [Step 3 flowchart](./FLOWCHART.md#step-3-api-contract-design)

| Attribute | Detail |
|-----------|--------|
| **Input** | Architecture (Step 1), data model (Step 2), user flows from the PRD |
| **Tool** | **Claude / Claude Code** (OpenAPI 3.1 generation) + free **Swagger UI** (browse/test) + **Prism** (mock servers) |
| **Output** | OpenAPI 3.1 specification, mock server endpoints, API documentation |
| **Human** | Validate endpoint design, review error envelope, approve contracts, gate frontend start |

**Workflow:**

**3.1 — Map flows to endpoints.** Use Claude with the Linear connector on to read each PRD user-flow section and propose the resource model and endpoints. Resolve naming and verb questions here before writing any spec — REST mistakes calcify fast.

**3.2 — Generate OpenAPI 3.1 spec.** Run the [`api-contract`](./PROMPTS.md#api-contract) prompt in Claude Code so the spec aligns with existing repo conventions. Claude produces a complete OpenAPI 3.1 YAML with: info + servers + security schemes, all CRUD endpoints, request bodies with JSON Schema, success and error responses (200/201/400/401/403/404/409/422/500), pagination on every list endpoint, reusable components, and consistent tagging. Commit to `/docs/api/openapi.yaml`.

**3.3 — View and refine in Swagger UI.** Spin up Swagger UI locally (Docker one-liner) or use Swagger Editor in the browser. Walk every endpoint, every example, every error response. The spec is the contract — anything missing here becomes a frontend/backend argument later.

**3.4 — Stand up mock servers with Prism.** Prism remains the recommended OSS mock server in 2026 with full OpenAPI 3.1 support ([stoplight.io/open-source/prism](https://stoplight.io/open-source/prism)): `prism mock docs/api/openapi.yaml`. Frontend can start integration the same day. Add the mock URL to the project README. Mockoon is acceptable as a desktop alternative when Prism is awkward (Windows-heavy teams).

**3.5 — Review and freeze.** Tech Lead reviews every endpoint against the [Gate 2 checklist](./QUALITY-GATES.md#gate-2-data-model--api-contracts): REST conventions, error envelope consistency, auth on every protected endpoint, pagination defined, idempotency keys on POSTs that need them. On approval, the spec is **frozen** — any subsequent change requires a documented change request and impact analysis on already-mocked frontend work.

**▼ GATE 2 — Data model + API contracts approved.** Tech Lead + one backend developer sign off. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-data-model--api-contracts).

**Escalation:** API contract changes after frontend development begins require PM approval and a documented impact analysis appended to the ADR for that contract. If a flow cannot be expressed cleanly in REST (long-lived workflows, real-time subscriptions), surface a separate ADR for the alternative (gRPC, GraphQL, WebSocket, server-sent events) — do not bend REST until it breaks.

---

### Step 4: UI/UX Wireframing — with Claude Design

> Visual: [Step 4 flowchart](./FLOWCHART.md#step-4-uiux-wireframing)

| Attribute | Detail |
|-----------|--------|
| **Input** | PRD user flows, personas, functional requirements, brand assets (if any), repo URL |
| **Tool** | **Claude Design** (primary — wireframes, interactive prototypes, design system) + Claude (accessibility review) |
| **Output** | Interactive prototypes (live HTML/CSS/React), component specifications, accessibility report, Claude Code handoff |
| **Human** | Validate UX, review accessibility, approve designs, drive iterations via inline comments |

**Workflow:**

**4.1 — Onboard the design system once per project.** In Claude Design, create a new project and link the codebase repo plus any existing design files. Claude Design reads the repo and design files to build a custom design system — colours, typography, components — that every subsequent prototype inherits automatically ([anthropic.com/news/claude-design-anthropic-labs](https://www.anthropic.com/news/claude-design-anthropic-labs)). For greenfield projects with no design system, brief Claude Design on the brand (or run the [`claude-design-system-bootstrap`](./PROMPTS.md#claude-design-system-bootstrap) prompt) and accept its proposed system before any wireframing begins.

**4.2 — Enable the frontend-design Skill.** In Claude.ai → Settings → Skills, enable the official **frontend-design** Skill ([claude.com/blog/improving-frontend-design-through-skills](https://claude.com/blog/improving-frontend-design-through-skills)). It biases Claude away from generic Inter-and-purple-gradient defaults and towards distinctive typography, intentional motion, and considered colour palettes — exactly what wireframes need to communicate intent. The Skill cooperates with Claude Design and Artifacts; no extra setup beyond toggling it on.

**4.3 — Generate wireframes per user flow.** For each PRD user flow, run the [`claude-design-wireframe`](./PROMPTS.md#claude-design-wireframe) prompt in Claude Design. Output is **live HTML/CSS/React in a Claude Design canvas**, not a pixel mockup — clickable, with real layout, real components, real behaviours. Claude Design produces all standard states out of the box: loading, empty, error, success, plus mobile and desktop variants when prompted.

**4.4 — Iterate via inline comments and chat.** Refine through two channels: **inline comments** (click a region of the canvas, request a targeted change — "tighten this header, increase tap target on the primary CTA") and **chat** (broad changes that affect the whole design). The Skill plus Claude Design's design-system inheritance keeps refinements visually consistent across flows.

**4.5 — Production-grade components when needed.** When a wireframe needs to evolve directly into production code, use Claude Design's **Handoff to Claude Code** action. Claude Design exports the React + Tailwind + shadcn-aligned components into the repo via Claude Code's local agent, preserving the design-system tokens. The output is editable code, not a screenshot — eliminating the wireframe-to-code translation step.

**4.6 — Accessibility review.** Run the [`accessibility-review`](./PROMPTS.md#accessibility-review) prompt against each generated component. Claude checks WCAG 2.1 AA: keyboard navigation, focus order, colour contrast, screen-reader labelling, semantic HTML, motion-reduction respect. Resolve every Critical and High; document accepted Mediums in the ADR for that flow.

**4.7 — Stakeholder review.** Share the Claude Design project link with PM and stakeholders. Stakeholders comment inline directly on the canvas. Iterate. Approve.

**▼ GATE 3 — Wireframes approved.** PM signs off in the Claude Design project. See [QUALITY-GATES.md → Gate 3](./QUALITY-GATES.md#gate-3-wireframes--tech-stack).

**Escalation paths (Claude Design fallbacks):**
- **Pixel-perfect brand work or designer-led flow** → Figma (the standard design tool, not the AI add-on). Designers in Figma, developers receive coded components from Claude Design or implement from Figma specs.
- **Full-stack interactive demo with real backend** → Bolt.new or Lovable. Treat as a throwaway exploration; do not ship the output.
- **A specific dropped-in component for an existing repo where Claude Design's handoff falters** → v0 by Vercel as a per-component fallback only.
- **Heavy IDE-driven UI work** → Cursor with the shadcn registry. Reserve for cases where the flow lives more in the codebase than in the design canvas.

If the team lands in fallback territory more than once a sprint, that is a signal — re-evaluate Claude Design coverage rather than normalising the workaround.

If wireframes reveal user flows not in the PRD, **loop back to Phase 1** — edit the Linear PRD Document (bumping the version per the changelog rule) and re-run `linear-gap-sweep` before resuming Phase 2.

---

### Step 5: Tech Stack Definition

> Visual: [Step 5 flowchart](./FLOWCHART.md#step-5-tech-stack-definition)

| Attribute | Detail |
|-----------|--------|
| **Input** | Architecture proposal, team expertise, constraints, PRD NFRs |
| **Tool** | **Claude** (trade-off analysis, ADRs) + Perplexity (current data, optional) |
| **Output** | Tech stack decisions + ADRs, training plan for unfamiliar choices |
| **Human** | Final technology decisions, team buy-in, training-need owner |

**Workflow:**

**5.1 — Frame each decision.** List every tech choice that needs to be made: language, framework, datastore(s), cache, broker, IaC, observability, auth provider. For each, note the constraint set: team expertise, performance, scalability, budget, ecosystem fit.

**5.2 — Generate scored comparisons.** Use Claude with the [`tech-stack-comparison`](./PROMPTS.md#tech-stack-comparison) prompt for each non-trivial decision. Claude produces a 1–5 scored matrix across criteria and a recommendation. For technology that ships fast (frontend frameworks, AI tooling), supplement with Perplexity for current benchmarks and adoption data — Claude's training data is good but not real-time.

**5.3 — Create ADRs.** Run the [`adr-generation`](./PROMPTS.md#adr-generation) prompt for each decision. Every ADR includes the rejected alternatives with reasons — future engineers thank you for this.

**5.4 — Team review and buy-in.** Walk the stack with the dev team. The blocker is rarely "is this the right tool" — it is "do we know how to operate it in production". For every choice the team has not shipped before, attach a training plan (book chapter, course, internal mentor, time budget) to the ADR.

**Escalation:** If a stack choice forces an architectural change (e.g., the chosen DB cannot meet the latency NFR), loop back to Step 1.6 design review with the new constraint. Do not paper over it in the stack ADR.

---

## Phase Handoff

| Artifact | Format | Location |
|----------|--------|----------|
| Architecture proposal + design review | Markdown + embedded Mermaid | `/docs/architecture/` |
| ADRs (architecture + stack + denormalisation) | Markdown | `/docs/adrs/` |
| System / cloud / sequence diagrams | Eraser.io DSL + PNG/SVG export | `/docs/diagrams/eraser/` (DSL committed; editor URL noted) |
| In-repo diagrams (C4, ER, key sequences) | Mermaid `.mmd` | `/docs/diagrams/mermaid/` |
| Database schema | ORM schema files | `/src/db/` or `/prisma/` |
| Migration scripts + seed data | ORM migration files | `/src/db/migrations/` |
| OpenAPI spec | YAML | `/docs/api/openapi.yaml` |
| Mock server URL | README entry | Project README |
| UI wireframes / prototypes | Claude Design project link + handoff branch | Project README + Claude Design |
| Accessibility review | Markdown | `/docs/accessibility/wcag-aa-report.md` |
| Tech stack ADRs + training plans | Markdown | `/docs/adrs/` |

**Handoff Checklist:**
- [ ] Architecture approved by Tech Lead (Gate 1)
- [ ] All significant decisions documented as ADRs, each citing the PRD section
- [ ] Eraser diagrams: system overview, cloud architecture, ≥ 2 key sequences — DSL in repo, exports linked
- [ ] Mermaid diagrams: C4 Context + Container, ERD — rendering in repo
- [ ] Schema reviewed by ≥ 1 backend developer; migrations run cleanly locally
- [ ] OpenAPI 3.1 spec covers every PRD user flow; mock server URL in README
- [ ] Claude Design wireframes cover every PRD user flow with all states; accessibility review passes WCAG 2.1 AA
- [ ] Tech stack ADRs complete; every unfamiliar choice has a training plan
- [ ] Linear: every Phase 2 ADR / spec / diagram URL is linked from the corresponding Linear Issue with the `phase:design` label

---

## Risks & Guardrails

| Risk | Mitigation |
|------|------------|
| **Diagram drift** — Eraser/Mermaid diagrams diverge from the implemented system over time | Commit Eraser DSL alongside exports so diagrams round-trip via MCP. For C4/ERD, use Mermaid in repo and regenerate from the codebase via Claude Code on every ADR change. Add a CI check that fails if `/docs/diagrams/` has not been touched in the same PR as a structural code change. |
| **Hallucinated architecture decisions** — Claude proposes a pattern that sounds plausible but does not match the actual NFRs (e.g., recommends event sourcing for a CRUD app) | Always run the [`design-review`](./PROMPTS.md#design-review) prompt before accepting a proposal; require the trade-off interrogation step (1.3) on every option; ADRs must list the **rejected** alternatives with the specific reason — if Claude cannot articulate why an alternative was rejected, it has not actually evaluated it. |
| **Hallucinated requirements in ADRs** — Claude writes consequences that read well but were never validated | ADRs require human "Status: Accepted" sign-off; PRs that touch `/docs/adrs/` need a non-author reviewer. Treat every numeric claim ("latency drops 40%") as a citation-needed flag — strike it or back it. |
| **OpenAPI spec drift** — the spec is correct on day one but the implemented API diverges by week three | Make the spec the source of truth: contract-test the running API against `openapi.yaml` in CI (e.g., Schemathesis or Dredd). Spec changes go through the [Gate 2](./QUALITY-GATES.md#gate-2-data-model--api-contracts) review path, not casual edits. |
| **Claude Design output ignores accessibility** — generated UI looks good but fails keyboard navigation, contrast, or screen-reader semantics | The accessibility review at Step 4.6 is mandatory, not optional. Resolve every Critical and High before Gate 3. The frontend-design Skill helps with intent but does not replace the audit. |
| **Wireframe-to-prod creep** — stakeholders fall in love with a Claude Design prototype and the "wireframe" is shipped without engineering review | Mark every Claude Design project explicitly as "wireframe — not production". Production code path is **Handoff to Claude Code → branch → PR → Phase 3 review**, never canvas-to-prod. |
| **Eraser permission overreach** — a connector with `delete` enabled wipes diagrams during an aggressive refactor prompt | Disable `delete` scope in the Anthropic Connectors policy until the team is mature. Audit logs must be on in Eraser. Off-board users by revoking the OAuth grant. |
| **Spec/schema/diagram fan-out without traceability** — three diagrams, two ADRs, an ER diagram, an OpenAPI spec, and nobody can find the PRD section that drove any of it | Every artifact cites the PRD section (Linear Document anchor) it traces to. The handoff checklist enforces this; the Phase 3 team rejects unlinked artifacts. |

---

## System-Design-to-Phase-3 Loop (end-to-end)

```
[Phase 1 handoff] Linear Project URL = approved PRD Document + epic Milestones + accepted backlog
   │
[Step 1.1] Pull PRD + Linear context (read-only via Linear MCP)
   │
[Step 1.2-1.3] Claude generates 2-3 architecture options + trade-off interrogation
   │
[Step 1.4] eraser-architecture-diagram (via Eraser MCP) → cloud + system diagrams
[Step 1.5] Mermaid C4 Context/Container committed to /docs/diagrams/mermaid/
   │
[Step 1.6] design-review prompt → severity-ranked issue list, resolved
[Step 1.7] adr-generation → /docs/adrs/ (every decision cites a PRD section)
[Step 1.8] Team review
   │
   ▼  GATE 1: Architecture approved by Tech Lead
   │
[Step 2.1-2.2] Claude Code → schema in ORM format with indexes + seeds
[Step 2.3] eraser-er-diagram (via Eraser MCP) → ER diagram in DSL + PNG
[Step 2.4] Mermaid erDiagram committed
[Step 2.5-2.6] Backend review → migrations run clean
   │
[Step 3.1-3.2] Claude Code → OpenAPI 3.1 YAML in /docs/api/openapi.yaml
[Step 3.3] Swagger UI walkthrough
[Step 3.4] Prism mock server live; URL in README
[Step 3.5] Tech Lead + backend dev review → spec frozen
   │
   ▼  GATE 2: Data model + API contracts approved
   │
[Step 4.1] Claude Design onboards design system from repo + brand
[Step 4.2] frontend-design Skill enabled
[Step 4.3-4.4] Wireframes per flow + iteration via inline comments
[Step 4.5] Handoff to Claude Code (when production-grade required)
[Step 4.6] accessibility-review prompt → WCAG 2.1 AA report
[Step 4.7] Stakeholder review on Claude Design canvas
   │
   ▼  GATE 3: Wireframes approved by PM
   │
[Step 5.1-5.4] Tech-stack scored comparisons + ADRs + training plans
   │
   ▼  GATE 4: Phase Handoff complete (all artifacts present, Linear-linked, traceable)
   │
[Phase 3: Development]
```

Four explicit human gates ensure that **no AI-generated artifact reaches Phase 3 without sign-off**, every diagram round-trips via DSL, every endpoint has a mock, and every wireframe is accessibility-checked.

---

## Related Documents

- [Prompt Templates →](./PROMPTS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Process Flowcharts →](./FLOWCHART.md) (six per-step diagrams)
- [ADR Template →](../templates/adr-template.md)

## External References

- [Eraser MCP server docs](https://docs.eraser.io/docs/mcp) — server URL, OAuth, available tools
- [Eraser AI agent integrations](https://docs.eraser.io/docs/using-ai-agent-integrations) — MCP vs Skills, recommended workflows
- [Eraser DiagramGPT](https://www.eraser.io/diagramgpt) — diagram types, DSL, export formats
- [Anthropic — Introducing Claude Design](https://www.anthropic.com/news/claude-design-anthropic-labs) — launch, capabilities, model
- [Get started with Claude Design](https://support.claude.com/en/articles/14604416-get-started-with-claude-design) — access, refinement, handoff
- [Anthropic — Improving frontend design through Skills](https://claude.com/blog/improving-frontend-design-through-skills) — frontend-design Skill
- [Mermaid C4 syntax](https://mermaid.js.org/syntax/c4.html) — Context/Container/Component/Dynamic
- [Stoplight Prism (OSS)](https://stoplight.io/open-source/prism) — OpenAPI 3.1 mock server
- [Phase 2 Tools Evaluation](../../docs/tools-evaluation/2.SystemDesign_Phase_Tools.md)
