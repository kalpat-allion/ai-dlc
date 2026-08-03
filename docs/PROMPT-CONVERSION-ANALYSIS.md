# Prompt → Skill / Subagent Conversion Analysis

> **What this is.** An assessment of all ~100 prompts across the seven `aidlc-phases/*/PROMPTS.md` files, classifying each one as a candidate Claude Code **skill**, **subagent**, **slash command**, or as a prompt that should stay exactly as it is.
>
> **Status:** **22 of the 34 proposed artifacts are built** and ship as templates under `aidlc-phases/*/`. Together with Phase 3's seven specialist subagents — which predate this analysis and are therefore outside the 34 — that is **29 shipped artifacts** in the repo, the figure [`ROUTING.md`](./ROUTING.md) scopes to. Phases 1, 2, 3 and 6 are complete; Phases 4, 5 and 7 are unbuilt. See [Build order](#build-order). *(This line previously read "analysis only. No skills, subagents, or commands have been authored" — false since the v2.4 Phase 6 build, and left standing through three subsequent ones.)*

>
> **Method.** Seven per-phase classification passes against the framework's own doctrine at [`03-development/PROCESS.md` §"Creating your own Claude Code skill"](../aidlc-phases/03-development/PROCESS.md) (L338-429), a cross-phase synthesis pass to collapse duplicates, and two adversarial review passes — one on routing quality, one on operational cost. 42 initial proposals survived down to 30.
>
> **Revision — Sentry removed from the framework.** Two prompts are deleted (`sentry-context-pull`, `Inherited Error Triage`), taking the corpus from 100 to **98**. The proposed `diagnose-production-bug` and `triage-inherited-errors` skills die with them; `reproduce-and-diagnose-bug` and `load-test-engineer` are added in Phase 4. Artifact count stays 30. See [Sentry removal — consequences](#sentry-removal--consequences).
>
> **Revision — third-party security tooling removed.** The prescribed security stack narrows to **GitGuardian + ggshield plus GitHub-native tooling only**, and **CodeQL is promoted from fallback to the prescribed SAST baseline**. Three Phase 5 prompts are deleted (`Reachability Triage`, `Container CVE Triage`, `OPA Policy Generation`), taking the corpus from 98 to **95**. `Semgrep Custom Rule Generation` is *rewritten* as `CodeQL Custom Query Generation`, so `semgrep-rule-author` is renamed **`codeql-query-author`** and survives intact; `author-opa-policy` and `dependency-risk-analyst` die, and `Dependency Upgrade Impact` reverts to a kept-as-prompt. Artifact count 30 → **28**. See [Security tooling removal — consequences](#security-tooling-removal--consequences).
>
> **Correction — one kill reversed, one gap closed.** Two corrections to the revision above, both in Phase 5. **(1) `Reachability Triage` was deleted in error and is restored.** The kill rested on a false premise: the prompt's own header note read *"use when Endor Labs / Snyk reachability is **unavailable** and you need a triage cut on a long CVE list"* — it was the **no-tool fallback**, not a tool-dependent prompt, and the reachability reasoning it prescribes was always Claude's job. Re-sourced on Dependabot, it comes back, and **`dependency-risk-analyst` is un-killed** wrapping it plus `Dependency Upgrade Impact` (which moves back from kept-as-prompt). `Container CVE Triage` stays deleted — that verdict was correct and is unchanged. **(2) A new subagent, `threat-model-verifier`, closes a gate defect**: two Phase Handoff checkboxes assert that P0 threat-model mitigations landed in code, and nothing verified them. It wraps a **newly authored** prompt, `Threat-Model Mitigation Verification`. Corpus 95 → **97**; artifacts 28 → **30**. See [Security tooling removal — consequences](#security-tooling-removal--consequences).
>
> **Revision — Phase 1's three P0 skills are now BUILT.** `publish-prd-to-linear`, `scaffold-linear-milestones` and `push-linear-stories` ship as templates under `aidlc-phases/01-requirement-gathering/skill-prompts/`, wired into that phase's PROCESS Step 0, Steps 2.2-2.4 and Stages 3a-3c, and into Gates 1 and 2 with a committed-and-smoke-tested checkbox each. **No prompt was deleted, renamed or rewritten** — the paste path stays valid, so this revision changes no counts: still 96 prompts and 34 artifacts, with **18 shipped artifacts rather than 15** — *shipped* rather than *converted*, which is the distinction the [Verdict](#verdict) now makes explicit; of the 34, this build took the figure from 8 to 11. `sweep-requirements-gaps` is deliberately not built; its own stated condition is that these three run on a real PRD first. **Two deferred decisions had their triggers fire on this build** — see [Build order](#build-order) Phase 0 item 3. See [Phase 1 build — consequences](#phase-1-build--consequences).
>
> **Revision — Phase 2 is now BUILT, and the slash-command tier exists.** All **seven** Phase 2 artifacts ship together: `solution-architect`, `architecture-reviewer` and `accessibility-auditor` under `subagent-prompts/`; `render-design-diagrams`, `data-model-design` and `api-contract-freeze` under `skill-prompts/`; and **`/adr` — the first slash command in the repo**, which establishes the `command-prompts/` convention two further commands were queued behind. **No prompt was deleted, renamed or rewritten**, so counts are unchanged: still 96 prompts and 34 artifacts, with **25 shipped artifacts rather than 18** — of the 34, this build took the figure from 11 to 18. See the [Verdict](#verdict) for why the two populations need naming. The `software-architect` coordinated edit landed with them. Phase 2 is the first phase to ship all three artifact tiers at once, and the first to **reuse** placeholder names rather than coin new ones. **One premise this document relied on in three places turned out to be false** — see [Phase 2 build — consequences](#phase-2-build--consequences).
>
> **Revision — Pulumi removed (framework v2.4), and Phase 6 is now BUILT.** Pulumi leaves the prescribed framework entirely; Terraform becomes the IaC primary with OpenTofu as the CLI-compatible fallback. One prompt is **deleted** (`Pulumi Cost Delta` — Infracost supports Terraform natively, so its own header stated its obsolescence condition), taking the corpus from 97 to **96**, and one is renamed and rewritten (`Pulumi IaC Generation` → `Terraform IaC Generation`, anchor `#pulumi-iac-generation` → `#terraform-iac-generation`). Phase 6's artifacts grow **4 → 8** and move from *proposed* to *shipped*: `pulumi-iac-engineer` becomes **`terraform-iac-engineer`**, the proposed single `iac-foundation-bringup` **splits** into `iac-state-backend-bringup` + `ci-identity-and-secrets-bringup`, and `cost-guardrails-bringup` + `deploy-and-rollback-bringup` are added so no capability removed with Pulumi silently drops. Artifacts 30 → **34**. `observability-bringup` is **unblocked** — the `/docs/runbooks/` conflict is resolved. See [Pulumi removal — consequences](#pulumi-removal--consequences).
>
> **Revision — Phase 3's remaining four artifacts are now BUILT, and it is the first phase to ship across two builds.** `open-pull-request` and `run-sprint-planning` ship under `03-development/skill-prompts/`; `/load-task-context` and `/write-module-readme` under `command-prompts/` — the second directory to adopt the convention `/adr` established. **No prompt was deleted, renamed or rewritten**, so counts hold: still 96 prompts and 34 artifacts, with **22 of the 34 built rather than 18**, and **29 shipped artifacts in the repo rather than 25**. *(Those are two different populations, and the banners above were conflating them — see the [Verdict](#verdict).)* The two bundled coordinated edits — `linear-task-agent` **(a)** and `code-reviewer` — land with them, and the phase's three known documentation defects are fixed in the same change. **Phase 3 is the first phase to have shipped its artifacts across separate builds**: its seven specialist subagents shipped before this analysis existed, its two skills and two commands now. Every other phase shipped as one set, and the split has a cost worth naming — the coordinated edits that close `open-pull-request`'s routing had to be made against templates consuming repos have already vendored, so an install that keeps an older copy of `linear-task-agent` or `code-reviewer` re-opens a gate bypass that looks like nothing at all. See [Phase 3 build — consequences](#phase-3-build--consequences).

---

## Verdict

**55 of 96 prompts convert into 34 artifacts** — 20 skills, 11 subagents, 3 slash commands. **41 stay exactly as they are.**

**22 of the 34 are built; 12 remain.** Phase 6's eight shipped alongside the v2.4 Pulumi removal — see [Pulumi removal — consequences](#pulumi-removal--consequences) for why a vendor migration turned out to be the right moment to build a phase's helpers — Phase 1's three P0 skills shipped next, as the first artifacts built in this document's own recommended order, Phase 2's seven shipped as one set, and Phase 3's remaining four shipped last. Counted as **shipped artifacts in the repo** rather than as conversion outputs the number is **29**, because Phase 3's seven specialist subagents predate this analysis and are excluded from the 34 — 29 is the figure [`ROUTING.md`](./ROUTING.md) scopes to. The **12 remaining** are all in Phases 4, 5 and 7, plus Phase 1's held `sweep-requirements-gaps`.

> **This paragraph previously read *"25 of the 34 are built … Nine remain"*, and both halves were wrong.** It put Phase 3's seven already-shipped specialists into the numerator while the 34 excludes them from the denominator, then derived the remainder by subtraction instead of by counting entries — which is why the enumeration that followed it ("Phases 4, 5 and 7 plus two Phase 3 commands") could not add up to nine under any reading. Recounted from the entries: **14 of the 20 skills, 5 of the 11 subagents and 3 of the 3 commands** are built, which is 22. The 12 unbuilt are `sweep-requirements-gaps`; `author-test-plan`, `reproduce-and-diagnose-bug`, `load-test-engineer`, `e2e-and-coverage-engineer`; `secret-leak-response`, `codeql-query-author`, `threat-modeler`, `dependency-risk-analyst`, `threat-model-verifier`; `finalise-documentation`, `handoff-agent`. **Two populations were being counted with one number**, and the fix is to name which one each figure belongs to rather than to pick a winner.

The governing insight came from the adversarial review. [PROCESS.md L404](../aidlc-phases/03-development/PROCESS.md) establishes that a skill's description **is the entire routing surface** — skills *auto-trigger* on user intent, while subagents are *name-invoked*. It follows that any skill whose trigger list reuses a verb one of the seven shipped Phase 3 subagents already claims **wins that route by default and hijacks ordinary work**. That single rule killed or narrowed six proposals that otherwise looked sound, and it is the main reason the conversion rate is 58% rather than 90%.

The second-largest constraint is input availability. Roughly a quarter of the prompts consume a blob no tool in the mandated stack can fetch — a meeting transcript, a vendor pricing sheet, a billing export, a dashboard reading. Wrapping those in a gated artifact does not remove toil; it makes fabricated figures look gate-approved.

### Coverage at a glance

| Phase | Prompts | → Skill | → Subagent | → Slash cmd | Kept as prompt |
|---|---|---|---|---|---|
| 1 — Requirement Gathering | 12 | 9 *(8 of them via 3 shipped skills)* | 0 | 0 | 3 |
| 2 — System & Architecture Design | 15 | 6 *(via 3 shipped skills)* | 4 *(via 3 shipped subagents)* | 1 *(shipped)* | 4 |
| 3 — Development | 18 | 6 *(via 2 shipped skills)* | 0 *(7 shipped before this analysis)* | 2 *(both shipped)* | 10 |
| 4 — Testing & QA | 14 | 5 | 4 | 0 | 5 |
| 5 — Security & Compliance | 13 | 2 | 5 | 0 | 6 |
| 6 — CI/CD & DevOps | 15 | 6 *(all shipped)* | 2 *(both shipped)* | 0 | 7 |
| 7 — Delivery & Handoff | 9 | 2 | 2 | 0 | 5 |
| **TOTAL** | **96** | **35 → 20 skills** | **17 → 11 subagents** | **3 → 3 commands** | **41** |

Grouping is where the value sits: 35 prompts collapse into 20 skills and 17 into 11 subagents. A skill that wraps four prompts is worth far more than four separate skills.

Priority split **of what remains**: **P0 = 1** — `e2e-and-coverage-engineer`, now the last of them, since `open-pull-request` and `run-sprint-planning` shipped with the Phase 3 set and Phase 1's three P0 skills and Phase 2's `/adr` before that. **P1 = 8**: `author-test-plan`, `reproduce-and-diagnose-bug`, `load-test-engineer`, `secret-leak-response`, `threat-modeler`, `dependency-risk-analyst`, `threat-model-verifier`, `finalise-documentation`. **P2 = 3**: `codeql-query-author`, `sweep-requirements-gaps`, `handoff-agent`. That is 12, which reconciles with the Verdict.

> **The previous split — *"P0 = 3, P1 = 9, P2 = 6 — plus Phase 6's 8 and Phase 2's 7"* — double-counted.** `accessibility-auditor` was carried as a P2 *and* inside Phase 2's seven, so the three tiers plus the two phase sets summed to 35 against a total of 34. Stating the split over **unbuilt artifacts only** removes the ambiguity permanently: a shipped artifact has no priority, and a priority queue that also lists shipped phase sets is two lists wearing one heading.

**Phase 6's skill count is 6, not 2, and only three of the six wrap a prompt at all.** `iac-state-backend-bringup`, `ci-identity-and-secrets-bringup`, `cost-guardrails-bringup` and `deploy-and-rollback-bringup` wrap **prose PROCESS steps**, not `PROMPTS.md` entries — which is why the phase's skill count exceeds what a prompt-by-prompt reading would predict. Worth noting as a limit of this document's method: **a conversion analysis that only reads the prompt library will systematically under-count the skills a phase needs**, because the procedures most worth encoding are often the ones nobody wrote a prompt for.

### Directory convention

Siblings to the existing `subagent-prompts/`, per phase:

```
aidlc-phases/<phase>/
  PROCESS.md  PROMPTS.md  QUALITY-GATES.md  FLOWCHART.md
  subagent-prompts/<name>.md          # Phases 3, 6 and 2
  skill-prompts/<name>/SKILL.md       # Phases 6, 1, 2 and 3 — folders, not files
  command-prompts/<name>.md           # Phases 2 and 3 — `/adr` established the convention
```

**Phase 3 is the only phase carrying all three directories**, and it is also the only one where the three tiers reference each other by slug: `open-pull-request` names five subagents in its body and two of those subagents name it back. The instantiation step therefore copies all three directories in one action rather than treating skills and commands as optional extras — see [Phase 3 build — consequences](#phase-3-build--consequences) §6.

Like `subagent-prompts/`, these are **templates, not active artifacts** — Claude Code only auto-discovers `.claude/agents/`, `.claude/skills/` and `.claude/commands/`. Each directory gets a README mirroring the Phase 3 one, and templates stay repo-agnostic with `{{UPPER_SNAKE_CASE}}` placeholders so they can be copied into consuming repos.

---

## Recommended new artifacts

### Skills

#### `open-pull-request` — **SHIPPED** · Phase 3

- **Wraps:** Self-Review Before PR, pr-description, **plus the rebase half of PROCESS 4.1, which has no prompt**
- **Path:** [`aidlc-phases/03-development/skill-prompts/open-pull-request/SKILL.md`](../aidlc-phases/03-development/skill-prompts/open-pull-request/SKILL.md)
- **Gate:** P3 [Gate 2 (PR Merge)](../aidlc-phases/03-development/QUALITY-GATES.md#gate-2-pr-merge), pre-merge half
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** PROCESS.md L358 uses `open-pull-request` verbatim as its exemplar skill name, and L349 routes "open a PR" to a skill. It sequences seven steps across four shipped subagents that no single agent owns.
- **What changed on build:**
  - **It came in at seven steps, not five, and the two additions are the ones the gate actually turns on.** A new **step 1** reconciles the identifier on the branch against the story the developer names and **stops on a mismatch without picking** — a one-character difference de-links the PR from the tracker and can auto-close an unrelated issue on merge. And the fix loop **re-runs the review against the fixed diff** before anything else proceeds: the first report's projected post-fix counts are a prediction, not a measurement, and *a count nobody re-measured is how a Critical reaches a reviewer.* Neither step exists in either wrapped prompt or in PROCESS 3.5.
  - **`Closes` is wrong for a multi-PR story, and the pre-build description above prescribed it unconditionally.** It read *"`Closes ENG-XXX` in the body"*; the source `pr-description` prompt offers `Closes` **or** `Part of`. Always writing `Closes` auto-closes the story at the first of several merges — a one-word default with a silent, irreversible effect. The skill asks *"is this the only PR for this story?"* before composing the body and is forbidden from assuming.
  - **Rebase invalidates the review, and neither the specification nor the source prompts noticed.** The prescribed order is review → rebase → open. A rebase that resolves a conflict introduces hunks the review never saw, and the conflict resolver produces a *resolution report*, not a review — so the branch reaches the PR with unreviewed code in it. The skill **re-reviews the resolved hunks** rather than reordering the steps, which keeps it consistent with the phase's own process: *resolution code is code no reviewer has ever read.*
  - **It hands `linear-task-agent` the finished title and body verbatim, to post unmodified** — not a payload to draft from. See the [Phase 3 build — consequences](#phase-3-build--consequences) §4 defect: the two specify incompatible PR bodies, and re-drafting is exactly where the closing keyword and the AI-generated-sections notation go missing.
  - **It does not claim two of Gate 2's lines and says so.** The coverage-threshold and SAST checkboxes are automated CI lines that run *after* the PR opens. The gate is annotated rather than left implying full coverage — the `container-image-engineer` seven-of-nine shape, reached again.

#### `run-sprint-planning` — **SHIPPED** · Phase 3

- **Wraps:** linear-sprint-pull, Estimation, estimates-to-linear, story-decomposition
- **Path:** [`aidlc-phases/03-development/skill-prompts/run-sprint-planning/SKILL.md`](../aidlc-phases/03-development/skill-prompts/run-sprint-planning/SKILL.md)
- **Gate:** P3 [Gate 1 (Sprint Commitment)](../aidlc-phases/03-development/QUALITY-GATES.md#gate-1-sprint-commitment) — **three of its five criteria**
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** **Verified in the shipped template** — `linear-task-agent` refuses sprint planning three times: its description says *"sprint-planning prompts run by humans"*, [line 19](../aidlc-phases/03-development/subagent-prompts/linear-task-agent.md) states *"You do NOT run sprint-planning prompts (sprint pull, estimates roll-up, story decomposition)"*, and it repeats as a refusal case at **line 40**. It cannot be a subagent extension and cannot route into the existing Linear writer. *(This entry previously cited line 37, which was wrong before this build and is now line 40 after it — the boundaries section gained a bullet.)*
- **What changed on build:**
  - **It answers three of Gate 1's five criteria, not five, and the report says which two it did not.** AC ≥ 3 and unambiguous, estimate set, XXL decomposed are evaluable. **Velocity calibration is a human conversation with data the skill does not hold**, and **owner assignment is a staffing decision** — both come back `not mine to judge`, and refusing to answer them *is* the deliverable. Three criteria out of five is not a commitment, and a report that quietly marked five would have manufactured one. Same shape as `container-image-engineer`'s seven-of-nine; the gate was annotated to match rather than left implying full coverage.
  - **One proposed trigger was cut as a routing defect.** *"Is this ready to commit"*, unqualified, is far more likely to be a question about a working tree than about a cycle — it would pull a cycle-planning procedure into an ordinary git turn. Replaced with *"are these stories ready for the cycle"*; **every surviving trigger names the sprint, the cycle or the backlog.** Contrast `render-design-diagrams`, where **no** subset of the proposed triggers survived: here the fix was one phrase, because the collision was with ordinary English rather than with an incumbent artifact.
  - **A state check per story, however the story arrived.** Step 1 pulls `Backlog` only, but a story handed to the skill directly bypasses that filter — so step 2 re-checks and drops anything already in flight. A story in flight carries a committed number, and re-sizing it mid-cycle rewrites the history the team calibrates against.
  - **It refuses to size below three AC bullets — which collides with `story-decomposition`'s floor of five.** See [Phase 3 build — consequences](#phase-3-build--consequences) §3: a 3-4-bullet story is estimable, can come back XXL, and then cannot legally be decomposed while Gate 1 requires XXL stories to be decomposed. The skill neither invents seams nor waives the line; it records the decomposition line as `not met` and sends the story back for AC.

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
- **Why:** Split out of an original five-prompt `build-linear-backlog` proposal at the human approval gate. Inlining five prompts under the repo-agnostic shipping rule produces ~160 lines of prompt body alone, against a two-screen limit (PROCESS.md L412) and an empirical house ceiling of 111 lines. Terminating at the existing approval stop is the natural, non-arbitrary cut. **The split is vindicated on build:** the two halves came in at 37 and 54 lines, so the merged version would have run to ~85 with two confirm-before-write stops and two human gates inside one procedure — past the limit and past the single-procedure rule, exactly as predicted.
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

#### `render-design-diagrams` — **SHIPPED** · Phase 2

- **Wraps:** Eraser Architecture Diagram (via MCP), Eraser ER Diagram (via MCP)
- **Path:** `aidlc-phases/02-system-design/skill-prompts/render-design-diagrams/SKILL.md`
- **Gate:** P2 Gates 1, 2, 4 (diagram artifacts and locations)
- **Description:**
  > Use when a system, cloud, sequence, or entity-relationship diagram needs to be produced and landed in the repo — generates it through the Eraser MCP server, commits the DSL under the repo's diagram directory, exports PNG/SVG, generates the equivalent in-repo Mermaid so it renders in markdown, and records the Eraser editor URL. Triggers on "diagram this schema", "generate the C4 container diagram", "ER diagram for the new tables", "render the sequence diagram for checkout", "update the diagrams to match the new design". Do NOT use for: deleting or reorganising an Eraser workspace, designing the architecture itself, or UI mockups.
- **Why:** Runs 7+ times per project and is the only concrete mitigation for the diagram-drift risk PROCESS.md names. Generation is one step; the gate-bearing obligation around it is six, across two surfaces and two PROCESS steps.
- **✅ Collision resolved at build, and the resolution is not the one the ⚠ predicted.** Every trigger originally proposed here collided with the project-scope `architecture-diagram` skill — "generate the C4 container diagram", "ER diagram for the new tables", "render the sequence diagram for checkout", "update the diagrams to match the new design" all fall inside its four claimed phrases. **No subset of the proposal survived.** A rename was considered and rejected on a distinction worth keeping: the two *names* never collided, so renaming leaves all four colliding phrases exactly where they were. **Renaming fixes name collisions; this was a trigger collision.** What made narrowing honest is that the artifacts genuinely differ — the incumbent produces in-repo Mermaid only, this skill produces Eraser DSL + PNG/SVG + editor URL + the Mermaid mirror, and the incumbent satisfies exactly one of the four gate lines that count them. Had both produced the same files, no wording would have turned two owners into one.
- **The residual risk was recorded rather than papered over.** In a repo with both installed, bare "draw the architecture diagram" still routes to the incumbent and produces the mirror only — **one gate line satisfied, three silently not, and it looks like the diagram got done.** That is an install-time decision (uninstall the incumbent, or reserve it for sketches), stated in the phase's Step 0 rather than left to be discovered at the gate.

#### `data-model-design` — **SHIPPED** · Phase 2

- **Wraps:** Entity Extraction, Schema Generation, Migration Generation
- **Path:** `aidlc-phases/02-system-design/skill-prompts/data-model-design/SKILL.md`
- **Gate:** P2 Gate 2 (five checkboxes — the densest Phase 2 coverage)
- **Description:**
  > Use when turning approved PRD requirements into a database schema — extracts entities, ownership and cardinalities from the PRD, generates the schema in the repo's ORM with indexes covering the stated top query patterns, stops for a mandatory backend-developer review, then generates the up/down migration plus seed data and runs it against a local database. Triggers on "design the data model from the PRD", "extract the entities from the PRD", "we need the data model before we build". Do NOT use for: schema changes or migrations inside a story mid-implementation (use `backend-engineer`), denormalising without a measured query and an ADR, running migrations against anything but a local database, or rendering the ER diagram.
- **Why:** Textbook "first, then, then, finally". **Triggers deliberately narrowed:** the original "write the migration for the new tables" / "generate the schema" collided with `backend-engineer`'s shipped trigger "add the migration for ENG-XXX" — and because skills auto-trigger, the skill would have won and dragged a greenfield PRD chain into an ordinary story turn.
- **What changed on build:**
  - **The wrapped `Schema Generation` prompt has a deliverable that belongs to another artifact.** Its item 7 asks for a Mermaid `erDiagram` alongside the schema — but `render-design-diagrams` owns both diagram surfaces and the gate lines that count them. Left in both, the ER diagram is written twice by two procedures and **the second writer wins silently.** The prompt is unchanged; the skill hands the schema over instead. This is the Phase 1 `Gap Analysis` finding in a new shape: **a prompt carrying a deliverable that belongs elsewhere is invisible until you inline it**, because as a paste prompt nobody else was ever going to write that file.
  - **The mid-procedure review stop is a human, and had to be defended as one.** PROCESS 2.5 says "walk the schema with a backend developer"; nothing in the framework hands that to an agent, and handing it to one would void it exactly as merging `architecture-reviewer` into `solution-architect` would void "not the author". The step carries an explicit *"do not continue because the reviewer is unavailable"* clause — **the failure mode is unavailability, not disagreement**, and only the first one looks like a reason to proceed.

#### `api-contract-freeze` — **SHIPPED** · Phase 2

- **Wraps:** API Contract
- **Path:** `aidlc-phases/02-system-design/skill-prompts/api-contract-freeze/SKILL.md`
- **Gate:** P2 Gate 2 (eleven API lines); metric: 0 contract change-requests after frontend start
- **Description:**
  > Use when an API contract needs to be designed, audited, mocked and frozen before frontend work starts — maps PRD user flows to a resource model, generates an OpenAPI 3.1 spec with the team's error envelope, pagination, auth and idempotency conventions, audits every operation mechanically against the API gate checklist, stands up the mock server and records its URL in the README, then records the freeze and the change-request path. Triggers on "design the API contract", "generate the OpenAPI spec", "stand up the mock server", "audit the spec before we freeze it". Do NOT use for: implementing endpoints, changing an already-frozen spec without the documented change request, or designing the data model behind it.
- **Why:** Wraps one prompt but absorbs four prompt-less PROCESS steps and a real business metric. Deliberately not merged with `data-model-design` — the combined chain is eight steps with two human stops, well past the two-screen limit.
- **Its drop condition was evaluated on build and did not fire — but only because of a step the spec did not have.** The condition was *"build it if the audit is not genuinely mechanical"*. Audited line by line against Gate 2's eleven API checkboxes: **8 are mechanical outright** (flow coverage, request/success/seven-error responses, single error envelope, pagination on lists, `x-prd-section`, `operationId` + naming, mock URL in README, freeze marker + change-request doc). **1 is not the skill's** — the Tech Lead and backend-developer signature. **And 2 are not mechanical as written:** *"auth on every **protected** endpoint"* and *"Idempotency-Key on POSTs that **require** it"*. Both turn on a product judgement — which endpoints are public, which POSTs are non-idempotent — and **an agent that infers them and then audits its own inference passes every time.**
  - The fix is a **new step 2** that refuses to proceed until a human declares the public-endpoint list and the non-idempotent-POST list. With both in hand the audit is genuinely mechanical and the artifact earns its keep; **without it, this skill is a fabrication engine wearing a checklist** — which is precisely what the drop condition was pointing at without being able to name it. The two Gate 2 lines were reworded to read against the declared lists, because as written they were unfalsifiable.
  - **It also grew a seventh step and the gap was real, not cosmetic.** The proposed six ended at the freeze note and came in under band at 27 lines. PROCESS 3.3's human Swagger walkthrough had no home, and the skill was implicitly claiming that a passing mechanical audit means the spec is *right*. It does not — **the audit proves completeness; only a reader proves correctness.** Step 6 now leaves the freeze note **unsigned** and step 7 hands over the audit result, the mock URL and the note.

#### `author-test-plan` — P1 · Phase 4

- **Wraps:** Test Plan Generation, test-plan-gap-analysis, test-plan-to-linear-document
- **Path:** `aidlc-phases/04-testing-and-qa/skill-prompts/author-test-plan/SKILL.md`
- **Gate:** P4 Gate 1 (Test Plan Approved)
- **Description:**
  > Use when a QA Lead or Tech Lead needs the test plan for a project or release produced and published. One procedure: fetch the PRD and architecture via Linear MCP rather than pasting, draft the plan with an AC-to-test map, gap-check it against the source ACs and NFRs, resolve every Critical/High, then publish it as a Linear Document with stable section anchors. Entering at "check the test plan for gaps" or "publish the test plan to Linear" resumes the procedure at that step; publication is gated on the gap check clearing. Triggers on "write the test plan", "what's our test strategy", "do we have a test for every AC". Do NOT use for: writing test code, auditing coverage of an existing suite, or silently revising an already-published plan.
- **Why:** The third prompt refuses to run without the second's clearance — hard-gated steps must not be advertised as alternatives (PROCESS.md L429). The "fetch via MCP rather than paste" instruction is what defeats the context-hungry objection.

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

#### `solution-architect` — **SHIPPED** · Phase 2 · `opus`

- **Wraps:** Architecture Proposal, Trade-off Interrogation
- **Path:** `aidlc-phases/02-system-design/subagent-prompts/solution-architect.md`
- **Gate:** P2 Gate 1 (≥2 options, 10x stress-test, no unmitigated SPOF)
- **Description:**
  > Use this agent for the system-level architecture pass at the start of a project or a major new subsystem: produce 2-3 candidate architectures (components, communication, data architecture, infrastructure, bounded contexts, trade-offs at launch and 10x, top risks), stress-test each seriously-considered option at 10x load for first-break point, single points of failure, cost cliffs, state contention and failure isolation, and end with a justified recommendation stopped at an explicit Tech Lead approval gate. Read-only apart from writing the proposal document. Invoke on "propose an architecture for this product", "what are our architecture options", "stress-test option 2 at 10x", "design the system before we pick a stack". Do NOT invoke for: per-story design inside an existing codebase (use `software-architect`), the independent production-readiness review (use `architecture-reviewer`), STRIDE / OWASP threat modelling (use `threat-modeler`), schema work, OpenAPI contracts, or writing the ADR (use `/adr`).
- **Why:** **Not a duplicate** — `software-architect`'s shipped description explicitly escalates *"system-wide architectural decisions / new tech choices / new service boundaries / new datastores"* **out** of its remit. Trade-off Interrogation folds in because PROCESS.md 1.3 permits it as an in-conversation follow-up.
- **What changed on build: the Trade-off Interrogation is a mandatory step, not the optional follow-up it folds in as.** It runs on **every seriously-considered option** and **before** the recommendation, for two reasons the spec did not state. A stress test run *after* the recommendation argues against a decision already anchored — the model has committed, and the test becomes a defence rather than a filter. And Gate 1 asks for the interrogation on *"the chosen option"*, which is circular: **the stress test is part of how you choose.** Running it only on the winner tests the one option whose weaknesses are least likely to change the outcome.

#### `architecture-reviewer` — **SHIPPED** · Phase 2 · `opus`

- **Wraps:** Design Review
- **Path:** `aidlc-phases/02-system-design/subagent-prompts/architecture-reviewer.md`
- **Gate:** P2 Gate 1 (0 Critical, 0 High open; reviewed by ≥1 senior engineer who is not the author)
- **Description:**
  > Use this agent for the independent pre-build production-readiness review of a chosen system architecture: produce a severity-ranked (Critical / High / Medium / Low) issue list across scalability, security, reliability, operability and cost, each with a concrete recommendation and the PRD section affected, ending in a one-line PASS / PASS WITH FIXES / FAIL verdict plus Critical and High counts. Read-only. Invoke on "review this architecture for production readiness", "is this design ready to build", "design review before the architecture gate". Do NOT invoke for: generating the architecture options (use `solution-architect`), STRIDE / OWASP threat modelling (use `threat-modeler`), reviewing a code diff (use `code-reviewer`), per-story design, WCAG audits, or applying the fixes.
- **Why:** Deliberately **not merged** into `solution-architect` — a same-session follow-up to the proposal cannot honestly satisfy "not the author"; merging would void the gate while appearing to satisfy it. Main failure mode is verdict inflation, so severity must be defined by consequence, not tone.
- **What changed on build — three, and the first is the one that mattered:**
  - **"Not merged" was a directory fact, not a behaviour, until the agent could detect the violation.** This document put the independence property in the *why*, and the description says "independent". Neither is enforcement: **the agent has no way to know it is reviewing something the same session produced unless it is told to check and stop.** That check now sits in the operating boundaries and again in the escalation list. Generalises past this pair — *a boundary that depends on session history cannot be expressed in a description, because a description constrains routing and this violation happens after routing succeeded.*
  - **The verdict is derived from the counts, not chosen.** The source prompt names three verdict words and never says which findings produce which — so verdict inflation was not a risk the prompt failed to *prevent*, it was **undefined, and any verdict was defensible.** Now: PASS = 0 Critical / 0 High; PASS WITH FIXES = 0 Critical and every High has a pre-build fix; FAIL otherwise. Paired with a consequence-based severity table and one demotion rule — *a finding with no attachable consequence is Medium at most.*
  - **A mandatory `Unevaluated` section.** The source prompt carries *"flag anything you cannot evaluate from the inputs given"* as one sentence among ten. As prose it evaporates under a thin input set — which is exactly when it matters. Promoted to a numbered step and a named report section, the `container-image-engineer` seven-of-nine precedent reached independently.

#### `accessibility-auditor` — **SHIPPED** · Phase 2 · `opus`

- **Wraps:** Accessibility Review (Claude)
- **Path:** `aidlc-phases/02-system-design/subagent-prompts/accessibility-auditor.md`
- **Gate:** P2 Gate 3 (WCAG 2.1 AA, 0 Critical / 0 High); Gate 4 artifact
- **Description:**
  > Use this agent to audit a component, screen, or page against WCAG 2.1 Level AA: findings as Issue / Severity / Recommendation / WCAG reference across perceivable, operable, understandable, robust and tap-target criteria, ending in a PASS / PASS WITH FIXES / FAIL verdict. Read-only — never patches markup, styles, or ARIA. Invoke on "run an accessibility review on this screen", "is this component WCAG AA", "check keyboard navigation and contrast on the booking flow", "a11y audit before sign-off". Do NOT invoke for: fixing the violations (use `frontend-engineer`), general correctness / security / performance review of a diff (use `code-reviewer`), or visual / brand critique.
- **Why:** No existing or proposed agent covers WCAG, and its input is fetchable from the repo with no human paste. P2 on an honest limitation: a source-only audit cannot compute rendered contrast, so it must state its limits and require an automated axe run rather than asserting a PASS.
- **What changed on build — the honesty property needed teeth, and it needed a gate decision:**
  - **"State its limits and require an axe run" is satisfiable by a template that states its limits and then returns PASS anyway.** Built instead as a **verdict-vocabulary restriction**: from source alone the only reachable verdicts are `FAIL` and `PASS WITH FIXES` — both safe to be wrong about — plus a **fourth token, `PASS WITHHELD`**, when the rendered-page scan or the keyboard pass is missing. PASS is the verdict that stops anyone else looking, so it is the one that must be unreachable without evidence.
  - **The fourth token is not in the source prompt and was not in the gate.** Gate 3 read *"Accessibility review passes WCAG 2.1 AA (0 Critical, 0 High)"*, under which `PASS WITHHELD` is correctly not a pass — but only by inference, and a reader in a hurry infers the other way. The gate now says so explicitly. **A new verdict token is a gate change, not a template detail**, and shipping one without the matching gate edit would have been the unfalsifiable-checkbox pattern arrived at from a third direction.
  - **The unmeasurable set is five items larger than this document assumed.** It names contrast. Building the can/cannot table found reflow at 320 CSS px, text resize to 200%, computed tap-target size, focus order against *visual* reading order, and what an assistive technology actually announces — all equally unreachable from source. **An agent told only "you cannot do contrast" will confidently assert the other five.** The table is the enforcement, on the `container-image-engineer` nine-row precedent.

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

#### `/adr` — **SHIPPED** · Phase 2

- **Wraps:** Architecture Decision Record Generation
- **Path:** `aidlc-phases/02-system-design/command-prompts/adr.md`
- **Gate:** P2 Gates 1, 3, 4; also consumed by Phases 5 and 6
- **Description:**
  > Generate an Architecture Decision Record for a decision just made — takes the decision, options considered, constraints and the PRD anchor; reads the repo's ADR directory to allocate the next sequential number; emits Status (always Proposed) / Context / Decision / Consequences (positive, negative, neutral) / Alternatives Considered with a specific rejection reason per alternative / Validation signal; writes it and surfaces the number it picked. Refuses when no alternative was genuinely rejected, and never fabricates a numeric claim in Consequences.
- **Why:** **Highest leverage-per-unit-of-build-cost on the entire list** — deterministic, one argument, no branching, fired 11+ times per project, cited by three gates.
- **What changed on build — the premise behind "no auto-trigger competition" was false, and is now true only because the template makes it true.** Verified against the current documentation: **custom commands have been merged into skills.** A file at `.claude/commands/adr.md` and a skill at `.claude/skills/adr/SKILL.md` both create `/adr` and work the same way, and **by default both the user and Claude can invoke either.** A command's description is therefore a routing claim exactly like a skill's. The shipped template sets **`disable-model-invocation: true`**, which is what makes it user-invocable only.
  - The claim was not softened — it was **made true and the mechanism written down.** But three statements in the repo were inaccurate as written and are corrected: this document's *"the only artifact that faces no auto-trigger competition at all"*, `03-development/PROCESS.md` L324's tier table (which distinguished a command from a skill on invocation shape alone — **since fixed with the Phase 3 build, now L350**), and `ROUTING.md`'s framing of commands as inherently explicitly-fired.
  - **The tier is now a convention held in one frontmatter field, not a property of the directory.** `/load-task-context` and `/write-module-readme` inherit the field, not just the folder — and the `command-prompts/README.md` says so in place, because the next author will otherwise reasonably assume the directory grants it.
- **Sequential numbering needed a git read, not just a directory listing.** A record added on an unmerged branch is invisible to `ls`, and that is exactly how two ADRs written the same afternoon collide. The command checks `git log --all --diff-filter=A` over the ADR directory, reports the number **and what it allocated from**, and states the resolution rule for the residual two-unpushed-branches case: **rename the new file at merge, never renumber a record someone has already read.**

#### `/load-task-context` — **SHIPPED** · Phase 3

- **Wraps:** task-context
- **Path:** [`aidlc-phases/03-development/command-prompts/load-task-context.md`](../aidlc-phases/03-development/command-prompts/load-task-context.md)
- **Gate:** none directly — feeds P3 [Gate 2](../aidlc-phases/03-development/QUALITY-GATES.md#gate-2-pr-merge) "Architecture alignment verified"
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** The only part of the story-start choreography no shipped subagent owns (steps 2.1-2.3 already live in `linear-task-agent`). Was P2 because it is downstream of tracker hygiene — built after the Phase 1 backlog skills, so the deep-links it reads exist.
- **What changed on build:**
  - **It ships a third conflict value, `unchecked`, and names it the load-bearing one.** The proposed shape had a conflict column and a "not found" report, which leaves an unreadable source with a blank or absent conflict — and blank reads as *fine*. Every row now carries exactly one of a **named conflict** (quoting the AC bullet and the contradicting line), **`none found`** (the source was read in full), or **`unchecked`** (not found, or unreadable). **A record nobody opened cannot agree with anything, and `none found` on a row nobody read is the precise failure this command exists to prevent.** The gate line it feeds was reworded to match: *a row marked `unchecked` is a source nobody read, not a source that agreed.*
  - **One row per endpoint, never one row for "the API".** The source prompt says "the OpenAPI section for any endpoint touched", which a model satisfies with a single summary paragraph. A single row cannot carry a per-endpoint conflict, so the one endpoint whose contract disagrees with the AC disappears into the average.
  - **The absent pointer is itself a finding.** A story that cites no requirement section must be reported as citing none — not resolved by going to look for the section it probably meant and then reporting that as the citation. That is the fabricated-citation failure in its most helpful-looking form.

#### `/write-module-readme` — **SHIPPED** · Phase 3

- **Wraps:** Documentation Generation (P3)
- **Path:** [`aidlc-phases/03-development/command-prompts/write-module-readme.md`](../aidlc-phases/03-development/command-prompts/write-module-readme.md)
- **Gate:** P3 [Gate 2](../aidlc-phases/03-development/QUALITY-GATES.md#gate-2-pr-merge) (READMEs updated at the documentation threshold); [Gate 3](../aidlc-phases/03-development/QUALITY-GATES.md#gate-3-phase-completion)
- **Description:** see the shipped template; the frontmatter is the authoritative version.
- **Why:** **Reinstated after review.** It was killed for failing the auto-trigger test, but PROCESS.md L404 scopes that requirement to *skills*; L350 defines a slash command as "one prompt with arguments" with no trigger obligation. Threshold detection is the human's job at the DoD step. Marginal cost was near zero once `/adr` established the convention.
- **What changed on build:**
  - **An unenforceable judgement was replaced with a lookup.** *"Omit any section with nothing real to say"* is satisfiable by a run that omits nothing, and **a padded README reads as a thorough one** — the reviewer's incentive points the wrong way. Each of the seven sections now names the artifact that licenses it: import edges in **both** directions for Architecture, the line that actually reads the env var for Configuration, a real exported symbol with its real signature for Usage, an in-code `TODO`/`FIXME` or a labelled tracked issue for Limitations, a test invocation **that was actually run** for Testing. *"Nothing real to say"* becomes *"the grep returned nothing"*, which is checkable.
  - **All seven sections are reported back, written-with-evidence or omitted-with-what-was-looked-for.** A section that simply vanishes is indistinguishable from one nobody considered, which is how padding returns on the next run. **The report is half the deliverable**, and Gate 3's line now says so: a README with four sections and three documented omissions satisfies it; one with all seven and no report does not.
  - **The shape test was the closer call of the two commands, and it passes.** Seven evidence rows look like a branching checklist. They are not: they are **output admission criteria applied uniformly in one pass**, with no ordering between them and no human gate. The line for the next author is recorded in place — a `--section` argument, an ask-the-developer step, or a threshold-detection step moves it to `skill-prompts/`.

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
| post-mortem | 4 | **Re-evaluated on merit and settled.** Fails on *shape*, not convention cost: six input blocks and two refusal rules is not "one prompt with arguments" (PROCESS.md L350). Its two refusals are exactly what `reproduce-and-diagnose-bug` guarantees — fold its checklist in as that skill's terminal step. Two of six input fields also lose their source post-Sentry |
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
| `wireframe-flow` (skill) | Both | No supported hand-off mechanism — `frontend-design` is a globally-enabled official Skill, and skills match the **user's** prompt (L404), not another skill's instruction. Its Gate 3 discipline already lives in `02-system-design/QUALITY-GATES.md:56,99,103`. Stack-locked to React+Tailwind+shadcn by the gate text. |
| `triage-refactor-candidates` (skill) | Routing lens | **Verified payload mismatch.** Its terminal step files through `linear-task-agent`, whose file-followup flow requires a parent issue, refuses on a missing parent identifier, and refuses follow-ups resolvable in the active diff — a module-wide scan has no parent. PROCESS.md L420 calls this class "worse than no skill". |
| `quarterly-kb-review` (skill) | Ops lens | Runs 90+ days after the delivery team has left, needing three simultaneous OAuth grants plus a vendored `linear-task-agent`. P7 PROCESS.md:706 names MCP non-transfer as the phase's own top risk. |
| `build-linear-backlog` (5-prompt skill) | Ops lens | Cannot satisfy the two-screen rule (L412) once framework links are stripped — ~160 lines of prompt body before procedure, against an empirical house ceiling of 111. **Split** into `scaffold-linear-milestones` + `push-linear-stories` at the existing human gate. |
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

**Eight** coordinated edits to already-shipped `subagent-prompts/` templates — the header read "Six" until the `linear-task-agent` row split in two and the three-Phase-1-skills row was added, neither of which updated it. Each must land in the **same PR** as the artifact it unblocks, because these files ship into consuming repos.

> **The rule is narrower than it reads, and the repo has now applied it that way four times.** What it forbids is shipping a **pointer to an artifact the consuming repo does not have** — a dangling reference. It does not reach an edit that is a **pure refusal**, which names no artifact, unblocks nothing, and is strictly safer standalone than absent. The `software-architect.md` row was split on exactly this basis in v2.4 (its `pr-reviewer` dangling-slug fix shipped alone while its `solution-architect` exclusion stayed bundled), the `linear-task-agent.md` row was split the same way, the **Phase 2 build split the remaining `software-architect` half too** — restating it by work type rather than naming `solution-architect` — and the **Phase 3 build did the same to `refactor-specialist`**, which had been prescribed to name `file-followup-bug`, a template that does not exist. Stated as a test: **if removing the artifact from the sentence leaves the sentence still true and still useful, the edit can ship alone.**
>
> **Four instances is enough to invert the default.** The "Ships with" column should read *standalone* for any edit that is a pure refusal or a work-type restatement, and bundle only where a slug is genuinely named. **After Phase 3 the bundled set is down to one row** — the `threat-modeler` ↔ `architecture-reviewer` mutual exclusion, which is bundled because both sides name a slug and neither sentence survives its removal. The two rows that named `open-pull-request` have now landed with it.

| Shipped template | Edit | Ships with |
|---|---|---|
| ~~`linear-task-agent.md` **(a)**~~ | ~~pr-open flow gains *"if the pre-PR self-review has not been run, load `open-pull-request` first"*.~~ **LANDED with `open-pull-request`, as five touches rather than one.** The prescribed edit was to the pr-open flow alone; shipped as the description, the boundaries section, the **request-classification step**, the pr-open flow and the escalation list. The classification step is the load-bearing one for the same reason it was in half (b): `pr-open` is chosen before any boundary is read, so `pr-open` itself now carries *"presupposes the pre-PR gate has already run"*. Its description contains "open the PR for this branch" verbatim, so **without this the subagent wins the route and the entire gate is bypassed**. | `open-pull-request` — shipped together |
| ~~`linear-task-agent.md` **(b)**~~ | ~~Boundaries name Documents / Projects / Milestones as out-of-scope **by design**, not by omission.~~ **LANDED, split from (a).** Shipped as four additive touches — description, request-classification step, operating boundaries, escalation list — plus the reciprocal clause in all three Phase 1 skill descriptions. See [Splitting the `linear-task-agent` row](#splitting-the-linear-task-agent-row). | — (shipped standalone) |
| ~~`code-reviewer.md`~~ | ~~Do-NOT list gains *"the full pre-PR gate including DoD, rebase and PR open (load `open-pull-request`, which hands the diff review back here as step 1)"*.~~ **LANDED — and the prescribed edit was in the wrong place, which is this build's sharpest finding.** It was description-only; the agent's **step-5 output format** ended by recommending *"`linear-task-agent` to open the PR"* — the gate bypass written into the agent's own output, firing **after** routing had already succeeded, where no description can reach it. Shipped as three touches: the description, step 5, and the escalation list. Two corrections of fact: the diff review comes back **as step 2**, not step 1 (step 1 is the branch/story identifier reconciliation), and the skill has seven steps, so a developer who stops at a clean review has run **step 2 of 7**. | `open-pull-request` — shipped together |
| `backend-engineer.md` | Operating boundaries gain the three security-remediation constraints (minimal backward-compatible diff; mandatory regression test failing before and passing after; no tautological symptom-masking fix). **Replaces** the killed `security-fix-engineer`. | — |
| ~~`software-architect.md`~~ | ~~Mutual *"Do NOT invoke for: system-wide / new-tech / new-service-boundary decisions (use `solution-architect`)"*.~~ **LANDED with the Phase 2 build — and it does not name `solution-architect`.** A cross-phase slug in a vendored template dangles in a repo that installed Phase 3 but not Phase 2, which is the same defect the v2.4 `pr-reviewer` fix removed from the line immediately before it in the same sentence. Shipped as *"escalate to human architect review, **or to the team's system-level architecture pass where one is set up**"*. **By this table's own narrowing test it therefore passes as standalone** — it names no artifact and unblocks nothing. | — (shipped with Phase 2, but no longer needed bundling) |
| ~~`refactor-specialist.md`~~ | ~~Description gains the filer's name: route candidate-identification to the `file-followup-bug` skill (canonised at PROCESS.md L336), not to an unnamed actor.~~ **LANDED — but not by name, because `file-followup-bug` does not exist as a shipped template.** It appears only as PROCESS.md's worked teaching example (L363), in `ROUTING.md`, in this document and in a Phase 1 README — never as a `SKILL.md`. Naming it would have reproduced the exact `pr-reviewer` dangling-slug defect v2.4 removed. **Shipped restated by work type** — *"the repo's follow-up-issue filing procedure, the one that files a single tracked issue arising from work in flight and hands the tracker write to `linear-task-agent`"* — the Phase 2 `software-architect` form, which passes this table's own standalone test. **The same defect appeared a second time in that template's escalation list**, where *"file as a new Linear issue"* instructed an agent explicitly forbidden from calling the tracker; fixed in the same form. **Replaces** the killed `triage-refactor-candidates`. | — (shipped standalone) |
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

> **On the `03-development/PROCESS.md` line numbers throughout this document.** The Phase 3 build added ~46 lines to that file, so every citation into it moved. **Live citations have been re-read against the current file and updated** — the doctrine landmarks are now L349/L350 (tier table), L358 (`open-pull-request` as exemplar name), L404 (description-as-routing-surface), L408 (open with the role), L412 (two-screen limit), L420 (hand-off integrity), L429 (single-procedure rule), L442 (verification checklist). **Struck entries below keep their pre-fix numbers**, which is where the defect was. A line number is a citation with a shelf life, and this is the second build to invalidate a batch of them; the durable fix is to cite by section heading, which the Method note above now does.

- ~~**`03-development/PROCESS.md` L353** — the worked skill example has a **broken relative path** (`../..` from `.claude/skills/<name>/` resolves to `.claude/`, not the repo root) **and** an anchor `#bug-report` that matches no heading in `PROMPTS.md`.~~ **FIXED with the Phase 3 build — and the fix covered three references, not two.** A third lived on the worked example's own `description:` frontmatter line (*"formatted to PROMPTS.md → bug-report"*), which no link check would ever have caught because it is prose, not a link — and it is the copy that ships, since the frontmatter travels into the consuming repo while the body's links do not. All three are restated inline. The authoring rule that produced them went too: item 4 read *"Reference, do not duplicate — link to PROMPTS.md, PROCESS.md and template files by relative path"*, which is the instruction that generates a dangling link **every time it is followed**, and now reads *"Inline the procedure; never link out of the skill folder into the framework."*
- ~~**`03-development/PROCESS.md` L412** — the skill verification checklist lists **four** pre-commit tests, omitting **hand-off integrity** (documented at L392) — the exact test that killed `triage-refactor-candidates`.~~ **FIXED with the Phase 3 build.** The checklist now names **five**, and the fifth earned its place in the same build: the `pr-description` ↔ `linear-task-agent` PR-body mismatch is a hand-off-integrity failure that no other test would have surfaced. *(Now at L442; the test it was omitting is at L420.)*
- ~~**`03-development/subagent-prompts/software-architect.md:3`** — routes users to `pr-reviewer`, a subagent slug that exists nowhere in the repo.~~ **FIXED in v2.4** — now routes to "the team's post-open PR review tooling", matching the convention `code-reviewer.md:3` already used and accurate for a repo that may have chosen CodeRabbit, the Claude bot, or both.
- ~~**`/docs/runbooks/` ownership is contested** across Phases 3, 6 **and** 7, with three filename schemes.~~ **RESOLVED in v2.4.** The diagnosis was wrong in a useful way: this was never three competing schemes for one artifact, but **two distinct artifact classes collapsed into one flat directory, plus one typo**. Phase 3's `<service>.md` is a *service operational* runbook; Phase 6's `<alert-name>.md` is a *per-alert response* runbook; Phase 7's `<alert>.md` was a shortened token, not a competing claim. Resolution: `/docs/runbooks/alerts/<alert-name>.md` (Phase 6 owns) and `/docs/runbooks/services/<service>.md` (Phase 3 owns), with Phase 7 a **consumer, never an owner**. Subdirectories rather than a filename fix because a flat directory admits a real collision — a service named `checkout` and an alert named `checkout` produce the same path, with different audiences and reviewers. The load-bearing half is the derivation rule: `<alert-name>` is the alert's name **as configured in the backend, lowercased and kebab-cased**, which turns Gate 4's *"every alert links to a runbook"* from an eyeball check into a script. **Unblocks `observability-bringup`.**
- **`docs/planning/` does not exist** but is referenced by `README.md` and by the `ai-sdlc-framework-architect` agent's mandatory-reading table.
- ~~**Links into a `templates/` directory are broken framework-wide**~~ — **FIXED. Surfaced by the Phase 2 build as three dead ADR links; the real scope was 23 links across all seven phases and the root `README.md`.** Two distinct defects were tangled together:
  1. **Wrong relative depth.** `../templates/` from `aidlc-phases/<phase>/` resolves to `aidlc-phases/templates/`, **which has never existed**. That is every link in Phases 1, 3, 4 and 7. The `../../templates/` form used by Phases 5 and 6 resolves correctly to the repo root.
  2. **The target is gitignored.** Root `templates/` *does* exist locally and holds exactly three files — `code-review-checklist.md`, `prd-template.md`, `user-story-template.md` — but `templates` is in `.gitignore`, so **none of it is tracked**. The other seven referenced files (`adr-template`, `test-plan-template`, `bug-report-template`, `runbook-template`, `post-mortem-template`, `handoff-document-template`, `release-notes-template`) do not exist anywhere.
  
  **Why this was invisible:** on the author's machine, six of the links resolve. Depth bugs and gitignored targets both fail only for someone who cloned the repo, which is everyone the framework ships to. All 23 links are removed — bullet entries deleted, inline prose rewritten to keep its meaning, and the three ADR references re-pointed at `/adr`, which now carries the record shape inline. **The directory was deliberately not created:** a template that lives in the framework repo is a template a consuming repo never receives, which is the same reasoning that puts skill procedures inline rather than behind a link.
- ~~**Four broken `PROMPTS.md` anchors in `02-system-design/PROCESS.md`**~~ — **FIXED with the Phase 2 build.** `#adr-generation` (×2, real heading `#architecture-decision-record-generation`), `#eraser-architecture-diagram` and `#eraser-er-diagram` (both missing the `-via-mcp` suffix), and `#accessibility-review` (real heading `#accessibility-review-claude`). Five instances. All four were pointing at prompts this phase's own build wraps, which is how they surfaced: **verifying a Wraps column is the only routine that reads every inbound anchor.**
- ~~**`03-development/PROCESS.md` L324's tier table is now inaccurate.** It distinguishes a slash command from a skill on invocation shape alone.~~ **FIXED with the Phase 3 build**, in the same change that shipped the phase's two commands. The slash-command row now reads *"…that only a human should be able to fire — `.claude/commands/<name>.md` **plus `disable-model-invocation: true`**"*, followed by a blockquote stating that the tier is that field and not the directory. *(Now at L350.)* See [Phase 2 build — consequences](#phase-2-build--consequences) for where the premise was falsified and [Phase 3 build — consequences](#phase-3-build--consequences) §11 for its second use.
- **Phase 5 Gate 6 scoping inconsistency** — surfaced while evaluating the MCP Enforcement Policy prompt; fix in `QUALITY-GATES.md` before any MCP-policy artifact is considered.
- **The AC-floor deadlock between two Phase 3 prompts.** [`Estimation`](../aidlc-phases/03-development/PROMPTS.md#estimation) refuses below **3** acceptance-criteria bullets; [`story-decomposition`](../aidlc-phases/03-development/PROMPTS.md#story-decomposition) refuses below **5**. A story with 3-4 bullets is therefore estimable, can come back XXL, and then **cannot legally be decomposed** — while [Gate 1](../aidlc-phases/03-development/QUALITY-GATES.md#gate-1-sprint-commitment) requires *"story decomposed into sub-issues if AI-estimated XXL"*. **Recorded, not fixed** — no prompt text was changed in this build, and the correct repair is a product decision about which floor is right, not an edit that silently moves one. `run-sprint-planning` neither invents seams nor waives the line: it records the decomposition line as `not met` and sends the story back for AC. See [Phase 3 build — consequences](#phase-3-build--consequences) §3.
- **[`estimates-to-linear`](../aidlc-phases/03-development/PROMPTS.md#estimates-to-linear) stores only half the estimate its gate asks for.** Gate 1 wants *"Estimate set (T-shirt + Fibonacci)"*; only the Fibonacci value lands in a structured field, and the T-shirt size is prose inside a comment — **so the gate line cannot be checked mechanically.** The skill pins the T-shirt size to the comment's **first line** so it is at least greppable, which is a mitigation and not a fix. The real fix is a second tracker field, which is a workspace change rather than a template one.
- **`frontend-engineer.md:20` and `backend-engineer.md:21` assume a context load that nothing performs.** Both open *"Read the AC, PRD section, ADRs, and OpenAPI section…"*, and the frontend one attributes it to *"the developer (or `linear-task-agent`)"*. **`linear-task-agent` returns deep-links and never dereferences them** — its find flow outputs the PRD deep-link and the linked ADRs alongside the identifier and branch name, which is a pointer list, not the sources. The step that actually reads them is Step 2.4, now `/load-task-context`. Left unfixed because the honest repair is to name the command, and a cross-tier slug in a vendored subagent dangles in a repo that installed `subagent-prompts/` alone — the same constraint that shaped the `refactor-specialist` row above.
- **`software-architect.md:21` re-reads Step 2.4's sources rather than consuming them.** *"Read the Linear issue, the cited PRD section, the acceptance criteria, and any cited ADRs"* is Step 2.4 run a second time, one step later, by an agent that has just been handed a report containing exactly those four things. Harmless when it agrees with the first read and invisible when it does not — **two independent reads of the same sources, with no rule for which wins.**

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

Phase 6's build deleted a prompt, renamed another, and dropped a deliverable. Phase 1's changed **no prompt text at all**: 12 prompts before, 12 after, all anchors intact, and the paste path documented as still valid in `PROMPTS.md`'s header. Counts are unchanged — 96 prompts, 34 artifacts, 11 of them built, 18 shipped artifacts in the repo.

That is the expected outcome when a phase's tooling is stable, and it is a useful control. **Every edit Phase 6's build forced was caused by the vendor migration, not by the act of building a skill.** A team reading that build might reasonably have concluded that converting prompts churns the prompt library; it does not.

### What building nonetheless surfaced

Three things that specification could not, all recorded in the per-artifact "What changed on build" notes above:

1. **`Gap Analysis` is two prompts wearing one heading.** The pre-publish self-review and the post-publish backlog sweep share a name, a body and nothing else — different input, different output, different gate. The analysis had this half-recorded (the sweep's Do-NOT list excluded the self-review), but only inlining it forced the split to be *enforced* rather than described. **The general lesson: a prompt whose header note says "this form is also valid for X" is a candidate for being two prompts**, and the conversion pass is where that gets settled.
2. **A predicted anchor map and a read-back anchor map are not the same artifact.** The source prompt returns headings extracted from the draft at confirm time; what downstream issues need is the anchors the tracker actually minted. Both look like a list of anchors in the output. Only one of them resolves.
3. **The most dangerous refusals are the ones the source prompt left implicit.** "Never set state beyond Triage" implies "never clear `needs-human-review`" to a human reader and does not imply it to a model that has just been thanked for its work. Gate 3 is that label; making it explicit cost one line.

### One property carried over deliberately

Every one of the three terminates at a **human gate**, not at the next skill — publish stops for stakeholder review, scaffold stops for PM approval, push stops for the AI Inbox. This is the same shape as Phase 6's split at the storage/identity seam, and it has a routing benefit that was not the reason for it: **a mis-triggered skill in this set costs a confirmation prompt, not a backlog.** Three auto-triggering skills that write to a shared tracker would be a genuinely risky thing to ship without that property.

---

## Phase 2 build — consequences

Phase 2 shipped all seven of its artifacts as one set — the first phase to ship **all three tiers** at once, and the first build where the stack did not change underneath it *and* the phase was not the smallest available. Like Phase 1's, it deleted, renamed and rewrote **no prompt**: 15 before, 15 after, every anchor intact, the paste path still valid. Counts are unchanged at 96 prompts and 34 artifacts, with 18 of them built and 25 shipped artifacts in the repo.

### 1. A premise this document used in three places was false

**Custom commands have been merged into skills.** Verified against the current documentation: a file at `.claude/commands/adr.md` and a skill at `.claude/skills/adr/SKILL.md` both create `/adr` and work the same way, and **by default both the user and Claude can invoke either.** A command's `description` is a routing claim exactly like a skill's.

Three statements were wrong as a result, and all three are corrected: this document's *"the only artifact that faces no auto-trigger competition at all"*; `03-development/PROCESS.md` L324's tier table, which separated a command from a skill on invocation shape alone — **fixed with the Phase 3 build, and now L350**; and `ROUTING.md`'s framing of commands as inherently explicitly-fired.

**The response was to make the claim true rather than soften it.** `/adr` ships with `disable-model-invocation: true`. But the consequence generalises and is worth stating plainly: **the command tier is a convention held in one frontmatter field, not a property of a directory.** Delete that field and `/adr` re-enters every collision rule in `ROUTING.md`. `/load-task-context` and `/write-module-readme` inherit the field, not just the folder.

This also **re-prices decision #4** (the slash-command tier). Commands were partly killed on "not worth establishing a convention", a cost this build has now paid — but the tier's cheapness is real while its no-competition property costs one deliberate field per command. That is still cheap; it is just not free, and it is not automatic.

### 2. Four artifacts needed a token for "I could not tell"

The strongest pattern in this build, arrived at independently in four places:

- `architecture-reviewer` — a mandatory **`Unevaluated`** section, promoted from one sentence among ten in the source prompt.
- `accessibility-auditor` — a fourth verdict token, **`PASS WITHHELD`**, plus a can/cannot table covering **six** unmeasurable-from-source criteria where this document named one.
- `api-contract-freeze` — a **refusal** to audit the two Gate 2 lines that rest on a product judgement, until a human declares the public-endpoint and non-idempotent-POST lists.
- `render-design-diagrams` — a refusal to substitute the Mermaid mirror when the Eraser MCP server is unreachable, because the mirror satisfies one gate line of four.

Stated as a rule: **an agent that can only return pass-or-fail will manufacture whichever one the room wants.** It is the same property `container-image-engineer` reached with seven-of-nine and `dependency-risk-analyst` with *undetermined* — but this is the first build where it emerged in four artifacts at once without being specified in any of them, which suggests it belongs in the authoring doctrine rather than being rediscovered per artifact.

### 3. Two boundaries could not live in a description

A description is a claim on *routing*. Two of Phase 2's boundaries are violated **after** routing has already succeeded, and no description reaches them:

- **`architecture-reviewer` reviewing a design its own session produced.** Routing was correct; the input is the problem. It has to *check and stop*, which is a step, not a description.
- **`data-model-design`'s backend-developer review stop.** Nothing downstream refuses to proceed, so the skill must refuse for itself — and specifically against *unavailability*, which is the excuse that reads like a reason.

This is the same lesson as `linear-task-agent` (b)'s classification-step finding, generalised: **if a boundary depends on session history or on a human who is not in the room, it must be a step in the body. The frontmatter cannot hold it.**

### 4. Inlining found a deliverable that belonged to a different artifact

`Schema Generation`'s item 7 asks for a Mermaid `erDiagram` — but `render-design-diagrams` owns both diagram surfaces and the gate lines counting them. Left in both, the file is written twice and the second writer wins silently. **As a paste prompt this was invisible**, because no other procedure was ever going to write that file; it only becomes a conflict once both are artifacts that run.

This is the Phase 1 `Gap Analysis` finding in a second shape, and the pair now supports a stronger claim than either alone: **the conversion pass is where a prompt library's internal overlaps get settled, and they are not visible from reading the library.** Phase 1 found one prompt that was two; Phase 2 found one prompt carrying another artifact's deliverable.

### 5. The placeholder file started paying off

Phase 2 uses **16 placeholders and coined only 8.** Eight are existing names adopted from Phases 3 and 6 — the first time literal overlap between phases exists, which is the intended end state rather than a regression. **The pair count is still six, not fourteen.**

Two near-pairs were caught at authoring time and recorded rather than minted: `{{AUTH_STACK}}` now carries the auth *provider* and the *wire security scheme* under one name, applying the reuse-even-if-you'd-have-named-it-differently rule; and `{{A11Y_SCAN_COMMAND}}` deliberately does **not** reuse `{{TEST_RUNNER}}`, because filling it with a unit-test command makes the accessibility PASS permanently unreachable — and the symptom looks like a pedantic agent rather than a bad fill, so nobody goes looking.

### 6. A collision was closed by declining to claim a verb

`render-design-diagrams` had **no surviving subset** of its proposed triggers — all four collided with a project-scope `architecture-diagram` skill. A rename was considered and rejected on a distinction worth keeping: **renaming fixes name collisions, and this was a trigger collision.** The names never collided.

Narrowing was honest only because the artifacts genuinely differ. The residual risk is recorded rather than dissolved: with both installed, bare "draw the architecture diagram" produces the Mermaid mirror only — **one gate line satisfied, three not, and it looks done.** That is an install-time decision, and it now lives in the phase's Step 0.

`ROUTING.md` gained a row for the **deliberately unclaimed** lane. A lane left to an incumbent on purpose reads identically to an oversight unless it is written down, and the next author who thinks "we should trigger on *draw the diagram*" re-opens a closed collision.

---

## Phase 3 build — consequences

Phase 3 shipped its remaining four artifacts — two skills and two commands — as one set, and is **the first phase whose artifacts landed across separate builds**: its seven specialist subagents predate this analysis entirely. Like Phases 1 and 2 it deleted, renamed and rewrote **no prompt**: 18 before, 18 after, every anchor intact, the paste path still valid. Counts hold at 96 prompts and 34 artifacts, with 22 built and 29 shipped.

It is also the first build where the phase being converted **already had shipped templates in it**, and that turned out to be the difference that mattered. Four of the findings below exist only because the incumbents were real files with real wording rather than a specification.

### 1. The "I could not tell" pattern has a fourth confirmation, and this time it arrived in every artifact

[Phase 2 §2](#phase-2-build--consequences) recorded four artifacts independently needing a token for *"I could not tell"* and concluded it belonged in the authoring doctrine. **All four Phase 3 artifacts reached it again, none of them told to:**

- **`open-pull-request`** — the Definition-of-Done walk marks each line met / not met / **`cannot verify`**. Two of the five lines attract it structurally: *"all AC verified (by tests **or manual check**)"* and *"no known bugs introduced"* **cannot be established from a diff at all.**
- **`run-sprint-planning`** — the commitment-readiness report marks each criterion ready / not ready / **`not mine to judge`**.
- **`/load-task-context`** — ships a third conflict value, **`unchecked`**, and names it the load-bearing one.
- **`/write-module-readme`** — reports all seven sections back as written-with-evidence or omitted-with-what-was-looked-for.

**Eight artifacts across two phases have now converged on this without being told to.** It has stopped being a finding and is a rule: **every artifact that returns a judgement ships a token for the judgement it could not make, and that token is never rounded up.** An artifact that can only answer yes-or-no will answer whichever one the room wants, and the answer will be indistinguishable from a real one.

### 2. Two gate claims did not survive counting — both the `container-image-engineer` shape

Neither skill covers its gate, and both say so in the gate rather than in a footnote:

- **`run-sprint-planning` answers three of [Gate 1](../aidlc-phases/03-development/QUALITY-GATES.md#gate-1-sprint-commitment)'s five criteria.** Velocity calibration is a human conversation with data the skill does not hold; owner assignment is a staffing decision. Both come back `not mine to judge`. **Three criteria out of five is not a commitment**, and a report that marked five would have manufactured one.
- **`open-pull-request` deliberately does not claim [Gate 2](../aidlc-phases/03-development/QUALITY-GATES.md#gate-2-pr-merge)'s coverage-threshold and SAST lines.** Those are automated CI lines that run *after* the PR opens — outside the pre-merge half the skill owns.

Both gates were annotated rather than left implying full coverage. This is the third and fourth instance of `container-image-engineer`'s seven-of-nine, and the pattern is now stable enough to state as a build step: **count the gate lines your artifact actually reaches before writing its description, and edit the gate when the answer is not all of them.**

### 3. A new defect class — two prompts whose refusal thresholds do not compose

[`Estimation`](../aidlc-phases/03-development/PROMPTS.md#estimation) refuses below **3** AC bullets. [`story-decomposition`](../aidlc-phases/03-development/PROMPTS.md#story-decomposition) refuses below **5**. A story with 3-4 bullets is therefore estimable, can come back XXL, and then **cannot legally be decomposed — while Gate 1 requires XXL stories to be decomposed.** The gate line becomes unsatisfiable by any legal path.

**This is invisible while both are paste prompts**, because a human sizing a story and a human decomposing it are the same person on two afternoons, and they adjust around it without noticing there was anything to adjust around. It becomes a hard stop the moment both are steps in one procedure.

It is the third distinct shape of the same meta-finding, and the three together are stronger than any one: **Phase 1 found one prompt that was two** (`Gap Analysis`); **Phase 2 found one prompt carrying another artifact's deliverable** (`Schema Generation`'s ER diagram); **Phase 3 found two prompts that cannot both be obeyed.** None of the three is visible from reading the library — the first needs the prompt inlined, the second needs a second artifact to exist, and the third needs both prompts inside one procedure.

### 4. A defect *between* a prompt and a shipped template

[`pr-description`](../aidlc-phases/03-development/PROMPTS.md#pr-description) and `linear-task-agent`'s pr-open flow specify **incompatible PR bodies**. The prompt's shape is Summary / `Closes` / Approach / AC checklist / AI-generated sections / Notes for reviewer; the shipped agent drafts Summary / AC coverage / Test plan / Out of scope / Reviewer notes. **The agent's shape drops the closing keyword and the AI-generated-sections notation — two distinct Gate 2 lines.**

This is a class **neither** of the repo's existing checks can surface. The per-phase READMEs' Wraps column verifies that a prompt exists and that its anchor resolves; it says nothing about whether a *different* template contradicts it. A prompt-by-prompt read never opens the subagent. **The defect lives in the gap between the two, and the only routine that crosses that gap is building a skill that hands one to the other.**

Fixed in the skill rather than in either source: `open-pull-request` hands `linear-task-agent` the **finished title and body verbatim, to post unmodified**, not a summary to draft from. That is a concrete instance of PROCESS.md's **hand-off-integrity test** (L420) — the fifth pre-commit test the framework's own checklist omitted, corrected in the same change. **The test that would have caught this was the one the checklist was missing**, which is a tidier vindication than it deserves to be.

### 5. The bypass was in the emitter, not only the router — the sharpest finding of the build

The prescribed `code-reviewer` edit was **description-only**: add the pre-PR gate to its Do-NOT list. But that agent's **step-5 output format** ended by recommending *"`linear-task-agent` to open the PR"*.

**The gate bypass was written into the agent's own output.** It fires *after* routing has already succeeded — the developer asked for a review, got the right agent, and was then told by that agent to skip four of the remaining five steps. No description reaches this: a description constrains which artifact loads, and this happens once one has.

This generalises the `linear-task-agent` (b) classification-step lesson from a new direction. That finding was *a boundary the classifier runs past*; this one is *a boundary the output walks around*. Together: **the load-bearing surface is wherever an artifact emits a recommendation, not only where it accepts a request.** A boundary check must read an agent's **outputs** — its report templates, its "recommended next agent" lines, its escalation lists — and not only its triggers. Every shipped subagent in this repo that ends by naming a next actor is now a place this defect could be sitting.

### 6. The same-phase dangling-pointer rule is under-specified

The repo's rule reads: *a cross-phase slug dangles, a same-phase slug does not.* `open-pull-request` is the first shipped skill to name sibling slugs in its body — `code-reviewer`, `frontend-engineer`, `backend-engineer`, `conflict-resolver`, `linear-task-agent` — on exactly that basis.

**"Same-phase, so it never dangles" assumes a repo installs *the phase*. Installation is per-directory.** A repo that copies `subagent-prompts/*.md` and skips `skill-prompts/` now holds two templates naming a skill it does not have; the reverse install holds a skill naming five agents it does not have.

Mitigated in the wiring — the mutual dependency is stated in `subagent-prompts/README.md` and in `skill-prompts/README.md`, and the phase's Step 0 copies all three directories in one action — but **the rule as this document states it needs the qualifier**: *a same-phase slug does not dangle **provided the phase's directories are installed together**, which is a property of the install instruction, not of the phase.*

### 7. `file-followup-bug` does not exist as a shipped template

The [Extend, don't create](#extend-dont-create) table prescribed routing `refactor-specialist`'s candidate-identification to `file-followup-bug` **by name**. It appears as PROCESS.md's worked teaching example (L363), in [`ROUTING.md`](./ROUTING.md), in this document, and in a Phase 1 README — **never as a `SKILL.md`.** It is a name three documents treat as an artifact and no directory contains.

Naming it would have reproduced the exact `pr-reviewer` dangling-slug defect v2.4 removed, in the same file class. **The edit shipped restated by work type** — the Phase 2 `software-architect` form — and passes the table's own standalone test. The table row is corrected accordingly, and `ROUTING.md`'s reserved-verb entry, which listed it as though it were shipped, is corrected too.

The same defect appeared a **second** time in that template, in its *escalation* bullet: *"file as a new Linear issue"*, instructed to an agent whose own boundaries forbid it from calling the tracker at all. Fixed in the same form. **One prescribed edit, two instances, and the second was not in the row that prescribed it** — which is the argument for reading the whole template rather than the line the table points at.

### 8. Rebase invalidates the review, and neither the specification nor the source prompts noticed

The prescribed order is review → rebase → open. **A rebase that resolves a conflict introduces hunks the review never saw**, and `conflict-resolver` produces a *resolution report*, not a review — so the branch reaches the PR carrying code no reviewer has read, with a clean review report attached to it.

Reordering to rebase → review → open was considered and rejected: it contradicts PROCESS 4.1's own sequence and would have put the skill out of step with the phase it ships into. The skill **re-reviews the resolved hunks** instead, and Gate 2's rebase line now carries the same requirement.

### 9. `Closes` is wrong for a multi-PR story

The pre-build description named only `Closes`. The source prompt offers `Closes` **or** `Part of`. **Always writing `Closes` auto-closes the story at the first of several merges** — the tracker's git integration does it, nobody sees it happen, and the remaining PRs land against a closed issue.

The skill asks *"is this the only PR for this story?"* and is forbidden from assuming. Worth recording as a class: **a one-word default with a silent, irreversible effect is the cheapest thing in a conversion to get wrong**, because it reads as a formatting detail in the specification and is a state change in production.

### 10. One proposed trigger was cut as a routing defect

`run-sprint-planning`'s proposed *"is this ready to commit"* is, unqualified, **far more likely to be about a working tree than about a cycle**. It would have pulled a cycle-planning procedure into an ordinary git turn. Replaced with *"are these stories ready for the cycle"*, and **every surviving trigger names the sprint, the cycle or the backlog.**

Worth contrasting with `render-design-diagrams`, where **no** subset of the proposed triggers survived. There the collision was with an incumbent artifact that claimed the same four phrases, and narrowing meant surrendering the lane. Here the collision was with **ordinary English**, and the fix was one phrase — because nothing else owns the utterance, only the reader's default reading of it. **Two different failures wearing the same word "collision"**: one is contested ownership, the other is ambiguity with no second claimant.

### 11. The command tier's one-field property held on its second use

Both commands carry **`disable-model-invocation: true` deliberately, not by inheritance**, and both READMEs say why in place — because the next author will otherwise reasonably assume the directory grants it. That is the [Phase 2 §1](#phase-2-build--consequences) finding surviving contact with a second author, which is the only evidence that a convention has actually been established.

**`/write-module-readme` was the closer shape-test call and it passes.** Seven evidence rows look like a branching checklist, which would have promoted it to `skill-prompts/`. They are not: they are **output admission criteria applied uniformly in one pass** — no ordering between them, no human gate, no second argument. The line for the next author is recorded in the directory README rather than left to be re-derived: **a `--section` argument, an ask-the-developer step, or a threshold-detection step moves it to `skill-prompts/`.**

### 12. `/write-module-readme` replaced an unenforceable judgement with a lookup

*"Omit any section with nothing real to say"* is **satisfiable by a run that omits nothing** — and a padded README reads as a thorough one, so the reviewer's instinct rewards the failure. It is the same unfalsifiable shape as an unverified gate checkbox, in a document nobody thinks of as a gate.

Each of the seven sections now names the artifact that licenses it: **import edges in both directions** (Architecture), **the line that actually reads the env var** (Configuration), **a real exported symbol with its real signature** (Usage), **an in-code `TODO`/`FIXME` or a labelled tracked issue** (Limitations), **a test invocation that was actually run** (Testing). *"Nothing real to say"* becomes *"the grep returned nothing"*, **which is checkable.**

The generalisation is worth carrying to the remaining twelve artifacts: **wherever a specification asks an artifact to exercise restraint, replace the restraint with a lookup.** An instruction to be honest is satisfied by a model that believes it is being honest; an instruction to cite a file and a line is not.

### 13. A Gate 2 checkbox was already unfalsifiable, for a reason nobody had connected to it

*"PR links story, describes approach, notes AI-generated sections"* is **exactly the pair `linear-task-agent`'s own PR-body shape omits** — the closing keyword that makes the link and the AI-generated-sections notation. The gate has been asking for two things the phase's only PR writer was never going to produce, and it has presumably been ticked anyway, because the PR looks completely normal.

It was **annotated, not rewritten**: the line is correct as a requirement, and the defect was in the producer. It now names `open-pull-request` as the composer and `linear-task-agent` as the verbatim poster, and says what a re-drafted body silently costs. **The checkbox was not wrong; nothing in the framework could satisfy it**, which is a failure mode the repo has now met from four directions and has no cheaper detector for than building the artifact.

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

4. ~~**`/adr` (½ day).** Best item on the list: explicitly fired, no auto-trigger competition, no coordinated edit, no unmandated MCP, cited by three gates, fires 11+ times per project. If the command convention works here it works everywhere.~~ **SHIPPED**, with the Phase 2 set rather than ahead of it. The convention works — but *"explicitly fired, no auto-trigger competition"* turned out to be a property of one frontmatter field rather than of the tier, so the convention that now exists is **one field plus a directory**, not a directory. See [Phase 2 build — consequences](#phase-2-build--consequences).
5. ~~**`publish-prd-to-linear` (2 days).**~~ **SHIPPED.** Two prompts, one gate, one MCP server that Phase 1 already mandates. It came in wrapping three prompts, not two — see its entry.

### Phase 2 — the P0 tier (~3 weeks)

One artifact per PR, so the router can be re-tested each time.

6. ~~`scaffold-linear-milestones` + `push-linear-stories` (3 days — ship together; the second consumes the anchor map)~~ **SHIPPED**, together, as predicted — the second refuses to run without the first's anchor map, so shipping them apart would have shipped one skill that could not execute its own procedure.
7. ~~`open-pull-request` + the `linear-task-agent` and `code-reviewer` edits (3 days — three files, one PR)~~ **SHIPPED**, as one PR with **four** shipped-template edits rather than two — `refactor-specialist` and `subagent-prompts/README.md` came with it — and both prescribed edits turned out to be in the wrong place or the wrong shape. See [Phase 3 build — consequences](#phase-3-build--consequences) §5 and §7.
8. ~~`run-sprint-planning` (2 days)~~ **SHIPPED**, in the same PR as item 7 rather than after it. Bundling was right for a reason the queue did not anticipate: the two skills share a directory README whose routing section is the only place `open-pull-request`'s collision is explained, and splitting them would have shipped that README twice.
9. `e2e-and-coverage-engineer` (2 days — the rename resolves the roombook collision; no shipped-template edit needed after the narrowing) — **the last remaining P0.**

### Phase 3 — P1 (~4-6 weeks)

No-coordinated-edit artifacts first: **`load-test-engineer`** (zero routing collisions — the cheapest P1 to land), **`dependency-risk-analyst`** (also collision-free; its only dependency is that Dependabot is switched on), `threat-modeler`, `secret-leak-response`. **`threat-model-verifier` ships after `threat-modeler`, never before it** — it audits that agent's output, so building it first gives it nothing to verify, and the two descriptions must be written as a mutually-excluding pair in the same PR. Then the remaining multi-step skills: `author-test-plan`, `reproduce-and-diagnose-bug`, `finalise-documentation`.

> **Phase 2's seven are already built and are not in this queue** — `solution-architect`, `architecture-reviewer`, `accessibility-auditor`, `render-design-diagrams`, `data-model-design`, `api-contract-freeze` and `/adr` shipped as one phase set. **The `threat-modeler` ↔ `architecture-reviewer` mutual exclusion is now one-sided and must be completed when `threat-modeler` ships:** the shipped reviewer already excludes threat modelling, but by work type ("the repo's security-review tooling") rather than by slug, because naming an unbuilt artifact is the dangling-pointer defect. `threat-modeler`'s own description must carry the reciprocal, and whoever builds it should decide then whether both sides can safely name each other — they will be in different phases, so the cross-phase rule says probably not.

> **Phase 6's eight helpers are already built and are not in this queue.** They shipped with the v2.4 Pulumi removal rather than in rollout order. The rule that generalises: **when a phase undergoes a stack change, build its helpers then** — not before, when the shape is speculative, and not after, when the losses have already been forgotten.

### Phase 4 — P2, conditional

Build only if the frequency materialises. Several name their own drop condition: `sweep-requirements-gaps` now carries a **falsifiable release trigger and a numeric drop condition** rather than a felt one (see its entry), `handoff-agent` after the first real engagement close, `reproduce-and-diagnose-bug` if teams already ship regression tests in the fix diff unprompted.

> **Both Phase 3 commands have left this queue** — `/load-task-context` and `/write-module-readme` shipped with the Phase 3 set. `/load-task-context`'s condition was *"build it if tracker hygiene does not supply the deep-links"*, and building it settled the question in the opposite direction from the one the condition anticipated: **the deep-links are not the hard part — dereferencing them is.** The Phase 1 skills do supply and verify them, and the command still earns its keep, because a pointer that resolves and a source that was actually read are different facts and only one of them is checkable at the gate. That is why it ships a third conflict value. **A drop condition phrased against an input can be satisfied and still miss why the artifact exists.**

> **`api-contract-freeze` has left this queue** — it shipped with the Phase 2 set, and its drop condition (*"if the audit is not genuinely mechanical"*) was **evaluated on build and did not fire**, but only after a step was added that the specification did not contain. That is the more useful outcome than either building or dropping it blind: **a drop condition is a question, and building the artifact is how you answer it.** Two of its eleven audit lines were genuinely not mechanical, and the fix was a refusal rather than an abandonment. See its entry.

---

## Decisions still open

1. **Does the `roombook/` dry-run happen before or after authoring?** Doing it first costs half a day and will change at least three descriptions (and possibly two names). Doing it after means authoring templates that then need edits in the consuming repo. **Recommendation: before** — it is the only empirical routing evidence available anywhere in this repo.

2. **`security-fix-engineer` and `triage-refactor-candidates`: absorb into shipped agents, or keep as gaps?** Both are resolved above as boundary edits to `backend-engineer` and `refactor-specialist`. The alternative is to accept the routing risk and build them as separate artifacts — which is what four of the seven per-phase analysts wanted. The deciding question is whether teams actually mis-route on "fix this SAST finding" today.

3. ~~**Do the Phase 1 outer-loop skills perform their own Linear writes, or does `linear-task-agent` widen?**~~ **SETTLED by shipping them.** The three skills perform their own writes, each behind its own confirm-then-`go`, and both directions of the boundary are now written down where a maintainer will hit them: `01-requirement-gathering/skill-prompts/README.md` states that these are the outer-loop writers and that the dev-loop agent refuses this work, and Phase 1's PROCESS Step 0 repeats it as "by design, not by omission". **The reciprocal edit to `linear-task-agent.md` is still outstanding** — it ships with `open-pull-request`, and until it lands the guarantee is documented on only one of the two sides. Original resolution, unchanged: `linear-task-agent` stays sole writer for the **inner dev loop** (verified in its own boundary text), and outer-loop writes — Documents, Projects, Milestones, Triage issues, estimates, the test-plan Document — are performed by the phase skills, each borrowing its literal-`go` confirm-before-write protocol. This is correct but non-obvious; if it is not written into that agent's boundaries, a future maintainer will "fix" it and break the dev-loop-only guarantee.

4. **The slash-command tier.** Commands were killed partly on "not worth establishing a convention" — **a cost `/adr` has now paid.** `/write-module-readme` was reinstated; **`/post-mortem` is now settled and stays killed**, on shape rather than convention cost (see the not-converted table). Should the remaining three (`/promote-component`, `/infra-runbook`, `/linear-context-pull`) be re-evaluated on their own merits, or is the smaller command surface itself the goal?
   > **Re-priced by the build, in both directions.** The convention cost is genuinely gone — a second command is now a file plus a README row. But the merged-into-skills finding means the tier's defining property is **not free**: each command must carry `disable-model-invocation: true` deliberately, and a command that omits it is a skill with a worse description. So the honest framing of this question changed. It is no longer *"is a command cheap enough to bother with"* — it is **"is this prompt one argument and no branching, given that the tier no longer confers explicit-firing by itself?"** That is a question about shape, which is the same test that settled `/post-mortem`.

5. ~~**Do skill templates inline their procedure, or link it?**~~ **SETTLED by shipping six of them in v2.4.** Resolved as **inline with `{{PLACEHOLDERS}}`**, and the corollary that was still open — where the framework pointer goes — is now answered too: **shipped skill templates carry no framework path in their body; the pointer lives in `skill-prompts/README.md`, which never ships.** The reasoning is that PROCESS.md L408's "open with the role and the source of truth" presupposes the skill was authored *in* the consuming repo, where `aidlc-phases/` exists. So each shipped skill opens by naming a **repo artifact** that does resolve there — the conventions file, the recorded architecture decisions, the existing workflows, the service's exposed metrics. Framework provenance is not lost, it is relocated to the README's Wraps column, which is strictly better placement: the person who needs the provenance is the one instantiating, not the agent executing. **One declared exception:** `cicd-pipeline-bringup` references the four agentic-workflow templates rather than inlining ~65 lines of YAML that would breach the two-screen limit, and the README makes that a one-time human paste at instantiation.
   > The decision also cost what it predicted it would: inlining is what forced the `iac-foundation-bringup` split, exactly as it forced the `build-linear-backlog` one. Two splits from the same rule is enough evidence to call it load-bearing rather than incidental.

6. **How does a shipped artifact get revised when its phase's stack changes?** New, raised by v2.4. Phase 6's helpers were built during a vendor migration and four of the eight exist only to carry capabilities the departing vendor took with it. That worked — but the eight are now *shipped templates* in a repo, and the next stack change to Phase 6 has to revise them rather than author them. There is no convention yet for versioning a template, deprecating one, or telling a consuming repo that its vendored copy is stale. **Recommendation: do nothing until it bites once.** A versioning scheme designed before a single template has ever needed revising will be wrong.

---

## Appendix: full per-prompt classification

All 96 prompts, in file order. Line numbers refer to each phase's `PROMPTS.md`. **Five** prompts are deleted and shown struck from their tables: `sentry-context-pull` (Phase 4) and `Inherited Error Triage` (Phase 7) with the Sentry removal, `Container CVE Triage` and `OPA Policy Generation` (Phase 5) with the security-tooling removal, and `Pulumi Cost Delta` (Phase 6) with the v2.4 Pulumi removal. `Reachability Triage` was struck in the same revision as the Phase 5 pair and is **restored**, so it appears live. One Phase 5 row — `Threat-Model Mitigation Verification` — is a **newly authored** prompt with no pre-revision counterpart. **101 rows, 5 struck, 96 live.**

> **This header read "All 97 … Four … 101 rows, 4 struck, 97 live" until the Phase 3 build recounted it.** The v2.4 Pulumi removal struck `Pulumi Cost Delta` in the Phase 6 table and updated the corpus figure in the revision banner and the Verdict, but not here — so the appendix has been asserting a corpus one larger than the rest of the document for two builds. Recounted per phase against the tables below: 12 + 15 + 18 + 15 + 15 + 16 + 10 = **101 rows**, less 5 struck = **96 live**, which reconciles with the coverage table's 12 + 15 + 18 + 14 + 13 + 15 + 9.

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
| Architecture Proposal | 7 | subagent | `solution-architect` — **shipped** |
| Trade-off Interrogation | 54 | subagent | `solution-architect` — **shipped** *(folded in as a mandatory step, not the optional follow-up)* |
| Eraser Architecture Diagram (via MCP) | 84 | skill | `render-design-diagrams` — **shipped** |
| Eraser ER Diagram (via MCP) | 115 | skill | `render-design-diagrams` — **shipped** |
| Architecture Decision Record Generation | 135 | slash-command | `/adr` — **shipped**; the repo's first slash command |
| Entity Extraction | 177 | skill | `data-model-design` — **shipped** |
| Schema Generation | 215 | skill | `data-model-design` — **shipped** *(its item 7 Mermaid deliverable belongs to `render-design-diagrams`; prompt unchanged, skill hands over)* |
| Migration Generation | 250 | skill | `data-model-design` — **shipped** |
| API Contract | 281 | skill | `api-contract-freeze` — **shipped** *(gained a refusal step the spec did not have)* |
| Tech Stack Comparison | 315 | keep-as-prompt | — |
| Design Review | 348 | subagent | `architecture-reviewer` — **shipped** |
| Design System Bootstrap | 384 | keep-as-prompt | — |
| UI Wireframe | 413 | keep-as-prompt | `wireframe-flow` killed |
| Production Component | 446 | keep-as-prompt | `/promote-component` killed |
| Accessibility Review (Claude) | 474 | subagent | `accessibility-auditor` — **shipped** |

### Phase 3 — Development

| Prompt | Line | Verdict | Artifact |
|---|---|---|---|
| Estimation | 7 | skill | `run-sprint-planning` — **shipped** *(its 3-bullet AC floor deadlocks against `story-decomposition`'s 5 — recorded, not fixed)* |
| Feature Scaffolding (for Claude Code) | 39 | keep-as-prompt | embedded in shipped `frontend-engineer` / `backend-engineer` |
| Test Generation | 81 | keep-as-prompt | embedded in three shipped subagents |
| Self-Review Before PR | 114 | skill | `open-pull-request` — **shipped** |
| Refactoring | 164 | keep-as-prompt | embedded in shipped `refactor-specialist` |
| Documentation Generation | 196 | slash-command | `/write-module-readme` — **shipped** *("omit anything with nothing real to say" replaced by a per-section evidence rule)* |
| Debugging (for Claude Code) | 222 | keep-as-prompt | — |
| MCP Integration (for Claude Code) | 254 | keep-as-prompt | `add-mcp-server` killed |
| linear-sprint-pull | 281 | skill | `run-sprint-planning` — **shipped** |
| estimates-to-linear | 301 | skill | `run-sprint-planning` — **shipped** *(writes only the Fibonacci half of Gate 1's "T-shirt + Fibonacci"; the skill pins the T-shirt size to the comment's first line as a mitigation)* |
| story-decomposition | 321 | skill | `run-sprint-planning` — **shipped** *(5-bullet AC floor; see `Estimation` above)* |
| linear-next-task | 345 | keep-as-prompt | shipped `linear-task-agent` `find` flow |
| task-context | 363 | slash-command | `/load-task-context` — **shipped** *(gained a third conflict value, `unchecked`, and one row per endpoint)* |
| architecture-design | 387 | keep-as-prompt | system prompt of shipped `software-architect` |
| linear-progress-comment | 445 | keep-as-prompt | shipped `linear-task-agent` `progress` flow |
| pr-description | 473 | skill | `open-pull-request` — **shipped** *(specifies a PR body incompatible with `linear-task-agent`'s; the skill hands its finished body over verbatim)* |
| refactor-candidates | 503 | keep-as-prompt | `triage-refactor-candidates` killed; filing routed **by work type**, not to `file-followup-bug` — that template does not exist |
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
