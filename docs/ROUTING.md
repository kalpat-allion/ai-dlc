# Artifact Routing Map

> **What this is.** One map of every shipped subagent and skill in this repository, organised by **the work it owns** rather than by the phase directory it happens to live in. Its job is to answer one question: *when a developer says X, which artifact should take it, and which must not?*
>
> **Why it is not per-phase.** Artifacts are **not scoped to the phase that ships them.** `linear-task-agent` is used whenever someone works a story, in month one or month nine. `publish-prd-to-linear` is used whenever requirements are written, including a mid-project change request. A boundary between two artifacts used across phases cannot be expressed inside either phase's README — which is exactly why this file exists.
>
> **Scope: 38 shipped artifacts** — 7 development subagents, 2 DevOps subagents, 3 design subagents, 2 testing subagents, **3 security subagents**, 6 DevOps skills, 3 requirement skills, 3 design skills, 2 development skills, 2 testing skills, **2 security skills**, and **3 commands** (1 design, 2 development). *(Counted from `aidlc-phases/*/{skill,subagent,command}-prompts/`, not incremented.)* Proposed-but-unbuilt artifacts are listed in [PROMPT-CONVERSION-ANALYSIS.md](./PROMPT-CONVERSION-ANALYSIS.md) and enter this map when they ship, not before.

---

## The one rule

**Skills auto-trigger from their description; subagents are name-invoked but can still be auto-selected.** So the `description:` field is a claim on user utterances, and two artifacts claiming the same utterance is a defect — the router picks one, silently, and the loser's guarantees never run.

**Commands are not a third mechanism.** Custom commands have been merged into skills: a file at `.claude/commands/adr.md` and a skill at `.claude/skills/adr/SKILL.md` both create `/adr` and work the same way, and **by default both you and Claude can invoke either**. What makes a command explicitly-fired is the frontmatter field `disable-model-invocation: true`, not the directory it sits in. Remove that field and the command's description becomes a claim on user utterances exactly like a skill's, and it inherits every rule below. Any command shipped from this repo carries it; the tier is a convention held in frontmatter.

Boundaries are therefore drawn on **three axes, in this order**:

1. **Write type** — what the artifact creates or mutates (a Linear Document vs. an issue state; a `backend` block vs. resource HCL). This is the most durable axis, because it does not move when the project does.
2. **File ownership** — which paths it writes. Two artifacts touching `.github/workflows/` need a file-level split, not a topic-level one.
3. **Verb** — the literal words a user says. Least durable, most immediate: a stolen verb hijacks ordinary work on day one.

**Never draw a boundary on phase.** "Use this during Phase 1" is not a boundary; a team in month six doing requirements work is still doing requirements work.

---

## Master map

The **Provenance** column records which phase ships the template. It is a lookup path, **not a usage restriction**.

### Tracker writes (Linear MCP)

| Artifact | Type | Owns | Never owns | Provenance |
|---|---|---|---|---|
| `publish-prd-to-linear` | Skill | The requirements Document, the Project that carries it, the section-anchor map | Milestones, Issues, marking anything approved | P1 |
| `scaffold-linear-milestones` | Skill | Milestones — creation, order, target dates | The Project, the Document, Issues | P1 |
| `push-linear-stories` | Skill | First-time story creation from a requirements document: Triage issues, their AC, labels, deep-links | Milestones, any state move past Triage, clearing `needs-human-review` | P1 |
| `linear-task-agent` | Subagent | Writes **around a story already in flight**: find, start, progress comment, PR-open, done, single follow-up filing, field updates | Requirements Documents, Projects, Milestones, bulk issue creation from a requirements document; sprint planning; code; **opening a PR before the pre-PR gate has run** | P3 |
| `run-sprint-planning` | Skill | Outer-loop cycle preparation: the read-only candidate pull, the sizing, the estimate field and its comment, XXL decomposition into sub-issues of an existing backlog story | Every write around a story already in flight; backlog construction from a requirements document; velocity calibration; owner assignment; opening or committing the cycle | P3 |

**The tracker now has three write types, not two.** The requirement skills **construct** the backlog from a document; `run-sprint-planning` **prepares** what is already in it for commitment; `linear-task-agent` **operates** on one story in flight. All three create issues, so "does it create an issue?" separates none of them. The tests that do: *where does the issue come from* (a requirements document vs. a parent backlog story vs. the work in hand), and *what state is the story in* — `run-sprint-planning` writes only to `Backlog` and drops anything that has left it, because a story in flight already carries a committed number and re-sizing it mid-cycle rewrites the history the team calibrates against.

**The tie-break, stated as a write type:** the three skills **construct the backlog**; `linear-task-agent` **operates on a story that already exists in it**. Both write issues, so "does it create an issue?" is not the test. The test is *where the issue comes from* — a requirements document (skills) or a developer working a story who found something (agent).

> **✅ Enforced in both directions, in the shipped templates.** `linear-task-agent` refuses backlog construction in four places — its description, its request-classification step, its operating boundaries and its escalation list — and the three skills each refuse writes around a story in flight. Neither side names the other by slug: a cross-phase slug reference dangles in a repo that installed only one of the two sets, so both sides describe the boundary **by write type**.
>
> **What made this necessary rather than tidy:** the agent's scope-discipline rule fences writes to *other* projects, while the skills write into the *same* one — so before this edit nothing refused it. An issue the agent created from a requirements document would carry no `needs-human-review` label and no verified deep-link, making it invisible to **two** gate checkboxes at once: "AI Inbox cleared" and "every issue's PRD deep-link still resolves". Both would pass while the issue sat outside the mechanism they measure.
>
> **The load-bearing half is the classification step, not the boundaries section.** The misroute happens when a request is classified as `file-followup` or `update` before any boundary is consulted, so those two classifications now carry their own limits — one issue only, traced to the story in flight; and `update` never creates.

### Application code

| Artifact | Type | Owns | Never owns | Provenance |
|---|---|---|---|---|
| `software-architect` | Subagent | Per-story design pass inside an existing codebase, read-only, stops at developer approval | System-wide architecture, new service boundaries, new datastores, implementation | P3 |
| `frontend-engineer` | Subagent | UI implementation **and its component tests**, in the same commit | Server-side code, E2E suites, review | P3 |
| `backend-engineer` | Subagent | Server-side implementation **and its integration tests**, migrations inside a story | UI code, E2E suites, review | P3 |
| `refactor-specialist` | Subagent | Refactoring executed against an existing `tech-debt` issue | Identifying candidates for filing, behaviour changes | P3 |
| `code-reviewer` | Subagent | Pre-PR self-review of a diff, read-only | Applying fixes, post-open PR review, architecture review | P3 |
| `conflict-resolver` | Subagent | Merge / rebase / cherry-pick conflict resolution on the working tree | Anything not conflict-shaped; never `--abort`s unasked | P3 |
| `open-pull-request` | Skill | The **ordered pre-PR gate** as one procedure: identifier reconciliation, self-review, fixes **and their re-review**, the Definition-of-Done walk, the rebase and the re-review of resolved hunks, the composed title and body | The diff review itself, the fixes, the conflict resolution, **any tracker write**, the merge, anything after the PR is open | P3 |
| `/load-task-context` | Command | **Dereferencing** a story's pointers — the requirement section, the governing decision records, the API-spec section per endpoint, the module's existing tests — and reporting each with a conflict verdict | Any tracker write, writing a missing source, resolving a conflict it found, starting the implementation | P3 |
| `/write-module-readme` | Command | One module's `README.md`, from seven candidate sections each admitted only on evidence found in the module | API reference content (it renders from the spec), clearing the `TODO`/`FIXME` markers it surfaces, documenting a repo or a module it was not pointed at | P3 |

**Test-writing is deliberately not a separate owner.** The implementation specialists claim unit and integration tests by **positive** trigger, because the test ships in the same commit as the code. An artifact that claimed "write the tests" generally would take that work away from them mid-story.

#### The `open-pull-request` collision set — **CLOSED**, and worth reading phrase by phrase

The sharpest collision in this repo so far. `open-pull-request` auto-triggers, and **two shipped subagents claimed the same utterances verbatim in their own descriptions.** Both are now closed by coordinated edits that landed in the same PR as the skill — recorded here rather than deleted, because the closure is three sentences in three files and the next edit to any of them re-opens it.

| Skill phrase | Was claimed by, verbatim | What the incumbent winning costs |
|---|---|---|
| *"I'm done, open the PR"* · *"raise the PR for ENG-XXX"* | `linear-task-agent` — **"open the PR for this branch"** | The PR opens with **no self-review, no Definition-of-Done walk and no rebase.** Gate 2's entire pre-merge half is bypassed — **and the PR looks completely normal.** Nothing downstream re-runs any of it, and the reviewer assumes the pass already happened |
| *"ready to PR"* · *"run the pre-PR checks"* · *"can this go up for review"* | `code-reviewer` — **"anything to fix before I PR"** / **"review my diff before I PR"** | The developer gets **step 2 of 7** and reads a clean report as *ready*. Steps 3-7 — the verified fixes, the DoD walk, the rebase, the composed body, the hand-off — never run |
| *"ship this branch"* | **nothing** | Free. Claimed by no other artifact, which is why it survived the narrowing intact |

**How each was closed, and where the closure lives:**

- **`linear-task-agent`** — five additive touches, not one. Its description, its operating boundaries, its **request-classification step** (`pr-open` now presupposes the gate has run), its pr-open flow, and its escalation list. The classification step is the load-bearing one: a request is classified into one of eight flows *before* any boundary is consulted, so a boundary written only in the boundaries section is one the classifier has already run past.
- **`code-reviewer`** — three touches, and the prescribed edit was in the wrong place. A description-only fix would have missed the real bypass: the agent's **step-5 output format** ended by recommending *"`linear-task-agent` to open the PR"* — **the bypass written into the agent's own output, firing after routing had already succeeded.** Generalise it: *the load-bearing surface is wherever an artifact emits a recommendation, not only where it accepts a request.* Any boundary check that reads only descriptions will miss this class.

> **A lower-severity adjacency that needed no edit, recorded so nobody "fixes" it.** `conflict-resolver` claims *"merge main into my branch"* and *"fix the conflicts on this rebase"*; `open-pull-request` has **no rebase trigger**, so there is no collision. But *"rebase onto main and open the PR"* speaks both artifacts' language in one sentence, and **the conflict resolver has no notion of the gate around it** — it resolves, reports, and stops, and nothing tells the developer that the resolved hunks are now unreviewed code. The mitigation is inside the skill (it re-reviews resolved hunks) rather than in either description, because the utterance is genuinely ambiguous and narrowing either side would cost a real trigger.

> **`run-sprint-planning` collides with nothing, and that is a decision rather than luck.** Its lane was reserved in advance from two directions: the Phase 1 `skill-prompts/README.md` negative-routing check already lists *"estimate these stories for the sprint"* as work the requirement skills must not take, and `linear-task-agent` refuses sprint planning in **three** places — its description, its operating boundaries and its escalation list. **A deliberately reserved lane reads identically to an oversight unless it is written down**, which is the same reasoning that gave the deliberately-unclaimed diagram lane its row in the Phase 2 build. Do not add a bare tracker verb to it, and **do not restore the trigger *"is this ready to commit"*** — unqualified, it reads as a question about a working tree.

> **Both Phase 3 commands carry `disable-model-invocation: true`, so neither has an auto-trigger surface today.** The phrase-level check below is therefore the state of the world **if that field were removed** — which is precisely the scenario the two `command-prompts/README.md` files exist to prevent. `/load-task-context` would claim *"what's the context for ENG-247"* and *"pull the ADRs for this story"*, overlapping `linear-task-agent`'s read-only find flow, which returns those pointers without dereferencing them. `/write-module-readme` would claim *"document this module"* and *"write the README for src/orders"*, which nothing else claims — its exposure is the smaller of the two. **Verify the field is present rather than assuming the directory grants it.**

### Design-time artifacts

| Artifact | Type | Owns | Never owns | Provenance |
|---|---|---|---|---|
| `solution-architect` | Subagent | System-level architecture options, the 10x stress test on each, the recommendation, the proposal document | Per-story design, reviewing its own proposal, decision records, the schema, the API contract, diagrams | P2 |
| `architecture-reviewer` | Subagent | The independent production-readiness verdict on a design it did **not** author | Producing or revising the design, code diffs, WCAG, applying any fix it recommends | P2 |
| `accessibility-auditor` | Subagent | The WCAG 2.1 AA verdict on one rendered surface | Fixing violations, diff review, brand critique, **PASS without a rendered-page scan** | P2 |
| `render-design-diagrams` | Skill | The Eraser DSL, the PNG/SVG exports, the recorded editor URL, and the in-repo Mermaid mirror | The architecture decision itself, the schema, wireframes, Eraser workspace admin | P2 |
| `data-model-design` | Skill | Greenfield schema, its indexes, the migrations and the seed data | The ER diagram, the API contract, any schema work inside a story already in flight | P2 |
| `api-contract-freeze` | Skill | The OpenAPI spec, the mechanical audit, the mock URL, the **unsigned** freeze note | The schema underneath it, implementation, the freeze signature | P2 |
| `/adr` | Command | One decision record, with the next sequential number allocated and reported | Choosing between options, accepting a decision (Status is always `Proposed`) | P2 |

**`solution-architect` and `architecture-reviewer` split on direction of travel, not on topic, and the split is a gate requirement.** One produces the design; the other judges it. Merging them would void Gate 1's *"reviewed by ≥ 1 senior engineer who is not the author"* while appearing to satisfy it. Both descriptions exclude the other, and **`architecture-reviewer` additionally detects and refuses a same-session self-review** — because a description constrains routing, and once an agent is loaded no description constrains what it agrees to do.

**Three artifacts here own a *verdict*, and a verdict is a write.** `architecture-reviewer` and `accessibility-auditor` return one, and `api-contract-freeze` deliberately does not — it leaves the freeze note unsigned. The pattern worth keeping: an agent may compute a verdict from evidence it can actually obtain, and must have a token for "I could not obtain it" (`Unevaluated`, `PASS WITHHELD`). An agent that can only return pass-or-fail will manufacture whichever the room wants.

### Testing and quality

| Artifact | Type | Owns | Never owns | Provenance |
|---|---|---|---|---|
| `author-test-plan` | Skill | The test plan Document, its AC-to-test map, the gap check that gates publication, its read-back section anchors | Test code in any layer, auditing an existing suite's coverage, ownership assignment, the approval signature, any bug | P4 |
| `e2e-and-coverage-engineer` | Subagent | End-to-end specs, trace-file diagnosis of a failing run, the coverage-gap audit against a real coverage artefact. **Writes test files only** | Unit and integration tests, production code — including a `data-testid` — load/mobile tests, fixing the bug a failing test found, **any tracker write** | P4 |
| `load-test-engineer` | Subagent | One journey's load script, thresholds wired to cited NFR numbers, the run → read → tune loop, the verdict | Every other test type, profiling or optimising what fails, capacity planning, cost, **defining the SLO it is measured against** | P4 |
| `reproduce-and-diagnose-bug` | Skill | One failing regression test named after one triaged bug, the proof it fails for the right reason, the ranked root cause, the blast radius, the post-mortem seed | The fix, triage and severity, any tracker write, module and story tests, end-to-end suite health, the published post-mortem | P4 |

**Test-writing now has four claimants and the split is by *what the test is evidence about*, not by who writes it.** The implementation specialists write unit and integration tests **in the same commit as the code**; `e2e-and-coverage-engineer` writes end-to-end specs for **code that already shipped**; `load-test-engineer` writes one script that measures **the system under load**; `reproduce-and-diagnose-bug` writes **exactly one test, named after a bug, that must fail before it can pass**. "Does it write a test file?" separates none of them. The tests that do: *when relative to the code* and *what the assertion is about*.

**Three of the four refuse to hold both ends of their own loop, and that is the pattern to preserve.** `e2e-and-coverage-engineer` cannot edit the system its tests are evidence about; `load-test-engineer` cannot choose the target it measures against or fix what misses it; `reproduce-and-diagnose-bug` cannot apply the fix its own test is the acceptance criterion for. Each would be more convenient merged, and each merge deletes the only independent signal in its loop.

### Security and compliance

| Artifact | Type | Owns | Never owns | Provenance |
|---|---|---|---|---|
| `threat-modeler` | Subagent | The threat model for one system, service or feature: the STRIDE walk, the OWASP and AI/agentic cross-references, and the **P0/P1 mitigation list** the rest of the phase is measured against | Verifying its own mitigations landed, CVE triage, the fix, any Dockerfile or IaC verdict, production-readiness review, **any tracker write** | P5 |
| `dependency-risk-analyst` | Subagent | Reachability per dependency advisory — symbol, import sites, a named call path from a confirmed entry point, verdict with `file:line` — and major-version upgrade impact. Read-only | Container-image or base-layer CVEs, the manifest edit, the upgrade PR, dismissing an alert, **signing the risk acceptance** | P5 |
| `threat-model-verifier` | Subagent | The per-mitigation `file:line` evidence table and its verdict: Landed / Partial / Not landed / Unevaluated. Read-only | Producing or revising the threat model, hunting new vulnerabilities, implementing a missing mitigation, granting a deferral, **auditing a list from its own session** | P5 |
| `codeql-query-author` | Skill | One project-specific CodeQL query, its two fixtures, the proof it fires on one and not the other, its pack and suite wiring | The vulnerability itself, the built-in query packs, any severity downgrade, judging an existing finding true or false | P5 |
| `secret-leak-response` | Skill | One credential incident: the rotation runbook, the revocation proof, blast radius, the audit window, the notification list, the post-mortem seed | Executing the rotation, preventative scanner or secret-manager setup, **any history rewrite before revocation is proven**, the disclosure decision | P5 |

**`threat-modeler` and `threat-model-verifier` are the `solution-architect` / `architecture-reviewer` split in a second place, and it is load-bearing for the same reason.** One states what should be built; the other reads code and says whether it was. Merging them would satisfy the phase-handoff checkbox while voiding it, because an author reads intent as evidence. `threat-model-verifier` additionally **detects and refuses a same-session self-audit** — a description constrains routing, and once an agent is loaded nothing in its description constrains what it agrees to do.

**Four of the five carry a "cannot evaluate" token, and that is now the house pattern rather than a per-artifact choice.** `dependency-risk-analyst` returns **undetermined**, which escalates exactly like reachable; `threat-model-verifier` returns **Unevaluated** per row and **`PASS WITHHELD`** as a verdict; `threat-modeler` lists the components it could not reach by name; `codeql-query-author` states what its two fixtures did **not** prove. Same property as `PASS WITHHELD` in testing and seven-of-nine in containers: **an artifact that can only return pass-or-fail will manufacture whichever one the room wants.**

**Nothing in this section reviews a container image or an IaC diff, and that is deliberate.** Phase 5's Gate 4 has no scanner, no policy engine and no admission controller behind it — it is a named human reviewer against a fixed required-controls list. All three security subagents refuse that question explicitly, on the same footing as `container-image-engineer`'s scan-verdict refusal one section down. **A helper here would make an unscanned surface look gate-approved**, which is the one failure this whole map exists to prevent.

### Infrastructure and pipelines

| Artifact | Type | Owns | Never owns | Provenance |
|---|---|---|---|---|
| `terraform-iac-engineer` | Subagent | Resource HCL, modules, plan-only tests; loops until plan is clean. **Holds no apply credential** | The `backend` block, the state store, CI credentials, anything in `.github/workflows/`, `apply` | P6 |
| `iac-state-backend-bringup` | Skill | The `backend` block, state store, lock, state IAM roles, `.gitignore` state entries | Resource HCL, workflows, CI credentials | P6 |
| `ci-identity-and-secrets-bringup` | Skill | OIDC provider and trust policy, secret-manager wiring, the drift workflow | The state store, build workflows, the Infracost workflow, resource HCL | P6 |
| `cost-guardrails-bringup` | Skill | The Infracost workflow, budget alerts, the unit-economics metric, the cost reviewer checklist | The cost estimate itself, any other workflow, approving a costly PR | P6 |
| `cicd-pipeline-bringup` | Skill | Build / test / deploy-to-staging workflows, `@claude` wiring, agentic workflows, branch protection | The drift workflow, the Infracost workflow, the production gate | P6 |
| `deploy-and-rollback-bringup` | Skill | The production approval gate, deploy strategy, rollback trigger, the drill | The build pipeline, SLO definitions | P6 |
| `container-image-engineer` | Subagent | One service's Dockerfile and `.dockerignore`; self-certifies 7 of 9 standards, names the other 2 | CVE counts, scan verdicts, image-size comparisons — **under any circumstances** | P6 |
| `observability-bringup` | Skill | Dashboards, SLOs, alerts, alert runbooks under `/docs/runbooks/alerts/` | Instrumentation code, service runbooks under `/docs/runbooks/services/`, incident work | P6 |

**`.github/workflows/` has four claimants and the split is by file, not topic** — build/test (`cicd-pipeline-bringup`), drift (`ci-identity-and-secrets-bringup`), Infracost (`cost-guardrails-bringup`), production gate (`deploy-and-rollback-bringup`). A topic-level boundary here would produce four artifacts that each believe they own "the CI setup".

---

## Contested surfaces

Where two artifacts could plausibly take the same sentence. These are the rows worth re-reading before editing any description.

| Utterance | Goes to | Not to | Because |
|---|---|---|---|
| "create the issues for this epic" | `push-linear-stories` | `linear-task-agent` | Backlog construction from a requirements document, not work on a story in flight |
| "file a bug for that auth race" | `linear-task-agent` | `push-linear-stories` | Discovered while working a story; no requirements document behind it |
| "write the tests for this" | `frontend-engineer` / `backend-engineer` | anything else | Ships in the same commit as the code |
| "review this PR for cost" | *nothing yet* | `code-reviewer` | `cost-guardrails-bringup` deliberately contains **no** form of "review" in its triggers, precisely to avoid this collision. It produces a checklist a human uses |
| "set up terraform for this repo" | `iac-state-backend-bringup` | `terraform-iac-engineer` | The backend must exist before resources; the subagent assumes it does |
| "add the S3 bucket module" | `terraform-iac-engineer` | `iac-state-backend-bringup` | Resource HCL, not backend bring-up |
| "write the runbook" | `observability-bringup` **if alert-triggered** | — | Alert runbooks live under `alerts/`; service runbooks under `services/` are Phase 3-owned and are a different artifact class |
| "how many CVEs in this image" | *nothing* | `container-image-engineer` / `dependency-risk-analyst` | **No scanner exists in this stack.** Both agents must refuse rather than answer; nothing downstream would contradict a fabricated figure. **Two claimants now, and neither takes it** — recorded as unowned-on-purpose so the next author does not read the gap as an oversight and fill it |
| "design the architecture for ENG-247" | the per-story design specialist | `solution-architect` | A story identifier against an existing codebase. The seam is **scope**, not the word "design" |
| "is this design ready to build" | `architecture-reviewer` | `solution-architect` | A proposal cannot review itself and still satisfy "not the author" |
| "add the migration for ENG-412" | `backend-engineer` | `data-model-design` | In-story schema work, shipped in the story's own commit. The design skill claims **greenfield only**, which is why its triggers name the requirements document |
| "review this screen" | `accessibility-auditor` **if against WCAG** | `code-reviewer` | Split on what is reviewed: a rendered interface against a published standard, never a diff |
| "is the contrast OK here" | `accessibility-auditor` — **answered as unmeasured** | — | No agent can compute rendered contrast from source. The honest answer names the colour pair and asks for the scan |
| "draw the architecture diagram" | a project-scope `architecture-diagram` skill where installed | `render-design-diagrams` | Deliberately left with the incumbent — see the collision table below. The framework skill claims the Eraser round-trip, not the bare phrasing |
| "freeze the API contract" | `api-contract-freeze` | a project-scope `system-design-architect` agent | Skill beats agent on auto-trigger; decide at install which one the repo keeps |
| "open the PR for this branch" | `open-pull-request` | `linear-task-agent` | The tracker agent **posts** the PR; it does not run the gate in front of it. Closed in the agent's classification step, not only its description — `pr-open` is chosen before any boundary is read |
| "anything to fix before I PR" | `code-reviewer` | `open-pull-request` | A diff review with **no PR at the end of it**. The skill excludes bare "review my diff" by name, which is what leaves the reviewer its own lane in the reciprocal direction |
| "ready to PR" · "can this go up for review" | `open-pull-request` | `code-reviewer` | There *is* a PR at the end of it. A clean review is step 2 of 7, and it reads as *ready* to anyone in a hurry |
| "rebase onto main and open the PR" | `open-pull-request` | `conflict-resolver` | Both artifacts' language in one sentence. The resolver owns the conflict; it has no notion of the gate around it, and resolved hunks are unreviewed code until the skill re-reviews them |
| "estimate these stories for the sprint" | `run-sprint-planning` | `linear-task-agent` / the requirement skills | Outer-loop cycle preparation. The lane was reserved in advance in three places on one side and one on the other — see the collision set above |
| "re-estimate ENG-247, it's bigger than we thought" | *nothing* | `run-sprint-planning` | The story has left `Backlog`. The skill refuses: a committed number is what velocity is calibrated against, and overwriting it makes the machine guess and the team's number indistinguishable |
| "document this module" | `/write-module-readme` — **explicitly fired only** | — | The command carries `disable-model-invocation: true`, so today this utterance routes nowhere. Listed because removing that field would make it a live claim |
| "do we have a test for every AC" | `author-test-plan` | `e2e-and-coverage-engineer` | Plan-time, against the acceptance criteria as published. The map is a table in a document, not a measurement of code |
| "where are our test gaps" | `e2e-and-coverage-engineer` | `author-test-plan` | Shipped code plus a **real coverage artefact**. Same word, opposite end of the phase — and the skill wins any genuine overlap by auto-trigger, so the seam has to hold in the skill's description, not only the agent's |
| "write the test plan" | `author-test-plan` | `frontend-engineer` / `backend-engineer` | One word from the reserved *"write the tests"*. **Every trigger on this skill carries "test plan" or "test strategy"** — that qualifier is the entire boundary, exactly as the tracker skills always name the requirements document |
| "this E2E spec is flaky in CI" | `e2e-and-coverage-engineer` | `reproduce-and-diagnose-bug` | Suite health, diagnosed from a trace. The bug skill starts from a reported **product defect** with a tracker issue behind it |
| "the checkout page is broken in prod, find out why" | `reproduce-and-diagnose-bug` **once triaged** | `e2e-and-coverage-engineer` | A bug report with no failing test attached. The agent reads traces from test runs; it must not take this |
| "can we handle 10k concurrent users" | `load-test-engineer` | — | Measurement against cited targets. **"How many instances will we need at 10k"** is capacity planning and routes nowhere — the agent measures, it does not size |
| "just tell me if staging passes the p95" | `load-test-engineer` — **answered as `PASS WITHHELD`** | — | Unless staging is declared performance-matched. The honest answer reports every measured number and withholds the verdict, in the `PASS WITHHELD` / `Unevaluated` family above |
| "what are the security risks in this design" | `threat-modeler` | `architecture-reviewer` | Adversarial walk versus production readiness across scalability / reliability / operability / cost. Same input, different lens — **and both sides were already deconflicted by work type before `threat-modeler` shipped**, so no coordinated edit was needed |
| "did the P0 mitigations land" · "close out the threat model" | `threat-model-verifier` | `threat-modeler` | The one seam that must never close for convenience: an author reading their own recommendation will accept intent as evidence, and two gate checkboxes turn on this table |
| "is this CVE actually reachable" · "which of these 40 alerts matter" | `dependency-risk-analyst` | `threat-modeler` | A third-party advisory against code that exists, not a design that may not be built yet |
| "can we accept the risk on the not-reachable ones" | *nothing* | `dependency-risk-analyst` | The agent supplies the call path; the acceptance is a **Tech Lead signature**. An acceptance resting on an agent's summary has nobody behind it |
| "add a CodeQL rule so this can't recur" | `codeql-query-author` | — | Free — nothing else claims CodeQL. **But bare "lint for this pattern" was cut from its triggers**: it fires on somebody wanting an ESLint rule and returns a committable, plausible, entirely wrong artefact |
| "make sure nobody does this again" | *nothing* | `codeql-query-author` | Cut from the skill's triggers at build time. It is the closing sentence of a bug post-mortem, where the prevention action belongs to the bug procedure and its named owner — a query author hijacking it substitutes a static-analysis rule for a process decision |
| "do we need to rewrite git history" | *nothing* | `secret-leak-response` | Unqualified, this is squashing commits or dropping a large binary. The skill's trigger is *"scrub **this key** out of git history"* — **every history-related trigger names the credential**, or a fifteen-minute incident procedure lands in an ordinary git turn |
| "we need to rotate our secrets" | split on **incident vs setup** | — | A leaked credential is `secret-leak-response`; wiring the secret manager and CI identity is `ci-identity-and-secrets-bringup`, which already excludes *"rotating a leaked credential during an incident"* by name. **Lane reserved from the other side before the skill existed** |
| "scan the Dockerfile for misconfigurations" | *nothing* | any Phase 5 helper | The Gate 4 human reviewer. All three security subagents refuse by name — see the section note above |

---

## Reserved verbs

A verb in this list is spoken for. **Never add it to a new artifact's trigger phrases** without resolving the collision first.

| Verb / phrase | Owner |
|---|---|
| implement, build the feature, scaffold the component | `frontend-engineer` / `backend-engineer` |
| review my diff, anything to fix before I PR | `code-reviewer` — **a diff review with no PR at the end of it** |
| ready to PR, run the pre-PR checks, can this go up for review, ship this branch | `open-pull-request` |
| refactor | `refactor-specialist` |
| resolve the conflicts, fix the rebase | `conflict-resolver` |
| my next task, move ENG-XXX to, comment progress, open the PR | `linear-task-agent` — **but "open the PR" only after `open-pull-request` has run the gate** |
| plan the next sprint, size the backlog, decompose this XXL story | `run-sprint-planning` — every trigger names the sprint, the cycle or the backlog. **Never add "is this ready to commit"** |
| file a bug, log a follow-up, track this for later | `linear-task-agent`. **Not `file-followup-bug`** — see the note below |
| apply, destroy, terraform state | **nobody** — no agent holds an apply credential by design |
| commit the DSL, the editor URL, the PNG/SVG export | `render-design-diagrams` |
| freeze the spec, stand up the mock server | `api-contract-freeze` |
| write the ADR, record the decision, document why we chose X | `/adr` |
| write the test plan, what's our test strategy, gap-check the plan | `author-test-plan` — **every trigger carries "test plan" or "test strategy"; never shorten one to a bare "write the tests"** |
| add an E2E test, why is this Playwright test failing, run the coverage audit, where are our test gaps | `e2e-and-coverage-engineer` |
| write the load test, run the k6 script, are we meeting our p95 | `load-test-engineer` |
| reproduce ENG-XXX, write a failing test for this bug, rank the root causes | `reproduce-and-diagnose-bug` — **on an already-triaged bug only.** "How bad is this one" stays a human severity call |
| threat model this, STRIDE this service, review the AI surface for prompt injection | `threat-modeler` |
| did the mitigations land, verify the P0 mitigations, close out the threat model | `threat-model-verifier` — **never the agent that wrote the list** |
| triage these Dependabot alerts, is this CVE reachable, what breaks if we go to v5 | `dependency-risk-analyst` |
| write a CodeQL query, add a custom CodeQL rule, this query is too noisy | `codeql-query-author` — **every trigger names CodeQL, the query, or code scanning. Never shorten one to a bare "lint for this pattern" or "make sure nobody does this again"** |
| we leaked a key, a token got committed, the secret scanner fired, is this key still live | `secret-leak-response` — **every history-related trigger names the credential; never a bare "rewrite git history"** |
| accept the risk, sign the risk acceptance, defer this mitigation | **nobody** — a dated Tech Lead signature by design, on the same footing as the apply credential |
| how many CVEs in this image, scan the Dockerfile, is this IaC compliant | **nobody** — no scanner, policy engine or admission controller exists in this stack. Claiming it is the failure the container-and-IaC gate is built around |
| draw the architecture diagram, diagram this flow, generate a C4 diagram | **deliberately unclaimed** — left to a project-scope diagram skill. Claiming it is what re-opens the collision above |

> **`file-followup-bug` is not a shipped artifact, and this file said it was.** The row above previously read *"`linear-task-agent` (and `file-followup-bug` where built)"*, which reads as a slug awaiting a build. It is not: it exists only as the **worked teaching example** in `03-development/PROCESS.md` (L363) — the illustration used to show what a `SKILL.md` looks like — plus references in this file, in [PROMPT-CONVERSION-ANALYSIS.md](./PROMPT-CONVERSION-ANALYSIS.md) and in a Phase 1 README. **No directory contains it and none is planned to.** The follow-up-filing capability is `linear-task-agent`'s `file-followup` flow, capped at one issue arising from the story in flight. The Phase 3 build's `refactor-specialist` edit was prescribed to route to this name and shipped **restated by work type** instead, precisely to avoid minting a fourth reference to a template that does not exist — the same dangling-slug defect the `pr-reviewer` fix removed in v2.4. **Do not name it in a shipped template.**

Two naming patterns keep the surface small and are worth preserving:

- **The `-bringup` suffix is a routing asset.** No ordinary development utterance contains "bring up", and it makes the subject a *repository capability* rather than a *unit of code*.
- **Tracker-writing skills always name the requirements artifact** ("from the PRD", "the backlog structure") rather than a bare tracker noun. That is what keeps them clear of the story-loop agent sharing the same MCP server.

---

## Known collisions in consuming repos

Predicted, not observed — both trees below are gitignored, so these are what happens in *any* repo that already followed the framework, not a live defect here.

| New artifact | Collides with | Status |
|---|---|---|
| the three requirement skills | a project-scope `requirement-gathering` skill | Unverified — run the negative-routing check after install |
| `cicd-pipeline-bringup` | a project-scope `cicd-devops` skill | Mitigated at build: its triggers deliberately do **not** lead with "CI/CD". Any edit reintroducing "set up CI/CD for this repo" re-opens it |
| `render-design-diagrams` | a project-scope `architecture-diagram` skill | **Mitigated at build, with a residual risk that is an install-time decision.** Every trigger now names the Eraser round-trip or the two-surface obligation; bare "draw the architecture diagram", "diagram this flow", "generate a C4 diagram" and "update the diagram for X" are left with the incumbent. **Residual:** in a repo with both, that bare phrasing produces the Mermaid mirror only — one gate line satisfied, three (DSL, editor URL, exports) silently not, and it looks done. Either uninstall the incumbent or reserve it for sketches. **Re-opened by** adding any of those four phrases, or stripping the Eraser/DSL/export qualifier off a trigger that has one |
| `data-model-design`, `api-contract-freeze` | a project-scope `system-design-architect` agent | **Newly recorded.** That agent claims "produce the data model / OpenAPI contract / ADRs" and "freeze the API contract" as invocation phrases. Skills beat agents on auto-trigger, so the skills take those utterances — the gate-favourable outcome — but the agent's own boundaries stop running for that work. A silent behaviour change in any repo with both; decide at install |
| `solution-architect` | a project-scope `system-design-architect` agent | Same agent, third overlap — it also claims system-level option generation. Its scope is the union of three framework artifacts, so a repo keeping it should retire it rather than run both |
| `e2e-and-coverage-engineer` | a project-scope `qa-test-engineer` agent | **Closed at build by the rename**, which is why it shipped under this name and not the obvious one. The incumbent carries the **opposite** scope — it writes unit and integration tests alongside the code — so a same-name ship would have replaced it with an agent that refuses exactly the work it existed to do, and the swap would be silent |
| `author-test-plan` | **a partial Phase 4 install** | Same shape as the `open-pull-request` row below, one directory over. `author-test-plan` names `e2e-and-coverage-engineer` by slug in two refusal cases; installing `skill-prompts/` without `subagent-prompts/` leaves both pointing at an agent the repo does not have. Same-phase slugs never dangle **only if the phase is installed as a phase** — installation is per-directory |
| `open-pull-request` | **an already-vendored copy of `linear-task-agent` or `code-reviewer`** | **A collision class the others do not have, because Phase 3 shipped across two builds.** Its seven subagents vendored into consuming repos long before the skill existed, so the edits that close the collision set above are in templates those repos already have an older copy of. **This is the only entry in this table that is not predicted — it is certain** for any repo that installed Phase 3 before this build and does not re-copy. Symptom: the PR opens with no self-review, no DoD walk and no rebase, and looks entirely normal. Re-copy `subagent-prompts/` alongside `skill-prompts/`, or treat `/open-pull-request` as explicitly invoked |
| `threat-modeler` | a project-scope `security-reviewer` agent | **Newly recorded, and the prediction that opened it is now testable.** That agent claims broad security-review utterances, and *"what are the security risks here"* is inside both surfaces. Both are subagents, so neither auto-triggers over the other — the exposure is an operator invoking the wrong one by name and getting a generic review where a classification-gated STRIDE walk was needed, or the reverse. Decide at install which one the repo keeps; running both means two P0 lists and no agreed one |
| `threat-model-verifier` | **a partial Phase 5 install** | Same shape as the `author-test-plan` row above. `threat-model-verifier` reads the mitigation list `threat-modeler` writes, and both directory READMEs describe the pair as a pair. Installing the verifier without the modeller leaves an auditor with nothing to audit; installing the modeller alone re-opens the unfalsifiable handoff checkbox this pair was built to close. **Copy `subagent-prompts/` as a directory, not by file** |
| `secret-leak-response` | *nothing* | **Recorded as a non-collision on purpose.** `ci-identity-and-secrets-bringup` already excludes *"rotating a leaked credential during an incident"* in its own description, so the incident lane was reserved from the other side before this skill existed. **A deliberately reserved lane reads identically to an oversight unless it is written down** — do not "tidy" that exclusion out of the Phase 6 skill |
| `open-pull-request` | **a partial Phase 3 install** | Not a name collision but the same failure. `open-pull-request` names five sibling subagents by slug; `linear-task-agent` and `code-reviewer` name it back. **Installing `subagent-prompts/` without `skill-prompts/` leaves two templates pointing at a skill the repo does not have** — and the reverse leaves a skill pointing at five absent agents. The repo's "same-phase slugs never dangle" rule assumes a repo installs *the phase*; installation is per-directory, so the rule needs that qualifier |

---

## Adding a new artifact

1. Write the description first. If you cannot state what it owns **and** what it must never take, it is not one artifact.
2. Grep this file's reserved-verb table for every trigger phrase you drafted.
3. Check the write-type axis: does anything already write the same object? If yes, the boundary goes in **both** descriptions, in the same PR.
4. Run the negative-routing check — ask for ordinary work in plain language and confirm the new artifact does not load. **For a command this check does not apply** — but only because `disable-model-invocation: true` is set; verify the field is present rather than assuming the directory grants it.
5. Add a row here. **An artifact that ships without a row in this file has no recorded boundary**, which means the next author cannot avoid colliding with it.

> **A deliberately unclaimed lane is a routing decision and belongs in the tables above.** Phase 2 left bare diagram phrasing to a project-scope incumbent rather than fighting for it. Recorded in the reserved-verb table as unclaimed-on-purpose, it survives the next author; recorded nowhere, the first person who thinks "we should trigger on *draw the diagram*" re-opens a collision that was closed on purpose.

> **Maintenance rule.** This file is only worth what its last update is worth. Any PR that ships, renames, or retires an artifact edits this file in the same PR — the same rule the per-phase READMEs' Wraps columns already carry.
