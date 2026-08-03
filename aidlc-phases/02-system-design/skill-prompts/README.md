# Phase 2 System Design Skill Prompts

This directory holds the **`SKILL.md` templates** for the Phase 2 design skills referenced in [`../PROCESS.md`](../PROCESS.md). They are templates, not active skills — Claude Code only auto-discovers folders under `.claude/skills/`.

Each skill encodes **one ordered procedure ending at a human gate**. Unlike the [Phase 2 subagents](../subagent-prompts/), which a developer invokes by name, **a skill auto-triggers from its description alone.** That makes the `description:` field the entire routing surface — read [Routing](#routing) before editing one, because one of these three ships into a **known trigger collision** and stays clear of it only by how its description is worded.

None of the three writes to a tracker. They write **repository artifacts** — diagram DSL and exports, the schema and its migrations, the OpenAPI spec — and each stops at a named human reviewer rather than at the next skill.

## Files

| Folder | Procedure | Gate | Wraps |
|--------|-----------|------|-------|
| [`render-design-diagrams/`](./render-design-diagrams/SKILL.md) | Read the decision or the schema, generate through the Eraser MCP server, title for version-identifiability, commit DSL + PNG/SVG + editor URL under `docs/diagrams/eraser/`, commit the Mermaid mirror under `docs/diagrams/mermaid/`, report assumptions, stop for visual refinement | Gates 1, 2 and 4 (every diagram artifact and location line) | [`eraser-architecture-diagram`](../PROMPTS.md#eraser-architecture-diagram-via-mcp), [`eraser-er-diagram`](../PROMPTS.md#eraser-er-diagram-via-mcp) |
| [`data-model-design/`](./data-model-design/SKILL.md) | Extract entities, relationships and cross-context crossings from approved requirements, demand the real query patterns, generate the schema, **hard stop for backend-developer review**, then migrations + seed data, run up/down/up locally | Gate 2 (the five schema and migration checkboxes — the densest Phase 2 coverage) | [`entity-extraction`](../PROMPTS.md#entity-extraction), [`schema-generation`](../PROMPTS.md#schema-generation) **(minus its ER-diagram deliverable — see note)**, [`migration-generation`](../PROMPTS.md#migration-generation) |
| [`api-contract-freeze/`](./api-contract-freeze/SKILL.md) | Map user flows to a resource model, collect the two judgements the audit cannot make, generate the OpenAPI 3.1 spec, audit it mechanically line by line, stand up and verify the mock server, write the unsigned freeze note, stop for the human walkthrough and signature | Gate 2 (the eleven API lines); metric: **0 contract change-requests after frontend start** | [`api-contract`](../PROMPTS.md#api-contract) — plus four prompt-less PROCESS steps (3.1 resource modelling, 3.4 mock server, 3.5 audit, 3.5 freeze) |

The **Wraps** column is this directory's purpose. Because shipped templates must stand alone in a consuming repo, each skill **inlines** its procedure with `{{PLACEHOLDERS}}` rather than linking back to `PROMPTS.md` — so the table above is the only record of where a procedure came from. Keep it current, or the round-trip is lost.

> **Note — `data-model-design` drops one deliverable from `schema-generation`.** That prompt's item 7 asks for a Mermaid `erDiagram` alongside the schema. In the skill set that deliverable belongs to `render-design-diagrams`, which owns both diagram surfaces and the gate lines that count them. Left in both, the ER diagram gets written twice, in two places, by two procedures, and the second writer wins silently. The prompt stays whole for anyone pasting it; the skill hands the finished schema over instead.

> **Note — `api-contract-freeze` is deliberately not merged with `data-model-design`.** The combined chain is eight steps with two human stops, past the two-screen limit and past the single-procedure rule. The seam is real: different reviewers at the stop, a strict ordering dependency (the spec is generated over the reviewed schema), and a natural terminal verification of its own (the mock server answering).

## How to instantiate per repo

1. Copy the chosen **folder** into `.claude/skills/<skill-name>/` at the consuming repo root, preserving the folder name. **The folder name must equal the `name:` field** — Claude Code uses the folder name as the invocation slug, and a mismatch means the skill silently never loads. Use `cp -r`; globbing the Markdown files flattens the structure and produces zero working skills.
2. Replace the placeholders with the repo's values:
   - `{{ERASER_MCP_SERVER}}` — the name the Eraser MCP server is registered under in this repo's committed `.mcp.json`, usually `eraser` but deliberately distinct (e.g. `eraser-acmeco`) where a project uses its own Eraser account (render-design-diagrams)
   - `{{ORM}}` — e.g. `Prisma`, `TypeORM`, `Drizzle`, `SQLAlchemy`, or `raw SQL` (data-model-design)
   - `{{DATASTORE}}` — **with its version**, e.g. `PostgreSQL 16`; the version changes the available DDL syntax (data-model-design)
   - `{{SCHEMA_PATH}}` — where the schema lives, e.g. `prisma/schema.prisma`, `src/db/schema.ts` (data-model-design)
   - `{{MIGRATIONS_PATH}}` — e.g. `services/api/migrations` (data-model-design)
   - `{{API_BASE_PATH}}` — e.g. `/api/v1` (api-contract-freeze)
   - `{{AUTH_STACK}}` — how requests authenticate, e.g. `JWT bearer tokens`, `Clerk`, `Auth0` (api-contract-freeze)
   - `{{MOCK_SERVER}}` — e.g. `Prism`, or `Mockoon` on Windows-heavy teams (api-contract-freeze)
3. **The Eraser MCP server must be connected at project scope before `render-design-diagrams` runs**, with `delete` withheld in Claude Code's tool-permission settings — see [`../PROCESS.md` → Step 0](../PROCESS.md#step-0-one-time-setup--connect-claude-to-eraser-via-mcp). **When it is unreachable the skill says so and stops.** It does not fall back to producing the Mermaid mirror and presenting that as the diagram set: the DSL, the exports and the editor URL are three separate gate lines, and a mirror satisfies none of them. A silent fallback here reads as a passing gate.
4. Commit `.claude/skills/<skill-name>/SKILL.md` to the repo — skills at project scope are shared team infrastructure; treat edits as code changes requiring review.
5. **Verify. `/agents` does not list skills** — skills are a different surface and are verified differently. Run all four checks:
   - **Discovery.** Open a Claude Code session; the skill appears in the available-skills list under its slug. If not, the folder name and the `name:` field are out of alignment.
   - **Explicit invocation.** Type `/<skill-name>` with a representative input. The procedure should run top-to-bottom and stop at its human gate — including `data-model-design`'s mid-procedure review stop, which must not be walked past.
   - **Auto-trigger, in messy language.** This is the check that matters, because the description is the whole routing surface:
     - `render-design-diagrams` — *"can you get the architecture into eraser and commit the dsl and a png somewhere sensible"*
     - `data-model-design` — *"the PRD's signed off, we need a schema before anyone starts coding"*
     - `api-contract-freeze` — *"frontend want to start monday, can we lock the api down and give them something to hit"*
   - **Refusal.** Drive each skill into its sharpest refusal and confirm it redirects rather than complies:
     - `render-design-diagrams` — *"eraser's down, just do the mermaid ones and we'll call the diagrams done"*
     - `data-model-design` — *"denormalise the bookings table, it'll be faster"* and *"just run the migration on staging so QA can look at it"*
     - `api-contract-freeze` — *"assume the obvious endpoints are public and mark it frozen"*
6. **Negative-routing check, once, after installing more than one.** Ask for ordinary work these skills must not claim — *"implement the checkout endpoint"*, *"review my diff before I PR"*, *"add the migration for ENG-412"*, *"draw me a quick diagram of this function's control flow"* — and confirm none of the three loads. The third is the one to watch: a migration inside a story in flight belongs to the repo's server-side implementation specialist, and `data-model-design` losing that route is exactly why its triggers are as narrow as they are. If a skill does load, its description has stolen a verb; narrow it and re-test.

## Routing

Skills auto-trigger; subagents do not. A skill whose trigger phrases reuse a verb a specialist subagent already claims **wins that route by default and hijacks ordinary work.** Three consequences when editing anything in this directory:

- **Never add a trigger phrase containing a bare development verb** — implement, build the feature, write the tests, review the diff, refactor, open the PR. Those belong to the development specialists.
- **Never restore the wider data-model triggers.** "Write the migration for the new tables" and "generate the schema" were **cut before authoring**: they collide with the shipped server-side specialist's own trigger *"add the migration for ENG-XXX"*, and because skills auto-trigger, this skill would win and drag a greenfield requirements-to-schema chain into an ordinary story turn. The narrowing is the routing fix, not a stylistic preference.
- **The three must exclude each other**, and the boundary is *which artifact each writes*, not which topic each covers:

| Skill | Owns | Explicitly does not own |
|---|---|---|
| `render-design-diagrams` | Eraser DSL, PNG/SVG exports, the recorded editor URL, the committed Mermaid mirror | choosing the architecture, designing the schema it renders, wireframes, Eraser workspace administration |
| `data-model-design` | the schema, its indexes, the up/down migrations, the seed data | the ER diagram, the API contract, any schema work inside a story in flight, the denormalisation decision |
| `api-contract-freeze` | the OpenAPI spec, the mechanical audit, the mock server URL in the README, the unsigned freeze note | the schema underneath it, endpoint implementation, signing the freeze |

**These three are not scoped to a stage of the project.** Design work happens whenever it happens — a new subsystem in month nine uses the same three skills. The cross-phase version of this table lives in [`docs/ROUTING.md`](../../../docs/ROUTING.md); placeholder names that recur under other spellings elsewhere are reconciled in [`docs/PLACEHOLDERS.md`](../../../docs/PLACEHOLDERS.md).

### The `architecture-diagram` collision, and how it is resolved

A repo that already followed this framework is likely to carry a project-scope **`architecture-diagram`** skill whose triggers are *"draw the architecture diagram"*, *"diagram this flow"*, *"generate a C4/sequence/ER diagram"*, *"update the diagram for X"* — a **verbatim** collision with every trigger originally proposed for `render-design-diagrams`.

It is resolved the way `cicd-pipeline-bringup` resolved its own: **on the write-type axis first, then by not leading with the colliding phrase.**

| | `render-design-diagrams` | a project-scope `architecture-diagram` |
|---|---|---|
| Produces | Eraser DSL **and** exports **and** the editor URL **and** the in-repo Mermaid mirror | in-repo Mermaid only |
| Round-trips | yes — the DSL re-opens in the editor | no — the next change redraws from scratch |
| Gate lines satisfied | all of them | the Mermaid line only |

So `render-design-diagrams` never claims a bare diagram verb. Its triggers name the **Eraser round-trip or the two-surface obligation**: *"render the design diagrams in Eraser"*, *"commit the DSL"*, *"the editor URL and the PNG export"*, *"both the Eraser and the in-repo version"*, *"the full diagram set before the design gate"*. Bare *"draw the architecture diagram"* is left to the incumbent, and its description says so in place.

**A rename would not have fixed this and was rejected.** The two names do not collide; the *triggers* do, and a rename leaves every colliding phrase exactly where it was. Narrowing is honest here only because the artifacts genuinely differ — had both skills produced the same files, no wording would have made two owners into one.

**What re-opens it.** Adding *"draw the architecture diagram"*, *"diagram this flow"*, *"generate a C4 diagram"* or *"update the diagram for X"* to this skill's triggers; or dropping the Eraser/DSL/export qualifier from a trigger that has one.

**The residual risk, stated plainly.** In a repo where both are installed, *"draw the architecture diagram"* routes to the incumbent and produces Mermaid only — which satisfies one gate line and leaves the DSL, export and editor-URL lines unsatisfied while looking like the diagram got done. Pick one owner at install time: either uninstall the incumbent, or keep it for sketches and invoke `/render-design-diagrams` explicitly for anything the design gate counts.

> **A second collision, in the same consuming repos.** A project-scope `system-design-architect` **agent** may claim *"produce the data model / OpenAPI contract"* and *"freeze the API contract"*. Skills beat agents on auto-trigger, so `data-model-design` and `api-contract-freeze` take those utterances — which is the outcome the gates want, but it means the agent's own boundaries stop running for that work. Decide deliberately at install time rather than discovering it mid-project.
