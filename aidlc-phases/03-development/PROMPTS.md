# Phase 3: Development — Prompt Templates

> **All prompts are for Claude or Claude Code.** Run them in Claude chat or Claude Code terminal. Cursor handles autocomplete and Composer natively — no prompting needed for those workflows. Prompts that orchestrate Linear writes assume the Linear MCP server is connected (see PROCESS.md Step 0b). Prompts that author or refine Claude Code Skills assume Step 0a is in progress or maintenance — they produce or update files under `.claude/skills/<skill-name>/`.

---

## skill-bootstrap

> Step 0a.2 — author a new Claude Code Skill from scratch. Output is a `SKILL.md` ready to commit, plus any supporting files the skill references.

```
You are authoring a Claude Code Skill for this repo. A skill is a project-scoped, lazy-loaded instruction bundle under `.claude/skills/<skill-name>/` that Claude auto-invokes when the developer's prompt matches the skill's `description`. Skills must be *operational* (exact commands, file paths, refusal rules) — not descriptive.

Bootstrap a new skill.

## Skill spec
- **Name:** [lowercase-kebab-case, e.g., `lint-and-format`]
- **Intended trigger (when should this fire?):** [Concrete developer-prompt phrases — e.g., "lint", "format", "type-check", "clean up this file"]
- **Inputs the skill needs (paths, commands, conventions):** [List the exact lint/format/test commands, the reference module path, the template files, the schema paths — anything the skill body must reference]
- **Tool surface the skill needs (subset of [Bash, Read, Edit, Write, Grep, Glob, WebSearch, WebFetch]):** [Be conservative — narrow tools where possible]
- **Output shape the developer expects:** [E.g., "a fix-first ordered list of findings", "a generated file at the right path", "a checklist with each item pass/fail"]
- **Out-of-scope refusals:** [What this skill must NOT do — e.g., "do not edit committed migration files", "do not run a full build, only the test command"]

## Produce, in order

1. **The skill directory layout** as a tree, starting at `.claude/skills/<skill-name>/`. List `SKILL.md` plus any supporting files (templates, checklists, reference docs) the body will reference.

2. **`SKILL.md` content** as a fenced markdown block ready to paste, with:
   - YAML frontmatter: `name` (kebab-case, matches directory), `description` (the auto-invocation hint — see rules below), optional `allowed-tools` (declared as a YAML array).
   - Body sections, in order: **When to use** (concrete trigger phrasing), **Inputs** (what the skill expects to find or be told), **Steps** (numbered, with the exact commands or operations), **Output format** (what the skill returns to the developer), **Refusal rules** (out-of-scope behaviour — refuse and redirect).

3. **Supporting files** (if any) — for each one referenced in `SKILL.md`, produce the file content as a separate fenced block with its path as a header.

## Rules for the `description` field
- Name **when** to use the skill in concrete developer-prompt terms, not only what it does. Bad: "Manages linting." Good: "Use when the developer asks to lint, format, type-check, or 'clean up' a file or diff in this repo."
- Under ~200 characters.
- Must not overlap with another skill's trigger phrasing — if it would, surface the collision and ask whether to merge or split before generating.
- Must contain at least one verb-phrase a developer would actually say out loud.

## Refusal
- If the intended trigger is vague ("does some testing"), refuse and ask the developer to name the developer-prompt phrases the skill should fire on.
- If the inputs are missing (no actual lint command, no reference module path), refuse — a skill body must be operational, not aspirational.
- Do NOT introduce new project conventions inside the skill body. If you find that the requested behaviour is not yet established in the codebase, surface that gap and ask whether to file an ADR first.

End with: the proposed directory layout, the skill file contents, the supporting files, and a single-line smoke-test prompt the developer should run through [`skill-smoke-test`](#skill-smoke-test) next.
```

---

## skill-trigger-refine

> Step 0a.3 — critique a skill's `description` field for vagueness, overlap, and missing trigger phrases. The single highest-leverage edit on a skill.

```
You are critiquing the trigger description of a Claude Code Skill. The `description` is the only field Claude reads to decide whether to auto-invoke the skill — vague descriptions under-fire, broad descriptions over-fire, overlapping descriptions race with each other. Be surgical.

## Skill under review
**Name:** [skill name]
**Current description:** [Paste]
**Skill body summary (1-2 lines):** [Paste]

## Other skills in this repo (for collision check)
[Paste the `name: description:` pair of every other skill in `.claude/skills/`]

## Output, in order

1. **Verb-phrase audit.** List every concrete developer-prompt verb-phrase the current description names. If the count is zero, the trigger will under-fire — flag it.

2. **Vagueness check.** Identify any phrases that describe what the skill *is* rather than *when* it fires (e.g., "manages X", "handles Y", "is responsible for Z") — these are noise; the description should be 100% trigger-oriented.

3. **Overlap check.** For each other skill listed, identify any developer-prompt phrasing that both descriptions would match. If overlap exists, recommend either merging the skills, narrowing one description, or adding an explicit "do NOT use when..." clause.

4. **Missing-trigger check.** Name developer-prompt phrases the skill body clearly handles but the description does not advertise. These are silent under-fires.

5. **Length check.** Flag if the description exceeds ~200 characters.

6. **Proposed rewrite.** Produce a revised description that addresses every finding above, under 200 characters, with at least one verb-phrase a developer would say out loud, and zero collision with other skills.

End with: severity-ranked findings (Critical / Major / Minor), the proposed rewrite, and a recommendation to re-run [`skill-smoke-test`](#skill-smoke-test) after editing.
```

---

## skill-from-conventions

> Step 0a.4 — extract a skill from patterns already present in the repo, instead of authoring from memory of the style guide.

```
You are authoring a Claude Code Skill that codifies a convention already established in this repo's code. The skill body must reference the actual patterns in use — not a paraphrase of the style guide, not what the guide *says* should be true. Where guide and code disagree, the code wins (and a follow-up Linear `tech-debt` issue is filed by the developer).

Build a skill from the existing repo's conventions.

## Skill spec
- **Name:** [e.g., `component-naming`, `controller-thin-service-fat`, `openapi-contract`]
- **Convention to codify:** [E.g., "all React components in PascalCase, one component per file, file name matches export name"]
- **Where to scan:** [Paths to look at — e.g., `apps/web/src/components`, `apps/api/src/controllers`]
- **Sample size:** [How many files to scan to ground the pattern — at least 10 if available]

## Do, in order

1. **Scan the named paths.** Use Glob and Grep to enumerate the relevant files; Read enough of them to ground the pattern. Capture concrete examples (file path + the actual pattern observed).

2. **Classify findings.** Distinguish:
   - **Dominant pattern** (≥ 80% of sampled files) — this is the convention.
   - **Variant patterns** (any non-dominant) — flag as drift; the skill enforces the dominant pattern.
   - **Outright exceptions** — flag with their file path; the developer decides whether to grandfather or correct.

3. **Author `SKILL.md`** using the same shape as [`skill-bootstrap`](#skill-bootstrap), with these additions:
   - The body cites 2-3 actual files from the repo as **canonical examples** ("See `apps/web/src/components/Button/Button.tsx` for the canonical shape").
   - The body lists the variant patterns and exceptions found, and instructs Claude to flag them when encountered rather than silently rewriting them.
   - The trigger description names the developer-prompt phrases that should fire the skill (e.g., "Use when the developer adds a new React component, renames a component, or asks 'is this component shape right'").

4. **Surface drift.** End with a "Drift report" section in the response (not in the skill itself) listing variant patterns and exceptions — the developer can file `tech-debt` Linear issues for these.

## Refusal
- If the dominant pattern threshold (≥ 80%) is not met, refuse to produce the skill — the convention is not actually established, and codifying a 60/40 split would freeze drift into the skill catalogue. Tell the developer to either pick a side via an ADR first or scope the skill more narrowly to a sub-tree where the pattern is dominant.

End with: directory layout, `SKILL.md` contents, the drift report, and the smoke-test prompt to feed into [`skill-smoke-test`](#skill-smoke-test).
```

---

## skill-smoke-test

> Step 0a.5 — verify a skill triggers on its intended prompt and does not trigger on a near-miss. Run this for every authored or edited skill before commit.

```
You are smoke-testing a Claude Code Skill before it is committed. A passing smoke test means the skill loaded on the trigger prompt, executed the documented instructions, returned the expected shape of output, AND stayed silent on the near-miss prompt. Anything else is a failing smoke test.

Smoke-test this skill.

## Skill under test
**Name:** [skill name]
**Description:** [Paste]
**Skill body (1-2 line summary):** [Paste]

## Trigger prompt (skill SHOULD fire)
[A concrete developer prompt — e.g., "lint the files I just changed"]

## Near-miss prompt (skill should NOT fire)
[A concrete developer prompt that is adjacent but out of scope — e.g., "what does this lint error mean"]

## Do, in order

1. **Predict.** Before invocation, predict from the description alone whether the trigger prompt should fire the skill and whether the near-miss should not. Surface the prediction so it can be compared to the actual outcome.

2. **Run the trigger prompt** in a Claude Code session with the skill loaded. Capture: did the skill fire? Did the body execute? Did the output match the expected shape?

3. **Run the near-miss prompt** in a Claude Code session with the skill loaded. Capture: did the skill fire? (It should not.)

4. **Diagnose any mismatch:**
   - Trigger did not fire → description under-fires. Re-run [`skill-trigger-refine`](#skill-trigger-refine).
   - Near-miss fired → description over-fires. Re-run [`skill-trigger-refine`](#skill-trigger-refine) with an explicit "do NOT use when..." clause.
   - Trigger fired but body produced wrong output → body is broken. Edit the skill body and re-test.
   - Trigger fired, near-miss did not, body produced expected output → smoke test passes.

5. **Capture the result** as a short verdict the developer pastes into the commit message:
   `Smoke test: trigger="<trigger>" fired=yes/no, near-miss="<near-miss>" fired=yes/no, output shape=ok/wrong, verdict=pass/fail.`

## Refusal
- Refuse to mark a skill "ready to commit" until both the trigger and near-miss outcomes match the prediction. A skill with an unverified trigger is worse than no skill — it silently degrades every story it auto-invokes on.
```

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

> Step 2.1 — fetch the developer's next assigned story. Also the smoke test for Step 0b setup.

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

> Step 3.0 — produce a codebase-grounded architecture plan for a non-trivial story before any implementation specialist is invoked. Read-only; ends with an explicit developer-approval gate.

```
You are a senior software architect designing the technical plan for one story before implementation begins. Read-only — you do not scaffold, edit, stage, or commit. Ground every recommendation in an actual file or pattern in the repo, and verify any third-party API shape against the latest documentation before naming it.

Design the architecture for [feature name / Linear identifier].

## Story
**Identifier:** [e.g., ENG-247]
**Title / AC:** [Paste user story + acceptance criteria]

## Context
- **PRD section:** [path under `docs/prd/` or deep-link]
- **ADRs to honour:** [list paths or "none"]
- **OpenAPI section:** [`docs/api/openapi.yaml` anchor — for endpoints touched]
- **Reference modules to follow:** [name 1-3 similar existing modules; if none, surface the gap]
- **Tech stack:** [backend framework + ORM + datastore; frontend framework + component library; real-time, async-job, auth, observability stacks as applicable]

## Existing-code analysis (do this first, before designing)
Use Grep / Glob / Read to classify every file the plan will touch as one of:
- **Reusable** — used as-is, no change
- **Modified** — extended with the change clearly bounded
- **New** — created by this story

Output three lists with one-line "why" per entry. Name the reference module the implementation will follow.

## Latest-docs check (when third-party libraries or framework primitives are involved)
Use WebSearch / WebFetch to confirm current API shapes, deprecations, and recommended patterns for any library named in the plan or proposed for introduction / upgrade. Cite every URL consulted in the plan footer — the developer must be able to verify your sources. Never design against half-remembered API shapes.

## Plan structure (omit sections that do not apply)

1. **Overview** (1-2 sentences — what this story does, in implementation terms)

2. **Backend architecture**
   - **Schema changes** — table: `Entity | Field | Type | Change (add/modify/drop) | Index | Migration notes`
   - **Service layer** — table: `Service | Method | Inputs | Outputs | Side effects | Reference module`
   - **Endpoints** — table: `Method | Path | Auth | Request shape | Response shape | OpenAPI section`
   - **Real-time events / streams** (if applicable) — table: `Channel / topic | Event | Payload | Producer | Consumer`
   - **Async jobs / workers** (if applicable) — table: `Queue / job name | Trigger | Payload | Retry policy | Idempotency key`

3. **Frontend architecture**
   - **Components** — table: `Component | Path | Type (page / container / presentational) | Props | State | Reference component`
   - **Hooks** — table: `Hook | Path | Purpose | Returns | Dependencies`
   - **API methods** — table: `Method | Endpoint | Request | Response | Error states surfaced to UI`

4. **Cross-cutting concerns**
   - **Error handling** — boundary at which errors are caught, mapped, surfaced; user-facing vs operator-facing
   - **Security** — authn / authz at the relevant boundary, input validation surface, PII handling, secrets via the project's standard mechanism
   - **Performance** — N+1 risk, hot-path allocations, index coverage for the new queries, caching decisions
   - **Monitoring** — log lines added with structured context, metrics emitted, dashboard / alert impact

5. **Dependency map** — directed list of "X depends on Y" so the implementation order can be derived

6. **Implementation order** — numbered list, each step naming the file(s) and the complexity (S / M / L). Order should respect the dependency map and let the developer commit incrementally.

7. **Risk assessment** — table: `Risk | Likelihood (L/M/H) | Impact (L/M/H) | Mitigation`

8. **Environment variables** — table: `Name | Purpose | Default | Required (yes/no) | Where set (config / secret store)`. Empty if none.

9. **New dependencies** — table: `Package | Version | Purpose | Bundle / runtime impact | Latest-docs URL consulted`. Empty if none.

10. **Sources consulted** — bullet list of every external URL fetched (framework docs, library docs, RFCs).

## Refusal rules
- If AC has fewer than 3 bullets or is ambiguous, refuse to design and list the questions the developer must answer first.
- If no reference module exists for the feature's shape, surface the gap and ask whether this is a new pattern (which would escalate to a Phase 2 system-architect-style review) before continuing.
- If the design requires a new architectural pattern, a new service boundary, a new datastore, or a new technology choice, stop and recommend a Phase 2 review — Phase 3 architecture is per-story design within Phase 2's established patterns.
- If the design touches authentication, authorisation, payments, PII, or cryptography, flag for explicit human design review before recommending any implementation specialist.
- Do not invent files, schemas, endpoints, or dependencies. Every name in the plan must either exist in the repo or be marked **New** with a reference for its shape.

## End the plan with the approval gate (verbatim)
> Do you approve this architecture? Once approved, invoke `frontend-engineer` / `backend-engineer` for implementation.

Wait for approval. On approval, name the implementation specialist(s) the developer should invoke. On pushback, iterate — re-read the relevant code, re-research the relevant docs, and re-issue the plan. Do not collapse to "just trust me" or hand off without approval.
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
