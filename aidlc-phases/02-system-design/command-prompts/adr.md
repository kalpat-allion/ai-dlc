---
description: "Write an Architecture Decision Record for a decision the team has already made. Reads the repo's decision-record directory to allocate the next sequential number and surfaces what it allocated from, then emits Status (always Proposed), Context citing the requirement that forced the decision, Decision, Consequences split positive / negative / neutral, Alternatives Considered with a specific rejection reason for each, and a measurable Validation signal. Refuses when no alternative was genuinely weighed and rejected, and states no figure it cannot source. Usage: /adr adopt row-level security for tenant isolation"
argument-hint: "[the decision, in a few words]"
disable-model-invocation: true
---

# Architecture Decision Record

You are adding to the permanent record of architecture decisions kept under `{{ADR_DIR}}`. This entry outlives everyone who was in the room, so it has to be specific enough that someone can disagree with it in two years.

**Decision to record:** $ARGUMENTS

## Procedure

1. **Collect four inputs.** The decision; the requirement or constraint that forced it, cited to its source (a requirements-document section anchor, an issue, or a spec); the options that were genuinely weighed; and the binding constraints — budget, expertise, compliance, latency. Ask for anything missing. Never supply it yourself: an invented constraint is read as a recorded fact by everyone downstream.
2. **Read the prior records.** Any record under `{{ADR_DIR}}` this decision contradicts or replaces is named in the Context by number and title. Never edit or renumber an existing record.
3. **Allocate the number.** Take the highest `NNN` prefix in `{{ADR_DIR}}`, add one, zero-pad to three. Then check `git log --all --diff-filter=A --name-only -- {{ADR_DIR}}`: a record added on an unmerged branch is invisible to a directory listing, and that is exactly how two records written the same afternoon end up sharing a number. If `{{ADR_DIR}}` does not exist, create it, start at `001`, and say that you created it.
4. **Write** `{{ADR_DIR}}/NNN-<kebab-title>.md` in the shape below, then **report the number, the file path, and what you allocated it from** — the highest number you saw and where you saw it. If it still collides at merge, the fix is to rename this file; never renumber a record someone has already read.

## Shape

    # ADR-NNN: <title>
    ## Status
    Proposed
    ## Context
    <the situation forcing the decision; cite the requirement source by anchor; name any record this supersedes>
    ## Decision
    <what was decided, and why this option rather than the others>
    ## Consequences — ### Positive / ### Negative / ### Neutral
    <what gets easier, what gets harder and is accepted anyway, what merely changes>
    ## Alternatives Considered — one `###` per alternative
    <Pros / Cons / **Reason rejected**: the specific constraint it failed and how — never "less suitable">
    ## Validation
    <the measurable signal that would show this decision was right, and the one that would show it was wrong>

## Refusals

- **No alternative was genuinely weighed and rejected → refuse to write the record.** If the only options you can name are "do nothing" or one invented to lose, say so and stop. A straw man makes an unconsidered decision look considered, and a reviewer reads that list as the evidence that it was considered.
- **Never state a figure you cannot source.** "Cuts p95 by 40%", "halves the bill" — strike it, or attribute it to the benchmark, quote or measurement it came from. Consequences is where invented numbers enter a repo and are never questioned again.
- **Status is `Proposed`, always.** Accepting a decision is a human act performed in a review; a record that arrives Accepted has skipped that review while looking like it passed one.
- Asked to *choose* between the options rather than record a choice already made → refuse. The record must not be a paraphrase of its own author's advocacy.
