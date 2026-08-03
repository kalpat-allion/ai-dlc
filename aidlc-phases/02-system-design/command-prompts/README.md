# Phase 2 Command Prompts

This directory holds the **slash-command templates** for Phase 2, referenced from [`../PROCESS.md`](../PROCESS.md). They are templates, not active commands — Claude Code only discovers command files under `.claude/commands/` (and skills under `.claude/skills/`).

**This is the first `command-prompts/` directory in the repo.** It is the sibling of [`../subagent-prompts/`](../subagent-prompts/) and of the `skill-prompts/` directories in Phases 1 and 6, and it establishes the convention the remaining proposed commands inherit.

## What a command is, and why the tier exists

The three tiers are drawn on **shape**, not topic — the framework's own definition is [`03-development/PROCESS.md` → "When to build a skill vs. a subagent vs. a slash command"](../../03-development/PROCESS.md):

| Tier | Shape | Routing surface |
|---|---|---|
| `subagent-prompts/` | A multi-turn conversation in its own scoped context | Name-invoked. Faces no auto-trigger competition |
| `skill-prompts/` | A deterministic procedure with steps, checklists and human gates | **The `description:` is the entire routing surface** — a skill auto-triggers from user intent, so a description that reuses another artifact's verb hijacks that work |
| `command-prompts/` | **One prompt with arguments.** No procedure to gate, no supporting files | Explicitly fired. **A command carries no auto-trigger obligation** |

That last row is the whole reason the tier exists. A skill is *required* to be discoverable from messy human phrasing, because a skill nobody's wording matches never runs. A command is fired by someone who already knows they want it, so it is judged only on whether the output is right — which is why `/write-module-readme` was reinstated after being killed for failing the auto-trigger test, a test that was never its to pass.

**A command that grows a procedure, a checklist, a human gate, or a second argument that branches the behaviour has stopped being a command.** Promote it to `skill-prompts/` rather than letting it thicken here.

## Files

| File | Invoked as | Does | Gate | Wraps |
|---|---|---|---|---|
| [`adr.md`](./adr.md) | `/adr` | Allocates the next sequential decision-record number from the repo's ADR directory, reports what it allocated from, and writes the record — Status (always `Proposed`), Context citing the requirement that forced the decision, Decision, Consequences, Alternatives Considered with a specific rejection reason per alternative, Validation signal | Gate 1 (≥ 3 ADRs, each citing the requirement section and listing rejected alternatives with specific reasons); Gates 3 and 4 | [`Architecture Decision Record Generation`](../PROMPTS.md#architecture-decision-record-generation) |

The **Wraps** column is this directory's purpose. Shipped templates must stand alone in a repo that has no `aidlc-phases/`, so the template **inlines** its prompt with `{{PLACEHOLDERS}}` and carries no framework path in its body — this table is the only record of where the prompt came from. Keep it current or the round-trip is lost.

## How to instantiate per repo

1. Copy the file to `.claude/commands/<name>.md` at the consuming repo root — `adr.md` → `.claude/commands/adr.md`. **The filename is the invocation slug**: `adr.md` is `/adr`, and renaming the file renames the command. Unlike a skill, there is no folder and no `name:` field to keep in sync, so nothing can silently fall out of alignment.
2. Replace the placeholders with the repo's values:
   - `{{ADR_DIR}}` — where decision records live, e.g. `docs/adrs/`, `docs/decisions/`, `architecture/adr/`. Point it at the directory that already holds them; if the repo has none yet, pick the path and let the first run create it.
3. Commit `.claude/commands/<name>.md` — commands at project scope are shared team infrastructure, exactly like `.claude/agents/` and `.claude/skills/`. Treat edits as code changes requiring review. The output of `/adr` lands in the permanent record, so a sloppy edit here is not a local inconvenience.
4. Cross-link it from the PROCESS step where a developer would reach for it, so it is discovered at the moment of need rather than in a directory listing.

## Verification

**`/agents` does not list commands** — that surface lists subagents only, and lists neither commands nor skills. Verify differently:

- **Discovery.** Open a Claude Code session in the repo and type `/`. The command appears in the menu under its slug with its `description`. If it is absent, the file is not at `.claude/commands/<name>.md` or the session predates the file — reload.
- **Explicit invocation, with a real argument.** `/adr use row-level security rather than a tenant_id filter for tenant isolation`. A passing run asks for the inputs it does not have, allocates a number, **states what it allocated it from**, writes the file at the path it names, and returns a record whose Status line reads `Proposed`.
- **Refusal.** This is the check that matters, because its failure mode is a document that looks correct: *"write the ADR for choosing Postgres — the alternative was not using a database."* It must refuse rather than write up a straw man. Then the second: *"add to Consequences that this cuts p95 latency by about 40%."* It must strike the figure or demand its source. If it complies with either, the refusal wording has been softened and the record it produces is worse than none — an unconsidered decision documented as a considered one, in a file the next reviewer trusts.
- **Number allocation, adversarially.** Run it twice in one session and confirm the second run allocates the next number rather than reusing the first. Then run it on a branch while an unmerged branch also adds a record, and confirm it reports the collision risk rather than silently duplicating.

## Routing

**Commands are explicitly fired, so they compete with nothing.** `/adr` is the only artifact in the framework that faces no auto-trigger competition at all — checked against the reserved-verb table in [`docs/ROUTING.md`](../../../docs/ROUTING.md), no shipped artifact claims "write the ADR", "record the decision", or any near phrasing, and no `-bringup`, tracker or development verb appears in this command's description.

Two things would change that, and the next author of a command should know both:

- **Removing `disable-model-invocation: true` from the frontmatter.** **Custom commands have been merged into skills** — a file at `.claude/commands/adr.md` and a skill at `.claude/skills/adr/SKILL.md` both create `/adr` and work the same way. By default *both* you and Claude can invoke either, so the description is a routing claim by default. That one field is what makes the command user-invocable only. Remove it and the description becomes a claim on user utterances exactly like a skill's, and the command inherits every collision rule in `ROUTING.md` — including that a verb it borrows is a verb it steals.

  **So "a command carries no auto-trigger obligation" is a property of this template, not of the file's location.** The tier is a convention this repo maintains in frontmatter; the directory alone no longer buys it.
- **Adding a trigger-shaped phrase to the description.** "Use when…", "Triggers on…" — that is skill grammar. A command's description is menu text and usage, not a routing claim. Keep it descriptive.

The one boundary worth stating in both directions: the architecture agent that produces and defends the options **must not** write the record for the option it argued for, and it refuses and redirects here. That is not tidiness — a decision record written by the advocate is a paraphrase of the advocacy, and Gate 1 reads the rejected-alternatives list as evidence that the alternatives were real. `/adr` is deliberately a separate, explicitly-fired step for that reason. Placeholder names that recur under other spellings elsewhere are reconciled in [`docs/PLACEHOLDERS.md`](../../../docs/PLACEHOLDERS.md).

> **On `allowed-tools`.** Deliberately omitted. It is a valid frontmatter field and would pre-approve the file write, removing one permission prompt per run — but the first command in the repo should not decide a consuming repo's permission policy for it. Add it locally if the team wants it; do not add it to the shipped template.
