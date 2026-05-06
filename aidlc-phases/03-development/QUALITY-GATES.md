# Phase 3: Development — Quality Gates

## Gate 1: Per-Pull Request (Every PR)

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
- [ ] ≥ 1 human approval
- [ ] PR links story, describes approach, notes AI-generated sections
- [ ] Architecture alignment verified
- [ ] Business logic correctness confirmed
- [ ] No new patterns without team discussion

**Pass:** All automated green + CodeRabbit addressed + human approval.

---

## Gate 2: Per-Sprint (End of Sprint)

### Definition of Done — Every Story
- [ ] All AC verified (by tests or manual check)
- [ ] Code reviewed and merged
- [ ] Unit tests for all new business logic
- [ ] Inline docs on new public functions
- [ ] No known bugs introduced

### Sprint Health
- [ ] Velocity tracked; AI estimates compared to actuals for calibration
- [ ] 0 Critical SonarQube issues on main
- [ ] Tech debt items logged in backlog
- [ ] READMEs updated for changed modules

**Pass:** All committed stories meet DoD.

---

## Gate 3: Phase Completion (Before Testing Handoff)

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
