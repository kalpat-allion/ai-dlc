---
name: data-model-design
description: Use when approved product requirements must become a database schema for the first time — extracting the domain entities, their ownership and their cardinalities from the requirements document, generating the schema in the repo's ORM with indexes covering the stated top query patterns, stopping for a mandatory backend-developer review, then producing the up and down migrations plus seed data and running them against a local database. Triggers on "design the data model from the PRD", "extract the entities from the PRD", "we need the data model before we start building", "turn the approved requirements into a schema". The triggers are deliberately narrow: this skill claims greenfield data-model design only. Do NOT use for: a schema change or a migration inside a story already in flight — that belongs to the repo's server-side implementation specialist, who ships it in the story's own commit, and this skill must not take it; denormalising without a measured query and a recorded architecture decision; running a migration against anything but a local database; rendering the ER diagram; or designing the API contract that sits over the schema.
---

# Design the data model

You are turning approved requirements into a schema the team will live with for years. The source of truth is the **approved requirements document** and the **conventions the repo already follows** — read the existing schema, naming style and migration history before writing anything, because a schema that fights the repo's conventions gets rewritten within a sprint.

This procedure has a hard stop in the middle. Steps 1-6 are design. Step 8 writes migrations, and must not begin until a human has read step 6's output.

## Procedure

1. Read the approved requirements — functional requirements and user flows — plus the bounded contexts and the cross-context communication pattern the architecture decisions recorded. **Refuse if the requirements are unapproved or unreadable.** Never invent entities.
2. Extract the entity list. For each: name (singular, PascalCase) · owning bounded context · one-sentence purpose · identity (natural key, surrogate, or composite) · attributes with type intent and nullability · lifecycle (created when, archived or deleted when).
3. Extract the relationships: source → target, cardinality (1:1, 1:N, N:M), and whether the link crosses a bounded-context boundary. **Flag every crossing** — those usually become events or duplicated read models rather than foreign keys — and recommend shared FK, duplicated read model, or event propagation, with a one-line reason each.
4. List the open questions: everything the requirements left ambiguous about identity, ownership or cardinality. Guess none of them.
5. Ask for the **top 5-10 queries the application will actually run**, and **refuse to proceed without them**. Indexes designed against imagined queries are the expensive kind of wrong.
6. Generate the schema in `{{ORM}}` for `{{DATASTORE}}` at `{{SCHEMA_PATH}}`, matching the repo's existing naming and structure: entities with fields, types, constraints and defaults; foreign keys carrying the step-3 cardinalities; indexes covering every query from step 5; enums; created and updated timestamps on every entity; soft-delete columns where the step-2 lifecycle calls for one. Flag any field the requirements leave ambiguous about type or nullability instead of picking one.
7. **Hard stop for backend-developer review.** A named backend developer walks the schema: third normal form unless a denormalisation is explicitly justified, cardinalities match the requirements, every list-page query has a covering index, every search field has a stated strategy (btree, gin, full-text). Do not continue on your own judgement, and do not continue because the reviewer is unavailable.
8. On approval, generate into `{{MIGRATIONS_PATH}}`: an **up** migration creating every table, index, enum, constraint and trigger, with DDL ordered so foreign keys resolve; a **down** migration reversing it, with an explicit comment on any step that would lose data irrecoverably; and **seed data** — 5-10 rows per entity, realistic values rather than foo/bar, referential integrity intact, deterministic IDs so tests are stable. All timestamps default to UTC; every script idempotent where the tool supports it.
9. Run the migration against a **local** database: up, down, then up again. Report the result verbatim. The schema is not done until it migrates cleanly twice. → It now goes to the data-model gate for Tech Lead and backend sign-off.

## Refusal cases

- **Never denormalise without a measured query and a recorded architecture decision.** An AI suggestion is not a measurement, and neither is "it'll be faster".
- **Never run a migration against anything but a local database.** Not staging, not a shared dev database, not "just to check" — hand the command to the human instead.
- **Never skip the step-7 review**, and never let your own re-read count as it. A second person is the entire point.
- Never emit a data-destructive operation — DROP COLUMN, a precision-losing ALTER TYPE, a table rename without a copy — without an explicit comment justifying it.
- Never add an entity, field or relationship the requirements do not contain. Report it as a gap; adding scope here bypasses requirements approval.
- Never stub an entity, enum or extension the schema references but you cannot see. Stop and list what is missing.
- A schema change or migration **inside a story already in flight is not this skill's work.** Hand it to the repo's server-side implementation specialist.
- The ER diagram is not yours either. Hand the finished schema to `render-design-diagrams`.
