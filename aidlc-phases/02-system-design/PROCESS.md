# Phase 2: System Design — Process Definition

## Overview

This document defines the AI-assisted workflow for the System Design phase, using the consolidated AI-DLC tool stack.

**Phase Duration:** 1–2 weeks (varies with system complexity)
**Phase Owner:** Tech Lead / Solutions Architect
**Tools Used:** Claude Code (design reasoning + the **frontend-design Skill** for UI), **Figma** (design canvas / handoff via the official Figma MCP), **Eraser.io** (diagrams, via the Eraser MCP), Mermaid (in-repo diagram-as-code), free Swagger UI + Prism (OpenAPI viewing & mocking)

> **Tool Philosophy:** Claude Code does **all** of the design reasoning — architecture proposals, ADRs, schema generation, OpenAPI specs, tech-stack trade-offs, and design review. **Eraser.io is the single specialist visual tool** for architecture and data diagrams, and it is driven from inside Claude Code through the official Eraser MCP server, so the architect never context-switches into a separate diagram app to do the first draft. For UI/UX, the **frontend-design Skill runs inside Claude Code and generates the interface as real code in the repo** (React + Tailwind + shadcn), inheriting the project's design system directly from the codebase — there is no separate canvas surface to round-trip through. When a design canvas is genuinely needed (designer-led flows, pixel-perfect brand work, or a shared design file), **Figma is the paired tool, connected over the official Figma MCP** so Claude Code reads designs out of Figma and writes them back without leaving the terminal. v0 / Bolt remain narrower escalation paths (see Step 4.7). Mermaid stays in scope for diagrams that must live in git.

---

## Process Steps

### Step 0: One-Time Setup — Connect Claude to Eraser via MCP

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | Eraser.io workspace admin access (Business tier recommended for SOC 2 Type II + audit logs), Claude subscription that funds Claude Code (Pro / Max / Team / Enterprise), Claude Code installed |
| **Tool** | **Eraser MCP server** (`https://app.eraser.io/api/mcp`) — official, hosted by Eraser, OAuth 2.1, wired into Claude Code at **project scope**; API-key fallback for CI |
| **Output** | Claude Code can `generate`, `generateEdit`, `export`, `search`, `create`, `update`, and `delete` Eraser diagrams, files, folders, and presets from inside the repo |
| **Human** | Workspace admin confirms the MCP endpoint is reachable; each user authorises once per project via OAuth (`/mcp`) |

The Eraser MCP is the integration mechanism — do **not** wrap the Eraser REST API by hand. It is centrally hosted, OAuth-authenticated, and driven entirely from **Claude Code** ([docs.eraser.io/docs/mcp](https://docs.eraser.io/docs/mcp)). Register it once per repo (the registration is committed to `.mcp.json`), then each user authenticates once per project.

#### Set up the project-scoped Eraser MCP server in Claude Code

All Claude interaction in the AI-DLC runs through **Claude Code**, and the Eraser MCP server is wired **per project** so each repo carries its own committed MCP config — letting you keep multiple projects open concurrently, each authenticated independently (even under a **different Eraser account per project**).

1. **Workspace admin (Eraser):** confirm the workspace is on a plan that exposes the MCP endpoint and that no IP allow-list blocks `app.eraser.io`. Enable the audit log under **Settings → Security**.
2. **Add the server at project scope** (run from the repo root). This writes the registration to the repo's committed `.mcp.json`:
   ```bash
   claude mcp add --transport http --scope project eraser https://app.eraser.io/api/mcp
   ```
   Because `.mcp.json` is committed, the whole team shares one per-project MCP config. For a **different Eraser account per project**, give it a **distinct server name** (e.g., `eraser-acmeco`) so registrations and OAuth tokens never collide. For agentic CI pipelines that cannot do interactive OAuth, use the API-key path: `claude mcp add eraser -- npx -y @eraserlabs/eraser-mcp` with `ERASER_API_KEY` in the environment.
3. **Authenticate once per project:** open Claude Code (`claude`) in the repo → `/mcp` → select **eraser** (or the distinct name from step 2) → approve OAuth. The token is stored against this project's server registration, so authenticating one project never disturbs another.
4. **Verify:** `claude mcp list` should show `eraser: connected`. In a session: `Via Eraser MCP, generate a cloud architecture diagram for an AWS-based three-tier web app and export it as PNG.` Confirm the diagram appears in your Eraser workspace.
5. **Restrain write scope until the team is confident.** For Phase 2, keep Claude Code to **`generate` + `generateEdit` + `export` + `search`**, with **`delete` withheld** — enforced through Claude Code's tool-permission settings (see [Risks & Guardrails](#risks--guardrails)).
6. **Revoke when done:** `claude mcp remove eraser`, and revoke the OAuth grant in Eraser under **Settings → Security → Connected applications**.

#### Install the Phase 2 design artifacts

Phase 2 ships **seven templates** — three subagents, three skills and one slash command. Copy them into the consuming repo, fill the placeholders, and commit; they are project artefacts, not personal config, and they transfer with the repo at Phase 7. Install them after the Eraser MCP server is connected, because `render-design-diagrams` calls it on its second step.

```bash
# From the consuming repo root
mkdir -p .claude/agents .claude/skills .claude/commands
cp    <ai-dlc>/aidlc-phases/02-system-design/subagent-prompts/*.md   .claude/agents/
cp -r <ai-dlc>/aidlc-phases/02-system-design/skill-prompts/*/        .claude/skills/
cp    <ai-dlc>/aidlc-phases/02-system-design/command-prompts/adr.md  .claude/commands/
```

Skills are **folders**, not files — `.claude/skills/<skill-name>/SKILL.md`, where the folder name must equal the `name:` field. A mismatch means the skill silently never loads, so copy the directories with `cp -r` rather than globbing the Markdown. Subagents and commands are single files, and for a command **the filename is the invocation slug** (`adr.md` → `/adr`).

| Artifact | Type | Owns | Step | Gate |
|---|---|---|---|---|
| [`solution-architect`](./subagent-prompts/solution-architect.md) | Subagent | 2–3 architecture options, the 10x stress test on each, the recommendation, the proposal document | 1.2–1.3, 1.5 | Gate 1 |
| [`architecture-reviewer`](./subagent-prompts/architecture-reviewer.md) | Subagent | The independent production-readiness verdict on a design it did not author | 1.6 | Gate 1 |
| [`accessibility-auditor`](./subagent-prompts/accessibility-auditor.md) | Subagent | The WCAG 2.1 AA verdict on one rendered surface | 4.6 | Gate 3 |
| [`render-design-diagrams`](./skill-prompts/render-design-diagrams/SKILL.md) | Skill | Eraser DSL, PNG/SVG exports, the editor URL, and the in-repo Mermaid mirror | 1.4–1.5, 2.3–2.4 | Gates 1, 2, 4 |
| [`data-model-design`](./skill-prompts/data-model-design/SKILL.md) | Skill | The schema, its indexes, the migrations and the seed data | 2.1–2.2, 2.5–2.6 | Gate 2 |
| [`api-contract-freeze`](./skill-prompts/api-contract-freeze/SKILL.md) | Skill | The OpenAPI spec, the mechanical audit, the mock URL, the unsigned freeze note | 3.1–3.5 | Gate 2 |
| [`/adr`](./command-prompts/adr.md) | Command | One decision record, with the next sequential number allocated and reported | 1.7, 5.3 | Gates 1, 3, 4 |

Fill the placeholders before committing — full lists in each directory's README ([subagents](./subagent-prompts/README.md), [skills](./skill-prompts/README.md), [commands](./command-prompts/README.md)). Verify subagents with `/agents`; **`/agents` lists neither skills nor commands** — confirm a skill by its slug in the available-skills list, and a command by typing `/` and finding it in the menu.

> **Settle the diagram-skill question before the first diagram, not after.** If the repo already has an in-repo-Mermaid-only diagram skill, bare *"draw the architecture diagram"* routes there and produces the Mermaid mirror **only** — one gate line satisfied, three (DSL committed, editor URL recorded, PNG/SVG exported) silently not, and the run looks like the diagram got done. Either uninstall the incumbent, or keep it for sketches and invoke `render-design-diagrams` explicitly for anything gate-bearing.

#### Verification checklist

- [ ] `claude mcp list` shows `eraser: connected` in the repo (registration committed to `.mcp.json`)
- [ ] The three subagents are listed by `/agents`; the three skills appear under their slugs; `/adr` appears in the `/` menu
- [ ] Smoke-test diagram appears in your Eraser workspace
- [ ] Claude Code can `search` your workspace and `export` to PNG/SVG
- [ ] Audit logging is on in Eraser
- [ ] Claude Code tool-permission settings keep Eraser to `generate` / `generateEdit` / `export` / `search` (`delete` withheld)

> **Permission inheritance:** Claude Code inherits the connecting user's Eraser permissions. There is no privilege escalation. Off-board by revoking the OAuth grant.

> **UI/UX tooling (Step 4):** the frontend-design Skill needs no setup beyond enabling it in Claude Code (Step 4.2). If the project uses **Figma** as its design canvas, register the Figma MCP server at project scope too — `claude mcp add --transport http --scope project figma <figma-mcp-url>` — then authenticate per project via `/mcp`, exactly like Eraser above. Figma is optional: repos with no design file drive Step 4 entirely from the frontend-design Skill.

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

**1.1 — Pull inputs.** In Claude Code with the **Linear MCP server connected**, fetch the approved PRD Document and any related initiatives/issues from the Phase 1 Project. Do not paste — let Claude Code read via MCP so the trace stays clean. Record the PRD Document URL and PRD version (`v1.0`, etc.) — every ADR and diagram must cite both.

> **Steps 1.2, 1.3 and 1.5 are one procedure, and the [`solution-architect`](./subagent-prompts/solution-architect.md) subagent runs them as one** — options, stress test, recommendation, stop at the Tech Lead gate. The sub-steps below remain the definition of what it does and the paste-prompt path for teams that have not installed it. **One difference on purpose:** 1.3 reads as an optional follow-up; inside the agent it is a mandatory step run on *every* seriously-considered option and *before* the recommendation. A stress test run after the recommendation argues against a decision already anchored, and Gate 1 asking for it on "the chosen option" is circular — the stress test is part of how you choose.

**1.2 — Generate architecture options with Claude Code.** Use the [`architecture-proposal`](./PROMPTS.md#architecture-proposal) prompt. Feed in the PRD reference, NFRs, team profile, budget, and timeline. Claude Code produces 2–3 candidate architectures with components, communication patterns, data architecture, and trade-off tables. The output stays local — no diagrams or ADRs yet.

**1.3 — Trade-off interrogation.** For each option, run the [`trade-off-interrogation`](./PROMPTS.md#trade-off-interrogation) prompt (or follow up in Claude Code with `Stress-test option 2 at 10x current load — where does it break first, and what is the mitigation?`). Force Claude Code to surface failure modes, single points of failure, and cost cliffs. Capture the answers; they become inputs to the design review prompt at 1.6.

> **Steps 1.4 and 1.5 are one procedure, and the [`render-design-diagrams`](./skill-prompts/render-design-diagrams/SKILL.md) skill runs them as one** — generate through Eraser, commit the DSL, export PNG/SVG, record the editor URL, then commit the Mermaid mirror. Keeping them as two steps is what lets a team ship the Eraser half and forget the mirror; the skill will not.

**1.4 — Render diagrams in Eraser.io (via MCP).** With the Eraser MCP server connected, run the [`eraser-architecture-diagram`](./PROMPTS.md#eraser-architecture-diagram-via-mcp) prompt. Claude Code calls Eraser MCP `generate` to produce a cloud architecture diagram and a system overview diagram in Eraser's diagram-as-code DSL. Eraser supports flowcharts, sequence diagrams, ER diagrams, cloud architecture, and BPMN/swimlane natively ([eraser.io/diagramgpt](https://www.eraser.io/diagramgpt)). For each generated diagram, Claude Code returns the editor URL and the DSL — open the URL, refine in the visual editor or via inline AI requests, then export PNG/SVG. The DSL is checked into `/docs/diagrams/eraser/` alongside the export so the diagram round-trips.

**1.5 — Render in-repo diagrams in Mermaid.** For diagrams that must live in version control and render in GitHub/GitLab markdown (sequence diagrams in ADRs, C4 Context/Container diagrams), use Claude to produce Mermaid directly. Mermaid in 2026 supports the full C4 model — Context, Container, Component, Dynamic ([mermaid.js.org/syntax/c4](https://mermaid.js.org/syntax/c4.html)). Commit these to `/docs/diagrams/mermaid/` and reference them from the ADR.

**1.6 — Design review with Claude.** Invoke [`architecture-reviewer`](./subagent-prompts/architecture-reviewer.md) **in a session that did not produce the design** — it refuses a same-session self-review, which is exactly what Gate 1's "not the author" line measures. It returns a severity-ranked issue list across scalability, security, reliability, operability and cost, a list of anything the inputs did not let it evaluate, and a verdict derived from the counts rather than chosen. Resolve every Critical and High before proceeding. The paste path via the [`design-review`](./PROMPTS.md#design-review) prompt stays valid.

**1.7 — Create ADRs.** For each significant decision (chosen pattern, datastore selection, sync vs async boundaries, deployment topology) run [`/adr <the decision>`](./command-prompts/adr.md). It allocates the next sequential number, reports what it allocated from, and writes the record to `/docs/adrs/` with Status `Proposed`. **It refuses when no alternative was genuinely weighed and rejected** — Gate 1 reads that rejected-alternatives list as the evidence that the alternatives were real, so a straw man is worse than no record. It also refuses to state a figure it cannot source. The paste path via the [`architecture-decision-record-generation`](./PROMPTS.md#architecture-decision-record-generation) prompt stays valid. Each ADR cites the PRD section that drove the decision and embeds the relevant Mermaid diagram inline.

> **The architecture agent does not write the record for the option it argued for.** `solution-architect` supplies the options and the specific rejection reasons and then refuses, redirecting here. A decision record written by the advocate is a paraphrase of the advocacy.

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

> **Steps 2.1, 2.2, 2.5 and 2.6 are one procedure, run by the [`data-model-design`](./skill-prompts/data-model-design/SKILL.md) skill** — extract, generate, stop for the backend-developer review, then migrate and seed. **2.5 is the skill's own hard stop, not a step after it**, and the skill will not continue because the reviewer is unavailable. **2.3 and 2.4 are not this skill's** — both diagram surfaces belong to `render-design-diagrams`, which owns the gate lines that count them.
>
> The skill claims **greenfield data-model design only**. A schema change or migration inside a story already in flight belongs to the repo's server-side implementation specialist, who ships it in the story's own commit — the triggers are deliberately narrow so an ordinary story turn cannot pull this chain in.

**2.1 — Extract entities.** Run the [`entity-extraction`](./PROMPTS.md#entity-extraction) prompt in Claude Code with the Linear MCP server connected. Claude Code reads the PRD Document and produces a structured entity list with attributes, relationships, cardinalities, and a cross-context map. Entities that span bounded contexts may need duplication or an event contract, not a foreign key — the prompt flags these explicitly.

**2.2 — Generate schema with Claude Code.** From inside the repo, run the [`schema-generation`](./PROMPTS.md#schema-generation) prompt. Specify the ORM (Prisma / TypeORM / Drizzle / raw SQL) and naming convention. Claude Code reads existing repo conventions and produces a schema that matches them — this is the key reason to drive schema generation from Claude Code rather than chat. Output includes constraints, indexes covering the top query patterns, soft-delete columns where appropriate, and timestamps on every entity.

**2.3 — Render ER diagram in Eraser.io.** Covered with 2.4 by [`render-design-diagrams`](./skill-prompts/render-design-diagrams/SKILL.md), which takes the committed schema as its source. Run the [`eraser-er-diagram`](./PROMPTS.md#eraser-er-diagram-via-mcp) prompt. Claude calls Eraser MCP `generate` with the schema as input — Eraser's ER syntax is purpose-built for this and accepts paste of either DDL or natural language ([eraser.io/use-case/api-diagrams](https://www.eraser.io/use-case/api-diagrams)). Refine in the Eraser editor, export PNG/SVG, commit DSL to `/docs/diagrams/eraser/`.

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

> **Steps 3.1 to 3.5 are one procedure, run by the [`api-contract-freeze`](./skill-prompts/api-contract-freeze/SKILL.md) skill** — map flows to resources, generate the spec, audit it mechanically, stand up the mock, write the freeze note, stop. **3.3's walkthrough and 3.5's signature are the human half of its terminal stop:** the audit proves the spec is *complete*, only a reader proves it is *right*, so the skill leaves the freeze note **unsigned** and hands over the audit result, the mock URL and the note.
>
> **One step the sub-steps below do not have.** Before generating anything, the skill demands two declarations it refuses to infer: **which endpoints are public**, and **which POSTs are non-idempotent**. Gate 2's "auth on every *protected* endpoint" and "Idempotency-Key on POSTs that *require* it" are product judgements, and an agent that infers them then audits its own inference passes every time. With both declared, the audit is genuinely mechanical.

**3.1 — Map flows to endpoints.** Use Claude Code with the Linear MCP server connected to read each PRD user-flow section and propose the resource model and endpoints. Resolve naming and verb questions here before writing any spec — REST mistakes calcify fast.

**3.2 — Generate OpenAPI 3.1 spec.** Run the [`api-contract`](./PROMPTS.md#api-contract) prompt in Claude Code so the spec aligns with existing repo conventions. Claude produces a complete OpenAPI 3.1 YAML with: info + servers + security schemes, all CRUD endpoints, request bodies with JSON Schema, success and error responses (200/201/400/401/403/404/409/422/500), pagination on every list endpoint, reusable components, and consistent tagging. Commit to `/docs/api/openapi.yaml`.

**3.3 — View and refine in Swagger UI.** Spin up Swagger UI locally (Docker one-liner) or use Swagger Editor in the browser. Walk every endpoint, every example, every error response. The spec is the contract — anything missing here becomes a frontend/backend argument later.

**3.4 — Stand up mock servers with Prism.** Prism remains the recommended OSS mock server in 2026 with full OpenAPI 3.1 support ([stoplight.io/open-source/prism](https://stoplight.io/open-source/prism)): `prism mock docs/api/openapi.yaml`. Frontend can start integration the same day. Add the mock URL to the project README. Mockoon is acceptable as a desktop alternative when Prism is awkward (Windows-heavy teams).

**3.5 — Review and freeze.** Tech Lead reviews every endpoint against the [Gate 2 checklist](./QUALITY-GATES.md#gate-2-data-model--api-contracts): REST conventions, error envelope consistency, auth on every protected endpoint, pagination defined, idempotency keys on POSTs that need them. On approval, the spec is **frozen** — any subsequent change requires a documented change request and impact analysis on already-mocked frontend work.

**▼ GATE 2 — Data model + API contracts approved.** Tech Lead + one backend developer sign off. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-data-model--api-contracts).

**Escalation:** API contract changes after frontend development begins require PM approval and a documented impact analysis appended to the ADR for that contract. If a flow cannot be expressed cleanly in REST (long-lived workflows, real-time subscriptions), surface a separate ADR for the alternative (gRPC, GraphQL, WebSocket, server-sent events) — do not bend REST until it breaks.

---

### Step 4: UI/UX Wireframing — in Claude Code

> Visual: [Step 4 flowchart](./FLOWCHART.md#step-4-uiux-wireframing)

UI/UX in the AI-DLC is generated **as real code in the repo** by Claude Code with the **frontend-design Skill** — not on a separate canvas. Wireframes are runnable React/Tailwind/shadcn screens you preview in the project's own dev server (or Storybook), so there is no wireframe-to-code translation step and no surface to round-trip through. **Figma is the paired canvas tool** for designer-led or pixel-perfect work, connected over the official **Figma MCP** so Claude Code reads a Figma file into code (design-to-code) or pushes generated screens back into Figma (code-to-design) from inside the terminal.

| Attribute | Detail |
|-----------|--------|
| **Input** | PRD user flows, personas, functional requirements, brand assets (if any), the repo itself (existing components / Tailwind config / tokens), an existing Figma file (optional) |
| **Tool** | **Claude Code + frontend-design Skill** (primary — generates screens as repo code) + **Figma via the Figma MCP** (optional canvas / designer handoff) + Claude Code (accessibility review) |
| **Output** | Runnable wireframe screens in the repo (React + Tailwind + shadcn), all states, desktop + mobile; component specifications; accessibility report; PR per flow |
| **Human** | Validate UX in the running preview, review accessibility, approve designs, drive iterations by prompt |

**Workflow:**

**4.1 — Establish the design system in the repo (once per project).** Point Claude Code at the repo so the frontend-design Skill infers the existing design system — colours, typography, spacing, component library — from the code (Tailwind config, tokens, existing components). Every subsequent screen inherits it automatically. For greenfield projects with no design system, run the [`design-system-bootstrap`](./PROMPTS.md#design-system-bootstrap) prompt to generate the token set + a reference screen **as committed repo code** (e.g., a `/design-system` route or a Storybook story), and approve it before any wireframing begins. If the project already has a **Figma** design system, connect the Figma MCP and pull tokens/variables into the repo with `get_variable_defs` / `get_design_context` so the code and the Figma file agree.

**4.2 — Enable the frontend-design Skill.** Enable the official **frontend-design** Skill ([claude.com/blog/improving-frontend-design-through-skills](https://claude.com/blog/improving-frontend-design-through-skills)) in Claude Code. It biases Claude Code away from generic Inter-and-purple-gradient defaults and towards distinctive typography, intentional motion, and considered colour palettes — exactly what wireframes need to communicate intent. No setup beyond enabling it.

**4.3 — Generate wireframes per user flow.** For each PRD user flow, run the [`ui-wireframe`](./PROMPTS.md#ui-wireframe) prompt in Claude Code. Output is **live React/Tailwind/shadcn code committed to the repo** — clickable, with real layout, real components, real behaviours, previewed in the project's dev server. Produce all standard states (loading, empty, error, success) plus desktop (1440) and mobile (375) variants. For designer-led or pixel-perfect flows, drive the same result through **Figma via the Figma MCP** — either implement an existing Figma frame with `get_design_context`, or generate the screen and push it into Figma with `generate_figma_design` / `use_figma` for a designer to refine.

**4.4 — Iterate by prompt (and in Figma when used).** Refine through Claude Code: targeted edits ("tighten this header, increase the tap target on the primary CTA") and broad restyles, previewing each change in the running app. The frontend-design Skill plus the repo's design-system tokens keep refinements consistent across flows. When a designer is iterating on the Figma canvas, re-sync the change into the repo with the Figma MCP so code stays the source of truth.

**4.5 — Promote to production-grade components.** Because screens are already real code, "handoff" is just promotion: lift the mature component into the shared component library (`src/components/ui/…`), tighten its typed props and variants, add the render + a11y tests, and open a PR. There is no canvas-to-code export — the [`production-component`](./PROMPTS.md#production-component) prompt drives this directly in Claude Code.

**4.6 — Accessibility review.** Invoke [`accessibility-auditor`](./subagent-prompts/accessibility-auditor.md) per component. It checks WCAG 2.1 AA — keyboard navigation, focus order, contrast, screen-reader labelling, semantic HTML, motion-reduction — and is explicit about what a source read cannot establish. **It cannot reach PASS from source alone:** PASS requires the automated scan run against the *rendered* page with no serious or critical violations, plus a described keyboard traversal. Without both it returns **`PASS WITHHELD`**, which is a gate blocker, not a soft pass. Resolve every Critical and High; document accepted Mediums in the ADR for that flow. The paste path via the [`accessibility-review`](./PROMPTS.md#accessibility-review-claude) prompt stays valid.

**4.7 — Stakeholder review.** Share a running preview — a deploy preview, a Storybook link, or a screen recording of the flow — with PM and stakeholders (or the Figma file if the flow was designed there). Capture feedback, iterate in Claude Code, approve.

**▼ GATE 3 — Wireframes approved.** PM signs off on the previewed screens (deploy preview / Storybook / Figma). See [QUALITY-GATES.md → Gate 3](./QUALITY-GATES.md#gate-3-wireframes--tech-stack).

**Escalation paths (beyond the frontend-design Skill + Figma):**
- **Pixel-perfect brand work or designer-led flow** → **Figma** is the primary canvas here, wired over the Figma MCP so code and design stay in sync (not a manual spec handoff).
- **Full-stack interactive demo with real backend** → Bolt.new or Lovable. Treat as a throwaway exploration; do not ship the output.
- **A specific dropped-in component that is faster to start from a generator** → v0 by Vercel as a per-component fallback; bring the code back into the repo and reconcile it to the design-system tokens.

If the team lands in fallback territory more than once a sprint, that is a signal — re-evaluate whether the frontend-design Skill + repo design system is being used to its full extent rather than normalising the workaround.

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

**5.3 — Create ADRs.** Run [`/adr <the decision>`](./command-prompts/adr.md) for each decision. Every ADR includes the rejected alternatives with reasons — future engineers thank you for this, and the command refuses to write the record without them.

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
| UI wireframes / prototypes | Runnable screens in the repo (React + Tailwind + shadcn) + deploy-preview / Storybook link; Figma file link if used | Repo (`/src`) + Project README |
| Accessibility review | Markdown | `/docs/accessibility/wcag-aa-report.md` |
| Tech stack ADRs + training plans | Markdown | `/docs/adrs/` |

**Handoff Checklist:**
- [ ] Architecture approved by Tech Lead (Gate 1)
- [ ] All significant decisions documented as ADRs, each citing the PRD section
- [ ] Eraser diagrams: system overview, cloud architecture, ≥ 2 key sequences — DSL in repo, exports linked
- [ ] Mermaid diagrams: C4 Context + Container, ERD — rendering in repo
- [ ] Schema reviewed by ≥ 1 backend developer; migrations run cleanly locally
- [ ] OpenAPI 3.1 spec covers every PRD user flow; mock server URL in README
- [ ] frontend-design Skill wireframes (repo code) cover every PRD user flow with all states; accessibility review passes WCAG 2.1 AA
- [ ] Tech stack ADRs complete; every unfamiliar choice has a training plan
- [ ] Linear: every Phase 2 ADR / spec / diagram URL is linked from the corresponding Linear Issue with the `phase:design` label

---

## Risks & Guardrails

| Risk | Mitigation |
|------|------------|
| **Diagram drift** — Eraser/Mermaid diagrams diverge from the implemented system over time | [`render-design-diagrams`](./skill-prompts/render-design-diagrams/SKILL.md) is the mitigation's owner: it commits the DSL, the exports, the editor URL **and** the Mermaid mirror in one pass, so the two surfaces cannot drift apart from each other, and it titles every diagram with the requirements version so a stale one is identifiable without opening it. It also refuses to certify that a diagram matches production without reading the code. Add a CI check that fails if `/docs/diagrams/` has not been touched in the same PR as a structural code change. |
| **Hallucinated architecture decisions** — Claude proposes a pattern that sounds plausible but does not match the actual NFRs (e.g., recommends event sourcing for a CRUD app) | [`architecture-reviewer`](./subagent-prompts/architecture-reviewer.md) before accepting a proposal — its severity is assigned from **consequence and the load at which it occurs**, never from how serious a finding sounds, and its verdict is derived from the counts rather than chosen, which is what stops a genuine blocker being written politely enough to read as Medium. `solution-architect` runs the trade-off interrogation on every seriously-considered option before recommending. ADRs must list the **rejected** alternatives with the specific reason, and [`/adr`](./command-prompts/adr.md) refuses to write one without them — if Claude cannot articulate why an alternative was rejected, it has not actually evaluated it. |
| **Hallucinated requirements in ADRs** — Claude writes consequences that read well but were never validated | ADRs require human "Status: Accepted" sign-off; PRs that touch `/docs/adrs/` need a non-author reviewer. Treat every numeric claim ("latency drops 40%") as a citation-needed flag — strike it or back it. |
| **OpenAPI spec drift** — the spec is correct on day one but the implemented API diverges by week three | Make the spec the source of truth: contract-test the running API against `openapi.yaml` in CI (e.g., Schemathesis or Dredd). Spec changes go through the [Gate 2](./QUALITY-GATES.md#gate-2-data-model--api-contracts) review path, not casual edits. |
| **Generated UI ignores accessibility** — frontend-design output looks good but fails keyboard navigation, contrast, or screen-reader semantics | The accessibility review at Step 4.6 is mandatory, not optional. Resolve every Critical and High before Gate 3. The frontend-design Skill helps with intent but does not replace the audit. [`accessibility-auditor`](./subagent-prompts/accessibility-auditor.md) cannot return PASS from source alone — contrast ratio, reflow at 320 CSS px, resize to 200%, computed tap-target size, focus order against *visual* reading order, and what an assistive technology actually announces are all unmeasurable from source, and it reports them unmeasured rather than asserting them. Without the rendered-page scan and a keyboard pass it returns `PASS WITHHELD`. |
| **Wireframe-to-prod creep** — stakeholders fall in love with a wireframe screen and it ships without engineering review | Mark every wireframe screen explicitly as "wireframe — not production" (e.g., a route flag or Storybook tag). The production path is **promote component → PR → Phase 3 review**, never wireframe-branch-to-prod. |
| **Eraser permission overreach** — an MCP server with `delete` enabled wipes diagrams during an aggressive refactor prompt | Withhold the `delete` action in Claude Code's tool-permission settings until the team is mature. Audit logs must be on in Eraser. Off-board users by revoking the OAuth grant. |
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
[Step 1.7] /adr → /docs/adrs/ (every decision cites a PRD section)
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
[Step 4.1] frontend-design Skill infers design system from repo (+ Figma tokens if used)
[Step 4.2] frontend-design Skill enabled in Claude Code
[Step 4.3-4.4] Wireframe screens per flow as repo code + iteration by prompt
[Step 4.5] Promote mature components to the shared library → PR
[Step 4.6] accessibility-review prompt → WCAG 2.1 AA report
[Step 4.7] Stakeholder review on deploy preview / Storybook / Figma
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
- [ADR Command Template →](./command-prompts/adr.md) (`/adr` — carries the record shape inline)
- [Subagent Templates →](./subagent-prompts/README.md) · [Skill Templates →](./skill-prompts/README.md) · [Command Templates →](./command-prompts/README.md)

## External References

- [Eraser MCP server docs](https://docs.eraser.io/docs/mcp) — server URL, OAuth, available tools
- [Eraser AI agent integrations](https://docs.eraser.io/docs/using-ai-agent-integrations) — MCP vs Skills, recommended workflows
- [Eraser DiagramGPT](https://www.eraser.io/diagramgpt) — diagram types, DSL, export formats
- [Anthropic — Improving frontend design through Skills](https://claude.com/blog/improving-frontend-design-through-skills) — the frontend-design Skill in Claude Code
- [Figma MCP server](https://www.figma.com/blog/introducing-figmas-dev-mode-mcp-server/) — design-to-code and code-to-design from Claude Code
- [Mermaid C4 syntax](https://mermaid.js.org/syntax/c4.html) — Context/Container/Component/Dynamic
- [Stoplight Prism (OSS)](https://stoplight.io/open-source/prism) — OpenAPI 3.1 mock server
- [Phase 2 Tools Evaluation](../../docs/tools-evaluation/2.SystemDesign_Phase_Tools.md)
