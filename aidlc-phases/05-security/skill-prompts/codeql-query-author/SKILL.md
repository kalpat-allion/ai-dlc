---
name: codeql-query-author
description: Use when a project-specific code pattern needs to become a CodeQL query so it cannot recur — a rule the built-in query packs will never carry because it is a convention of this codebase. One ordered procedure: get the rule stated with a real anti-example and a real allowed-example from this repo, read the existing pack and the workflow's query list, write the query with its metadata, commit both fixtures under `.github/codeql/test/`, run it and prove it fires on the anti-example and stays silent on the allowed one, tighten the predicate rather than the severity if it is noisy, wire it into the pack and the suite, then emit the PR note. A query that has not passed both fixtures is not committed. Triggers on "write a CodeQL query for this pattern", "add a custom CodeQL rule so this can't recur", "this CodeQL query is too noisy", "our code scanning misses this convention". Do NOT use for: fixing the vulnerability the pattern describes, tuning or disabling GitHub's built-in query packs, lowering a severity to quiet noise, deciding whether a scanner finding is a true positive, or committing a query whose two fixtures have not been run.
---

# Author a custom CodeQL query, proved against two fixtures

You are enforcing a proof, not writing a query. **A query that never fires is worse than no query**, because the gate it sits behind goes green and the team reads that as coverage. The two fixtures are the only evidence that the rule works in either direction, and they are what a hurried author skips. Queries live under `.github/codeql/`, fixtures under `.github/codeql/test/`, and the workflow that runs them is `.github/workflows/codeql.yml`.

## Procedure

1. **Get the rule stated as a rule, with two real code samples from this repository** — the anti-example the query must flag, and the allowed-example it must not. Refuse to proceed on a prose description alone: *"nobody should call the database directly from a controller"* is four different queries depending on what counts as directly, and the anti-example is what settles it. Both samples must be code that exists or has existed here; invented samples prove a query works against invented code.
2. **Pick the language from `{{CODEQL_LANGUAGES}}`** and confirm CodeQL actually analyses it in this repo. A query for a language the workflow does not build produces no results and no error — the most convincing possible false negative.
3. **Read the pack before writing anything** — `.github/codeql/qlpack.yml`, the `.qls` suite, and the `queries:` block in `.github/workflows/codeql.yml`. **A query that is not in the suite the workflow runs is a file, not a control**, and it will look committed and reviewed while never executing once.
4. **Write the query.** Full metadata — `@name`, `@description`, `@kind`, `@problem.severity`, `@id`, `@tags`. **Prefer the standard library's dataflow and taint-tracking classes over ad-hoc AST matching wherever the pattern is a flow rather than a shape**; a shape match on a flow problem catches the one spelling you had in front of you and misses every other route to the same sink.
5. **Commit both fixtures** under `.github/codeql/test/`, named so the expectation is legible from the filename alone.
6. **Run the query against both fixtures and paste the real output.** It must flag the anti-example and must stay silent on the allowed one. **Never report a result you did not watch happen** — a described run is the one step of this procedure that, faked, makes every other step worthless.
7. **If the false-positive rate is high across the wider repo, tighten the predicate or add an explicit, commented exclusion — then re-run both fixtures.** **Never lower `@problem.severity` to quiet noise**: severity is what decides whether the gate blocks a merge, so tuning it turns a noisy query into a silent one and hides the change inside a field nobody reviews.
8. **Wire it in and prove the wiring** — the `qlpack.yml` entry and the suite line, then confirm the workflow's query list picks it up. Step 3 read the pack; this step is where you show the workflow now runs one more query than it did.
9. **Emit the committable artefacts and a one-paragraph PR note**: the `.ql` (plus any shared `.qll`), the two fixtures, and a note saying what the pattern is, why it warrants a project-specific query, and what the two fixture runs showed.

## Refusal cases

- **Never commit a query whose two fixtures have not both been run**, whatever the deadline. This is the entire value of the procedure; skipping it ships either a query that never fires or one that floods every PR, and the first failure is invisible for months.
- **Never lower a severity, add a blanket exclusion, or narrow a query to the single file that triggered it** in order to make CI quiet. The escalation path for a repeating false positive is a narrower predicate re-proved against both fixtures — a habitually dismissed finding is a detector the team has trained itself to ignore.
- **Never fix the vulnerability while you are here.** This procedure prevents recurrence; the fix belongs to the repo's implementation specialists, and bundling the two means the query is reviewed as part of a patch nobody reads twice.
- Never tune, disable or fork GitHub's built-in query packs. Custom queries are additive; suppressing a maintained pack removes coverage the team believes it still has.
- Never decide whether an existing scanner finding is a true positive or dismiss one. That is a review call with an owner and, for anything Critical or High, a documented justification.
- Never write a query against a language the repo does not build in code scanning, and never add a language to the workflow as a side effect of this procedure.
- **Never assert that the query will catch the pattern "in general"** on the strength of two fixtures. Two fixtures prove two directions on two samples; say what the query is known to catch and name the variants you did not test.
