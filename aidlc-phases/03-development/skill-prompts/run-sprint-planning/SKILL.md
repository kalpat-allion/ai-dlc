---
name: run-sprint-planning
description: Use when a Tech Lead or PM is preparing the upcoming sprint or cycle for commitment — pulling the candidate backlog from the tracker read-only, sizing each story that has usable acceptance criteria, posting the estimates and their subtask / risk / dependency breakdown back onto the issues behind a confirm-then-go stop, and decomposing anything sized XXL into sub-issues before the cycle opens. Ends in a commitment-readiness report marking every criterion ready / not ready / not mine to judge — velocity calibration and owner assignment are human calls this skill will not make on their behalf. Triggers on "plan the next sprint", "pull the candidate backlog for the cycle", "size the backlog", "estimate these stories for the sprint", "are these stories ready for the cycle", "decompose this XXL story". Performs its tracker writes itself: this is outer-loop planning, deliberately outside the story-loop tracker agent's remit. Do NOT use for: daily story pickup, state transitions, progress comments, PR opening or any other write around a story already in flight; re-estimating a story that has left Backlog; building the backlog from a requirements document; or opening, closing or committing the cycle itself.
---

# Size the backlog for cycle commitment

You are preparing a backlog for commitment. Everything here is reversible except the estimates you post — those are read as the team's number for the rest of the cycle. The source of truth is the acceptance criteria already on the issues and the code that already exists. **Nothing is estimated from a title.**

## Procedure

1. Pull the candidates **read-only**: project `{{LINEAR_PROJECT}}`, team `{{TEAM_PREFIX}}`, `state = Backlog`, ordered by priority then age, capped at 50. Per issue report identifier · title · priority · existing estimate · `branchName` · AC bullet count · milestone · labels; then the totals — candidates, count without AC, count without an estimate. **If exactly 50 come back, say the list is truncated** rather than reporting a total that looks complete. Zero results is a wrong filter, not an empty backlog: surface it and ask. Never invent an issue.
2. **Check state per story, however the story reached you** — including one handed to you directly rather than found by step 1. Anything outside `Backlog` is dropped from the pass and named. A story in flight already carries a committed number, and re-sizing it mid-cycle rewrites the history the team calibrates against.
3. **Refuse to size any story with fewer than three AC bullets, or with any ambiguous bullet.** List the questions the PM has to answer instead. Never invent an AC to make a story estimable — a fabricated bullet becomes a test somebody writes.
4. Size each remaining story against `{{TEAM_STACK}}` and the code already in the repo: T-shirt size (S <0.5d · M 0.5-1d · L 1-3d · XL 3-5d · XXL >5d) · Fibonacci points · subtasks with hours · risks, each **naming the file or pattern that creates it** · dependencies. Prefer the smaller size wherever you are uncertain.
5. **Confirm before writing** (see below), then wait for a literal `go`. On `go`, per story: set the estimate field to the Fibonacci points and **change no other field**, then post exactly one comment carrying the T-shirt size on its first line, followed by subtasks, risks and dependencies, signed `_(via Claude Code MCP)_`. One output line per story. **Stop the batch at the first failure** and surface it verbatim — a half-written batch nobody can see is worse than a failed one.
6. For each story sized **XXL**, propose 2-5 children: title · the parent AC bullets that child takes, **copied verbatim** · a size that must be ≤ L · the sequencing between them and why. Only after the developer approves, create them as sub-issues of the parent; the parent stays open as the tracking story. **An XXL story with fewer than five AC bullets has no seams to slice.** Do not invent them and do not decompose — send it back for the AC to be written, and record its decomposition line as `not met`.
7. Return the **commitment-readiness report**, one row per story, each criterion marked **ready**, **not ready**, or **not mine to judge**. You can answer three: AC ≥ 3 and unambiguous · estimate set · XXL decomposed. **Calibration against real team velocity and the owner assignment are always `not mine to judge`**, and saying so is the deliverable — three criteria out of five is not a commitment. → the Tech Lead calibrates against velocity, assigns owners, and opens the cycle.

## Confirm-before-write

Echo the whole batch as one table and require a literal `go`:

| Identifier | Title | T-shirt | Points | Existing estimate | Comment to post |

Any story that already carries an estimate is shown with the old value beside the new one and is **excluded from the write** until the developer confirms that row by identifier. Step 6's decomposition is a **separate** approval — never fold it into this one.

## Refusal cases

- **Never estimate a story you had to complete first.** Missing or ambiguous acceptance criteria is a stop, not an input to work around.
- **Never overwrite an existing estimate without per-issue confirmation.** That number may be the team's calibrated one rather than an earlier machine guess, and once overwritten the two are indistinguishable.
- **Never write to a story outside `Backlog`**, and never transition state, assign, comment on progress, or touch a branch or PR. Every write around a story already in flight belongs to the story-loop tracker agent.
- Never create issues from a requirements document, and never create Milestones or Projects. Step 6 creates children **of a backlog story that already exists**, and that is the only issue creation in this procedure.
- **Never open, close or commit the cycle, and never assign an owner.** Both are commitment criteria — doing them yourself removes the check they exist to be.
- Never report a story as ready to commit on the criteria you can evaluate while staying silent about the two you cannot.
