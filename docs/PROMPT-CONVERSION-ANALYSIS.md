# Prompt → Skill / Subagent Conversion Analysis

> **What this is.** An assessment of all ~100 prompts across the seven `aidlc-phases/*/PROMPTS.md` files, classifying each one as a candidate Claude Code **skill**, **subagent**, **slash command**, or as a prompt that should stay exactly as it is.
>
> **Status:** analysis only. No skills, subagents, or commands have been authored. Building them is a separate decision — see [Build order](#build-order).

>
> **Method.** Seven per-phase classification passes against the framework's own doctrine at [`03-development/PROCESS.md` §"Creating your own Claude Code skill"](../aidlc-phases/03-development/PROCESS.md) (L312-401), a cross-phase synthesis pass to collapse duplicates, and two adversarial review passes — one on routing quality, one on operational cost. 42 initial proposals survived down to 30.
>
> **Revision — Sentry removed from the framework.** Two prompts are deleted (`sentry-context-pull`, `Inherited Error Triage`), taking the corpus from 100 to **98**. The proposed `diagnose-production-bug` and `triage-inherited-errors` skills die with them; `reproduce-and-diagnose-bug` and `load-test-engineer` are added in Phase 4. Artifact count stays 30. See [Sentry removal — consequences](#sentry-removal--consequences).
>
> **Revision — third-party security tooling removed.** The prescribed security stack narrows to **GitGuardian + ggshield plus GitHub-native tooling only**, and **CodeQL is promoted from fallback to the prescribed SAST baseline**. Three Phase 5 prompts are deleted (`Reachability Triage`, `Container CVE Triage`, `OPA Policy Generation`), taking the corpus from 98 to **95**. `Semgrep Custom Rule Generation` is *rewritten* as `CodeQL Custom Query Generation`, so `semgrep-rule-author` is renamed **`codeql-query-author`** and survives intact; `author-opa-policy` and `dependency-risk-analyst` die, and `Dependency Upgrade Impact` reverts to a kept-as-prompt. Artifact count 30 → **28**. See [Security tooling removal — consequences](#security-tooling-removal--consequences).
>
> **Correction — one kill reversed, one gap closed.** Two corrections to the revision above, both in Phase 5. **(1) `Reachability Triage` was deleted in error and is restored.** The kill rested on a false premise: the prompt's own header note read *"use when Endor Labs / Snyk reachability is **unavailable** and you need a triage cut on a long CVE list"* — it was the **no-tool fallback**, not a tool-dependent prompt, and the reachability reasoning it prescribes was always Claude's job. Re-sourced on Dependabot, it comes back, and **`dependency-risk-analyst` is un-killed** wrapping it plus `Dependency Upgrade Impact` (which moves back from kept-as-prompt). `Container CVE Triage` stays deleted — that verdict was correct and is unchanged. **(2) A new subagent, `threat-model-verifier`, closes a gate defect**: two Phase Handoff checkboxes assert that P0 threat-model mitigations landed in code, and nothing verified them. It wraps a **newly authored** prompt, `Threat-Model Mitigation Verification`. Corpus 95 → **97**; artifacts 28 → **30**. See [Security tooling removal — consequences](#security-tooling-removal--consequences).
>
> **Revision — Phase 1's three P0 skills are now BUILT.** `publish-prd-to-linear`, `scaffold-linear-milestones` and `push-linear-stories` ship as templates under `aidlc-phases/01-requirement-gathering/skill-prompts/`, wired into that phase's PROCESS Step 0, Steps 2.2-2.4 and Stages 3a-3c, and into Gates 1 and 2 with a committed-and-smoke-tested checkbox each. **No prompt was deleted, renamed or rewritten** — the paste path stays valid, so this revision changes no counts: still 96 prompts and 34 artifacts, with **18 built rather than 15**. `sweep-requirements-gaps` is deliberately not built; its own stated condition is that these three run on a real PRD first. **Two deferred decisions had their triggers fire on this build** — see [Build order](#build-order) Phase 0 item 3. See [Phase 1 build — consequences](#phase-1-build--consequences).
>
> **Revision — Pulumi removed (framework v2.4), and Phase 6 is now BUILT.** Pulumi leaves the prescribed framework entirely; Terraform becomes the IaC primary with OpenTofu as the CLI-compatible fallback. One prompt is **deleted** (`Pulumi Cost Delta` — Infracost supports Terraform natively, so its own header stated its obsolescence condition), taking the corpus from 97 to **96**, and one is renamed and rewritten (`Pulumi IaC Generation` → `Terraform IaC Generation`, anchor `#pulumi-iac-generation` → `#terraform-iac-generation`). Phase 6's artifacts grow **4 → 8** and move from *proposed* to *shipped*: `pulumi-iac-engineer` becomes **`terraform-iac-engineer`**, the proposed single `iac-foundation-bringup` **splits** into `iac-state-backend-bringup` + `ci-identity-and-secrets-bringup`, and `cost-guardrails-bringup` + `deploy-and-rollback-bringup` are added so no capability removed with Pulumi silently drops. Artifacts 30 → **34**. `observability-bringup` is **unblocked** — the `/docs/runbooks/` conflict is resolved. See [Pulumi removal — consequences](#pulumi-removal--consequences).

---

## Verdict

**55 of 96 prompts convert into 34 artifacts** — 20 skills, 11 subagents, 3 slash commands. **41 stay exactly as they are.**

**18 of the 34 are built.** Phase 3's seven specialists shipped first; Phase 6's eight shipped alongside the v2.4 Pulumi removal — see [Pulumi removal — consequences](#pulumi-removal--consequences) for why a vendor migration turned out to be the right moment to build a phase's helpers — and Phase 1's three P0 skills shipped next, as the first artifacts built in this document's own recommended order.

The governing insight came from the adversarial review. [PROCESS.md L376](../aidlc-phases/03-development/PROCESS.md) establishes that a skill's description **is the entire routing surface** — skills *auto-trigger* on user intent, while subagents are *name-invoked*. It follows that any skill whose trigger list reuses a verb one of the seven shipped Phase 3 subagents already claims **wins that route by default and hijacks ordinary work**. That single rule killed or narrowed six proposals that otherwise looked sound, and it is the main reason the conversion rate is 58% rather than 90%.

The second-largest constraint is input availability. Roughly a quarter of the prompts consume a blob no tool in the mandated stack can fetch — a meeting transcript, a vendor pricing sheet, a billing export, a dashboard reading. Wrapping those in a gated artifact does not remove toil; it makes fabricated figures look gate-approved.

### Coverage at a glance

| Phase | Prompts | → Skill | → Subagent | → Slash cmd | Kept as prompt |
|---|---|---|---|---|---|
| 1 — Requirement Gathering | 12 | 9 *(8 of them via 3 shipped skills)* | 0 | 0 | 3 |
| 2 — System & Architecture Design | 15 | 6 | 4 | 1 | 4 |
| 3 — Development | 18 | 6 | 0 *(7 already shipped)* | 2 | 10 |
| 4 — Testing & QA | 14 | 5 | 4 | 0 | 5 |
| 5 — Security & Compliance | 13 | 2 | 5 | 0 | 6 |
| 6 — CI/CD & DevOps | 15 | 6 *(all shipped)* | 2 *(both shipped)* | 0 | 7 |
| 7 — Delivery & Handoff | 9 | 2 | 2 | 0 | 5 |
| **TOTAL** | **96** | **35 → 20 skills** | **17 → 11 subagents** | **3 → 3 commands** | **41** |

Grouping is where the value sits: 35 prompts collapse into 20 skills and 17 into 11 subagents. A skill that wraps four prompts is worth far more than four separate skills.

Priority split: **P0 = 4 remaining** (`open-pull-request`, `run-sprint-planning`, `e2e-and-coverage-engineer`, `/adr` — Phase 1's three P0 skills have shipped), **P1 = 15**, **P2 = 8** — plus Phase 6's **8 shipped**, which left the priority queue by shipping out of order (see [Pulumi removal — consequences](#pulumi-removal--consequences)).

**Phase 6's skill count is 6, not 2, and only three of the six wrap a prompt at all.** `iac-state-backend-bringup`, `ci-identity-and-secrets-bringup`, `cost-guardrails-bringup` and `deploy-and-rollback-bringup` wrap **prose PROCESS steps**, not `PROMPTS.md` entries — which is why the phase's skill count exceeds what a prompt-by-prompt reading would predict. Worth noting as a limit of this document's method: **a conversion analysis that only reads the prompt library will systematically under-count the skills a phase needs**, because the procedures most worth encoding are often the ones nobody wrote a prompt for.

### Directory convention

Siblings to the existing `subagent-prompts/`, per phase:

```
aidlc-phases/<phase>/
  PROCESS.md  PROMPTS.md  QUALITY-GATES.md  FLOWCHART.md
  subagent-prompts/<name>.md          # Phases 3 and 6
  skill-prompts/<name>/SKILL.md       # Phases 6 and 1 — folders, not files
  command-prompts/<name>.md           # still proposed; no slash command has shipped
```

Like `subagent-prompts/`, these are **templates, not active artifacts** — Claude Code only auto-discovers `.claude/skills/` and `.claude/agents/`. Each directory gets a README mirroring the Phase 3 one, and templates stay repo-agnostic with `{{UPPER_SNAKE_CASE}}` placeholders so they can be copied into consuming repos.

---

## Recommended new artifacts

### Skills

#### `open-pull-request` — **P0** · Phase 3

- **Wraps:** Self-Review Before PR, pr-description
- **Path:** `aidlc-phases/03-development/skill-prompts/open-pull-request/SKILL.md`
- **Gate:** P3 Gate 2 (PR Merge), pre-merge half
- **Description:**
  > Use when a developer has finished implementing a story and is ready to open the PR: runs the pre-PR self-review, requires every Critical/High to be fixed first, walks the per-story Definition of Done, rebases onto current main, and only then opens the PR with the Linear identifier in the title and `Closes ENG-XXX` in the body. Triggers on "I'm done, open the PR", "ready to PR", "ship this branch", "raise the PR for ENG-XXX", "can this go up for review". Hands the diff review to `code-reviewer`, fixes to `frontend-engineer` / `backend-engineer`, rebase conflicts to `conflict-resolver`, and the PR write to `linear-task-agent`. Do NOT use for: reviewing an already-open PR, merging, or a refactor branch with no tech-debt issue.
- **Why:** PROCESS.md L330 uses `open-pull-request` verbatim as its exemplar skill name, and L323 routes "open a PR" to a skill. It sequences five steps across three shipped subagents that no single agent owns.

#### `run-sprint-planning` — **P0** · Phase 3

- **Wraps:** linear-sprint-pull, Estimation, estimates-to-linear, story-decomposition
- **Path:** `aidlc-phases/03-development/skill-prompts/run-sprint-planning/SKILL.md`
- **Gate:** P3 Gate 1 (Sprint Commitment)
- **Description:**
  > Use when a Tech Lead or PM is preparing the upcoming sprint/cycle for commitment: pulling the candidate backlog from Linear, sizing each story, posting the estimates back onto the issues, and decomposing anything AI-sized XXL into sub-issues before the Cycle opens. Triggers on "plan the next sprint", "pull the backlog for the cycle", "estimate these stories", "size the backlog", "is this ready to commit". Performs the Linear writes directly — this is outer-loop planning, deliberately outside `linear-task-agent`'s dev-loop remit. Do NOT use for: daily story pickup (`linear-task-agent`), re-estimating a story already In Progress, or opening the Cycle itself.
- **Why:** **Verified in the shipped template** — `linear-task-agent` refuses sprint planning twice: its description says *"sprint-planning prompts run by humans"*, and [line 19](../aidlc-phases/03-development/subagent-prompts/linear-task-agent.md) states *"You do NOT run sprint-planning prompts (sprint pull, estimates roll-up, story decomposition)"*, repeated as a refusal case at line 37. It cannot be a subagent extension and cannot route into the existing Linear writer.

#### `publish-prd-to-linear` — **SHIPPED** · Phase 1

- **Wraps:** PRD Generation, PRD-to-Linear Document, **Gap Analysis (pre-publish self-review half only)**
- **Path:** `aidlc-phases/01-requirement-gathering/skill-prompts/publish-prd-to-linear/SKILL.md`
- **Gate:** P1 Gate 1 (PRD Completeness & Published in Linear)
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** The section-anchor map it returns is the phase's single point of failure — every downstream deep-link at Gates 2 and 3 depends on it. The publish step already encodes a confirm-then-`go` protocol, a state rule, a label rule and a refusal boundary: skill shape, not prompt shape.
- **What changed on build:**
  - **It wraps three prompts, not two.** `Gap Analysis` has two uses on two different inputs — the Step 2.3 self-review of a *local draft*, and the Step 4 sweep of the *published* Document against the backlog. Only the first belongs here, and it belongs here as a **step the skill cannot skip**, because Gate 1's completeness list is exactly what it checks. The doc's own not-converted table already recorded `Gap Analysis` as split across two artifacts; this build is where the split became load-bearing rather than notional. `sweep-requirements-gaps` still owns the other half.
  - **The anchor map is read back, not predicted.** The source prompt returns "the section headings extracted from the PRD" at confirm time, which is a prediction of what Linear *will* produce. The shipped skill re-reads the created Document and extracts the real anchors. A predicted map produces links that resolve to the wrong heading — visibly clickable, silently wrong, and the failure surfaces two gates later.
  - **The framework attribution line was genericised.** The prompt writes `Source: Generated by Claude via MCP — AI-DLC Phase 1` into every Linear artifact. In a shipped template that string names a framework the consuming repo does not have; it becomes "Generated by Claude via the Linear MCP server".

#### `scaffold-linear-milestones` — **SHIPPED** · Phase 1

- **Wraps:** Epic Decomposition, PRD-to-Linear Scaffold (Milestones)
- **Path:** `aidlc-phases/01-requirement-gathering/skill-prompts/scaffold-linear-milestones/SKILL.md`
- **Gate:** P1 Gate 2 (Linear scaffolding half)
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** Split out of an original five-prompt `build-linear-backlog` proposal at the human approval gate. Inlining five prompts under the repo-agnostic shipping rule produces ~160 lines of prompt body alone, against a two-screen limit (PROCESS.md L384) and an empirical house ceiling of 111 lines. Terminating at the existing approval stop is the natural, non-arbitrary cut. **The split is vindicated on build:** the two halves came in at 37 and 54 lines, so the merged version would have run to ~85 with two confirm-before-write stops and two human gates inside one procedure — past the limit and past the single-procedure rule, exactly as predicted.
- **What changed on build:**
  - **It gained a two-way coverage check** that neither source prompt has. Every functional-requirement section must be claimed by exactly one epic; sections claimed by none, and sections claimed by two, are both **reported rather than resolved**. This is the cheapest possible place to catch a coverage gap — before it becomes an absent story that Gate 3's per-story review cannot notice, because you cannot review an issue that was never created.
  - **It stops when Milestones already exist** rather than adding a parallel set. The source prompt is silent on the re-run case; a second scaffold on the same Project is the milestone-level analogue of the duplicate-issue risk that PROCESS.md already names as the phase's top risk.
  - **"Never invent a target date"** is a refusal, not a preference. The source prompt says to set dates "if the PRD timeline gives them", which a model reading helpfully will satisfy by interpolating. An invented date is read as a commitment by everyone downstream.

#### `push-linear-stories` — **SHIPPED** · Phase 1

- **Wraps:** User Story Generation, Acceptance Criteria, Stories-to-Linear Push, Linear Context Pull *(absorbed as the duplicate pre-flight)*
- **Path:** `aidlc-phases/01-requirement-gathering/skill-prompts/push-linear-stories/SKILL.md`
- **Gate:** P1 Gate 2 (story-content half); feeds Gate 3
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** Absorbs the killed `/linear-context-pull` command as its read-only duplicate pre-flight, which is where that prompt actually earns its keep — PROCESS.md names duplicate issues as the phase's top risk. A 20-story PRD is currently 20 separate acceptance-criteria pastes.
- **What changed on build:**
  - **"Never clear `needs-human-review`, never move a story to `Backlog`" is now an explicit refusal.** It was implicit in the source prompt's "never set state beyond Triage/Backlog", and implicit is not enough: an agent that has just created twenty issues and been told "great, accept them" will read label removal as tidying rather than as the acceptance gate it is. Gate 3 *is* that label.
  - **Duplicates stay visible in the confirmation table.** The source prompt lists them separately and creates the rest; the shipped skill keeps flagged rows in the same table marked as excluded, so the human sees what is being skipped rather than only what is being made. A silently omitted story looks identical to a story nobody wrote.
  - **Deep-links are verified after creation**, against the anchor map read back by `publish-prd-to-linear`. The refusal case is sharpened accordingly: a story with no valid anchor entry is not created, **and does not fall back to linking the Document root** — the fallback is what turns "no citation" into "a citation that resolves and proves nothing", which is worse, because Gate 3's reviewer clicks it and sees a real page.

#### `render-design-diagrams` — P1 · Phase 2

- **Wraps:** Eraser Architecture Diagram (via MCP), Eraser ER Diagram (via MCP)
- **Path:** `aidlc-phases/02-system-design/skill-prompts/render-design-diagrams/SKILL.md`
- **Gate:** P2 Gates 1, 2, 4 (diagram artifacts and locations)
- **Description:**
  > Use when a system, cloud, sequence, or entity-relationship diagram needs to be produced and landed in the repo — generates it through the Eraser MCP server, commits the DSL under the repo's diagram directory, exports PNG/SVG, generates the equivalent in-repo Mermaid so it renders in markdown, and records the Eraser editor URL. Triggers on "diagram this schema", "generate the C4 container diagram", "ER diagram for the new tables", "render the sequence diagram for checkout", "update the diagrams to match the new design". Do NOT use for: deleting or reorganising an Eraser workspace, designing the architecture itself, or UI mockups.
- **Why:** Runs 7+ times per project and is the only concrete mitigation for the diagram-drift risk PROCESS.md names. Generation is one step; the gate-bearing obligation around it is six, across two surfaces and two PROCESS steps.
- **⚠ Collision:** the trigger "draw the architecture diagram" collides verbatim with a live skill at `roombook/.claude/skills/architecture-diagram`. Deconflict before authoring.

#### `data-model-design` — P1 · Phase 2

- **Wraps:** Entity Extraction, Schema Generation, Migration Generation
- **Path:** `aidlc-phases/02-system-design/skill-prompts/data-model-design/SKILL.md`
- **Gate:** P2 Gate 2 (five checkboxes — the densest Phase 2 coverage)
- **Description:**
  > Use when turning approved PRD requirements into a database schema — extracts entities, ownership and cardinalities from the PRD, generates the schema in the repo's ORM with indexes covering the stated top query patterns, stops for a mandatory backend-developer review, then generates the up/down migration plus seed data and runs it against a local database. Triggers on "design the data model from the PRD", "extract the entities from the PRD", "we need the data model before we build". Do NOT use for: schema changes or migrations inside a story mid-implementation (use `backend-engineer`), denormalising without a measured query and an ADR, running migrations against anything but a local database, or rendering the ER diagram.
- **Why:** Textbook "first, then, then, finally". **Triggers deliberately narrowed:** the original "write the migration for the new tables" / "generate the schema" collided with `backend-engineer`'s shipped trigger "add the migration for ENG-XXX" — and because skills auto-trigger, the skill would have won and dragged a greenfield PRD chain into an ordinary story turn.

#### `api-contract-freeze` — P2 · Phase 2

- **Wraps:** API Contract
- **Path:** `aidlc-phases/02-system-design/skill-prompts/api-contract-freeze/SKILL.md`
- **Gate:** P2 Gate 2 (eleven API lines); metric: 0 contract change-requests after frontend start
- **Description:**
  > Use when an API contract needs to be designed, audited, mocked and frozen before frontend work starts — maps PRD user flows to a resource model, generates an OpenAPI 3.1 spec with the team's error envelope, pagination, auth and idempotency conventions, audits every operation mechanically against the API gate checklist, stands up the mock server and records its URL in the README, then records the freeze and the change-request path. Triggers on "design the API contract", "generate the OpenAPI spec", "stand up the mock server", "audit the spec before we freeze it". Do NOT use for: implementing endpoints, changing an already-frozen spec without the documented change request, or designing the data model behind it.
- **Why:** Wraps one prompt but absorbs four prompt-less PROCESS steps and a real business metric. Deliberately not merged with `data-model-design` — the combined chain is eight steps with two human stops, well past the two-screen limit.

#### `author-test-plan` — P1 · Phase 4

- **Wraps:** Test Plan Generation, test-plan-gap-analysis, test-plan-to-linear-document
- **Path:** `aidlc-phases/04-testing-and-qa/skill-prompts/author-test-plan/SKILL.md`
- **Gate:** P4 Gate 1 (Test Plan Approved)
- **Description:**
  > Use when a QA Lead or Tech Lead needs the test plan for a project or release produced and published. One procedure: fetch the PRD and architecture via Linear MCP rather than pasting, draft the plan with an AC-to-test map, gap-check it against the source ACs and NFRs, resolve every Critical/High, then publish it as a Linear Document with stable section anchors. Entering at "check the test plan for gaps" or "publish the test plan to Linear" resumes the procedure at that step; publication is gated on the gap check clearing. Triggers on "write the test plan", "what's our test strategy", "do we have a test for every AC". Do NOT use for: writing test code, auditing coverage of an existing suite, or silently revising an already-published plan.
- **Why:** The third prompt refuses to run without the second's clearance — hard-gated steps must not be advertised as alternatives (PROCESS.md L401). The "fetch via MCP rather than paste" instruction is what defeats the context-hungry objection.

#### `reproduce-and-diagnose-bug` — P1 · Phase 4

> Replaces the retired `diagnose-production-bug`. `sentry-context-pull` is deleted; the two remaining prompts are retooled to anchor on the Linear bug report.

- **Wraps:** bug-reproduction, Debugging (Bug Investigation)
- **Path:** `aidlc-phases/04-testing-and-qa/skill-prompts/reproduce-and-diagnose-bug/SKILL.md`
- **Gate:** P4 Gate 3 (Regression L61 and Bug Status L64-66); PROCESS steps 8.3-8.5
- **Description:**
  > Use when working a triaged bug that already has a tracker issue and reproduction steps. One ordered procedure: choose the cheapest reliable test layer and write the failing regression test named after the bug, verifying it fails for the right reason; then rank root-cause hypotheses against that evidence, state the blast radius, and hand the fix to the implementation specialist so the test and the fix ship in one diff. Triggers on "reproduce ENG-512 locally", "write a failing test for this bug", "I can't reproduce this bug", "rank the root causes here". Do NOT use for: triaging or assigning severity to an untriaged bug, picking up or transitioning the issue (use `linear-task-agent`), writing tests for a module or story (use `frontend-engineer` / `backend-engineer`), debugging a failing E2E test in CI (use `e2e-and-coverage-engineer`), or applying the fix.
- **Why:** Its value proposition **inverts** from toil-removal to gate-enforcement — the same basis on which `secret-leak-response` was accepted. The rule it enforces (regression test written *before* the fix, verified to fail for the right reason, shipped in the same diff) is named as a top phase risk twice, at `04-testing-and-qa/PROCESS.md:318` and in the risk table at `:446`, and it is exactly what humans invert under S1 pressure.
- **Mandatory edit when built:** the source prompt refuses when "affected-user segmentation" is missing (`PROMPTS.md:403`). That is unobtainable post-Sentry, so it would refuse on most real bugs. Relax to "stack trace **or** a deterministic reproduction"; demote segmentation to optional.
- **Its own drop condition:** post-Sentry this is 100% procedural discipline and 0% context fetch. If the team already ships regression tests in the fix diff unprompted, do not build it.

#### `load-test-engineer` — P1 · Phase 4 · `sonnet` *(subagent — listed here for adjacency; see Subagents)*

See the [Subagents](#subagents) section.

#### `secret-leak-response` — P1 · Phase 5

- **Wraps:** Secrets Incident Response
- **Path:** `aidlc-phases/05-security/skill-prompts/secret-leak-response/SKILL.md`
- **Gate:** P5 Gate 5 (Secrets Hygiene — sub-one-hour MTTR)
- **Description:**
  > Use the moment a credential is suspected or confirmed leaked — a scanner alert, a key spotted in a diff or a log, a token pasted into a prompt, or an external notification. Drives the rotate-first ordering: rotate with the exact steps for that credential type, verify the replacement in every environment, revoke the old one and confirm it is dead, assess blast radius, audit logs for unauthorised use, notify downstream owners, and only then consider history scrubbing. Produces the runbook and post-mortem skeleton; the human executes console and CLI steps. Triggers on "we leaked an API key", "a token got committed", "the secret scanner just fired", "is this key still live", "do we need to rewrite git history". Do NOT use for: preventative scanning setup, rewriting history before rotation, or deciding a leaked production credential is low-risk.
- **Why:** The clearest case of a conversion earning its keep by *enforcing a gate* rather than removing volume — it fires under a fifteen-minute clock, and the ordering rule it enforces (rotate before scrub) is exactly what people invert under pressure.

#### `codeql-query-author` — P2 · Phase 5

> Renamed from `semgrep-rule-author`. The wrapped prompt was **rewritten**, not deleted — the skill survives with the same shape, gate and verification discipline.

- **Wraps:** CodeQL Custom Query Generation
- **Path:** `aidlc-phases/05-security/skill-prompts/codeql-query-author/SKILL.md`
- **Gate:** P5 Gate 2 (SAST Quality Gate — custom CodeQL queries in `.github/codeql/` pass against the diff)
- **Description:**
  > Use when the team needs a project-specific static-analysis rule so a pattern cannot recur. Drafts the CodeQL query with its `qlpack.yml` entry and suite line so the workflow actually runs it, commits the anti-example and allowed-example fixtures under `.github/codeql/test/`, verifies the query fires on the anti-example and stays silent on the allowed example, tightens the predicate or adds an explicit exclusion if the false-positive rate looks high, and only then emits the committable `.ql` plus a one-paragraph PR note. Triggers on "write a CodeQL query for this", "lint for this pattern", "make sure nobody does this again", "this query is too noisy". Do NOT use for: fixing the vulnerability itself, tuning GitHub's built-in query packs, lowering a severity to hide noise, or committing a query that has not passed both fixtures.
- **Why:** The two-fixture verification is a genuine gate hurried humans skip, and skipping it ships queries that either never fire — manufacturing confidence — or flood CI. Gate 2's escalation clause prescribes exactly this loop: a repeating false positive is fixed *at the query*, re-proved against both fixtures, and committed, never habitually dismissed.
- **Portability improves with the rename:** the prompt is authored in Claude Code against the repo with **no MCP server involved**, where the Semgrep version depended on an MCP server the framework no longer mandates. One less setup precondition in every consuming repo.

#### `iac-state-backend-bringup` — **SHIPPED** · Phase 6

> *New in v2.4. The first half of the split described below.*

- **Wraps:** no prompt — absorbs prose PROCESS steps (project bootstrap, remote state)
- **Path:** `aidlc-phases/06-cicd-devops/skill-prompts/iac-state-backend-bringup/SKILL.md`
- **Gate:** P6 Gate 1 (state + repo layout)
- **Why:** every item it covers came free with the previous vendor's control plane and none comes free now. Three in particular are the ones teams discover during an incident rather than during setup: **state locking** (two concurrent applies corrupt state), **the read/write role split** (a plan running with write credentials is an accident waiting for a typo), and **version lifecycle** (an un-lifecycled versioned bucket grows without bound, and the recovery procedure has nothing to recover *to* if versioning was never on).

#### `ci-identity-and-secrets-bringup` — **SHIPPED** · Phase 6

> *New in v2.4. The second half of the split.*

- **Wraps:** no prompt — absorbs the old ESC-setup step, the CI half of the apply-path step, and drift detection
- **Path:** `aidlc-phases/06-cicd-devops/skill-prompts/ci-identity-and-secrets-bringup/SKILL.md`
- **Gate:** P6 Gate 1 (CI auth + secrets + drift)
- **Why:** **this skill is the compensating control for losing the managed apply runner.** With no external system able to refuse an apply, the only remaining enforcement of "no laptop applies to staging or prod" is that CI holds a role nobody's laptop can assume — which is exactly what OIDC with a branch-and-environment-scoped trust policy buys. Its drift workflow is then the only automated signal that somebody applied out-of-band anyway. Neither exists by default; both are quiet until the day they matter.

> **Why the proposed single `iac-foundation-bringup` became two skills.** Once the managed control plane was replaced by a raw cloud-native backend, one skill would have inherited everything the vendor used to configure — versioning, encryption, deny-public, version lifecycle, the lock, split read/write IAM, OIDC trust, secret-manager wiring, audit coverage — on top of the original root-module and drift work. Drafted honestly that is **11 steps across five surfaces and ~70 lines**, past the two-screen limit and past the point where the single-procedure rule holds.
>
> **The cut is at the storage/identity seam**, and it is non-arbitrary on four counts: the two halves have different reviewers (a cloud or platform admin vs whoever owns CI), different failure modes (data loss vs credential leak), a real ordering dependency (the OIDC trust policy references the bucket and lock table the first skill creates), and a natural terminal verification — drift detection is the first thing that exercises the whole chain end to end, so it belongs at the *end* of the second skill rather than stranded as step 7 of the first, where it could not yet run. Same reasoning that split `build-linear-backlog` at its existing human gate: **terminate at a real seam, not at a line count.** Both halves feed one Gate 1 block, so they ship in the same PR.

#### `cost-guardrails-bringup` — **SHIPPED** · Phase 6

> *New in v2.4. Absorbs the surviving residue of the deleted `Pulumi Cost Delta` prompt.*

- **Wraps:** no generation prompt
- **Path:** `aidlc-phases/06-cicd-devops/skill-prompts/cost-guardrails-bringup/SKILL.md`
- **Gate:** P6 Gate 1 (the four cost checkboxes)
- **Why:** the Infracost gap closed with v2.4, so cost-on-PR became a five-minute wiring job instead of a bespoke LLM pipeline — but Gate 1's other three cost checkboxes (budget alerts, unit economics, the estimate) are still audited by hand weeks later, and the deleted prompt's anomaly checks would otherwise have vanished silently. **This skill is where the Pulumi removal's capability accounting is settled:** the costing goes to a better tool, and the judgement Infracost cannot make — unjustified cost-bearing change, oversize against the stated performance target, missing mandatory tags — becomes a reviewer checklist here and a self-check in `terraform-iac-engineer`.
- **Routing near-miss, resolved:** it produces a *reviewer checklist* and deliberately contains no form of the verb "review" in its triggers. Had it said "review this PR for cost", it would have collided head-on with `code-reviewer`.

#### `cicd-pipeline-bringup` — **SHIPPED** · Phase 6

- **Wraps:** CI/CD Pipeline Generation, Claude Code Action — PR Review Workflow, Agentic Workflow Templates
- **Path:** `aidlc-phases/06-cicd-devops/skill-prompts/cicd-pipeline-bringup/SKILL.md`
- **Gate:** P6 Gate 2 (24 items — densest checklist in the phase)
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** Four Gate 2 items ride on the AI-in-CI workflow alone (exact version pin, fix-on-mention verified, bot-actor gate, spend cap) and are today audited by hand weeks after the YAML was written. Two rules a generator will violate are hard constraints: never make the AI review job a required check; never grant `pull_request_target`.
- **What changed on build:** the proposed trigger list led with **"set up CI/CD for this repo"**, which was the predicted verbatim collision with the `cicd-devops` skills in `demo/` and `roombook/`. The shipped description **does not lead with "CI/CD"** — its triggers are "set up the GitHub Actions workflow for this repo", "generate our build and test workflow", "wire branch protection". That removes the overlap while staying the phrasing a developer actually uses. Any future edit that reintroduces the original trigger re-opens the collision.
- **Deliberate exception to the inline-with-placeholders rule:** the four agentic-workflow templates are committed **by reference** to `PROMPTS.md` rather than inlined, because inlining ~65 lines of YAML would put the skill past the two-screen limit outright. The `skill-prompts/README.md` instantiation step calls this out as a one-time human paste.

#### `observability-bringup` — **SHIPPED** · Phase 6

- **Wraps:** Observability / Dashboard Generation, SLO and Alert Generation, Runbook Generation from Alert
- **Path:** `aidlc-phases/06-cicd-devops/skill-prompts/observability-bringup/SKILL.md`
- **Gate:** P6 Gate 4 — the **Dashboards and Alerts blocks only**
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** Three strictly dependent prompts — SLO targets populate Dashboard 2, alerts derive from SLOs, runbooks derive from alerts — previously ran as three sessions with metric names re-pasted between them, which is exactly where invented metric names enter. The dependency chain is the whole value.
- **✅ Unblocked.** The `/docs/runbooks/` conflict is resolved (see [Repo defects](#repo-defects-found-along-the-way)).
- **Scope correction made on build:** Gate 4's **Instrumentation block is not this skill's**. OpenTelemetry SDK integration, auto-instrumentation, structured logging and trace propagation are application code and belong to the implementation specialists. The skill says so rather than reporting that block as passing — the same honesty property as `container-image-engineer`'s seven-of-nine.

#### `deploy-and-rollback-bringup` — **SHIPPED** · Phase 6

> *New in v2.4. Replaces the managed-runner auto-deploy and vendor-action rollback wiring.*

- **Wraps:** no generation prompt
- **Path:** `aidlc-phases/06-cicd-devops/skill-prompts/deploy-and-rollback-bringup/SKILL.md`
- **Gate:** P6 Gate 5 (8 items)
- **Why:** Gate 5's items are the phase's least-verified, because seven of them are only exercised during a real production deploy and the eighth — the drill — is the one everybody defers. The rollback path decaying untested is named as a phase risk in its own right. **This skill runs the drill during the bring-up rather than scheduling it**, and reports the measured wall-clock time honestly: if it exceeds five minutes, it says the runbook is wrong rather than rounding down.
- **The caveat it is required to record:** Terraform has no previous-revision primitive, so rollback *is* re-applying an earlier commit — which only works while that commit is still applyable. The image-digest rollback is the always-safe half and fires first.

#### `finalise-documentation` — P1 · Phase 7

- **Wraps:** Documentation Generation (per page), Doc Completeness Check
- **Path:** `aidlc-phases/07-delivery-handoff/skill-prompts/finalise-documentation/SKILL.md`
- **Gate:** P7 Gate 2 (Documentation Complete)
- **Description:**
  > Use when finishing the customer-facing documentation set before delivery. One procedure: structure, per-page generation, API-reference render, completeness check, gap resolution, publish — stopping for human accuracy review before every publish. Entering at "check our docs for gaps" resumes at the audit step and loops back into page generation for every Critical/High finding. Triggers on "write the getting-started page", "generate the local dev setup docs", "are the docs complete", "we need docs before handoff". Do NOT use for: module READMEs or inline comments inside a feature story (use `/write-module-readme`), hand-writing API reference pages, or auditing a live knowledge base against production signal.
- **Why:** The completeness check gates publication and its findings re-enter page generation — splitting them breaks the loop that satisfies the gate. Description written as one procedure, not an either/or, so "check our docs for gaps" cannot terminate before the publish gate.

#### `sweep-requirements-gaps` — P2 · Phase 1 · **held, not built**

> **Deliberately left unbuilt when the other three Phase 1 skills shipped**, and its hold condition has been **replaced with a falsifiable one** — the original, *"build it after the Gate 1/2 skills have run on a real PRD"*, cannot be checked by anyone reading the repo, which is the same defect this document spent a whole revision removing from quality gates. Its scope also narrowed on that build: `publish-prd-to-linear` now owns the *pre-publish* half of `Gap Analysis`, leaving this skill the post-publish half only.
>
> **Why holding is right rather than merely cheap.** This skill's whole job is verification, and its four classifications all reduce to one operation — parse the `**PRD section:** [§X.Y](url#anchor)` line out of real issue descriptions and re-resolve it against a Document that Gate 3 has since re-versioned. **A sweep that mis-parses that line returns clean, and a clean sweep is Gate 3's evidence.** Building it from the spec re-runs the predicted-vs-read-back anchor experiment — the defect the Phase 1 build found by inlining against real data — in the one artifact where getting it wrong manufactures a gate pass. Same basis as `author-opa-policy` and `Pulumi Cost Delta`.
>
> **Release when either fires:**
> 1. **Evidence trigger.** A build-evidence note is committed recording one real post-handoff sweep against a Project the three skills built, stating: counts per category; whether the `**PRD section:**` line parsed out of real issue descriptions without hand-editing; whether any deep-link failed to re-resolve after a Document version bump; and wall-clock run time. **Falsifiable by `grep`** — the note is in the tree or it is not.
> 2. **Structural trigger.** Any artifact ships that edits an approved PRD Document after publication. Gate 3's *"every issue's PRD deep-link still resolves"* then has a producer of orphan anchors and no checker, and this is the only proposed artifact that checks it.
>
> **Drop condition:** two recorded sweeps returning zero Critical/High findings and no anchor failures. The prompt is then sufficient, and the skill would be a gate-shaped wrapper around a clean result.
>
> **The lane does not decay while empty.** All three shipped skills already exclude this territory by phrase, so nothing needs re-deconflicting when it is eventually built. What *is* currently unowned is the utterance "diff the backlog against the PRD" — it routes to no artifact and runs ungated. That was weighed and accepted: its output is a comment, which is recoverable, and its one dangerous escalation — filing issues for the gaps — is now closed from two directions by `push-linear-stories`' refusal case and the `linear-task-agent` (b) edit.

- **Wraps:** Gap Analysis *(post-publish half only)*, Linear Gap Sweep
- **Path:** `aidlc-phases/01-requirement-gathering/skill-prompts/sweep-requirements-gaps/SKILL.md`
- **Gate:** P1 Gate 3 (Final Review / Phase Handoff — names the wrapped prompt verbatim)
- **Description:**
  > Use when an approved PRD Linear Document and its Linear backlog need to be diffed before phase handoff — reading the Document and the Project's Milestones and Issues via the Linear MCP server, classifying findings as missing coverage, weak coverage, orphan issues, or out-of-scope drift, prioritising them, re-resolving every issue's PRD deep-link, and posting one consolidated comment on the Linear Project after explicit confirmation. Triggers on "run the gap analysis", "what's missing from the PRD", "diff the backlog against the PRD", "gap sweep before handoff". Do NOT use for: creating issues for the gaps (this comments only), the pre-publish PRD self-review on a local draft, or cost review.
- **Why:** Real gate coverage, but the thinnest Phase 1 skill; build it after the Gate 1/2 skills have run on a real PRD. The comment-only boundary is the point — an over-helpful run that files its own findings bypasses per-story acceptance.

---

### Subagents

#### `load-test-engineer` — P1 · Phase 4 · `sonnet`

- **Wraps:** k6 Load Test Generation
- **Path:** `aidlc-phases/04-testing-and-qa/subagent-prompts/load-test-engineer.md`
- **Gate:** **P4 Gate 3 Performance block — all five checkboxes (`QUALITY-GATES.md:52-56`)**, the largest orphaned block in the phase; also Gate 1 L10 and PROCESS steps 6.1-6.4
- **Description:**
  > Use this agent to build and run the performance test for one service or user journey: turns the stated non-functional requirements plus a supplied traffic model into a k6 script that exercises a full journey rather than hammering one endpoint, wires thresholds directly to the NFR numbers, runs it, reads p95/p99 and error rate against those thresholds, and tunes until the run is clean. Refuses to run without NFR targets cited to their source, refuses to invent a traffic model, and refuses to report a PASS against an environment not declared performance-matched to production. Invoke on "write the load test for checkout", "can we handle 10k concurrent users", "run the k6 script and tune the thresholds", "are we meeting our p95 target". Do NOT invoke for: unit, integration or E2E tests (the implementation specialists and `e2e-and-coverage-engineer` own those), profiling or optimising the code that fails the test, capacity planning or cost modelling, or defining the SLOs themselves (use `observability-bringup`).
- **Why:** The "traffic model is business judgement" objection holds for only **one of four** input blocks — endpoints come from the Phase 2 frozen OpenAPI, performance targets are Phase 1 NFRs, auth is repo-knowable. The deciding argument is shape: as a subagent it is name-invoked and faces **zero** auto-trigger competition, and its author → run → read p95 → tune → re-run loop is the same voluminous multi-turn shape that carried `terraform-iac-engineer`.
- **Routing:** no collision among the other 36. `e2e-and-coverage-engineer` already excludes "load or performance tests" — pre-deconflicted, no coordinated edit needed.
- **Two hard boundaries, or the objection wins:** (1) refuse without both NFR numbers cited to their Phase 1 source and a human-supplied traffic profile; (2) **refuse to report a PASS against staging not declared performance-matched** — this closes the risk `04-testing-and-qa/PROCESS.md:451` names verbatim: *"Load tests pass against an under-provisioned staging environment — green tests, red production."*

#### `e2e-and-coverage-engineer` — **P0** · Phase 4 · `sonnet`

- **Wraps:** Playwright E2E Test Generation, playwright-failure-debug, Test Coverage Gap Analysis
- **Path:** `aidlc-phases/04-testing-and-qa/subagent-prompts/e2e-and-coverage-engineer.md`
- **Gate:** P4 Gate 2 (per-sprint E2E, flakiness <2%, no untracked skips); feeds Gate 3
- **Description:**
  > Use this agent to own end-to-end coverage and the coverage audit for already-shipped code: author Playwright E2E tests for critical journeys with role-based locators and no fixed waits; diagnose a failing Playwright run from its trace file and classify it as app bug / test bug / environment / flake; and run the sprint-end coverage-gap audit producing a risk-weighted, ready-to-file gap list. Writes test files only — never edits production code to make a test pass, never calls the issue tracker. Invoke on "add an E2E test for checkout", "why is this Playwright test failing", "run the coverage audit", "where are our test gaps". Do NOT invoke for: unit or integration tests (the implementation specialists own those in the same commit), load or performance tests, mobile / device-matrix tests, fixing the production bug a failing test uncovered, or any issue-tracker write.
- **Why:** **Narrowed on both reviewers' verdict.** Unit and Integration Test Generation were cut because `backend-engineer`'s shipped description already lists *"write the integration tests"* as a **positive** trigger and `frontend-engineer` lists *"write the component tests"* — a description asserting both "invoke on X" and "do not invoke on X" is not a routing fix. The three surviving prompts fire on intent nothing else claims.
- **⚠ Renamed** to avoid colliding with the live `roombook/.claude/agents/qa-test-engineer.md`, which carries the opposite scope.

#### `solution-architect` — P1 · Phase 2 · `opus`

- **Wraps:** Architecture Proposal, Trade-off Interrogation
- **Path:** `aidlc-phases/02-system-design/subagent-prompts/solution-architect.md`
- **Gate:** P2 Gate 1 (≥2 options, 10x stress-test, no unmitigated SPOF)
- **Description:**
  > Use this agent for the system-level architecture pass at the start of a project or a major new subsystem: produce 2-3 candidate architectures (components, communication, data architecture, infrastructure, bounded contexts, trade-offs at launch and 10x, top risks), stress-test each seriously-considered option at 10x load for first-break point, single points of failure, cost cliffs, state contention and failure isolation, and end with a justified recommendation stopped at an explicit Tech Lead approval gate. Read-only apart from writing the proposal document. Invoke on "propose an architecture for this product", "what are our architecture options", "stress-test option 2 at 10x", "design the system before we pick a stack". Do NOT invoke for: per-story design inside an existing codebase (use `software-architect`), the independent production-readiness review (use `architecture-reviewer`), STRIDE / OWASP threat modelling (use `threat-modeler`), schema work, OpenAPI contracts, or writing the ADR (use `/adr`).
- **Why:** **Not a duplicate** — `software-architect`'s shipped description explicitly escalates *"system-wide architectural decisions / new tech choices / new service boundaries / new datastores"* **out** of its remit. Trade-off Interrogation folds in because PROCESS.md 1.3 permits it as an in-conversation follow-up.

#### `architecture-reviewer` — P1 · Phase 2 · `opus`

- **Wraps:** Design Review
- **Path:** `aidlc-phases/02-system-design/subagent-prompts/architecture-reviewer.md`
- **Gate:** P2 Gate 1 (0 Critical, 0 High open; reviewed by ≥1 senior engineer who is not the author)
- **Description:**
  > Use this agent for the independent pre-build production-readiness review of a chosen system architecture: produce a severity-ranked (Critical / High / Medium / Low) issue list across scalability, security, reliability, operability and cost, each with a concrete recommendation and the PRD section affected, ending in a one-line PASS / PASS WITH FIXES / FAIL verdict plus Critical and High counts. Read-only. Invoke on "review this architecture for production readiness", "is this design ready to build", "design review before the architecture gate". Do NOT invoke for: generating the architecture options (use `solution-architect`), STRIDE / OWASP threat modelling (use `threat-modeler`), reviewing a code diff (use `code-reviewer`), per-story design, WCAG audits, or applying the fixes.
- **Why:** Deliberately **not merged** into `solution-architect` — a same-session follow-up to the proposal cannot honestly satisfy "not the author"; merging would void the gate while appearing to satisfy it. Main failure mode is verdict inflation, so severity must be defined by consequence, not tone.

#### `accessibility-auditor` — P2 · Phase 2 · `opus`

- **Wraps:** Accessibility Review (Claude)
- **Path:** `aidlc-phases/02-system-design/subagent-prompts/accessibility-auditor.md`
- **Gate:** P2 Gate 3 (WCAG 2.1 AA, 0 Critical / 0 High); Gate 4 artifact
- **Description:**
  > Use this agent to audit a component, screen, or page against WCAG 2.1 Level AA: findings as Issue / Severity / Recommendation / WCAG reference across perceivable, operable, understandable, robust and tap-target criteria, ending in a PASS / PASS WITH FIXES / FAIL verdict. Read-only — never patches markup, styles, or ARIA. Invoke on "run an accessibility review on this screen", "is this component WCAG AA", "check keyboard navigation and contrast on the booking flow", "a11y audit before sign-off". Do NOT invoke for: fixing the violations (use `frontend-engineer`), general correctness / security / performance review of a diff (use `code-reviewer`), or visual / brand critique.
- **Why:** No existing or proposed agent covers WCAG, and its input is fetchable from the repo with no human paste. P2 on an honest limitation: a source-only audit cannot compute rendered contrast, so it must state its limits and require an automated axe run rather than asserting a PASS.

#### `threat-modeler` — P1 · Phase 5 · `opus`

- **Wraps:** Threat Model STRIDE, AI Agent Threat Review
- **Path:** `aidlc-phases/05-security/subagent-prompts/threat-modeler.md`
- **Gate:** P5 Gate 1 (Threat Model & Baseline); Gate 6 (AI Agent Security)
- **Description:**
  > Use this agent for a full threat-modelling pass on a system, service, or feature before a release candidate is built: walks every component through STRIDE with OWASP Top 10 and API Top 10 cross-references, and — when the product ships AI or agentic features — through the OWASP LLM Top 10 and Agentic Top 10, emitting Issue / category / severity / affected component / recommendation / priority for every finding, naming the field, the tool and the control rather than generic advice. Refuses to proceed when data classification, compliance scope, trust boundaries, or the agent's tool inventory are blank. Invoke on "threat model this architecture", "STRIDE this service", "is our agent feature safe", "review the AI surface for prompt injection". Do NOT invoke for: fixing a scanner-confirmed finding, triaging dependency CVEs or planning a major upgrade (use `dependency-risk-analyst`), checking whether the P0 mitigations it produced actually landed in code (use `threat-model-verifier`), reviewing a Dockerfile or IaC diff (Gate 4 is a named human reviewer, not an agent), general production-readiness review across scalability / reliability / operability / cost (use `architecture-reviewer`), or filing findings into the tracker.
- **Why:** Two prompts, one verb, one output shape, one commit target — the STRIDE prompt already hands off to the AI review, so grouping turns two invocations into one continuation. Both STOP-and-ask guards must be lifted verbatim into boundaries or the agent invents severity from an empty classification.

#### `dependency-risk-analyst` — P1 · Phase 5 · `opus`

> **Un-killed.** This agent was struck with the security-tooling removal on a premise that does not survive reading the source prompt. Stated plainly: **the earlier kill was wrong.** `Reachability Triage`'s own header note read *"Use when Endor Labs / Snyk reachability is **unavailable** and you need a triage cut on a long CVE list"* — it was the **no-tool fallback**, the thing you reach for *because* there is no reachability engine, not a wrapper around one. Deleting it as "input-orphaned" inverted its purpose. It is restored and re-sourced on Dependabot.

- **Wraps:** Reachability Triage *(restored, re-sourced on Dependabot)*, Dependency Upgrade Impact *(moved back from kept-as-prompt)*
- **Path:** `aidlc-phases/05-security/subagent-prompts/dependency-risk-analyst.md`
- **Gate:** P5 Gate 3 (Dependency Hygiene — 0 Critical CVEs in production dependencies; High CVEs carry documented Tech Lead risk acceptance)
- **Description:**
  > Use this agent to decide what a dependency finding actually costs you: takes the Dependabot alert list and, for each CVE, reads the advisory to identify the vulnerable symbol, locates every import of the package, and traces whether that symbol is reachable from a real entry point — returning reachable / not-reachable / undetermined with the `file:line` evidence for each verdict, so the Tech Lead's risk acceptance rests on a call path rather than a CVSS score. Also sizes a major-version upgrade from its changelog: breaking changes mapped to the call sites that will break, migration order, and rollback. Never edits a manifest, never opens the upgrade PR, and marks a CVE **undetermined** rather than guessing when the call path runs through reflection, dynamic dispatch, or a framework it cannot follow. Invoke on "triage these Dependabot alerts", "is this CVE actually reachable in our code", "what breaks if we go to v5", "which of these 40 CVEs matter". Do NOT invoke for: container-image or base-layer CVEs (nothing in the stack scans image layers — Gate 4 is a named human reviewer), authoring the SAST query that prevents recurrence (use `codeql-query-author`), applying the fix or the upgrade (use `backend-engineer`), threat modelling a design (use `threat-modeler`), or accepting the risk — that is the Tech Lead's signature, not the agent's.
- **Why:** Gate 3's bar is not "zero alerts", it is *zero Critical **in production dependencies**, with documented acceptance for the rest* — and nothing in the prescribed stack tells you which alerts clear that bar. Dependabot reports the CVE and the package; it does not report the call path. Working that out by hand across a long alert list is exactly the toil the framework converts, and it is reasoning, not tooling: read the advisory, find the import, trace to an entry point. **Narrowed to two prompts:** `Container CVE Triage` stays deleted and does not come back — it is hard-bound to `trivy image --format json`, and no GitHub-native tool scans image layers, so its input genuinely never exists.
- **The boundary that makes it honest:** it must be able to return **undetermined**. A reachability call with no call-graph dump is a judgement under uncertainty; an agent that only ever answers reachable/not-reachable will quietly manufacture the second answer, which is the fabrication pattern this document rejects everywhere else. Undetermined must escalate to the Tech Lead as if it were reachable.

#### `threat-model-verifier` — P1 · Phase 5 · `opus`

> **The only artifact in this document that wraps a prompt written for it**, rather than converting one that already existed. Every other entry here groups prompts the framework already ships. This one exists because the gap it closes has no prompt at all — so `Threat-Model Mitigation Verification` is authored new, in `05-security/PROMPTS.md`, alongside the subagent.

- **Wraps:** Threat-Model Mitigation Verification *(newly authored)*
- **Path:** `aidlc-phases/05-security/subagent-prompts/threat-model-verifier.md`
- **Gate:** P5 Gate 7 (`QUALITY-GATES.md:149`) and P5 Phase Handoff (`QUALITY-GATES.md:162`, `PROCESS.md:442`) — both now name this helper as their producer and require `file:line` evidence per mitigation with **0 not landed**. The old *"or backlog"* loophole — the very thing that made the checkbox unfalsifiable — is gone; anything still not landed now carries a dated Tech Lead deferral. Traces back to Gate 1 Step 1.4 (`PROCESS.md:183`), whose loop this closes at Step 1.5
- **Description:**
  > Use this agent to prove that the P0 and P1 mitigations a threat model called for actually exist in the code: takes the threat model's mitigation list plus the diff or the codebase and, for each mitigation, locates the implementing code and cites it as `file:line`, or reports it **not landed**. Returns one row per mitigation — mitigation, threat it answers, status (landed / not landed / partial), evidence — and nothing else. Read-only. Will not mark a mitigation landed on the strength of a comment, a TODO, a test name, or a backlog ticket; only implementing code counts. Refuses when the threat model lists no P0 items, and refuses when it cannot read the diff or the codebase. Invoke on "did the threat-model mitigations land", "verify the P0 mitigations before handoff", "check ADR-014's mitigation is actually implemented", "close out the threat model". Do NOT invoke for: finding new vulnerabilities (use `/security-review`), producing or revising the threat model (use `threat-modeler`), implementing or fixing a missing mitigation (use `backend-engineer` / `frontend-engineer`), triaging dependency CVEs (use `dependency-risk-analyst`), or signing the risk acceptance for a mitigation that did not land.
- **Why:** **Two checkboxes assert that P0 mitigations landed, and nothing in the framework verifies either one.** `threat-modeler` produces the P0 list but never reads code. `/security-review` reads code but has never heard of your threat model — it finds generic vulnerability classes, not *"did the specific control ADR-014 called for get implemented"*. Step 1.4 (`PROCESS.md:183`) says findings *"have to land in the backlog or they don't ship"*, and then no step checks. That is an unfalsifiable checkbox — the precise pattern this whole revision has been eliminating, and the same basis on which `secret-leak-response` and `reproduce-and-diagnose-bug` were accepted: value from **enforcing a gate**, not from saving pastes.
- **Routing:** name-invoked, so no auto-trigger competition. The one pair worth deconflicting is `threat-modeler` ↔ `threat-model-verifier` — same noun, opposite direction of travel (one writes the mitigation list, the other audits it against code). Both descriptions must exclude the other explicitly, or the pair repeats the `solution-architect` / `architecture-reviewer` problem in miniature.
- **Its own drop condition:** if teams already convert P0 mitigations into Phase 3 acceptance criteria at Step 1.4 — which is what that step instructs — the verification is done by the story's own DoD and this agent audits an already-closed loop. Build it only if the handoff checkbox is being ticked without evidence today.

#### `terraform-iac-engineer` — **SHIPPED** · Phase 6 · `sonnet`

> *Was `pulumi-iac-engineer`. Renamed and rescoped when Pulumi left the framework in v2.4.*

- **Wraps:** Terraform IaC Generation (`#terraform-iac-generation`, renamed from `#pulumi-iac-generation`)
- **Path:** `aidlc-phases/06-cicd-devops/subagent-prompts/terraform-iac-engineer.md`
- **Gate:** P6 Gate 1 (Infrastructure Foundation — IaC repo and modules)
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** Wraps one prompt but earns a subagent on shape: the iterate-until-plan-clean loop is a multi-turn tool-calling session, and its output volume would swamp the main context. Terraform is the mandated IaC after v2.4, so this is not vendor-lock dead weight. Regulated teams should override to `opus` per-repo.
- **What changed on build:**
  - **The "and a policy pack" deliverable was dropped.** CrossGuard left with the security tooling in v2.3 and no policy engine replaced it, so a policy-pack output would have targeted nothing.
  - **It gained a `## Hard rules` section above `## Operating boundaries`** — the `conflict-resolver` precedent, and warranted more strongly here than anywhere in Phase 3. With Pulumi Deployments gone there is **no managed runner that can refuse an apply**, so that section is the enforcement rather than a restatement of one. It ships at ~59 lines, over the 32–56 band, deliberately.
  - **A loophole the original spec missed:** a `terraform test` run block with `command = apply` *is* an apply — it provisions real infrastructure. A model reading only "never apply" would still write one, so it is called out explicitly and is the wording of the social-pressure refusal.
  - **Language-native testing did not survive the move to HCL.** The original wrapped "language-native component tests"; the shipped agent runs `terraform test` with `mock_provider` and plan-mode run blocks only, and is required to state what plan-time assertions cannot prove. This is a **capability regression**, recorded as such — the framework previously used infra testing as an argument *for* Pulumi.

#### `container-image-engineer` — **SHIPPED** · Phase 6 · `sonnet`

- **Wraps:** Dockerfile Generation (kept verbatim — zero prompt edits)
- **Path:** `aidlc-phases/06-cicd-devops/subagent-prompts/container-image-engineer.md`
- **Gate:** P6 Gate 3 — **seven of the nine per-image criteria**
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** **Narrowed.** The original `container-platform-engineer` had two "and"s in its first clause, which PROCESS.md resolves mechanically to *split*. Dockerfile generation is universal to any containerised service; the eight-manifest K8s set is conditional on a target most consuming repos lack and reverts to a prompt.
- **⚠ Correction made on build — the most important one in this document.** The spec above said "the eleven per-image criteria", then "the container standards checklist". Gate 3 has **nine** non-Kubernetes criteria, and the agent can self-certify only **seven**. Criterion 8 (image size compared against the previous release) is the exact claim its hard boundary forbids it from asserting, and criterion 9 requires Docker Gordon — a different tool. The shipped agent reports seven verdicts and **names the other two by owner**, enforced by a nine-row table rather than a bullet the model can drift past. Building it to claim nine would have manufactured a gate pass, which is the failure mode this whole revision exists to prevent.
- **Hard boundary, now load-bearing:** never state a CVE count, scan verdict, or image-size comparison. The original spec said *"only the CI scanner can"* — **there is no CI scanner any more.** Trivy and Docker Scout left in v2.3, so this boundary went from "defer to the scanner" to "nothing downstream will contradict a fabricated claim."

#### `handoff-agent` — P2 · Phase 7 · `opus`

- **Wraps:** Handoff Document Generation, Post-Handoff Retrospective
- **Path:** `aidlc-phases/07-delivery-handoff/subagent-prompts/handoff-agent.md`
- **Gate:** P7 Gate 3 (Handoff Package Ready); feeds Gate 5 Retrospective
- **Description:**
  > Use this agent to synthesise one source-cited Markdown deliverable from an engagement's artefacts — a handoff document or a post-handoff retrospective — reading across repos, ADRs, runbooks, coverage and security reports, and the issue tracker. Read-only across every source; writes only the named deliverable file. Every claim carries an inline source citation; missing facts are marked `[VERIFY: ...]` rather than invented; content breaching the customer-facing/internal split is marked `[REDACT-BEFORE-SHARE]`, never silently dropped. Invoke on "generate the handoff document", "synthesise the handoff from our artefacts", "produce the post-handoff retrospective". Do NOT invoke for: publishing to the docs site or knowledge base, any issue-tracker write, seeding KB content or writing KT scripts (those stay paste prompts), the customer documentation site (use `finalise-documentation`), or code review and architecture design.
- **Why:** **Narrowed from four deliverables to two.** The "framework prescribes it" justification does not survive reading the source — P7 PROCESS.md:176 says *"Do not create the file in this task"* and :203 lists it as "(If adopted)". Four deliverables behind an "or" is four jobs; two share one honest responsibility.

---

### Slash commands

#### `/adr` — **P0** · Phase 2

- **Wraps:** Architecture Decision Record Generation
- **Path:** `aidlc-phases/02-system-design/command-prompts/adr.md`
- **Gate:** P2 Gates 1, 3, 4; also consumed by Phases 5 and 6
- **Description:**
  > Generate an Architecture Decision Record for a decision just made — takes the decision, options considered, constraints and the PRD anchor; reads the repo's ADR directory to allocate the next sequential number; emits Status (always Proposed) / Context / Decision / Consequences (positive, negative, neutral) / Alternatives Considered with a specific rejection reason per alternative / Validation signal; writes it and surfaces the number it picked. Refuses when no alternative was genuinely rejected, and never fabricates a numeric claim in Consequences.
- **Why:** **Highest leverage-per-unit-of-build-cost on the entire list** — deterministic, one argument, no branching, fired 11+ times per project, cited by three gates. It is also the only artifact that faces no auto-trigger competition at all.

#### `/load-task-context` — P2 · Phase 3

- **Wraps:** task-context
- **Path:** `aidlc-phases/03-development/command-prompts/load-task-context.md`
- **Gate:** none directly — feeds P3 Gate 2 "Architecture alignment verified"
- **Description:**
  > Load the working context for a story before any code is written: reads the cited PRD section, the governing ADR(s), the OpenAPI section for every endpoint touched, and the existing tests in the affected module, and reports each as source path, relevance summary, and any conflict with the AC. Never fabricates a missing source — reports "not found" instead. Usage: `/load-task-context ENG-247`
- **Why:** The only part of the story-start choreography no shipped subagent owns (steps 2.1-2.3 already live in `linear-task-agent`). P2 because it is downstream of tracker hygiene — build it after the Phase 1 backlog skills, so the deep-links it reads exist.

#### `/write-module-readme` — P2 · Phase 3

- **Wraps:** Documentation Generation (P3)
- **Path:** `aidlc-phases/03-development/command-prompts/write-module-readme.md`
- **Gate:** P3 Gate 2 (READMEs updated at the documentation threshold); Gate 3
- **Description:**
  > Generate a Module README (overview, architecture, key concepts, usage, configuration, testing, limitations) for the module at the given path, omitting any section with nothing real to say and never hand-writing API reference content that renders from the OpenAPI spec. Usage: `/write-module-readme src/orders`
- **Why:** **Reinstated after review.** It was killed for failing the auto-trigger test, but PROCESS.md L376 scopes that requirement to *skills*; L324 defines a slash command as "one prompt with arguments" with no trigger obligation. Threshold detection is the human's job at the DoD step. Marginal cost is near zero once `/adr` establishes the convention.

---

## Deliberately NOT converted

| Prompt | Phase | Why it stays a prompt |
|---|---|---|
| Feature Scaffolding | 3 | A shipped subagent already embeds it by reference — `frontend-engineer` / `backend-engineer` |
| Test Generation | 3 | Embedded by three shipped subagents; PROMPTS.md is deliberately its single source of truth |
| Unit Test Generation | 4 | `backend-engineer` / `frontend-engineer` claim it by **positive** trigger; the post-hoc/test-first seam is invisible in any user utterance |
| Integration Test Generation | 4 | Same — `backend-engineer.md:3` lists "write the integration tests" as a positive trigger |
| Refactoring | 3 | The execution prompt inside the shipped `refactor-specialist` |
| architecture-design | 3 | The system prompt of the shipped `software-architect` |
| linear-next-task | 3 | `linear-task-agent`'s `find` flow |
| linear-progress-comment | 3 | `linear-task-agent`'s `progress` flow |
| Security Fix Generation | 5 | Overlaps `backend-engineer` almost exactly; its three constraints are three bullets on that agent, not a new artifact |
| ~~Container CVE Triage~~ | 5 | **Deleted, not kept.** Hard-bound to `trivy image --format json`; with no container scanner in the stack the input never exists. Unlike `Reachability Triage`, this one really is input-orphaned — nothing GitHub-native reads image layers |
| ~~OPA Policy Generation~~ | 5 | **Deleted, not kept.** OPA / Gatekeeper / Conftest and Pulumi CrossGuard all left the stack — ConstraintTemplate, Rego, `gator` and `dryrun` have nothing left to target |
| Debugging (P3) | 3 | Refuses to proceed without a pasted stack trace; a description covering "debug this" fires on nearly every request |
| Interview Summary Structuring | 1 | Whole input is a meeting transcript no mandated tool can fetch |
| KT Session Summary | 7 | Raw timestamped transcript; transcript tools have no MCP server in the mandated roster |
| Scalability & Cost Assessment | 1 | Human projections + mandated out-of-band pricing-calculator validation; a gated skill would make invented figures look gate-approved |
| Cost Estimation · Cloud Provider Comparison · Cost Optimisation | 6 | Pricing sheets, billing exports, budget targets, 12-month projections; cloud billing APIs are not a mandated MCP server |
| Security Posture Report · Evidence Compilation · Compliance Checklist | 5 | Leadership metrics read off dashboards; an agent claiming evidence exists for a control produces an audit failure, not a bad document |
| Tech Stack Comparison | 2 | Team skills, prod operating history, budget ceiling and data-residency posture are org knowledge; criteria weights are already fixed |
| Design System Bootstrap | 2 | Pure taste ("products we admire and why"); fires once, greenfield only, already human-gated |
| Bug Triage & Severity Assignment | 4 | PROCESS.md names it *the human gate*; turns on revenue, sprint priorities, release dates. **Strengthened post-Sentry** — its numeric anchors (affected users, events/hour, auto-severity) now have no source, so an agent would be inventing the impact figures it reasons from |
| ~~Pulumi Cost Delta~~ | 6 | ~~Category error — it already *is* CI automation hosted by a workflow file~~ **Superseded: deleted outright in v2.4.** Infracost supports Terraform natively, so the prompt's own stated reason for existing expired. Reason-pattern #3 (fabrication risk) now outranks #4 (already automated) — see [Pulumi removal — consequences](#pulumi-removal--consequences) |
| Release Notes Polish | 7 | Already fully automated in CI; Gate 1 asks for it to remain a documented prompt |
| Kubernetes Manifests Generation | 6 | Conditional on a deploy target most consuming repos lack; dead weight in a shipped template |
| runbook-generation (P3) | 3 | Needs dashboard links, alert names, log queries; no observability MCP mandated in Phase 3 |
| MCP Enforcement Policy | 5 | Needs a browser OAuth click a skill cannot perform; surfaced a Gate 6 scoping inconsistency to fix first |
| AGENTS.md Authoring | 6 | Once per repo; nine human-supplied policy inputs |
| Infra Runbook | 6 | Once per project; DR section has no fetchable source |
| Mobile Test Generation | 4 | **Six of six** input blocks are human config (OS, framework, app type, credentials, device matrix, binary path); PROCESS.md says "skip if not applicable". Same class as the K8s-manifests revert |
| Knowledge Base Seeding · KT Session Script | 7 | Human-authored source material; narrowed out of `handoff-agent` |
| Quarterly KB Completeness Check | 7 | Runs 90+ days post-handoff when MCP grants have not transferred |
| UI Wireframe · Production Component | 2 | Depend on the global `frontend-design` Skill; no supported skill-to-skill hand-off |
| MCP Integration (P3) | 3 | Requires a browser OAuth click |
| post-mortem | 4 | **Re-evaluated on merit and settled.** Fails on *shape*, not convention cost: six input blocks and two refusal rules is not "one prompt with arguments" (PROCESS.md L324). Its two refusals are exactly what `reproduce-and-diagnose-bug` guarantees — fold its checklist in as that skill's terminal step. Two of six input fields also lose their source post-Sentry |
| Pre-Release Self-Review · Self-Review Before Production Deploy | 5, 6 | See the `release-readiness-reviewer` kill below |
| refactor-candidates | 3 | See the `triage-refactor-candidates` kill below |
| Linear Context Pull | 1 | Absorbed as the duplicate pre-flight step inside `push-linear-stories` |

**Four reason patterns dominate.**

1. **A shipped subagent already owns it** — six Phase 3 prompts plus two Phase 4 test prompts. Re-wrapping these is the explicit duplicate reject.
2. **The input is a blob no mandated tool can fetch** — transcripts, pricing sheets, billing exports, dashboard counts, device matrices.
3. **The output's fabrication risk exceeds its toil** — cost ranges, compliance evidence, severity ratings. Wrapping these makes invented figures look gate-approved.
4. **It is already automated elsewhere** — release notes polish and cost-delta are CI payloads, not session prompts.

---

## Killed during review

| Proposed artifact | Killed by | Reason |
|---|---|---|
| `diagnose-production-bug` (skill) | Sentry removal | Its earn-its-keep was *"steps 2 and 3 consume step 1's MCP output"* — delete step 1 and both remaining steps revert to paste-fed, which is reason-pattern #2 for **not** converting. Two of three wrapped prompts survive as `reproduce-and-diagnose-bug`; `sentry-context-pull` is deleted outright |
| `triage-inherited-errors` (skill) | Sentry removal | **Not rewritable.** Nine of twelve CSV columns are error-tracker fields (`SentryIssueURL`, `SeerConfidence`, FirstSeen/LastSeen, EventCount/UsersAffected), and both value-bearing rules are arithmetic over them — the bandwidth cap needs the ranked list, and "never `wontfix` a customer-facing issue with `UsersAffected > 0`" needs a field with **no other source in the mandated stack**. Verified at `07-delivery-handoff/PROMPTS.md:473-487` |
| `author-opa-policy` (skill) | Security-tooling removal | OPA / Gatekeeper / Conftest and Pulumi CrossGuard both left the prescribed stack. Every artifact it produced — the ConstraintTemplate, the Constraint pinned to `dryrun`, the two manifest fixtures, the audit-window promotion plan — targets an engine the framework no longer has. Its source prompt is deleted outright; unlike the Semgrep rule author there is no surviving rewrite target |
| `release-readiness-reviewer` (subagent) | Ops lens | **Fabricated gate anchor — independently verified.** `self-review` / `readiness` return **zero** matches in `06-cicd-devops/QUALITY-GATES.md`; Gate 5 is eight pipeline-mechanics checkboxes. Its 14 checks span five phases' MCP rosters plus seven tools with no MCP server anywhere — half would report UNVERIFIED under an authoritative halt-or-proceed verdict. The *merge finding* is correct and becomes a doc fix. |
| `wireframe-flow` (skill) | Both | No supported hand-off mechanism — `frontend-design` is a globally-enabled official Skill, and skills match the **user's** prompt (L376), not another skill's instruction. Its Gate 3 discipline already lives in `02-system-design/QUALITY-GATES.md:56,99,103`. Stack-locked to React+Tailwind+shadcn by the gate text. |
| `triage-refactor-candidates` (skill) | Routing lens | **Verified payload mismatch.** Its terminal step files through `linear-task-agent`, whose file-followup flow requires a parent issue, refuses on a missing parent identifier, and refuses follow-ups resolvable in the active diff — a module-wide scan has no parent. PROCESS.md L392 calls this class "worse than no skill". |
| `quarterly-kb-review` (skill) | Ops lens | Runs 90+ days after the delivery team has left, needing three simultaneous OAuth grants plus a vendored `linear-task-agent`. P7 PROCESS.md:706 names MCP non-transfer as the phase's own top risk. |
| `build-linear-backlog` (5-prompt skill) | Ops lens | Cannot satisfy the two-screen rule (L384) once framework links are stripped — ~160 lines of prompt body before procedure, against an empirical house ceiling of 111. **Split** into `scaffold-linear-milestones` + `push-linear-stories` at the existing human gate. |
| `container-platform-engineer` (subagent) | Routing lens | Two "and"s in the first clause; PROCESS.md resolves that mechanically to *split*. **Narrowed** to `container-image-engineer`; K8s manifests revert to prompt. |
| `security-fix-engineer` (subagent) | Ops lens | Verb overlaps `backend-engineer` almost exactly; the three constraints carrying its value are three bullets on that agent's boundaries. Three template edits to buy one P2 artifact fails the earn-its-keep bar. |
| `handoff-agent` (4 deliverables) | Ops lens | "Framework prescribes it" falsified — P7 PROCESS.md:176 says *"Do not create the file in this task"*. Four deliverables behind an "or" is four jobs. **Narrowed** to 2. |
| `qa-test-engineer` (5 prompts) | Both | The proposed fix is self-contradictory: it adds "do NOT invoke for integration tests" to a description whose positive trigger list already reads "write the integration tests". **Narrowed** to 3 prompts and renamed. |
| `mcp-allow-list-policy` (skill) | Synthesis | Confidence LOW; surfaced a Gate 6 scoping inconsistency that must be fixed in QUALITY-GATES.md first. |
| `add-mcp-server` + `devops-toolchain-bootstrap` (skills) | Synthesis | Cross-phase duplicates; both self-identified as their phase's weakest; neither maps to a gate; both need a browser OAuth click a skill cannot perform. |
| `/linear-context-pull` | Synthesis | Step 1.4 is optional; its load-bearing use (duplicate pre-flight) is absorbed into `push-linear-stories`. |
| `/promote-component` | Synthesis | Cannot open the PR (single-writer rule); refactor overlaps two shipped agents; its "one a11y assertion" would be mistaken for the accessibility review. |
| `/infra-runbook` | Synthesis | Once per project; DR section has no fetchable source and emits only TODOs. |
| `/post-mortem` | Synthesis | Kept killed on frequency — **but the stated reason ("not worth establishing a command convention") is falsified** by shipping `/adr` and `/load-task-context`. Reinstate as an optional follow-on, not a P2 commitment. |

---

## Extend, don't create

Six coordinated edits to already-shipped `subagent-prompts/` templates. Each must land in the **same PR** as the artifact it unblocks, because these files ship into consuming repos.

> **The rule is narrower than it reads, and the repo has already applied it that way twice.** What it forbids is shipping a **pointer to an artifact the consuming repo does not have** — a dangling reference. It does not reach an edit that is a **pure refusal**, which names no artifact, unblocks nothing, and is strictly safer standalone than absent. The `software-architect.md` row was split on exactly this basis in v2.4 (its `pr-reviewer` dangling-slug fix shipped alone while its `solution-architect` exclusion stayed bundled), and the `linear-task-agent.md` row is now split the same way. Stated as a test: **if removing the artifact from the sentence leaves the sentence still true and still useful, the edit can ship alone.**

| Shipped template | Edit | Ships with |
|---|---|---|
| `linear-task-agent.md` **(a)** | pr-open flow gains *"if the pre-PR self-review has not been run, load `open-pull-request` first"*. Its description contains "open the PR for this branch" verbatim, so **without this the subagent wins the route and the entire gate is bypassed**. | `open-pull-request` (P0) — **still bundled** |
| ~~`linear-task-agent.md` **(b)**~~ | ~~Boundaries name Documents / Projects / Milestones as out-of-scope **by design**, not by omission.~~ **LANDED, split from (a).** Shipped as four additive touches — description, request-classification step, operating boundaries, escalation list — plus the reciprocal clause in all three Phase 1 skill descriptions. See [Splitting the `linear-task-agent` row](#splitting-the-linear-task-agent-row). | — (shipped standalone) |
| `code-reviewer.md` | Do-NOT list gains *"the full pre-PR gate including DoD, rebase and PR open (load `open-pull-request`, which hands the diff review back here as step 1)"*. Its shipped triggers "anything to fix before I PR" / "review my diff before I PR" overlap the skill's "ready to PR" on the same utterance — a second, undetected gate bypass. | `open-pull-request` (P0) |
| `backend-engineer.md` | Operating boundaries gain the three security-remediation constraints (minimal backward-compatible diff; mandatory regression test failing before and passing after; no tautological symptom-masking fix). **Replaces** the killed `security-fix-engineer`. | — |
| `software-architect.md` | Mutual *"Do NOT invoke for: system-wide / new-tech / new-service-boundary decisions (use `solution-architect`)"*. **Also fixes a live bug:** its own frontmatter routes *"post-open PR review (use `pr-reviewer`)"* to a slug that exists nowhere in the repo. | `solution-architect` (P1) |
| `refactor-specialist.md` | Description gains the filer's name: route candidate-identification to the `file-followup-bug` skill (canonised at PROCESS.md L336), not to an unnamed actor. **Replaces** the killed `triage-refactor-candidates`. | — |
| the three Phase 1 skills | Each Do-NOT list gains the **reciprocal** of the `linear-task-agent` (b) edit: no writes around a story already in flight. Required by `ROUTING.md`'s own rule that a shared write object is fenced in **both** descriptions — before this, the boundary lived in `skill-prompts/README.md`, which never vendors, so it was in **zero** shipped descriptions. | landed with `linear-task-agent` (b) |
| `threat-modeler` ↔ `architecture-reviewer` | Mutual exclusion between the two **new** subagents. Both `opus`, both read-only, both emit severity-ranked design-time findings, and "what are the security risks in this design" vs "what breaks in this design before we code it" are paraphrases. The synthesis pass ran mutual-exclusion against shipped agents but never *between* its own proposals. | both (P1) |

### Splitting the `linear-task-agent` row

Half (b) shipped standalone rather than waiting for `open-pull-request`, which sits four build-order items away. Three things decided it.

**The gap was live, and it defeated two gate checkboxes rather than one.** The agent's scope-discipline rule fences writes into projects *outside* the developer's named scope; the three Phase 1 skills write into the *same* project, so nothing refused backlog construction. An issue the agent created from a requirements document carries no `phase:requirements` / `ai-generated` / `needs-human-review` labels and no verified deep-link — so it is invisible to **Gate 2's "AI Inbox cleared"** (`QUALITY-GATES.md:56`) *and* **Gate 3's "every issue's PRD deep-link still resolves"** (`:75`). Both pass while the issue sits outside the mechanism they measure. That is the unfalsifiable-checkbox pattern, arrived at from a new direction: not a checkbox nobody verifies, but a checkbox whose *population* can be silently under-filled.

**The enforcement point had to be the vendored file.** Three alternatives were considered and fail for the same structural reason. Sharpening the skills' descriptions does not work — a description is a routing *claim*, not a refusal, and once the subagent is loaded no skill description constrains it. `ROUTING.md` and the directory READMEs do not work either: instantiation copies the `.md` into `.claude/agents/` and leaves the README behind. **The gap is in the file that vendors, so only that file can close it.**

**The load-bearing touch is the classification step, not the boundaries section.** `linear-task-agent` classifies a request into one of eight flows *before* any boundary is consulted; "create the issues for this epic" lands in `file-followup` or `update`, both in-scope, and the boundary is never reached. So `file-followup` is now capped at **one issue arising from the story in flight**, and `update` is stated to **never create**. A boundary written only in the boundaries section is one the classifier has already run past — worth generalising to any subagent whose system prompt opens with a classification step.

Two properties the edit deliberately keeps:

- **Neither side names the other by slug.** A cross-phase slug reference dangles in a repo that installed only one of the two sets. Both sides describe the boundary by **write type**, which also makes it survive a rename.
- **No placeholder was renamed.** `{{TEAM_PREFIX}}` / `{{LINEAR_TEAM}}` is one of the six pairs in [`PLACEHOLDERS.md`](./PLACEHOLDERS.md), and both live in the files touched here. Converge-on-next-touch means the **next substantive rewrite**, not the next additive line — renaming inside a vendored template breaks every repo that already instantiated it.

---

## Repo defects found along the way

Independent of any conversion, this analysis surfaced the following. Each is a small, self-contained fix.

- **[`03-development/PROCESS.md` L353](../aidlc-phases/03-development/PROCESS.md)** — the worked skill example has a **broken relative path** (`../..` from `.claude/skills/<name>/` resolves to `.claude/`, not the repo root) **and** an anchor `#bug-report` that matches no heading in `PROMPTS.md`. The framework's own teaching example does not resolve.
- **[`03-development/PROCESS.md` L412](../aidlc-phases/03-development/PROCESS.md)** — the skill verification checklist lists **four** pre-commit tests, omitting **hand-off integrity** (documented at L392) — the exact test that killed `triage-refactor-candidates`.
- ~~**`03-development/subagent-prompts/software-architect.md:3`** — routes users to `pr-reviewer`, a subagent slug that exists nowhere in the repo.~~ **FIXED in v2.4** — now routes to "the team's post-open PR review tooling", matching the convention `code-reviewer.md:3` already used and accurate for a repo that may have chosen CodeRabbit, the Claude bot, or both.
- ~~**`/docs/runbooks/` ownership is contested** across Phases 3, 6 **and** 7, with three filename schemes.~~ **RESOLVED in v2.4.** The diagnosis was wrong in a useful way: this was never three competing schemes for one artifact, but **two distinct artifact classes collapsed into one flat directory, plus one typo**. Phase 3's `<service>.md` is a *service operational* runbook; Phase 6's `<alert-name>.md` is a *per-alert response* runbook; Phase 7's `<alert>.md` was a shortened token, not a competing claim. Resolution: `/docs/runbooks/alerts/<alert-name>.md` (Phase 6 owns) and `/docs/runbooks/services/<service>.md` (Phase 3 owns), with Phase 7 a **consumer, never an owner**. Subdirectories rather than a filename fix because a flat directory admits a real collision — a service named `checkout` and an alert named `checkout` produce the same path, with different audiences and reviewers. The load-bearing half is the derivation rule: `<alert-name>` is the alert's name **as configured in the backend, lowercased and kebab-cased**, which turns Gate 4's *"every alert links to a runbook"* from an eyeball check into a script. **Unblocks `observability-bringup`.**
- **`docs/planning/` does not exist** but is referenced by `README.md` and by the `ai-sdlc-framework-architect` agent's mandatory-reading table.
- **Phase 5 Gate 6 scoping inconsistency** — surfaced while evaluating the MCP Enforcement Policy prompt; fix in `QUALITY-GATES.md` before any MCP-policy artifact is considered.

---

## Sentry removal — consequences

Sentry is being removed from the framework. Two prompts are deleted and two proposed skills die with them; two new Phase 4 helpers replace them. Beyond the artifact churn, three things need a decision.

### 1. One quality gate becomes unsatisfiable

**[`06-cicd-devops/QUALITY-GATES.md:124`](../aidlc-phases/06-cicd-devops/QUALITY-GATES.md)** — *"Sentry + Seer wired for application errors; Sentry MCP server connected to Claude Code"*. Nothing in the remaining stack can satisfy this. **Delete or rewrite it in the same change, or Phase 6 Gate 4 can never pass.** Lines 125 and 176 name Sentry inside lists that stay satisfiable once the word is struck.

Two PROCESS.md checklist items go the same way: `04-testing-and-qa/PROCESS.md:358` (Sentry-Linear integration verified) and `07-delivery-handoff/PROCESS.md:684` (inherited-error CSV imported).

**Phase 4's own gates are unaffected** — `04-testing-and-qa/QUALITY-GATES.md` contains zero Sentry references, and all four gates survive verbatim.

### 2. A metric that will now lie

**[`04-testing-and-qa/QUALITY-GATES.md:110`](../aidlc-phases/04-testing-and-qa/QUALITY-GATES.md)** sets *defect escape rate < 5%, measured as production bugs / sprint stories*. That numerator was populated automatically by the Sentry Agent. Without an error tracker, production bugs are discovered only when a user reports one — so **the metric improves on paper while reality gets worse, and no gate catches it.**

This is the most dangerous consequence of the removal because it is invisible. Either name a replacement detection source, or explicitly re-baseline the metric and stop treating it as a quality signal.

### 3. Severity loses its quantitative anchor

Bug Triage previously *calibrated* a machine-assigned severity; now severity is assigned from scratch, and the prompt's numeric anchors (affected users, events/hour) have no replacement source. The `severity:auto` and `source:sentry` Linear labels die with them. This makes Bug Triage **more** clearly a human gate, not less.

> **On substituting a vendor.** `06-cicd-devops/PROCESS.md:39` and `07-delivery-handoff/PROCESS.md:48` both name **Bugsnag** as the documented alternative. Bugsnag has no MCP server in the mandated roster, so swapping the vendor name restores *detection* but not the *paste-free MCP pull* — which is precisely why `sentry-context-pull` and `Inherited Error Triage` cannot be revived by find-and-replace.
>
> **On `docs/tools-evaluation/`.** Leave it alone. `README.md:295` sets the precedent from the Cursor removal: *"the comparative reports still assess Cursor on its merits — this change is to the prescribed framework only."* Likewise the changelog: add a new version row, do not rewrite history.

---

## Security tooling removal — consequences

Third-party security tooling is removed framework-wide. The prescribed stack narrows to **GitGuardian + ggshield** plus GitHub-native tooling — **CodeQL / GitHub Advanced Security**, **Copilot Autofix**, **Dependabot**, **GitHub Actions** — and **Claude Code** with `/security-review` and the `anthropics/claude-code-security-review` Action. Semgrep, Trivy, Checkov, OPA / Gatekeeper / Conftest, Pulumi CrossGuard, Snyk, Vanta, Drata, Cycode, Aikido, Endor Labs, TruffleHog, Gitleaks, detect-secrets, Docker Scout, Renovate and OWASP Dependency-Check all leave. Pulumi itself is untouched — only CrossGuard, its policy-as-code product, was removed.

Net of the corrections recorded in the banner above: **two** Phase 5 prompts are deleted (`Container CVE Triage`, `OPA Policy Generation`), one is rewritten (Semgrep → CodeQL), one deleted prompt is **restored** (`Reachability Triage`), and one is **newly authored** (`Threat-Model Mitigation Verification`). `author-opa-policy` dies, `semgrep-rule-author` is renamed, `dependency-risk-analyst` returns narrowed from three prompts to two, and `threat-model-verifier` is added. Three things need saying plainly.

### 1. SAST and SCA survive — this is not a scanning capability cut

**CodeQL is promoted from fallback to the prescribed SAST baseline.** Everything `semgrep-rule-author` was accepted on survives in QL: deterministic whole-repo scanning, scheduled scans, SARIF evidence in the Security tab, and committable project-specific queries under `.github/codeql/`. The skill is renamed **`codeql-query-author`**, keeps its gate, its shape and its two-fixture verification, and now runs with **no MCP server at all** — a setup precondition removed, not added.

SCA survives on **Dependabot**. [`05-security/QUALITY-GATES.md:58`](../aidlc-phases/05-security/QUALITY-GATES.md) Gate 3 is intact under its new name **Dependency Hygiene** — same bar, read against the Dependabot alerts view plus a package-manager-native audit in CI. Gate 2 is likewise intact as **SAST Quality Gate**, and its custom-query checkbox is exactly what `codeql-query-author` serves.

**The reachability cut survives too — and the first pass through this revision got that backwards.** Dependabot reports the CVE and the package, not the call path, and that was read as reachability leaving with the commercial scanners. The reverse is true: `Reachability Triage` existed **because** reachability tooling was unavailable, and its own header note said so verbatim. Gate 3's bar is *0 Critical in production dependencies, with documented Tech Lead acceptance for the rest* — somebody has to work out which alerts clear it, and after the correction that somebody is `dependency-risk-analyst`. No departed vendor was substituted; a reasoning prompt the framework already owned was thrown out with them by mistake, and has been put back.

### 2. Container/IaC scanning and policy-as-code have no inheritor

This is the one real loss and it should not be dressed up. Trivy, Checkov, Docker Scout, OPA / Gatekeeper / Conftest and Pulumi CrossGuard leave with **nothing put in their place**.

**[`05-security/QUALITY-GATES.md:74`](../aidlc-phases/05-security/QUALITY-GATES.md) Gate 4 — Container & IaC Review — is now human-verified**, cut from nine lines to two, and it says so in its own words: *"No scanner, policy engine, or admission controller enforces it. If nobody signed off, the gate has not passed — a green pipeline says nothing about this surface."* PROCESS.md Step 4 was gutted to match: two human-review sub-steps against the `AGENTS.md` conventions and a fixed required-controls list.

**No proposed artifact can close that gap honestly.** A skill that read a Dockerfile and asserted a CVE verdict, or a policy author with no engine to target, is precisely the fabrication pattern reason-pattern #3 rejects everywhere else in this document — it would make an unscanned surface look gate-approved. That is why `author-opa-policy` is killed outright rather than retargeted, and why nothing replaces it.

One existing boundary becomes load-bearing as a result: `container-image-engineer` (P6) is already forbidden to state a CVE count, scan verdict, or image-size comparison. That was prudence when a CI scanner would have contradicted it. It is now the only thing standing between the agent and an invented verdict, because **no scanner exists to contradict it.** Treat it as a hard constraint at authoring time.

### 3. Phase 5's artifact surface holds at five

Phase 5 still proposes **five** artifacts — but not the same five. `author-opa-policy` is gone for good and `semgrep-rule-author` is now `codeql-query-author`; `threat-modeler`, `dependency-risk-analyst` and `secret-leak-response` stand; `threat-model-verifier` joins them. Prompts go from 14 to **13**. Four of the five are P1 and one is P2, so the phase is no cheaper and no dearer to land than before — it is pointed at different things: what the code *does*, rather than what a scanner *said*.

> **The first pass through this revision recorded a drop to three artifacts. That number was wrong twice over** — it killed `dependency-risk-analyst` on a misreading of its own source prompt, and it counted the unverified-mitigation gap as no gap at all. Both are corrected above. The container/IaC loss in §2 is the only genuine capability cut in this change.

`secret-leak-response` is **untouched, zero edits**: GitGuardian is the kept secrets tool, and the rotate-before-scrub ordering it enforces never depended on which scanner raised the alert. `threat-modeler` survives unchanged in substance — its Do-NOT list keeps routing CVE triage to `dependency-risk-analyst`, which exists again, and gains one line routing mitigation verification to `threat-model-verifier`.

> **On substituting a vendor.** Do not. The framework deleted the container-scan and policy-gate steps rather than rewording them into something unverifiable, and naming a replacement scanner here would put this analysis out of step with the framework it describes.
>
> **On `docs/tools-evaluation/`.** Leave it alone, on the same precedent the Sentry removal cited — the comparative reports still assess every removed tool on its merits; this change is to the prescribed framework only.

---

## Pulumi removal — consequences

Framework v2.4 removed the entire Pulumi AI stack — CLI, Neo, Copilot, MCP server, Cloud, ESC, Deployments, Insights and the `pulumi/*` Actions — and made Terraform the IaC primary with OpenTofu as the CLI-compatible fallback. **Phase 6's eight artifacts were built during that migration rather than in rollout order**, and the reason generalises: a vendor migration is exactly the moment the helpers pay for themselves, because writing them forces each lost capability to be named rather than quietly dropped. Four of the eight exist for no other reason.

### Corpus and artifact deltas

| Change | Effect |
|---|---|
| `Pulumi Cost Delta` **deleted** | Corpus 97 → **96** |
| `Pulumi IaC Generation` → `Terraform IaC Generation` | Anchor `#pulumi-iac-generation` → `#terraform-iac-generation`; the only rename, and the only edit that could have broken the repo if split from its inbound references |
| `pulumi-iac-engineer` → `terraform-iac-engineer` | Renamed; "policy pack" deliverable dropped. No count change |
| **4 skills added** — `iac-state-backend-bringup`, `ci-identity-and-secrets-bringup`, `cost-guardrails-bringup`, `deploy-and-rollback-bringup` | Artifacts **+4**; skills 16 → **20** |
| **Total** | Artifacts 30 → **34**; Phase 6 column 4 → **8** |

> **This document under-counted Phase 6 by four, and the reason is methodological.** All four additions wrap **prose PROCESS steps, not `PROMPTS.md` entries** — a state backend has no prompt, an OIDC trust policy has no prompt, a rollback drill has no prompt. A prompt-by-prompt conversion analysis cannot see them. The first two were then further shaped by the migration: what the implementation brief carried as one `iac-foundation-bringup` **split at the storage/identity seam**, because once the managed control plane was gone that single skill would have run ~70 lines across five surfaces.

### Why `Pulumi Cost Delta` was deleted, not ported

This prompt was previously classified **keep-as-prompt** under reason-pattern #4 (*already automated elsewhere — a CI payload, not a session prompt*). That classification does not survive the migration, and the honest verdict is deletion:

1. **Its stated reason for existing is gone.** Its own blockquote read *"Pulumi-on-PR cost-comment replacement for Infracost — Infracost does not yet support Pulumi natively."* Infracost supports Terraform natively. A prompt whose header states its own obsolescence condition should be deleted, not renamed.
2. **A renamed version would be strictly worse than the tool it replaces.** Infracost carries a maintained pricing database; the prompt asks Claude to look up unit costs from a pasted price list and file anything it cannot price under `Unpriced changes`. Against a purpose-built pricer that is a **second, less accurate number on the same PR** — and the wrong one will occasionally be the one a reviewer reads.
3. **Reason-pattern #3 now dominates #4:** *the output's fabrication risk exceeds its toil.* An LLM cost estimate posted as a gate-visible PR comment beside an authoritative one is precisely the "invented figures look gate-approved" failure this analysis exists to prevent.
4. **Its non-redundant residue is preserved, not lost.** The anomaly checks — cost-bearing change with no justification in the linked issue, oversize against the stated performance target, missing mandatory tags — are convention and requirement judgement, not costing, and Infracost does none of them. They survive as `cost-guardrails-bringup`'s reviewer checklist and `terraform-iac-engineer`'s step-8 self-check, landing in the same commit as the deletion so the capability is never absent even briefly.

**Deprecation is a conversion outcome.** This document previously had four verdicts — skill, subagent, command, keep-as-prompt. It now has five.

### What has no inheritor

Stated plainly, on the precedent the security-tooling removal set for container scanning:

1. **Cross-stack / org-wide resource search and compliance query** (Pulumi Insights + the MCP `resource-search` tool). The documented path is cloud-native inventory (AWS Resource Explorer / Config, Azure Resource Graph, GCP Cloud Asset Inventory), **unassessed in `docs/tools-evaluation/`** and therefore flagged as an evaluation gap rather than prescribed. Consequence: the mandatory tag set in `AGENTS.md` is promoted **from convention to load-bearing control**, because an untagged resource is invisible to the only inventory capability the phase has left.
2. **Dynamic, short-lived credentials and immutable secret revisions** (Pulumi ESC). GitHub OIDC is *better* than ESC for every cloud credential — but third-party API keys become **static and long-lived**, with rotation a scheduled task owned by a named human rather than a property of the system.
3. **Language-native infrastructure unit testing.** `terraform test` with `mock_provider` is first-party and free, and is materially less expressive: no fixtures, no parameterised cases, no coverage measurement. The framework previously used infra testing as an argument *for* Pulumi, so the old Step 3.3 claim — *"this is where Pulumi pulls ahead of Terraform — no Terratest gymnastics"* — was **deleted, not softened**. Terratest would recover some of it and is deliberately not prescribed: it needs real provisioned infrastructure and a Go harness the framework carries nowhere else.

**No IaC MCP server is prescribed.** `hashicorp/terraform-mcp-server` is registry/provider-docs oriented rather than state introspection, and is unassessed — so Phase 6's MCP roster shrinks to GitHub + observability, and the docs say so explicitly rather than implying a replacement exists.

### The offsetting gains

- **The Infracost gap closes.** One prompt and one risk row delete outright rather than being reworded.
- **`terraform plan` has machine-readable change symbols** (`-/+`, `+/-`, `destroy`) where `pulumi preview` had prose, so the line-by-line pre-deploy check gets strictly *more* precise.
- **$0 control plane**, in the cloud account the team already audits.
- **One fewer vendor in the credential path.**

### Two honesty properties that emerged from building rather than specifying

Both were invisible at spec time and only appeared when the templates were drafted against the real gates — which is the argument for building helpers during a migration rather than before one:

- **`container-image-engineer` can self-certify seven of Gate 3's nine criteria, not nine.** The spec said "eleven"; the gate has nine; two are unreachable. See its entry above.
- **A `terraform test` run block with `command = apply` *is* an apply.** With no managed runner to refuse it, a model reading only "never apply" would still write one. This is now an explicit hard rule and the wording of the social-pressure refusal.

---

## Phase 1 build — consequences

Phase 1's three P0 skills were built in this document's own recommended order, against a stack that did not change underneath them. That makes this the first build with **no migration pressure**, and the differences from the Phase 6 build are worth recording.

### Nothing was deleted, renamed, or rewritten

Phase 6's build deleted a prompt, renamed another, and dropped a deliverable. Phase 1's changed **no prompt text at all**: 12 prompts before, 12 after, all anchors intact, and the paste path documented as still valid in `PROMPTS.md`'s header. Counts are unchanged — 96 prompts, 34 artifacts, 18 built.

That is the expected outcome when a phase's tooling is stable, and it is a useful control. **Every edit Phase 6's build forced was caused by the vendor migration, not by the act of building a skill.** A team reading that build might reasonably have concluded that converting prompts churns the prompt library; it does not.

### What building nonetheless surfaced

Three things that specification could not, all recorded in the per-artifact "What changed on build" notes above:

1. **`Gap Analysis` is two prompts wearing one heading.** The pre-publish self-review and the post-publish backlog sweep share a name, a body and nothing else — different input, different output, different gate. The analysis had this half-recorded (the sweep's Do-NOT list excluded the self-review), but only inlining it forced the split to be *enforced* rather than described. **The general lesson: a prompt whose header note says "this form is also valid for X" is a candidate for being two prompts**, and the conversion pass is where that gets settled.
2. **A predicted anchor map and a read-back anchor map are not the same artifact.** The source prompt returns headings extracted from the draft at confirm time; what downstream issues need is the anchors the tracker actually minted. Both look like a list of anchors in the output. Only one of them resolves.
3. **The most dangerous refusals are the ones the source prompt left implicit.** "Never set state beyond Triage" implies "never clear `needs-human-review`" to a human reader and does not imply it to a model that has just been thanked for its work. Gate 3 is that label; making it explicit cost one line.

### One property carried over deliberately

Every one of the three terminates at a **human gate**, not at the next skill — publish stops for stakeholder review, scaffold stops for PM approval, push stops for the AI Inbox. This is the same shape as Phase 6's split at the storage/identity seam, and it has a routing benefit that was not the reason for it: **a mis-triggered skill in this set costs a confirmation prompt, not a backlog.** Three auto-triggering skills that write to a shared tracker would be a genuinely risky thing to ship without that property.

---

## Build order

### Phase 0 — prerequisites, zero artifacts (~1-2 days)

Nothing else ships until these land.

1. **Routing dry-run in `roombook/` (½ day).** `roombook/.claude/` holds 10 agents and 5 skills and is the only tree in this repo where skills and subagents coexist — the only place the auto-trigger test can actually run. Predicted collisions: `render-design-diagrams` vs `skills/architecture-diagram` (verbatim trigger), `cicd-pipeline-bringup` vs `skills/cicd-devops`, the Phase 1 skills vs `skills/requirement-gathering`, `solution-architect` vs `agents/system-design-architect`, `threat-modeler` vs `agents/security-reviewer`, and `e2e-and-coverage-engineer` vs `agents/qa-test-engineer` (same name, opposite scope). Both directories are gitignored, so these are predictive, not shipped — but they predict exactly what happens in a repo that already followed the framework.
2. ~~**Directory convention PR (½ day).**~~ **DONE in v2.4.** `skill-prompts/<name>/SKILL.md` is established as a sibling of `subagent-prompts/`, each with a README mirroring the Phase 3 one. `command-prompts/` remains unbuilt — no slash command has shipped yet. **One correction the convention needed:** skills are **folders**, not files, because Claude Code uses the folder name as the invocation slug, so a `cp skill-prompts/*.md` instantiation would silently produce zero working skills. Both the README and the framework's Step 0.6 now say `cp -r`.
3. **Doc fixes (½ day).** The defect list above — two of which are now fixed. **`PLACEHOLDERS.md` and `ROUTING.md` were deferred, each with a firm trigger** (see [Decisions still open](#decisions-still-open)): a central placeholder file for ~32 entries would be a *third* copy of information whose authoritative home is the README the instantiating operator actually opens, and a routing map written while only one phase ships skills is a map with one region. **Trigger for `PLACEHOLDERS.md`: a third phase ships templates. Trigger for `ROUTING.md`: a second phase ships skills.**
   >
   > **Both triggers fired with the Phase 1 build, and both files are now written** — [`ROUTING.md`](./ROUTING.md) and [`PLACEHOLDERS.md`](./PLACEHOLDERS.md). Phase 1 was the third phase to ship templates (after 3 and 6) and the second to ship skills (after 6). The deferral was conditional and the condition was met; re-deferring a fired trigger is the unfalsifiable-checkbox pattern this document rejects everywhere else. `ROUTING.md` was the clearer of the two — Phase 1's skills and Phase 3's `linear-task-agent` share one MCP server and one set of Linear nouns, which is the first real cross-phase collision surface in the repo.
   >
   > **Writing `ROUTING.md` immediately paid for itself by exposing an unenforced boundary.** Setting the 18 shipped artifacts side by side made it visible that `linear-task-agent`'s scope-discipline rule fences writes to *other* projects, while the three Phase 1 skills write into the *same* project — so nothing in the shipped template actually refuses backlog construction. The map records that row as **intent, not behaviour**, and says so in place rather than reading as verification. That is the class of defect a per-phase README structurally cannot surface, and it argues the trigger should have been "a second phase ships anything that writes what an existing artifact writes", not "a second phase ships skills".
   >
   > **`PLACEHOLDERS.md`'s original deferral reasoning does not survive counting.** The three shipped phases carry **36 distinct placeholders** — 2 in Phase 1, 22 in Phase 3, 12 in Phase 6 — and the "third copy of the same information" objection assumed they overlap. They do not: **literal overlap between the three sets is zero.** What exists instead is worse and invisible from inside any one README — **synonym pairs that name the same repo fact differently across phases**: `{{SECRETS_MECHANISM}}` (P3) / `{{SECRETS_MANAGER}}` (P6); `{{OBSERVABILITY_STACK}}` (P3) / `{{OBS_BACKEND}}` (P6); `{{TEST_RUNNER}}` (P3) / `{{TEST_COMMAND}}` (P6); `{{BACKEND_ROOT}}` + `{{FRONTEND_ROOT}}` (P3) / `{{SERVICE_ROOT}}` (P6). An operator instantiating two phases fills the same value twice under two names, and nothing catches it when they fill it inconsistently.
   >
   > **Both were written, and `PLACEHOLDERS.md` shipped as a name-reconciliation table rather than a value list** — one row per repo fact, the alias each template uses, and why the answers must agree. Values stay in the per-directory READMEs, which are what an operator has open while instantiating; the central file holds only what no single README can see. Drafting it against the real inventory turned up **six reconciliation rows, not four** — the two additions being `{{TEAM_PREFIX}}` / `{{LINEAR_TEAM}}` (the same Linear team, once as its key and once as its name) and the `{{RUNTIME}}` / `{{TEAM_STACK}}` pair, which are not synonyms but *constrain* each other: `Node 22` under a `FastAPI + Python 3.12` stack is a contradiction that ships a broken image. **No rename sweep was performed** — templates are vendored by copy, so renaming breaks every repo that already instantiated them; the file prescribes converge-on-next-touch instead.

### Phase 1 — validate the approach at lowest risk (~1 week)

4. **`/adr` (½ day).** Best item on the list: explicitly fired, no auto-trigger competition, no coordinated edit, no unmandated MCP, cited by three gates, fires 11+ times per project. If the command convention works here it works everywhere. **Still unbuilt — `command-prompts/` remains an empty convention.**
5. ~~**`publish-prd-to-linear` (2 days).**~~ **SHIPPED.** Two prompts, one gate, one MCP server that Phase 1 already mandates. It came in wrapping three prompts, not two — see its entry.

### Phase 2 — the P0 tier (~3 weeks)

One artifact per PR, so the router can be re-tested each time.

6. ~~`scaffold-linear-milestones` + `push-linear-stories` (3 days — ship together; the second consumes the anchor map)~~ **SHIPPED**, together, as predicted — the second refuses to run without the first's anchor map, so shipping them apart would have shipped one skill that could not execute its own procedure.
7. `open-pull-request` + the `linear-task-agent` and `code-reviewer` edits (3 days — three files, one PR)
8. `run-sprint-planning` (2 days)
9. `e2e-and-coverage-engineer` (2 days — the rename resolves the roombook collision; no shipped-template edit needed after the narrowing)

### Phase 3 — P1 (~4-6 weeks)

No-coordinated-edit artifacts first: **`load-test-engineer`** (zero routing collisions — the cheapest P1 to land), **`dependency-risk-analyst`** (also collision-free; its only dependency is that Dependabot is switched on), `threat-modeler` + `architecture-reviewer` (as a mutually-excluding pair), `accessibility-auditor`, `secret-leak-response`. Then `solution-architect` with its `software-architect` edit. **`threat-model-verifier` ships after `threat-modeler`, never before it** — it audits that agent's output, so building it first gives it nothing to verify, and the two descriptions must be written as a mutually-excluding pair in the same PR. Then the remaining multi-step skills: `author-test-plan`, `reproduce-and-diagnose-bug`, `data-model-design`, `render-design-diagrams`, `finalise-documentation`.

> **Phase 6's eight helpers are already built and are not in this queue.** They shipped with the v2.4 Pulumi removal rather than in rollout order. The rule that generalises: **when a phase undergoes a stack change, build its helpers then** — not before, when the shape is speculative, and not after, when the losses have already been forgotten.

### Phase 4 — P2, conditional

Build only if the frequency materialises. Several name their own drop condition: `sweep-requirements-gaps` now carries a **falsifiable release trigger and a numeric drop condition** rather than a felt one (see its entry), `/load-task-context` if tracker hygiene does not supply the deep-links, `api-contract-freeze` if the audit is not genuinely mechanical, `handoff-agent` after the first real engagement close, `reproduce-and-diagnose-bug` if teams already ship regression tests in the fix diff unprompted.

---

## Decisions still open

1. **Does the `roombook/` dry-run happen before or after authoring?** Doing it first costs half a day and will change at least three descriptions (and possibly two names). Doing it after means authoring templates that then need edits in the consuming repo. **Recommendation: before** — it is the only empirical routing evidence available anywhere in this repo.

2. **`security-fix-engineer` and `triage-refactor-candidates`: absorb into shipped agents, or keep as gaps?** Both are resolved above as boundary edits to `backend-engineer` and `refactor-specialist`. The alternative is to accept the routing risk and build them as separate artifacts — which is what four of the seven per-phase analysts wanted. The deciding question is whether teams actually mis-route on "fix this SAST finding" today.

3. ~~**Do the Phase 1 outer-loop skills perform their own Linear writes, or does `linear-task-agent` widen?**~~ **SETTLED by shipping them.** The three skills perform their own writes, each behind its own confirm-then-`go`, and both directions of the boundary are now written down where a maintainer will hit them: `01-requirement-gathering/skill-prompts/README.md` states that these are the outer-loop writers and that the dev-loop agent refuses this work, and Phase 1's PROCESS Step 0 repeats it as "by design, not by omission". **The reciprocal edit to `linear-task-agent.md` is still outstanding** — it ships with `open-pull-request`, and until it lands the guarantee is documented on only one of the two sides. Original resolution, unchanged: `linear-task-agent` stays sole writer for the **inner dev loop** (verified in its own boundary text), and outer-loop writes — Documents, Projects, Milestones, Triage issues, estimates, the test-plan Document — are performed by the phase skills, each borrowing its literal-`go` confirm-before-write protocol. This is correct but non-obvious; if it is not written into that agent's boundaries, a future maintainer will "fix" it and break the dev-loop-only guarantee.

4. **The slash-command tier.** Commands were killed partly on "not worth establishing a convention" — a cost that no longer exists once `/adr` ships. `/write-module-readme` was reinstated; **`/post-mortem` is now settled and stays killed**, on shape rather than convention cost (see the not-converted table). Should the remaining three (`/promote-component`, `/infra-runbook`, `/linear-context-pull`) be re-evaluated on their own merits, or is the smaller command surface itself the goal?

5. ~~**Do skill templates inline their procedure, or link it?**~~ **SETTLED by shipping six of them in v2.4.** Resolved as **inline with `{{PLACEHOLDERS}}`**, and the corollary that was still open — where the framework pointer goes — is now answered too: **shipped skill templates carry no framework path in their body; the pointer lives in `skill-prompts/README.md`, which never ships.** The reasoning is that PROCESS.md L380's "open with the role and the source of truth" presupposes the skill was authored *in* the consuming repo, where `aidlc-phases/` exists. So each shipped skill opens by naming a **repo artifact** that does resolve there — the conventions file, the recorded architecture decisions, the existing workflows, the service's exposed metrics. Framework provenance is not lost, it is relocated to the README's Wraps column, which is strictly better placement: the person who needs the provenance is the one instantiating, not the agent executing. **One declared exception:** `cicd-pipeline-bringup` references the four agentic-workflow templates rather than inlining ~65 lines of YAML that would breach the two-screen limit, and the README makes that a one-time human paste at instantiation.
   > The decision also cost what it predicted it would: inlining is what forced the `iac-foundation-bringup` split, exactly as it forced the `build-linear-backlog` one. Two splits from the same rule is enough evidence to call it load-bearing rather than incidental.

6. **How does a shipped artifact get revised when its phase's stack changes?** New, raised by v2.4. Phase 6's helpers were built during a vendor migration and four of the eight exist only to carry capabilities the departing vendor took with it. That worked — but the eight are now *shipped templates* in a repo, and the next stack change to Phase 6 has to revise them rather than author them. There is no convention yet for versioning a template, deprecating one, or telling a consuming repo that its vendored copy is stale. **Recommendation: do nothing until it bites once.** A versioning scheme designed before a single template has ever needed revising will be wrong.

---

## Appendix: full per-prompt classification

All 97 prompts, in file order. Line numbers refer to each phase's `PROMPTS.md`. **Four** prompts are deleted and shown struck from their tables: `sentry-context-pull` (Phase 4) and `Inherited Error Triage` (Phase 7) with the Sentry removal, and `Container CVE Triage` and `OPA Policy Generation` (Phase 5) with the security-tooling removal. `Reachability Triage` was struck in the same revision and is **restored**, so it appears live. One Phase 5 row — `Threat-Model Mitigation Verification` — is a **newly authored** prompt with no pre-revision counterpart. **101 rows, 4 struck, 97 live.**

### Phase 1 — Requirement Gathering

| Prompt | Line | Verdict | Artifact |
|---|---|---|---|
| PRD Generation | 9 | skill | `publish-prd-to-linear` — **shipped** |
| Epic Decomposition | 60 | skill | `scaffold-linear-milestones` — **shipped** |
| User Story Generation | 82 | skill | `push-linear-stories` — **shipped** |
| Acceptance Criteria | 117 | skill | `push-linear-stories` — **shipped** |
| Gap Analysis | 142 | skill | **split on build:** pre-publish self-review half → `publish-prd-to-linear` (**shipped**); post-publish sweep half → `sweep-requirements-gaps` (held) |
| Interview Summary Structuring | 184 | keep-as-prompt | — |
| Linear Context Pull | 207 | keep-as-prompt | absorbed as the duplicate pre-flight step in `push-linear-stories` — **shipped** |
| PRD-to-Linear Document | 235 | skill | `publish-prd-to-linear` — **shipped** |
| PRD-to-Linear Scaffold (Milestones) | 278 | skill | `scaffold-linear-milestones` — **shipped** |
| Stories-to-Linear Push | 308 | skill | `push-linear-stories` — **shipped** |
| Linear Gap Sweep | 361 | skill | `sweep-requirements-gaps` — held until the three above run on a real PRD |
| Scalability & Cost Assessment | 399 | keep-as-prompt | — |

### Phase 2 — System & Architecture Design

| Prompt | Line | Verdict | Artifact |
|---|---|---|---|
| Architecture Proposal | 7 | subagent | `solution-architect` |
| Trade-off Interrogation | 54 | subagent | `solution-architect` |
| Eraser Architecture Diagram (via MCP) | 84 | skill | `render-design-diagrams` |
| Eraser ER Diagram (via MCP) | 115 | skill | `render-design-diagrams` |
| Architecture Decision Record Generation | 135 | slash-command | `/adr` |
| Entity Extraction | 177 | skill | `data-model-design` |
| Schema Generation | 215 | skill | `data-model-design` |
| Migration Generation | 250 | skill | `data-model-design` |
| API Contract | 281 | skill | `api-contract-freeze` |
| Tech Stack Comparison | 315 | keep-as-prompt | — |
| Design Review | 348 | subagent | `architecture-reviewer` |
| Design System Bootstrap | 384 | keep-as-prompt | — |
| UI Wireframe | 413 | keep-as-prompt | `wireframe-flow` killed |
| Production Component | 446 | keep-as-prompt | `/promote-component` killed |
| Accessibility Review (Claude) | 474 | subagent | `accessibility-auditor` |

### Phase 3 — Development

| Prompt | Line | Verdict | Artifact |
|---|---|---|---|
| Estimation | 7 | skill | `run-sprint-planning` |
| Feature Scaffolding (for Claude Code) | 39 | keep-as-prompt | embedded in shipped `frontend-engineer` / `backend-engineer` |
| Test Generation | 81 | keep-as-prompt | embedded in three shipped subagents |
| Self-Review Before PR | 114 | skill | `open-pull-request` |
| Refactoring | 164 | keep-as-prompt | embedded in shipped `refactor-specialist` |
| Documentation Generation | 196 | slash-command | `/write-module-readme` |
| Debugging (for Claude Code) | 222 | keep-as-prompt | — |
| MCP Integration (for Claude Code) | 254 | keep-as-prompt | `add-mcp-server` killed |
| linear-sprint-pull | 281 | skill | `run-sprint-planning` |
| estimates-to-linear | 301 | skill | `run-sprint-planning` |
| story-decomposition | 321 | skill | `run-sprint-planning` |
| linear-next-task | 345 | keep-as-prompt | shipped `linear-task-agent` `find` flow |
| task-context | 363 | slash-command | `/load-task-context` |
| architecture-design | 387 | keep-as-prompt | system prompt of shipped `software-architect` |
| linear-progress-comment | 445 | keep-as-prompt | shipped `linear-task-agent` `progress` flow |
| pr-description | 473 | skill | `open-pull-request` |
| refactor-candidates | 503 | keep-as-prompt | `triage-refactor-candidates` killed; route filing to `file-followup-bug` |
| runbook-generation | 530 | keep-as-prompt | — |

### Phase 4 — Testing & QA

| Prompt | Line | Verdict | Artifact |
|---|---|---|---|
| Test Plan Generation | 7 | skill | `author-test-plan` |
| test-plan-gap-analysis | 72 | skill | `author-test-plan` |
| test-plan-to-linear-document | 107 | skill | `author-test-plan` |
| Unit Test Generation | 131 | keep-as-prompt | routing collision with shipped engineers |
| Integration Test Generation | 169 | keep-as-prompt | routing collision with shipped engineers |
| Playwright E2E Test Generation | 210 | subagent | `e2e-and-coverage-engineer` |
| playwright-failure-debug | 250 | subagent | `e2e-and-coverage-engineer` |
| Mobile Test Generation (Appium / BrowserStack) | 283 | keep-as-prompt | — |
| k6 Load Test Generation | 322 | subagent | `load-test-engineer` |
| Debugging (Bug Investigation) | 362 | skill | `reproduce-and-diagnose-bug` *(refusal rule relaxed)* |
| ~~sentry-context-pull~~ | ~~408~~ | **deleted** | Sentry removed — no non-Sentry residue |
| bug-reproduction | 439 | skill | `reproduce-and-diagnose-bug` |
| Bug Triage & Severity Assignment | 475 | keep-as-prompt | — |
| Test Coverage Gap Analysis | 528 | subagent | `e2e-and-coverage-engineer` |
| post-mortem | 583 | keep-as-prompt | `/post-mortem` killed on shape; fold into `reproduce-and-diagnose-bug`'s terminal step |

### Phase 5 — Security & Compliance

| Prompt | Line | Verdict | Artifact |
|---|---|---|---|
| Threat Model STRIDE | 9 | subagent | `threat-modeler` |
| Threat-Model Mitigation Verification | 52 | subagent | `threat-model-verifier` *(**newly authored** — the only row in this appendix with no pre-revision counterpart)* |
| Security Fix Generation | 101 | keep-as-prompt | `security-fix-engineer` killed; absorbed into `backend-engineer` boundaries |
| CodeQL Custom Query Generation | 143 | skill | `codeql-query-author` *(prompt rewritten from Semgrep Custom Rule Generation; skill renamed, verdict unchanged)* |
| Reachability Triage | 188 | subagent | `dependency-risk-analyst` *(**restored** — the deletion misread the no-tool fallback as tool-dependent; re-sourced on Dependabot)* |
| Dependency Upgrade Impact | 226 | subagent | `dependency-risk-analyst` *(**moved back** from kept-as-prompt)* |
| ~~Container CVE Triage~~ | ~~164~~ | **deleted** | Security tooling removed — hard-bound to `trivy image --format json`, an input that no longer exists. Verdict re-examined with the `Reachability Triage` correction and **upheld** |
| ~~OPA Policy Generation~~ | ~~198~~ | **deleted** | Security tooling removed — OPA / Gatekeeper and CrossGuard both gone; `author-opa-policy` dies with it |
| Secrets Incident Response | 265 | skill | `secret-leak-response` |
| AI Agent Threat Review | 312 | subagent | `threat-modeler` |
| MCP Enforcement Policy | 380 | keep-as-prompt | `mcp-allow-list-policy` killed |
| Compliance Checklist Generation | 414 | keep-as-prompt | — |
| Evidence Compilation | 468 | keep-as-prompt | — |
| Security Posture Report | 508 | keep-as-prompt | — |
| Pre-Release Self-Review | 550 | keep-as-prompt | `release-readiness-reviewer` killed; doc fix instead |

> Struck rows carry their **pre-deletion** line numbers; live rows are re-read against the current `05-security/PROMPTS.md`.

### Phase 6 — CI/CD & DevOps

| Prompt | Line | Verdict | Artifact |
|---|---|---|---|
| AGENTS.md Authoring | 9 | keep-as-prompt | `devops-toolchain-bootstrap` killed |
| Cloud Provider Comparison | 61 | keep-as-prompt | — |
| Cost Estimation | 102 | keep-as-prompt | — |
| ~~Pulumi Cost Delta~~ | 170 | **deleted (v2.4)** | Infracost supports Terraform natively; anomaly checks absorbed by `cost-guardrails-bringup` + `terraform-iac-engineer` |
| Cost Optimisation | 200 | keep-as-prompt | — |
| Terraform IaC Generation *(was Pulumi IaC Generation)* | 218 | subagent | `terraform-iac-engineer` — **shipped** |
| Infra Runbook | 294 | keep-as-prompt | `/infra-runbook` killed |
| CI/CD Pipeline Generation | 336 | skill | `cicd-pipeline-bringup` |
| Claude Code Action — PR Review Workflow | 391 | skill | `cicd-pipeline-bringup` |
| Agentic Workflow Templates | 415 | skill | `cicd-pipeline-bringup` (sequencing only; templates stay verbatim) |
| Dockerfile Generation | 490 | subagent | `container-image-engineer` |
| Kubernetes Manifests Generation | 527 | keep-as-prompt | narrowed out of the subagent |
| Observability / Dashboard Generation | 575 | skill | `observability-bringup` |
| SLO and Alert Generation | 622 | skill | `observability-bringup` |
| Runbook Generation from Alert | 653 | skill | `observability-bringup` |
| Self-Review Before Production Deploy | 715 | keep-as-prompt | `release-readiness-reviewer` killed; gate anchor was fabricated |

### Phase 7 — Delivery & Handoff

| Prompt | Line | Verdict | Artifact |
|---|---|---|---|
| Release Notes Polish | 7 | keep-as-prompt | already automated in CI |
| Documentation Generation (per page) | 44 | skill | `finalise-documentation` |
| Doc Completeness Check | 79 | skill | `finalise-documentation` |
| Handoff Document Generation | 135 | subagent | `handoff-agent` |
| Knowledge Base Seeding (FAQ / Glossary / Troubleshooting) | 222 | keep-as-prompt | narrowed out of `handoff-agent` |
| Quarterly KB Completeness Check | 282 | keep-as-prompt | `quarterly-kb-review` killed |
| KT Session Script | 323 | keep-as-prompt | narrowed out of `handoff-agent` |
| KT Session Summary | 371 | keep-as-prompt | — |
| ~~Inherited Error Triage~~ | ~~438~~ | **deleted** | Sentry removed — 9 of 12 CSV columns are error-tracker fields |
| Post-Handoff Retrospective | 495 | subagent | `handoff-agent` |
