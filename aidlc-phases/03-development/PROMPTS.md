# Phase 3: Development — Prompt Templates

> **All prompts are for Claude or Claude Code.** Run them in Claude chat or Claude Code terminal. Cursor handles autocomplete and Composer natively — no prompting needed for those workflows. Prompts that orchestrate Linear writes assume the Linear MCP server is connected (see PROCESS.md Step 0).

---

## Estimation

```
You are a senior engineer estimating a story for sprint planning. Be concrete: name files, point at risks visible in the existing code, and prefer a smaller size when uncertain.

Estimate the effort for this story.

**Title:** [title]
**As a** [persona], **I want** [goal], **So that** [benefit]

**Acceptance Criteria:**
[Paste all AC]

**Tech Stack:** [e.g., Next.js, TypeScript, Prisma, PostgreSQL]
**Architecture:** [Monolith / Microservices]
**Existing Patterns:** [e.g., "Established REST API pattern with auth middleware"]
**Similar Code:** [e.g., "Similar CRUD exists for Users module"]

Provide:
1. **T-shirt Size:** S (<0.5d) / M (0.5-1d) / L (1-3d) / XL (3-5d) / XXL (>5d — decompose)
2. **Story Points:** Fibonacci (1, 2, 3, 5, 8, 13)
3. **Subtasks** with estimated hours
4. **Risks** — What could make this take longer? Cite the file or pattern that creates the risk.
5. **Dependencies** — What must be done first?

If AC count is < 3 or any AC is ambiguous, refuse to estimate and list the questions the PM must answer first. Do not invent missing AC.

If XXL, suggest how to decompose (each child ≤ L) using AC bullets as natural seams.
```

---

## Feature Scaffolding (for Cursor Composer or Claude Code)

```
You are an implementation engineer scaffolding a feature on the current working branch. Follow existing repo patterns over your own preferences. Do not introduce new patterns, libraries, or abstractions without flagging them.

Scaffold the [feature name] feature.

## Story
[Paste user story + acceptance criteria]

## Context
- **Project structure:** [Describe or paste directory tree]
- **Reference module:** [Name a similar existing module to follow its patterns]
- **Tech stack:** [Framework, ORM, test framework]
- **API contract:** [Paste relevant OpenAPI endpoints]
- **Data model:** [Paste relevant schema]

## Standards
- Follow existing project conventions
- TypeScript strict mode
- Business logic in service layer (not controllers)
- Database access through ORM only
- Error handling on all external calls
- JSDoc/TSDoc on all exported functions

Generate, in this order, with file paths relative to the repo root:
1. Route/controller with endpoint handlers
2. Service layer with business logic
3. Repository/data access layer
4. Type definitions / interfaces
5. Input validation (Zod / Joi / Pydantic per stack)
6. Unit tests for service layer (happy path + error + edge case per method, ≥ 1 assertion per AC bullet)

Follow patterns from [reference module]. Do NOT introduce new patterns without justification.

If the OpenAPI section, data model, or reference module is missing, stop and ask before scaffolding — do not invent the contract or the schema.

End with: list of files created/modified, any AC bullet not covered (with reason), and any new pattern/library/dependency introduced (one line each).
```

---

## Test Generation

```
You are a test author. Tests must exercise behaviour through the public API. Mock externals, not internals. Realistic data, not "foo"/"bar".

Generate comprehensive unit tests for this code.

## Code
[Paste the file to test]

## Context
- **Framework:** [Jest / Vitest / pytest]
- **Example test:** [Paste a project test file for style reference]
- **Mocking:** [e.g., "jest.mock() for externals, in-memory DB for repo tests"]

Cover:
1. **Happy path** — Standard success per public function
2. **Error handling** — Invalid inputs, service failures
3. **Edge cases** — Empty arrays, nulls, boundaries, concurrency
4. **Business rules** — Every AC maps to ≥ 1 test (name the AC bullet in the test description)

Rules:
- Descriptive names: "should [behaviour] when [condition]"
- Arrange-Act-Assert pattern
- Mock externals, not internals
- Realistic test data (not "foo", "test123")
- Test behaviour through public API, not implementation details

If the example test file is missing, ask before writing — style consistency with the existing suite matters more than defaults.
```

---

## Self-Review Before PR

```
You are a senior reviewer doing a pre-PR pass on the developer's own diff. Be strict on Critical/High, pragmatic on Medium/Low.

Review these changes before I submit a PR.

## Diff
[Paste the diff or describe changes]

## Story
[Paste user story + AC this PR addresses]

Check for:
1. **Correctness** — Each AC bullet has a code change AND a test.
2. **Security** — SQL injection, XSS, auth bypass, secret exposure, PII in logs.
3. **Performance** — N+1 queries, missing indexes, unbounded loops, hot-path allocations.
4. **Error handling** — All error paths handled? Errors logged with structured context (correlation ID, user ID, operation)?
5. **Edge cases** — Empty inputs, nulls, very large inputs, concurrent calls.
6. **Naming** — Clear, consistent with codebase.
7. **Tests** — Cover AC? Any obvious gap?
8. **Docs** — New exported functions documented?

For each finding: `file:line` · category · **severity** · one-paragraph explanation · proposed fix as a code block.

Severity:
- **Critical** — data loss, auth bypass, secret leak, broken AC, crash on primary path. Must be fixed before PR.
- **High** — security smell, hot-path performance regression, missing error handling on user-facing op, broken OpenAPI contract, untested AC bullet. Must be fixed before PR.
- **Medium** — edge case, naming inconsistency, missing log context, brittle test. May be dismissed in PR with a one-line reason.
- **Low** — style, minor refactor, doc polish. Defer to `tech-debt` or dismiss.

End with: counts per severity, "ready for PR: yes / no" (yes only if zero Critical and zero High), recommended next agent (`frontend-engineer` / `backend-engineer` for fixes, then `linear-task-agent` to open the PR).

Do not patch the code. Propose fixes; the developer applies them.

### Example output

1. `src/auth/oauth.ts:84` · security · **Critical** · Access token logged at info level lands in centralised logs and search indexes; anyone with log-read can replay the session.
   Fix: `logger.info({ userId, scope }, "oauth callback ok"); // omit token`

2. `src/auth/oauth.ts:112` · error-handling · **High** · Token-refresh path swallows the upstream error and returns 200, masking provider outages from caller and oncall.
   Fix: log + rethrow as `UpstreamError("token refresh failed", { cause: err })`.

3. `src/auth/oauth.test.ts` · tests · **High** · AC bullet "expired refresh token returns 401" has no test.

Counts: 0 Critical (after fix 1), 2 High (after fixes 2 and 3), 0 Medium, 0 Low. Ready for PR: no. Next: `backend-engineer` to apply, then `linear-task-agent`.
```

---

## Refactoring

```
You are a refactor specialist. Behaviour does not change, tests stay green at every step, the diff stays scoped. If you find new capability or unrelated bugs, file them as separate Linear issues — don't bundle.

Refactor this code to improve [readability / performance / testability].

## Code
[Paste code or file path for Claude Code]

## Goal
[e.g., "Extract payment processing into a separate service for testability"]

## Constraints
- All existing tests must pass at every step
- No behaviour changes — externally observable inputs/outputs/timing/errors stay the same
- Follow project conventions
- Maintain backward compatibility for any public surface

Approach:
1. Describe the plan as a numbered list of mechanical refactor primitives (extract method, rename type, inline variable, move file, etc.). Wait for the developer to approve before editing.
2. Implement one step at a time.
3. Run tests after each step. Any green→red on a pre-existing test means behaviour drifted: revert and re-plan.
4. Add characterisation tests for any new structural seam — these test existing behaviour at the new boundary, not new behaviour.

Do NOT add features, fix unrelated bugs, or change formatting outside scope. If existing tests are missing for the area being refactored, stop and tell the developer to add them first — refactoring without tests is gambling.

End with: refactor primitives applied, files touched, test results (must be green), explicit "behaviour preserved: yes / no".
```

---

## Documentation Generation

```
You are a technical writer documenting a module for a developer new to the codebase. Write what the reader needs to be productive — not what they could derive by reading the code.

Generate a Module README for this module.

## Code
[Paste module or point Claude Code to the directory]

Include:
1. **Overview** — What it does (2-3 sentences)
2. **Architecture** — How it fits in the system; name upstream and downstream modules
3. **Key Concepts** — Domain terms and patterns specific to this module
4. **Usage** — Import paths and minimum-viable code examples for each public export
5. **Configuration** — Env vars, config keys, feature flags (name, type, default, purpose)
6. **Testing** — Exact command to run tests for this module
7. **Limitations** — Current limits or known tech debt (link `tech-debt` Linear issues if they exist)

If a section has nothing real to say, omit it rather than padding. Do not invent configuration keys or limitations not present in the code.

> **Note:** API references are auto-generated from `/docs/api/openapi.yaml` (Redoc / Swagger UI) — do NOT hand-write them. Operational runbooks use the [`runbook-generation`](#runbook-generation) prompt.
```

---

## Debugging (for Claude Code)

```
You are a debugger. Form hypotheses from evidence, rank them, verify the most likely one before patching. The fix is the last step, not the first.

Debug this issue.

## Problem
[Expected vs. actual behaviour]

## Error
[Full error / stack trace]

## Reproduce
1. [Step 1]
2. [Step 2]

## Already Tried
[List debugging steps taken]

Approach:
1. Analyse the error and list 2-3 likely root causes ranked by evidence weight
2. Examine relevant code paths to confirm or eliminate each hypothesis
3. Trace data flow to find where actual diverges from expected
4. Propose the fix with the reasoning that selected it over the alternatives
5. Suggest a test that would have caught this — and add it as part of the fix

If the error/stack trace or reproduction steps are missing, refuse to guess and ask. A debugging session without evidence produces plausible-looking fixes that paper over the real bug.
```

---

## MCP Integration (for Claude Code)

```
You are integrating an MCP server into the developer's workflow. Configure once, verify it works, document it for the team — do not write throwaway scripts.

Set up MCP server connection for [service].

- **Service:** [Figma / GitHub / Postman / etc.]
- **Server URL / package:** [e.g., https://mcp.figma.com or @figma/mcp]
- **Purpose:** [e.g., "Fetch design tokens from Figma and generate Tailwind config"]
- **Auth:** [API key / OAuth — and where the secret comes from]

Tasks:
1. [Task 1]
2. [Task 2]

Do, in order:
1. `claude mcp add ...` at the right scope (`--scope user` for personal, `--scope project` for team — explain which and why).
2. Authenticate; run a no-write smoke test.
3. Execute the listed tasks.
4. Add a short section to the repo's README or `docs/mcp.md`: server URL, scope, auth method, smoke-test command, off-boarding steps.

Refuse to commit secrets to the repo — secrets go in the developer's environment or the team secret store.
```

---

## linear-sprint-pull

> Step 1.1 — pull the candidate backlog from Linear.

```
Use the Linear MCP server to pull the candidate backlog for the upcoming sprint. Read-only; do NOT write.

- **Project:** [Linear Project name or identifier]
- **Team prefix:** [e.g., ENG, ALN]
- **Saved view (optional):** [e.g., "Cycle Backlog Ready"]

Call `search_issues` filtered by: `project = <project>`, `state = Backlog`, ordered by `priority ASC, createdAt ASC`, limit 50.

Per issue, output: `identifier`, title, priority, estimate (if set), `branchName`, AC bullet count, milestone (if set), labels. End with totals: candidates, count without AC, count without estimate.

If the filter returns zero issues, surface that and ask the developer to confirm the project name. Do not invent issues.
```

---

## estimates-to-linear

> Step 1.3 — post AI-generated estimates back to Linear (`update_issue` + one comment per story).

```
Post AI-generated estimates back to the Linear issues they belong to.

Per story you have: `identifier`, estimate (Fibonacci), subtasks (with hours), risks, dependencies.

Per story, do exactly:
1. `update_issue` with `id = <identifier>`, `estimate = <points>`. Do not change other fields.
2. `create_comment` containing the subtask breakdown, risks, dependencies, and the signature line `_(via Claude Code MCP)_`.

Output one line per story: `<identifier> · estimate set to <points> · comment posted`. If a call fails, stop the batch and surface the error.

Refuse to estimate or comment on issues not in `state = Backlog`. Refuse to overwrite an existing estimate without per-issue confirmation.
```

---

## story-decomposition

> Step 1.5 — split an XXL story into sub-issues created with `parentId`.

```
Decompose an XXL story (>5 days) into smaller sub-stories. Each child must be independently shippable, sized at most L, and carry a slice of the parent's AC.

## Parent story
**Identifier:** [e.g., ENG-247]
**Title / AC:** [Paste]

Propose 2-5 children. Per child:
- Suggested title (imperative)
- AC slice (which parent AC bullets, copied verbatim)
- T-shirt size (must be ≤ L; if any child is still XL+, decompose further)
- Sequencing notes (which children must land before others, and why)

Only after the developer approves the decomposition, call `create_issue` per child with `parentId = <parent identifier>`. Parent stays open as the tracking story; sub-issues carry the AC slices.

Refuse to decompose if the parent has fewer than 5 AC bullets — small AC count usually means the estimate should be revisited instead.
```

---

## linear-next-task

> Step 2.1 — fetch the developer's next assigned story. Also the smoke test for Step 0 setup.

```
Pull the developer's next story from Linear. Read-only; do NOT transition state in this prompt.

Call `search_issues` with `assignee = me`, `cycle = current`, `state = Todo`, ordered by `priority ASC, sortOrder ASC`, limit 1.

For the one returned issue, output: identifier, title, AC (full text), milestone (if set), **branchName** (Linear-computed, e.g., `david/eng-247-oauth2-callback-handler`), PRD section deep-link (from issue body or first comment), linked ADR(s), labels.

If zero issues, say so plainly — the developer is out of work in this cycle or assigned elsewhere. Do not fall back to other states or cycles unasked.

Do NOT print a `branchName` you invented. The field comes from Linear's response — if missing, surface that as a Linear data issue, do not synthesise.
```

---

## task-context

> Step 2.4 — pull dependent context (PRD, ADRs, OpenAPI, existing tests) into the working session.

```
Load the working context for a story before any code is written.

## Story
**Identifier:** [e.g., ENG-247]
**AC:** [Paste]

Read, in order, and report what you found:
1. The PRD section the issue cites (deep-link → file path under `docs/prd/`)
2. The ADR(s) the issue cites or that govern the affected module
3. The OpenAPI section for any endpoint touched (`docs/api/openapi.yaml`)
4. Existing tests in the affected module — list test file paths and the behaviours they cover

Per item, output: source path · one-paragraph summary of what's relevant · any apparent conflict with the AC.

Refuse to fabricate. If a referenced ADR or PRD section does not exist, say so and ask before coding starts. This is the last Linear-adjacent read until the PR opens.
```

---

## architecture-design

> Step 3.0 — design and approve the architecture for one story before scaffolding. Read-only, codebase-grounded, doc-calibrated, stops at an explicit developer-approval gate.

```
You are a senior software architect designing one story before implementation begins. You are read-only. You ground every recommendation in an actual file or pattern in this repo, you calibrate every external API claim against current documentation (cite the URL), and you stop at an explicit developer-approval gate before any implementation specialist is invoked.

Design the architecture for this story.

## Story
**Identifier:** [e.g., ENG-247]
**Title:** [title]
**Acceptance Criteria:** [Paste all AC]
**PRD section:** [path or deep-link]
**Related ADRs:** [identifiers and paths]
**OpenAPI references:** [paths and section names for any endpoint touched]
**Reference modules to follow:** [paths of existing modules whose patterns the implementation should mirror]

Do, in order:
1. **Ingest the story.** Read the AC, PRD section, and any cited ADRs. If AC is missing, ambiguous, or has fewer than 3 bullets, refuse and ask before designing.
2. **Analyse the existing codebase.** Identify reusable modules, schemas, services, controllers, components, and hooks. Classify every file the plan will touch as **Reusable** / **Modified** / **New**. Name the reference module the implementation will follow — if none exists, surface as a gap for human architect review.
3. **Research latest documentation.** For any framework, library, or language API the design relies on, confirm the current version, deprecations, and recommended patterns via web search. Cite the URL of every external doc consulted.
4. **Produce the architecture plan** in the format below. Omit tables that have no surface area for this story (e.g., omit the real-time table if no real-time channel is touched).
5. **Stop at the approval gate.** Do not recommend invoking any implementation specialist before the developer explicitly approves the plan.

Produce the plan with these sections, in order:

- **Overview** — 1–2 sentences naming the feature and its single most important design decision.
- **Existing Code Analysis** — three lists: **Reusable** (path · one-line why), **Modified** (path · what changes), **New** (path · what it does). Name the reference module the implementation will follow.
- **Backend Architecture**
  - **Schema changes** — `Schema | File | Change | Indexes`
  - **Service layer** — `Service | File | Dependencies | Key Methods`
  - **API endpoints** — `Method | Path | Guard | Request | Response`
  - **Real-time / Async events** — `Channel / Stream | Event | Payload | Direction` (omit if no real-time surface is touched)
  - **Async jobs / workers** — `Queue / Worker | Processor | Job Data | Trigger` (omit if no async surface is touched)
- **Frontend Architecture**
  - **Components** — `Component | Path | Props | State`
  - **Hooks** — `Hook | Returns | Side Effects`
  - **API methods** — `Method | Endpoint | Request | Response`
- **Cross-cutting concerns** — Error handling, Security (auth guards, input validation, sanitization, authorisation boundaries), Performance (caching, pagination, query optimisation, hot-path allocations), Monitoring (metrics, traces, structured log fields).
- **Dependency Map** — module-to-module dependencies introduced or modified, with circular-dependency risks and mitigations.
- **Implementation Order** — numbered steps with complexity per step (simple / moderate / complex) and a one-line "what this step enables" for the next step.
- **Risk Assessment** — `Risk | Impact | Mitigation`
- **Environment Variables** — `Variable | Purpose | Required`
- **New Dependencies** — `Package | Version | Purpose` (with the documentation URL consulted)

Refusal rules:
- If AC count is < 3 or any AC is ambiguous, refuse to design and list the questions the PM or PRD author must answer first.
- If a referenced ADR, PRD section, or OpenAPI section does not exist, refuse and ask — never invent it.
- If the design would require a new architectural pattern not present elsewhere in the repo, a new service boundary, a new datastore, or a new technology choice, stop and surface for human architect review. Per-story design within established patterns is in scope; system-wide architecture is not.
- If the design touches a security-sensitive surface (auth, authorisation boundaries, payment flows, PII handling, secrets, cryptography), flag for explicit human design review before recommending any implementation specialist.
- Do not edit, scaffold, stage, commit, push, or open a PR. You are read-only.

End with the verbatim line: "Do you approve this architecture? Once approved, invoke `frontend-engineer` / `backend-engineer` for implementation."
```

---

## linear-progress-comment

> Step 3.4 — post a progress comment at substantive checkpoints. Not every commit is a checkpoint.

```
Post a progress comment on the active Linear issue. One comment per substantive checkpoint — not per commit, not per Cursor save.

## Diff since last comment
[Paste `git diff <last-comment-sha>..HEAD` or describe]

## Story
**Identifier:** [e.g., ENG-247]
**AC bullets covered so far:** [list]

Compose one comment containing:
- 2-3 line summary of behaviour-level changes since the last checkpoint (not file count)
- AC bullets newly satisfied, by name
- Any blocker or open question (with file/line if applicable)
- Next planned step
- Signature line `_(via Claude Code MCP)_`

Then `create_comment` on the identifier. Output `<identifier> · comment posted` on success.

Refuse to comment if the diff is empty or only formatting. Refuse to comment more than once per hour without explicit confirmation — comment spam drowns the issue.
```

---

## pr-description

> Step 4.1 — open the PR with a Linear-compliant title and body.

```
Draft a PR description that satisfies Linear's git integration and the team's review checklist.

## Story
**Identifier:** [e.g., ENG-247]
**Title:** [Story title]
**AC:** [Paste]

## Diff summary
[Paste `git diff main...HEAD --stat` plus high-level notes]

Produce:
1. **PR title:** `[ENG-XXX] <short imperative — under 70 chars>`. The bracketed identifier is non-negotiable; Linear's git integration depends on it.
2. **PR body** with these sections, in order:
   - **Summary** (3-5 lines: what and why)
   - **Closes** line: `Closes ENG-XXX` (single-issue) or `Part of ENG-XXX` (multi-PR — manual close)
   - **Approach** (3-5 lines; cite the reference module if one was followed)
   - **AC checklist** (one bullet per AC, each with the test that proves it)
   - **AI-generated sections** (file paths and a one-line note per section — for reviewer focus)
   - **Notes for reviewer** (deferred Mediums from self-review, follow-up `tech-debt` issues filed)

Refuse to draft if the identifier is missing or the diff against main is empty. The PR title must contain the exact identifier from the working branch — do not let the developer typo `ENG-274` into a PR for `ENG-247`.
```

---

## refactor-candidates

> Step 5.1 — scan a module for behaviour-preserving refactor candidates worth filing as `tech-debt` issues.

```
Scan a module for refactor candidates. Read-only — identify and prioritise candidates the developer can file as `tech-debt` Linear issues. Do NOT refactor here.

## Scope
[Path or paths to scan]

## SonarQube findings (optional)
[Paste relevant findings]

Per candidate, output:
- One-line description (extract method, split god class, deduplicate pattern, simplify long conditional, replace bespoke util with stdlib, etc.)
- File path(s) and approximate line count
- **Behaviour-preservation risk** (Low / Medium / High — High means thin test coverage)
- **Estimated effort** (S/M/L)
- **Why now vs defer** (one line)

Rank by `(value / effort) * (1 - risk)`. Surface the top 5.

Do NOT invent issues not in the code. Do NOT propose changes that alter behaviour — those are features, file as feature stories. End with a one-line recommendation for the top candidate ("File as `tech-debt`, route to `refactor-specialist`").
```

---

## runbook-generation

> Step 6.4 — draft an on-call runbook from code + observability dashboards. SRE reviews before publishing.

```
Draft an operational runbook for an on-call engineer paged at 3am who has not seen this service before. Optimise for "what do I do right now" — not for explanation.

## Service / module
[Name and path]

## Inputs
- **Code paths:** [Entry points, error paths, retry/circuit-breaker logic]
- **Observability:** [Dashboards, log queries, alert names — paste links]
- **Known incidents:** [Past Linear/Jira incident links if available]

Sections, in order:
1. **What this service does** (3 lines max)
2. **Recent alerts that page on-call** (alert name → likely cause → first action)
3. **Diagnostics** (exact log query, dashboard link, CLI command — copy-pasteable)
4. **Common failure modes** (symptom → root cause → mitigation, ranked by frequency)
5. **Escalation** (who to page, when, with what evidence)
6. **Rollback** (exact command and verification step)
7. **Related runbooks** (links)

Terse. Ban filler. Omit empty sections. Refuse to invent dashboards, log queries, or alert names not in the inputs — a fabricated runbook is worse than no runbook at 3am.

Output as Markdown ready to commit to `docs/runbooks/<service>.md`. SRE reviews before merging.
```
