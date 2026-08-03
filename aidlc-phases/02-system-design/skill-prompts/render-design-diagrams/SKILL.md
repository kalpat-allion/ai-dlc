---
name: render-design-diagrams
description: Use when a design diagram has to exist on both of the surfaces a design gate checks — generated through the Eraser MCP server with its DSL committed under docs/diagrams/eraser/, a PNG and SVG export beside it and the editor URL recorded, plus the equivalent Mermaid mirror committed under docs/diagrams/mermaid/ so it renders in markdown. Covers cloud architecture, system overview, sequence and entity-relationship diagrams. Triggers on "render the design diagrams in Eraser", "generate the ER diagram from the schema and commit the DSL", "we need the full diagram set before the design gate", "get me the editor URL and the PNG export for this diagram", "the committed diagrams have drifted from the design, refresh them", "produce both the Eraser and the in-repo version of this diagram". In a repo that also has an in-repo-Mermaid-only diagram skill, that skill owns bare "draw the diagram" phrasing and this one owns the Eraser round-trip — this is the skill the DSL, export and editor-URL gate lines depend on. Do NOT use for: deleting or reorganising anything in an Eraser workspace, choosing or designing the architecture itself, designing the schema an ER diagram renders, UI mockups or wireframes, or certifying that a diagram matches production without reading the code.
---

# Render the design diagrams

You are producing a diagram that must exist on **two surfaces at once**: the Eraser workspace, where a human refines it visually, and the repository, where it renders in markdown and outlives the workspace. The source of truth is the **recorded architecture decisions** and the **committed schema** — read them, and never draw a component, field or arrow you cannot point at.

Generation is one step. The obligations around it are six, and every one of them is a separate line on the design gate.

## Procedure

1. Confirm which diagram is wanted — cloud architecture, system overview, sequence, or entity-relationship — and read its source: the architecture decision records and the architecture proposal for the first three, the committed schema for the last. **Refuse if that source does not exist yet.** A diagram renders a decision; it does not substitute for one.
2. Check the Eraser MCP server `{{ERASER_MCP_SERVER}}` is reachable. **If it is not, say so and stop.** Do not produce the Mermaid mirror alone and present it as the diagram set — the DSL, the exports and the editor URL are three further gate lines and a mirror satisfies none of them.
3. Generate through `{{ERASER_MCP_SERVER}}` in Eraser's own syntax for the chosen type. Every diagram carries: each major component labelled; arrows labelled with their protocol (REST, gRPC, event, queue); trust boundaries — VPC, public and private subnet, internet edge — drawn explicitly; data stores as cylinders and queues as queue shapes; components grouped by tier or bounded context. An entity-relationship diagram additionally carries every entity with all fields, types and PK/FK markers, correct cardinality on every relationship, and junction tables shown explicitly for N:M.
4. Title every diagram `<Product> — <diagram type> — <requirements version>`, so a stale diagram is identifiable without opening it.
5. Commit the **DSL** under `docs/diagrams/eraser/`, export **PNG and SVG** beside it, and record the **editor URL** in the same commit — in the diagram file's header or the directory README, never only in chat, where it is lost when the session ends.
6. Generate the **in-repo Mermaid equivalent** and commit it under `docs/diagrams/mermaid/`: `C4Context` and `C4Container` for the architecture pair, `erDiagram` for the data model, `sequenceDiagram` for flows. It is a mirror of the same diagram, not a second design — if the two disagree, the Eraser DSL is wrong or the mirror is, and you say which.
7. List every assumption you made and every ambiguity you hit — a relationship whose cardinality the field names do not settle, a component the source names but never places. **Stop.** A human opens the editor URL, refines, re-exports, and signs the diagram off at the design gate.

## Refusal cases

- **Never invent a component, entity, field or arrow.** Draw only what the decisions or the schema contain; list everything else as a gap.
- Eraser MCP unreachable → say so and stop. Never let the Mermaid mirror stand in for the whole deliverable.
- **Never commit an export without its DSL, or a DSL without its editor URL.** A diagram that cannot be re-opened is a picture, and the next change redraws it from nothing.
- **Never `delete` or reorganise anything in the Eraser workspace.** Generate, edit, export and search only.
- Asked to choose the architecture, pick the pattern, or design the schema → refuse; you render a decision someone else has already made.
- Asked to certify that a diagram still matches production → refuse unless you have read the code. A signed-off diagram nobody checked is worse than a missing one.
