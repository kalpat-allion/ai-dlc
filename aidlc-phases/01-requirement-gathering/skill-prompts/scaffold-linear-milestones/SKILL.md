---
name: scaffold-linear-milestones
description: Use when an approved PRD Linear Document must be decomposed into delivery epics and one Linear Milestone created per epic on the Project that already exists. Reads the Document through the Linear MCP server rather than asking for a paste, derives epics with personas, dependencies, T-shirt complexity and priority, orders them for implementation with cross-cutting concerns (auth, permissions, notifications) pulled out as epics in their own right, checks that every functional requirement section is claimed by exactly one epic, then creates the Milestones behind a confirm-before-write stop and halts at the milestone-approval gate. Triggers on "break the PRD into epics", "decompose the PRD into epics", "set up the milestones for this PRD", "scaffold the backlog structure", "what are the epics for this project", "add the milestones to the Linear project". Hands the epic-to-Milestone mapping to push-linear-stories once a human has approved the scaffold. Do NOT use for: creating issues or stories (use push-linear-stories), publishing or editing the PRD Document or creating the Linear Project (use publish-prd-to-linear), sizing or estimating stories for a sprint, diffing an existing backlog against the PRD, or approving the scaffold — that approval is the PM's.
---

# Scaffold the Milestones from an approved PRD

You are turning an approved PRD into the backlog's skeleton. The source of truth is the **published PRD Linear Document**, read directly over MCP — never a paste, because a paste can be a stale copy of a Document that has since been revised. This skill stops at a human gate: it builds the scaffold, it does not fill it.

## Procedure

1. Check the two preconditions and **refuse if either fails**: the PRD Document header reads **Approved v1.0** or higher, and the Linear Project is out of `Planned` state. Milestones built on an unapproved PRD are rebuilt, and the rebuild is manual.
2. Read the Document via the Linear MCP server. Extract its section structure — `§` numbers and headings — and the Functional Requirements groupings. Read the current Project too: if Milestones already exist, list them and stop for the human rather than adding a second, parallel set.
3. Decompose the functional requirements into epics. For each epic record: **name** (short title) · **description** (2–3 sentences) · **personas affected** · **key user stories** in full "As a … I want … so that …" form · **dependencies** on other epics · **complexity** as a T-shirt size (S/M/L/XL) with a one-line justification · **priority** (Must / Should / Nice).
4. Pull cross-cutting concerns — authentication, permissions, notifications, audit, i18n — into epics of their own rather than smearing them across feature epics. They are where hidden dependencies live.
5. Order the list in recommended implementation order: dependencies before dependents, Must Have before Should Have where the dependency graph allows.
6. **Map every epic to the PRD sections it covers**, and check the map both ways. Report any functional-requirement section claimed by **no** epic, and any claimed by **more than one**. Do not resolve either silently — an unclaimed section is a coverage gap and a double-claimed one is a boundary the PM has to draw.
7. **Confirm before writing** (see below), then wait for a literal `go`.
8. On `go`, create **one Milestone per epic** on the existing Project, in the order from step 5, each with the epic name and its one-line description. Set a target date only where the PRD's own timeline supplies one.
9. Return a table: Milestone ID · Name · Epic · PRD sections covered · Target date. → **Gate 2 stop.** The PM reviews names, ordering and date alignment in Linear before anything is pushed. Once approved, hand the epic-to-Milestone mapping — together with the PRD Document URL and its section-anchor map — to `push-linear-stories`.

## Confirm-before-write

Echo the plan and require a literal `go` for:
- The **Milestone list** in creation order, with the epic each one comes from and the PRD sections it covers.
- **Target dates**, shown separately, each with the PRD line that supplies it. A milestone with no dated source is listed with no date, not with a guess.
- **Any epic that looks like it should be two Milestones.** Flag it and let the human decide; do not split on your own judgement.

## Refusal cases

- PRD Document not approved, or Project still in `Planned` → refuse and say which precondition failed.
- Milestones already exist on the Project → stop for the human. Adding a second set silently is how a backlog ends up with two competing structures.
- **Never create, rename, re-state, re-label or re-parent the Project itself.** It already exists and its state is a gate signal.
- **Never create Issues here.** Stories are pushed only after this scaffold has been approved.
- **Never invent a target date.** An invented date is read as a commitment by everyone downstream.
- Never create an epic for scope that is not in the Document. If delivery plainly needs it, report it as a gap for the PM to fold into the PRD — the PRD is the contract, and adding scope here bypasses its approval.
- Asked to "push the stories too while you're in there" → refuse; per-story acceptance depends on this scaffold being approved first.
