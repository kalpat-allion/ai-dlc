---
name: author-test-plan
description: Use when a QA Lead or Tech Lead needs the test plan for a project or release produced and published. One procedure: fetch the approved requirements document and the architecture over the tracker's MCP server rather than pasting them, draft the plan with an explicit acceptance-criterion-to-test map, gap-check that draft against the source acceptance criteria and NFRs, resolve every Critical and High finding, then publish it as a tracker Document with stable section anchors behind a confirm-then-go stop. Entering at "check the test plan for gaps" or "publish the test plan" resumes the procedure at that step rather than skipping what precedes it — publication is gated on the gap check clearing. Triggers on "write the test plan", "what's our test strategy for this release", "do we have a test for every AC", "gap-check the test plan before it goes out", "publish the test plan". Do NOT use for: writing test code of any kind, auditing the coverage of an existing suite, assigning test ownership or recording the approval, or silently revising an already-published plan.
---

# Author, gap-check and publish the test plan

You are producing the document every test written this cycle is justified against. The source of truth is the **approved requirements document** and the **architecture** as published — never a summary of either. The number this procedure exists to protect is **zero acceptance criteria that enter the cycle with no test named against them**.

## Procedure

1. **Fetch the sources over MCP. Never work from a paste.** Read the approved requirements Document and the architecture and decision records attached to project `{{LINEAR_PROJECT}}` by URL, through the tracker's MCP server. A pasted excerpt is an excerpt *somebody chose*, and the acceptance-criteria list step 3 checks against has to be the published one — not the portion that fitted in a message. If a source cannot be reached, stop and say which one; never proceed from what the requester remembers.
2. Draft the plan locally in Markdown, nine sections in this order: **§1 Scope** · **§2 Strategy** per test type, naming the framework and coverage target for each — unit and integration under `{{TEST_RUNNER}}`, E2E under Playwright, load under k6, mobile only if the project has a mobile surface · **§3 Cases** · **§4 AC Map**, a table of *AC bullet · test type · test name*, every acceptance criterion carrying at least one row · **§5 Test Data**, including how it is seeded and torn down · **§6 Environments**, naming the performance-matched environment the load tests need · **§7 Entry/Exit** criteria · **§8 Risk Prioritisation**, Critical / High / Medium areas with the testing depth each gets · **§9 Resources**. Name critical journeys individually — never "all flows" — and cite NFR numbers verbatim. **Anything with no source becomes `[NEEDS INPUT: …]`**; an invented AC becomes a test somebody writes and a coverage figure somebody trusts.
3. **Gap-check the draft against the sources, in order:** every source AC with no row in §4 · every NFR number with no load or reliability test asserting against it · every service boundary and external integration with no contract or integration test · edge-case categories the draft never addresses (empty and max-length input, concurrency, timezone, locale, expired session, partial network failure) · test-data handling for sensitive data and teardown · the performance-matched environment. Per gap: **severity · one-line description · suggested addition**, then counts per severity. **Never report a gap you cannot point at a source line for** — a hallucinated gap costs a cycle.
4. **Resolve every Critical and every High in the draft before anything is published**, then **re-run step 3 against the revised draft**. Medium and Low go into an explicit deferred subsection, never silently dropped. The first pass's counts describe a draft that no longer exists, and an unrepeated count is how a Critical reaches a published plan.
5. Fill §9 with the ownership table — one row per §4 test group — and leave **every owner blank**. Who tests what is an assignment made against real availability, and filling it in yourself removes the check it exists to be.
6. **Confirm before writing** (see below), then wait for a literal `go`. On `go`, create the Document attached to project `{{LINEAR_PROJECT}}`, titled `Test Plan v1`, normalising every heading to the §1-§9 form first. Then **re-read the created Document and extract its real anchors**, returning the `§X → #anchor-id` map. Anchors are read back, never predicted — bugs and test issues deep-link into this plan by anchor, and a wrong entry lands the reader in the wrong section.
7. **Stop.** Hand over the Document URL, the anchor map, the step-4 gap counts and the empty ownership table. → the test-plan approval gate: the QA Lead assigns ownership inside the Document and the QA Lead and Tech Lead approve it. Neither signature is yours.

## Confirm-before-write

Echo, and require a literal `go`:

- The **Document** — title, the nine headings exactly as they will be created, and the section the ownership table sits in.
- The **step-4 counts**, which must read `0 Critical · 0 High`. Any other line is a stop, not a warning.
- **Any write onto an existing plan.** Never overwrite or edit `Test Plan v1`. A superseded plan is a version bump the QA Lead asks for by name, echoed before and after — a silent edit orphans every deep-link that cites a heading you moved.

## Refusal cases

- **Never publish while a Critical or High gap is open**, whatever reason is offered — *the Tech Lead will catch it*, *the cycle starts tomorrow*. Nothing downstream re-runs this check, and a published plan reads as an approved one.
- **Never invent an acceptance criterion, an NFR number, a user journey or a gap.** `[NEEDS INPUT: …]` is the answer; a plausible invented latency target becomes a load-test threshold nobody can trace.
- **Never publish a draft the gap check has not been run against.** Asked to publish directly, run steps 3 and 4 on whatever draft exists first, and say that is what you are doing.
- Never assign test ownership, never record an approval, and never mark the Document approved. All three are the gate.
- Writing test code is not yours in any layer — unit and integration tests ship in the same commit as the code they cover, and the end-to-end suite belongs to `e2e-and-coverage-engineer`.
- Auditing what an existing suite already covers is a different job with a different input; that is `e2e-and-coverage-engineer` too. This procedure plans coverage that does not exist yet.
