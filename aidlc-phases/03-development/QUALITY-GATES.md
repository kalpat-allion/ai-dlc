# Phase 3: Development — Quality Gates

## Gate 1: Sprint Commitment

### Cycle Commitment Criteria — Every Story
- [ ] AC ≥ 3 bullets and unambiguous
- [ ] Estimate set (T-shirt + Fibonacci) and calibrated against team velocity
- [ ] No blocking dependencies (or blocker explicitly linked)
- [ ] Clear owner assigned
- [ ] Story decomposed into sub-issues if AI-estimated XXL (> 5 days)

**Pass:** Tech Lead opens the Linear Cycle with all committed stories meeting the above.

---

## Gate 2: PR Merge

### Per-Story Definition of Done
- [ ] All AC verified (by tests or manual check)
- [ ] Unit tests for all new business logic
- [ ] Inline docs on new public functions
- [ ] No known bugs introduced
- [ ] READMEs updated for any module that crossed the documentation threshold

### Automated (CI Pipeline)
- [ ] Linting passes
- [ ] Type checking passes (TypeScript strict)
- [ ] All unit tests pass
- [ ] Coverage ≥ 80% for new code
- [ ] SonarQube quality gate passes (0 Critical, 0 High vulnerabilities)
- [ ] Build succeeds
- [ ] No secrets in code

### AI Review (CodeRabbit)
- [ ] CodeRabbit review completed
- [ ] No unresolved Critical/High comments
- [ ] Each comment addressed or dismissed with justification

### Human Review
- [ ] ≥ 1 human approval (2 for high-blast-radius changes — auth, payments, schema migrations)
- [ ] PR links story, describes approach, notes AI-generated sections
- [ ] Architecture alignment verified
- [ ] For non-trivial stories, the `software-architect` plan was approved by the developer at Step 3.0 and captured as a Linear comment before scaffolding (in-pattern stories that skipped Step 3.0 confirm the followed reference module in the kickoff comment instead)
- [ ] Business logic correctness confirmed
- [ ] No new patterns without team discussion
- [ ] Branch is rebased onto current main and the merge button is unblocked (use `conflict-resolver` when conflicts arise; never `--abort` without explicit reason — reflexive aborts have lost completed resolution work in past incidents)

**Pass:** All automated green + CodeRabbit addressed + human approval + DoD checked.

---

## Gate 3: Phase Completion

### Code
- [ ] All MVP stories implemented and merged
- [ ] Overall test coverage ≥ 80%
- [ ] 0 Critical/High SonarQube issues on main
- [ ] All TODO/FIXME resolved or tracked

### Integration
- [ ] API endpoints match OpenAPI spec
- [ ] Integration tests pass in staging
- [ ] Migrations run cleanly from empty DB
- [ ] Environment config documented

### Documentation
- [ ] Module READMEs for all major modules
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
| SonarQube Crit/High on main | 0 |
| AI estimation accuracy | ±30% of actual |
| Doc coverage | 100% public APIs |
