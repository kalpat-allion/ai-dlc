# Phase 4: Testing & QA — Quality Gates

## Gate 1: Test Plan Approved

Before writing tests, the test plan must pass these checks.

- [ ] Test plan covers all 7 activity areas (unit, integration, E2E, mobile if applicable, load, test plan, bug triage)
- [ ] Every user story acceptance criterion is mapped to at least one test case
- [ ] Critical user journeys for E2E are explicitly identified (not "test everything")
- [ ] Load test scenarios include specific NFR targets (p95 latency, error rate, concurrency)
- [ ] Test environments identified (dev, staging, performance)
- [ ] Test data strategy documented (how seeded, how cleaned up)
- [ ] Test ownership assigned (who writes what)
- [ ] Entry and exit criteria defined
- [ ] Plan reviewed by QA Lead + Tech Lead

**Pass:** All items checked. Plan approved and committed to `/docs/test-plan.md`.

---

## Gate 2: Per-Sprint — Test Coverage

Runs continuously during Development phase sprints.

### Every PR
- [ ] New code has corresponding unit tests
- [ ] Test coverage ≥ 80% on new code (SonarQube enforced)
- [ ] All AC for the story have test coverage
- [ ] No test is a "fake pass" (AI-generated tests with always-true assertions)

### Every Sprint End
- [ ] Integration tests exist for all new API endpoints
- [ ] E2E tests exist for all new critical user journeys
- [ ] Flakiness rate < 2% across all test suites
- [ ] No tests skipped without a tracking ticket

**Pass:** All items checked per PR + per sprint.

---

## Gate 3: Release Candidate Testing

Before a build is promoted to release candidate, these must pass.

### Functional Testing
- [ ] All unit tests pass on main branch
- [ ] All integration tests pass against staging DB
- [ ] All critical-path E2E tests pass
- [ ] Mobile tests pass on target device matrix (if applicable)

### Performance Testing
- [ ] Load tests run against staging
- [ ] p95 latency meets NFR target
- [ ] p99 latency meets NFR target
- [ ] Error rate < target under expected peak load
- [ ] No memory leaks detected in sustained load test

### Regression
- [ ] Full E2E regression suite passes
- [ ] No tests skipped or xfailed without approved tracking ticket
- [ ] All previously-fixed bugs have regression tests still passing

### Bug Status
- [ ] 0 S0 (Critical) bugs open
- [ ] 0 S1 (High) bugs open
- [ ] All S2 (Medium) bugs have documented fix plan (this sprint or next)

**Pass:** All items checked. Build is approved as release candidate.

---

## Gate 4: Phase Handoff

Before handing off to Security & Compliance phase.

- [ ] Test plan (final version) committed to `/docs/test-plan.md`
- [ ] Test suites committed for all test types
- [ ] Coverage report accessible (SonarQube dashboard + CI artefact)
- [ ] Load test results archived with timestamped reports
- [ ] Known issues log exists (or Linear saved view)
- [ ] Regression test baseline committed
- [ ] Test environments documented (how to provision, how to connect)
- [ ] Release candidate has passed Gate 3

**Pass:** All artefacts available for the Security & Compliance team to reference.

---

## AI-Specific Testing Standards

| Standard | Rationale |
|----------|-----------|
| **Review every AI-generated test line by line** | AI tests can appear correct but miss edge cases or have fake assertions |
| **Never merge tests that "always pass"** | Worse than no tests; gives false confidence |
| **Test behaviour not implementation** | Tests coupled to implementation details break on every refactor |
| **Realistic test data** | AI often generates "foo"/"bar"/"test123" — use data that resembles production |
| **One assertion per logical step** | Makes failures diagnosable; AI tends to pack multiple assertions per test |
| **Run tests after every AI edit** | AI may break existing tests without realising; CI catches this |

---

## Metrics to Track

| Metric | Target | Measurement |
|--------|--------|-------------|
| Unit test coverage (new code) | ≥ 80% | SonarQube per PR |
| Integration test coverage (endpoints) | ≥ 60% happy path + error | Manual audit per sprint |
| E2E test coverage (critical journeys) | 100% | Test plan mapping |
| Flakiness rate | < 2% | CI run history |
| Defect escape rate | < 5% | Production bugs / sprint stories |
| Time from bug report to fix | Median < 48h for S0/S1 | Linear analytics |
| Load test: p95 latency | Per NFR target | k6 output per release |
| AI test generation time saved | Track per sprint | Developer self-report |
