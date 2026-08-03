---
description: "Load the working context for one story before any code is written. Reads the requirement section the story cites, the decision record(s) governing the module it touches, the API-spec section for every endpoint the acceptance criteria touch, and the existing tests in the affected module — reporting each as source path, what in it is relevant, and any conflict with the acceptance criteria. Reports every source it could not open as `not found` with its conflict column marked `unchecked`, and never summarises a source it did not read. Read-only: no tracker writes, no state transitions, no code. Usage: /load-task-context ENG-247"
argument-hint: "[story identifier, e.g. ENG-247]"
disable-model-invocation: true
---

# Load task context

You are assembling the working context for one story before a line of its code exists. The design pass, the implementation and the review are all built on what you return here, and none of them re-check you. **A source you summarise without opening is worse than one you report missing** — missing reads as a blocker and stops someone; plausible reads as done and stops no one.

**Story:** $ARGUMENTS

## Procedure

1. **Read the story** in the tracker (identifier prefix `{{TEAM_PREFIX}}`) for its acceptance criteria and the pointers it carries — the requirement section it cites, the decision records it links, the endpoints it names. Read-only: never transition state, post a comment, or file anything. If the identifier matches no story, stop and say so rather than searching for the one you think was meant.
2. **Follow each pointer to its source, in this order.** Record where each pointer itself came from, so a broken pointer reads as broken rather than as absent:
   1. The requirement section the story cites, under `{{REQUIREMENTS_DIR}}`.
   2. The decision record(s) the story cites, **and** any under `{{ADR_DIR}}` governing the module being touched — a governing record the story forgot to cite is the one most likely to be violated.
   3. The section of `{{API_SPEC_PATH}}` for **every** endpoint the acceptance criteria touch — one row per endpoint, never one row for "the API".
   4. The existing tests in the affected module, found by `{{TEST_RUNNER}}`'s file convention: each test file path and the behaviours it covers.
3. **Report one row per source**, never a merged one: `source path` · `what in it is relevant to this story, in one paragraph` · `conflict with the AC`.
4. **Close with the count** — sources found, sources not found — and, where any is missing, the single question the developer must answer before coding starts.

## The three conflict values

Every row carries exactly one. The column is never blank.

- **A named conflict** — quote the AC bullet and the line of the source that contradicts it. "Seems inconsistent" is not a conflict; name both sides and let the developer judge.
- **`none found`** — you read the source in full and it agrees with the AC.
- **`unchecked`** — the source was not found, or you could not read it. **This is the load-bearing value.** A record you never opened cannot agree with anything, and `none found` on a row you never read is the precise failure this command exists to prevent.

## Refusals

- **Never summarise a source you did not open.** Not from its filename, not from the story's own paraphrase of it, not from what a record with that title usually says. Report `not found`, name the pointer you followed, and stop being helpful about it.
- **Never invent the pointer either.** If the story cites no requirement section, that absence *is* the finding — do not go looking for the section it probably meant and then report that as the citation.
- **You report; you do not fix.** Do not write the missing record, amend the spec, add the absent test, or start the implementation. A conflict you resolve quietly removes the one moment anyone would have noticed it.
- **Asked to begin coding once the context is loaded → decline and hand back.** This command ends at the report.
