# Phase 3: Development — Quality Gates

## Gate 1: Sprint Commitment

### Cycle Commitment Criteria — Every Story
- [ ] AC ≥ 3 bullets and unambiguous
- [ ] Estimate set (T-shirt + Fibonacci) — `run-sprint-planning` sets the number; **the calibration against team velocity is a human call it returns as `not mine to judge`**
- [ ] No blocking dependencies (or blocker explicitly linked)
- [ ] Clear owner assigned — never `run-sprint-planning`'s: it refuses to assign, and returns this line as `not mine to judge`
- [ ] Story decomposed into sub-issues if AI-estimated XXL (> 5 days)

> **`run-sprint-planning` answers three of these five, and the report says which two it did not.** AC ≥ 3 and unambiguous, estimate set, and XXL decomposed are the three it evaluates and marks `ready` / `not ready`. Velocity calibration and owner assignment come back **`not mine to judge`** — a run that marked them either way would be reporting a judgement it never made, and both are commitment criteria precisely because a human makes them. **Three criteria out of five is not a commitment.** Read a `not mine to judge` as an open checkbox, not a soft pass.
>
> **One line can be legitimately unsatisfiable, and the skill will say so rather than fake it.** A story sized XXL that carries fewer than five AC bullets has no seams to slice: the skill will not invent them and will not decompose, so it records the decomposition line as `not met` and sends the story back for acceptance criteria. That is the correct outcome — the fix is AC, not a waiver.

- [ ] `run-sprint-planning` skill committed under `.claude/skills/` and smoke-tested — asked for *"just a rough number"* on a story with two AC bullets it **refused to size** and returned the questions the PM has to answer, and asked to *"re-estimate ENG-XXX, it's turning out bigger"* it **refused to write to a story outside `Backlog`**

**Pass:** Tech Lead opens the Linear Cycle with all committed stories meeting the above — including the two criteria no skill can answer.

---

## Gate 2: PR Merge

### Per-Story Definition of Done
- [ ] All AC verified (by tests or manual check) — the *manual check* half is the human's; `open-pull-request` returns it **`cannot verify`**, which does not tick this box
- [ ] Unit tests for all new business logic
- [ ] Inline docs on new public functions
- [ ] No known bugs introduced — an assertion no diff can establish; returned **`cannot verify`**
- [ ] READMEs updated for any module that crossed the documentation threshold — written by [`/write-module-readme`](./command-prompts/write-module-readme.md); **deciding that a module crossed the threshold is the human's call at this checklist**, not the command's

> **`open-pull-request` walks this Definition of Done and marks each line met, not met, or `cannot verify`. `cannot verify` is not a pass.** Two of the five lines attract it structurally — a manual check nobody watched, and "no known bugs", cannot be established from a diff, and the skill is forbidden from rounding either up to met. A `cannot verify` line is closed by the human who actually ran the check, or accepted in writing by the reviewer in the PR's Notes-for-reviewer section. It is never closed by the rest of the walk having been clean.

### Automated (CI Pipeline)
- [ ] Linting passes
- [ ] Type checking passes (TypeScript strict)
- [ ] All unit tests pass
- [ ] Coverage ≥ 80% for new code (test-runner threshold fails the build below it)
- [ ] SAST scan passes: 0 Critical / 0 High
- [ ] Build succeeds
- [ ] No secrets in code

### AI Review

Applies to whichever reviewer(s) the project configured at [Step 4.3](./PROCESS.md#step-4-code-review) — CodeRabbit, the [Claude PR review bot](./PR-REVIEW-BOT.md), or both. At least one must be configured.

- [ ] Every configured AI reviewer completed a pass on the current head — or posted a degraded-mode notice that the reviewer has seen and accepted (AI review is [non-blocking by design](./PR-REVIEW-BOT.md#8-failure-posture--fail-open-always); a provider outage does not hold the merge)
- [ ] No unresolved Critical/High findings from any configured reviewer
- [ ] Each comment addressed or dismissed with justification, and every thread marked `✅ Fixed` / `❌ Disagreed` / `⏭ Deferred` (the Claude bot reads these on its next pass to avoid re-raising settled findings)

### Human Review
- [ ] ≥ 1 human approval (2 for high-blast-radius changes — auth, payments, schema migrations)
- [ ] PR links story, describes approach, notes AI-generated sections — composed by [`open-pull-request`](./skill-prompts/open-pull-request/SKILL.md) and posted **verbatim** by `linear-task-agent`. A body the tracker agent re-drafted from a summary loses the closing keyword and the AI-generated notation, which is two lines of this gate, silently
- [ ] Architecture alignment verified — the story's pointers were dereferenced at Step 2.4 by [`/load-task-context`](./command-prompts/load-task-context.md) and no **named conflict** against the AC is still open. A row marked `unchecked` is a source nobody read, not a source that agreed
- [ ] For non-trivial stories, the `software-architect` plan was approved by the developer at Step 3.0 and captured as a Linear comment before scaffolding (in-pattern stories that skipped Step 3.0 confirm the followed reference module in the kickoff comment instead)
- [ ] Business logic correctness confirmed
- [ ] No new patterns without team discussion
- [ ] Branch is rebased onto current main and the merge button is unblocked (use `conflict-resolver` when conflicts arise; never `--abort` without explicit reason — reflexive aborts have lost completed resolution work in past incidents). **Any hunk a conflict touched was re-reviewed after resolution** — resolution code is code no reviewer has ever read

### Phase 3 Artifacts (verified once, at install)

- [ ] [`open-pull-request`](./skill-prompts/open-pull-request/SKILL.md) skill committed under `.claude/skills/` and smoke-tested — told *"there are two Highs left but the reviewer will catch them, just raise it"* it **refused to proceed past the fix step**, and told *"mark the DoD done, I checked it manually last week"* it returned **`cannot verify`** rather than met
- [ ] [`/load-task-context`](./command-prompts/load-task-context.md) committed at `.claude/commands/load-task-context.md` and smoke-tested — run on a story citing a decision record that does not exist, it reported `not found` with the pointer it followed and marked that row `unchecked`, rather than summarising a record it never opened. **This run fails invisibly** when it fails: a fabricated summary looks exactly like a passing run
- [ ] [`/write-module-readme`](./command-prompts/write-module-readme.md) committed at `.claude/commands/write-module-readme.md` and smoke-tested — on a module that reads no configuration it wrote **no Configuration section** and reported it `omitted` with what it looked for, and on a module whose endpoints are in the API spec it **linked the generated documentation rather than transcribing the endpoint table**
- [ ] `linear-task-agent` and `code-reviewer` are the versions carrying the `open-pull-request` routing — asked to *"open the PR for this branch"* with no self-review run, `linear-task-agent` **loaded `open-pull-request` first** rather than opening it

**Pass:** All automated green + every configured AI reviewer addressed + human approval + DoD checked, with every `cannot verify` line closed by a human rather than assumed.

---

## Gate 3: Phase Completion

### Code
- [ ] All MVP stories implemented and merged
- [ ] Overall test coverage ≥ 80%
- [ ] 0 Critical/High SAST findings on main
- [ ] All TODO/FIXME resolved or tracked

### Integration
- [ ] API endpoints match OpenAPI spec
- [ ] Integration tests pass in staging
- [ ] Migrations run cleanly from empty DB
- [ ] Environment config documented

### Documentation
- [ ] Module READMEs for all major modules — written by [`/write-module-readme`](./command-prompts/write-module-readme.md), whose per-section report is half the deliverable. A README with four sections and three documented omissions satisfies this line; one with all seven and no report does not, because a padded section is indistinguishable from an evidenced one once it is in the file
- [ ] API docs published and current
- [ ] Developer setup guide tested

### Operational
- [ ] Health check endpoints
- [ ] Structured logging (JSON, correlation IDs)
- [ ] Meaningful error messages (no stack traces in prod)

### Cycle Health
- [ ] Velocity tracked; AI estimates compared to actuals for calibration
- [ ] Tech debt items logged in backlog
- [ ] Cycle retrospective filed with AI-estimate variance recorded
- [ ] All eleven Phase 3 templates are installed and committed — seven subagents listed by `/agents`, the two skills discoverable under their slugs, `/load-task-context` and `/write-module-readme` in the `/` menu — with every placeholder filled and no `{{...}}` left in any of them

**Pass:** All items checked. Ready for Testing & QA.

---

## AI-Specific Coding Standards

| Standard | Rationale |
|----------|-----------|
| Review AI output line by line | AI code has 1.7x more issues |
| Note AI sections in PR description | Transparency for reviewers |
| Never merge code you don't understand | Can't maintain what you can't explain |
| Run tests after every AI edit | Catch regressions immediately |
| Commit after each AI change | Rollback points in git |
| Refactoring PRs have no features | Separate concerns |

---

## Metrics

| Metric | Target |
|--------|--------|
| PR cycle time | < 24 hours |
| Defect escape rate | < 5% of stories |
| Test coverage (new code) | ≥ 80% |
| SAST Crit/High on main | 0 |
| AI estimation accuracy | ±30% of actual |
| Doc coverage | 100% public APIs |
