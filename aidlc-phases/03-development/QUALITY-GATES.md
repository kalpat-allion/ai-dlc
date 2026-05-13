# Phase 3: Development — Quality Gates

## Gate 0: One-Time Setup (Before First Story)

### Claude Code Skills (Step 0a)
- [ ] `.claude/skills/` directory exists at the repo root and is committed
- [ ] Each skill is its own directory under `.claude/skills/<skill-name>/` containing at minimum `SKILL.md`
- [ ] Every `SKILL.md` has YAML frontmatter with `name` (lowercase-kebab-case, matches the directory) and `description` set; `allowed-tools` declared where the skill should not have unfettered tool access
- [ ] Every `description` names **when** the skill triggers in concrete developer-prompt terms (not only what it does), under ~200 characters, with at least one verb-phrase a developer would say out loud
- [ ] No two skills have overlapping trigger phrases (collision check passed)
- [ ] Supporting files (templates, checklists, reference docs) live under the skill's own directory and are referenced from `SKILL.md` by relative path
- [ ] Each skill has been smoke-tested with [`skill-smoke-test`](./PROMPTS.md#skill-smoke-test): trigger prompt fires the skill, near-miss prompt does not, output shape matches expectations
- [ ] Skill count ≤ 10 (regulated-industry teams may add compliance / accessibility skills with justification)
- [ ] Repo README mentions the skill bundle and how to extend it
- [ ] Tech Lead has reviewed every skill before commit

### Linear MCP + Subagent Roster (Step 0b)
- [ ] `claude mcp list` shows `linear: connected` for every developer
- [ ] Linear ↔ git integration enabled (auto-link by branch name, auto In Review on PR open, auto Done on PR merge)
- [ ] Anthropic Connectors policy: `update_issue`, `assign_issue`, `create_issue` enabled; `delete_issue` disabled workspace-wide
- [ ] `.claude/agents/linear-task-agent.md` and the six Phase 3 specialists (`software-architect`, `frontend-engineer`, `backend-engineer`, `code-reviewer`, `refactor-specialist`, `conflict-resolver`) committed and listed by `/agents`
- [ ] `linear-task-agent` no-write smoke test returns a real `branchName`
- [ ] Subagent system prompts reference the Step 0a skills they rely on (no dangling references)

**Pass:** Both Step 0a and Step 0b checklists complete. Re-run Gate 0 whenever the stack, conventions, or subagent roster change.

---

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
- [ ] For non-trivial stories: architecture plan was approved by the developer at Step 3.0 and captured as a Linear comment before scaffolding
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
