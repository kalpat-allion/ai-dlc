---
name: finalise-documentation
description: Use when a project's published documentation set has to be finished and gap-checked before it is delivered to anyone — one ordered procedure: read the docs build configuration and the page tree that already exists, agree the section structure, generate each page from cited repository sources, render the API reference from the specification instead of hand-writing it, run the completeness audit against the codebase, resolve every Critical and High gap by looping back into page generation, run the link check, and stop for human accuracy review before anything publishes. Entering at "check our docs for gaps" resumes at the audit and loops back into generation for every Critical or High finding, so the audit cannot terminate before the publish gate. Triggers on "write the getting-started page", "generate the local dev setup docs", "are our docs complete", "what's missing from our docs site", "we need the documentation finished before handoff". Self-checks fifteen of the documentation gate's nineteen items and names the other four by owner rather than claiming them. Do NOT use for: one module's README or its inline code comments inside a feature story (that is the repo's module-documentation command), hand-writing API reference content or editing the API specification itself, writing or rewriting an alert's runbook, auditing a live knowledge base against production signal, or the handoff document and the post-handoff retrospective — those are one synthesised deliverable for a receiving team, not pages on the documentation site.
---

# Finish and gap-check the documentation set

You are closing out the documentation that ships with the project — the pages a receiving team, a new developer, or a customer reads after the people who built it have gone.

**Generation and the completeness audit are one loop, not two jobs.** Run separately, they produce a site that looks finished and is missing the page somebody needs at 2am: the audit's findings are the next generation pass's input, and the loop is what the gate measures. Whichever end you are entered at, you finish the loop.

Every page is a draft until a human has checked it for accuracy. **You never publish.**

## Procedure

1. **Read before writing.** The docs build configuration, the navigation, the existing page tree under `{{DOCS_DIR}}`, and the voice already in use. **Prefer editing an existing page over adding a near-duplicate** whenever it is already 70% right — a second page on the same subject is how a docs site starts contradicting itself.
2. **Agree the section structure with the Tech Lead** before generating pages: getting started, user guides, API reference, architecture, operations, developer setup, troubleshooting, glossary. Drop any section this project genuinely does not need and say which and why — a library needs no end-user guide, and a padded structure produces padded pages.
3. **Generate each page from cited repository sources.** Clear hierarchy; action-oriented headings ("Install X", not "Installation"); working examples traced to real source, never `foo`/`bar`; diagrams and screenshots referenced with descriptive alt text; a common-pitfalls section; next-step links. Voice: second person for guides, imperative for how-to, third person for reference. **Never "simply", "just", or "easy"** — each one tells a stuck reader the problem is them.
4. **Render the API reference from `{{API_SPEC_PATH}}`.** The specification is the source of truth and the page renders from it. Hand-written API docs drift from the spec within two sprints and nothing detects it.
5. **Link the decision records in `{{ADR_DIR}}` from the architecture section.** Link them; never copy, summarise, or re-word one into a page. A restated decision is a second source of truth that nobody updates.
6. **Run the completeness audit** — cross-reference the codebase against the expected set across user-facing, developer, operations and handoff documentation. One row per gap: the missing page by name, priority, a rationale tied to a named user, role or operational risk, and the source to build it from. **Do not flag what does not apply to this project; do not flag what already exists under another name — search first.** Where the inputs cannot tell you whether a page exists, emit `[INSUFFICIENT INPUT: <what would let you decide>]` for that row rather than guessing.
7. **Resolve every Critical and High gap by looping back to step 3.** Medium and Low are filed as tracked documentation debt with an owner — never dropped, never quietly promoted to done.
8. **Run the repository's link check and paste the real output.** If no link checker is configured, say so and report that gate item unverified rather than asserting the links work.
9. **Self-check and report using the table below**, then stop for human accuracy review. → **the documentation gate.**

## The nineteen gate items, and the four you cannot certify

Report fifteen verdicts as met or unmet, and **name these four with their owner**:

| Gate item | Verdict |
|---|---|
| Runbook per alert | **Cannot verify — whoever owns the alert inventory.** You can confirm the runbook set is linked and reachable. One-runbook-per-alert needs the live alert list, and an alert configured in the observability backend but not committed as code is invisible to you |
| All auto-generated content reviewed by a human | **Cannot verify — the Tech Lead.** You wrote it, which makes you the wrong reader of it |
| Screenshots and diagrams accurate and current | **Cannot verify — the Tech Lead.** You cannot see a rendered screen, so you cannot know a screenshot still matches one |
| Documentation site deployed and searchable | **Cannot verify — whoever publishes.** You stop before publish by design, and search behaviour is a property of the deployed site |

**A report claiming all nineteen has fabricated four of them.**

## Refusal cases

- **Never publish, deploy, or merge the documentation site.** You produce pages and a gap report; the accuracy review and the publish are human steps, and a page that publishes unread is the one failure this procedure exists to prevent.
- **Never hand-write API reference content that renders from the specification, and never edit the specification to make a page easier to write.** Both invert the direction the truth flows in.
- **Never invent a version number, an environment-variable default, a repository name, an endpoint, a metric name, or a command.** Cite the file it came from, or mark it `[INSUFFICIENT INPUT: ...]`. These are exactly the details a generated page states confidently and gets wrong, and the reader has no way to tell.
- **Never write or rewrite an alert's runbook.** Link to it. An alert runbook is derived from the alert and belongs to whoever owns the alert; a documentation pass rewriting one produces a runbook that no longer matches what fires.
- **Never edit a decision record, a requirement document, or a runbook to make the documentation set look complete.** They are read-only sources. Link to them and, where one is wrong, report it as a gap against its owner.
- **Never write a disaster-recovery plan from nothing.** Recovery objectives are stated by people, not derivable from a repository, and a generated DR plan reads authoritative while having never been tested. Report it as a gap with a named owner.
- **Never pad the gap list.** A fabricated gap costs the same review time as a real one and teaches the reader to skim, which is how the real Critical row gets missed.
- Asked to tick the four hand-back items because "they're obviously fine" → refuse and name their owners again. Every one of them is a check about something you cannot see.
