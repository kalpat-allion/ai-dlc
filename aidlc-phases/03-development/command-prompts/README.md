# Phase 3 Command Prompts

This directory holds the **slash-command templates** for Phase 3, referenced from [`../PROCESS.md`](../PROCESS.md). They are templates, not active commands — Claude Code only discovers command files under `.claude/commands/` (and skills under `.claude/skills/`).

It is the sibling of [`../subagent-prompts/`](../subagent-prompts/) and of [`../skill-prompts/`](../skill-prompts/), and it follows the convention established by [`02-system-design/command-prompts/`](../../02-system-design/command-prompts/README.md), which shipped the repo's first command.

## What a command is, and why the tier exists

The three tiers are drawn on **shape**, not topic — see [`../PROCESS.md` → "When to build a skill vs. a subagent vs. a slash command"](../PROCESS.md):

| Tier | Shape | Routing surface |
|---|---|---|
| `subagent-prompts/` | A multi-turn conversation in its own scoped context | Name-invoked. Faces no auto-trigger competition |
| `skill-prompts/` | A deterministic procedure with steps, checklists and human gates | **The `description:` is the entire routing surface** — a skill auto-triggers from user intent, so a description that reuses another artifact's verb hijacks that work |
| `command-prompts/` | **One prompt with arguments.** No procedure to gate, no supporting files | Explicitly fired. **A command carries no auto-trigger obligation** — but only because of one frontmatter field; see [Routing](#routing) |

**A command that grows a procedure, a human gate, a supporting file, or a second argument that branches the behaviour has stopped being a command.** Promote it to `skill-prompts/` rather than letting it thicken here. `/post-mortem` was killed on exactly this test — six input blocks is not "one prompt with arguments" — and both files below were checked against it on the way in.

The shape test is **not** the auto-trigger test. `/write-module-readme` was killed once for failing a discoverability requirement that applies to skills and never applied to it: a command is fired by someone who already knows they want it, and is judged only on whether the output is right. Deciding *when* a module has crossed the documentation threshold is the human's job at the Gate 2 checklist, not the artifact's.

## Files

| File | Invoked as | Does | Gate | Wraps |
|---|---|---|---|---|
| [`load-task-context.md`](./load-task-context.md) | `/load-task-context` | Follows a story's citations to their sources — the requirement section, the governing decision record(s), the API-spec section per endpoint touched, the existing tests in the affected module — and reports each as source path, relevance, and conflict with the AC. Every source it could not open reports `not found` with conflict `unchecked` | None directly; feeds Gate 2 "Architecture alignment verified" | [`task-context`](../PROMPTS.md#task-context) (Step 2.4) |
| [`write-module-readme.md`](./write-module-readme.md) | `/write-module-readme` | Writes a Module README into the module's own directory from seven candidate sections, each admitted only on evidence found in the module, then reports all seven as written-with-evidence or omitted-with-what-was-missing | Gate 2 "READMEs updated for any module that crossed the documentation threshold"; Gate 3 "Module READMEs for all major modules" | [`Documentation Generation`](../PROMPTS.md#documentation-generation) (Step 6.2) |

The **Wraps** column is this directory's purpose. Shipped templates must stand alone in a repo that has no `aidlc-phases/`, so each template **inlines** its prompt with `{{PLACEHOLDERS}}` and carries no framework path in its body — this table is the only record of where the prompt came from. Keep it current or the round-trip is lost.

## How to instantiate per repo

1. Copy the file to `.claude/commands/<name>.md` at the consuming repo root — `load-task-context.md` → `.claude/commands/load-task-context.md`. **The filename is the invocation slug**: renaming the file renames the command. Unlike a skill, there is no folder and no `name:` field to keep in sync, so nothing can silently fall out of alignment.
2. Replace the placeholders with the repo's values:
   - `{{TEAM_PREFIX}}` — the tracker's team key, e.g. `ENG`. Must be the same value the dev-loop tracker agent uses, or the command reads stories from a team the agent never writes to.
   - `{{REQUIREMENTS_DIR}}` — where requirement documents live, e.g. `docs/prd/`, `docs/requirements/`. If requirements live in the tracker rather than the repo, point it at the tracker Document set and say so in the fill.
   - `{{ADR_DIR}}` — where decision records live, e.g. `docs/adrs/`. Same value the repo's `/adr` command writes into.
   - `{{API_SPEC_PATH}}` — the API specification file, e.g. `docs/api/openapi.yaml`. Both commands use it, for opposite reasons: one reads a section from it, the other refuses to hand-copy anything that renders from it.
   - `{{TEST_RUNNER}}` — e.g. `vitest`, `pytest`. Not the CI command with coverage flags: `/load-task-context` needs the runner's **test-file convention** to locate tests, and `/write-module-readme` needs an invocation it can **scope to one module and actually run**. Filled with a whole-suite CI command, the README's Testing section documents a command that does not test this module.
   - `{{TECH_DEBT_LABEL}}` — e.g. `tech-debt`. The label whose issues count as recorded debt for the Limitations section.
3. Commit `.claude/commands/<name>.md` — commands at project scope are shared team infrastructure, exactly like `.claude/agents/` and `.claude/skills/`. Treat edits as code changes requiring review.
4. Cross-link each from the PROCESS step where a developer would reach for it — Step 2.4 and Step 6.2 — so they are discovered at the moment of need rather than in a directory listing.

## Verification

**`/agents` does not list commands** — that surface lists subagents only, and lists neither commands nor skills. Verify differently:

- **Discovery.** Open a Claude Code session in the repo and type `/`. Both commands appear under their slugs with their descriptions. If one is absent, the file is not at `.claude/commands/<name>.md` or the session predates it — reload.
- **`/load-task-context`, on a story whose citations are known-good.** A passing run returns one row per source with a real path, and one row **per endpoint** rather than one row for "the API".
- **`/load-task-context`, adversarially — this is the check that matters.** Run it on a story that cites a decision record which does not exist. It must report `not found` with the pointer it followed and mark that row's conflict `unchecked`. **A run that summarises the missing record, or that writes `none found` on a row it could not read, has failed** — and it fails invisibly, because the output looks exactly like a passing run. Then ask it to write the missing record: it must decline. It reports; it does not fix.
- **`/write-module-readme`, on a module with nothing to configure.** A passing run has **no Configuration section in the file** and an `omitted` line for it in the report. If the file carries a Configuration section listing plausible env vars, the evidence rule has been softened and the command is now inventing repo facts into a file new developers read first. Then run it on a module whose endpoints appear in the API spec: it must link the generated documentation rather than transcribe the endpoint table.
- **The report is half the output.** Both commands are verified as much on what they say they could *not* do as on what they produced. A run that returns only the artifact, with no found/not-found count and no per-section written/omitted list, has dropped the honesty surface and should be treated as a failure even if the artifact looks right.

## Routing

**These commands are explicitly fired, so they compete with nothing** — but that property costs one deliberate frontmatter field per command, and the next author of a command must know why it is there:

- **`disable-model-invocation: true` is what buys it.** **Custom commands have been merged into skills** — a file at `.claude/commands/load-task-context.md` and a skill at `.claude/skills/load-task-context/SKILL.md` both create `/load-task-context` and work the same way. By default *both* the user and Claude can invoke either, so a command's `description` is a routing claim exactly like a skill's. That one field is what makes these user-invocable only. Remove it and the description becomes a claim on user utterances, and the command inherits every collision rule in [`docs/ROUTING.md`](../../../docs/ROUTING.md) — including that a verb it borrows is a verb it steals.

  **So "a command carries no auto-trigger obligation" is a property of these templates, not of this directory.** The tier is a convention held in frontmatter; the folder alone does not grant it. Both files here carry the field deliberately, not by inheritance.
- **Never add a trigger-shaped phrase to a description.** "Use when…", "Triggers on…" — that is skill grammar. A command's description is menu text and usage, not a routing claim.

Two boundaries are worth stating in both directions, because both survive routing:

- **The dev-loop tracker agent surfaces the pointers; `/load-task-context` follows them.** The agent's read-only find flow already returns the story's requirement deep-link and linked decision records alongside its identifier, branch name and labels — that is Steps 2.1-2.3 and it stays there. This command owns Step 2.4 only: **dereferencing** those pointers and reporting what is actually in the sources. It performs no tracker write of any kind, which is the line that keeps the two apart even though both read the same issue.
- **`/write-module-readme` never touches API reference content.** The API specification is the single source for endpoints and payload shapes, and a hand-written copy in a module README competes with the generated documentation without announcing that it is doing so.

One adjacency to note rather than fence: **Limitations reads in-code `TODO` / `FIXME` markers, and Gate 3 requires every `TODO`/`FIXME` to be resolved or tracked.** These agree — the command surfaces the markers into a document a human reads, it does not clear them and must not be asked to.

Placeholder names that recur under other spellings elsewhere are reconciled in [`docs/PLACEHOLDERS.md`](../../../docs/PLACEHOLDERS.md).

> **On `allowed-tools`.** Deliberately omitted, following `/adr`. It is a valid frontmatter field and would pre-approve the file write in `/write-module-readme` and the reads in `/load-task-context`, removing permission prompts — but a shipped template should not decide a consuming repo's permission policy for it. Add it locally if the team wants it; do not add it to the template.
