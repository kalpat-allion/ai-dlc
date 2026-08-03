---
description: "Write a Module README for the module at the given path — overview, architecture, key concepts, usage, configuration, testing, limitations — where each section is written only from evidence found in the module itself, and a section without that evidence is omitted rather than padded. Reports all seven sections back as written-with-evidence or omitted-with-what-was-looked-for, so an omission is visible rather than silent. Never hand-writes API reference content that renders from the API specification. Usage: /write-module-readme src/orders"
argument-hint: "[path to the module, e.g. src/orders]"
disable-model-invocation: true
---

# Module README

You are documenting a module for a developer who has never opened it. Write what they need to be productive — not what they could derive by reading the code.

**Module:** $ARGUMENTS

## Procedure

1. **Read the module first** — its public exports and their real signatures, its imports in both directions, every line that reads configuration, its tests. You write from that reading and from nothing else.
2. **Write only the sections that survive the evidence rule.** Seven are available. Fewer than seven is the normal outcome; **seven every time means the rule is not being applied** — a README generator asked for seven sections will find seven sections' worth of things to say whether or not they are true.
3. **Write `README.md` into the module's own directory**, then **report all seven sections** — each as `written` with the evidence you cited, or `omitted` with what you looked for and did not find. A section that simply vanishes is indistinguishable from one nobody considered, which is how padding comes back.

## The seven sections, and the evidence each requires

A section is written **only** if its evidence exists in what you actually read. No evidence → omit. The evidence is what you name in the report.

| Section | Written only from | Omit when |
|---|---|---|
| **Overview** — 2-3 sentences on what it does | The public exports taken together | Never. A module whose purpose you cannot state from its exports is a finding — report that instead of guessing |
| **Architecture** — where it sits; named upstream and downstream | Actual import edges, both directions | Nothing local imports it and it imports nothing local |
| **Key Concepts** — domain terms and patterns particular to this module | Terms in the code whose meaning is not self-evident from the name | Every name already means what it says |
| **Usage** — import path and a minimum-viable example per public export | The real exported symbol and its real signature | There is no public export |
| **Configuration** — name, type, default, purpose | The line that actually reads the env var, config key or flag | Nothing in the module reads configuration |
| **Testing** — the exact command | A `{{TEST_RUNNER}}` invocation scoped to this module **that you ran** | You could not run it — omit, and say so in the report rather than printing an untested command |
| **Limitations** — current limits and known debt | An in-code `TODO` / `FIXME`, or a tracked issue labelled `{{TECH_DEBT_LABEL}}` | Neither exists |

## Refusals

- **Never invent to fill a section.** A configuration key nothing reads, an upstream module nothing imports, a limitation nobody recorded — these are the lines a new developer trusts most and can verify least. **Omission is the correct output here, not the degraded one.**
- **`TODO`, `TBD`, `<describe here>`, "this module likely…" — all padding.** Reaching for one means the section failed the evidence rule. Delete the section; do not soften the sentence.
- **Never hand-write API reference.** Endpoints, request and response shapes, status codes and field tables render from `{{API_SPEC_PATH}}`. Link to the generated documentation. A hand-copy is wrong the first time the spec changes, and nothing tells the reader which of the two is stale.
- **Never restate a signature the reader can read.** A README that lists every function is derivable from the code and rots faster than the code does.
- **One module, one path, one file.** Asked to document a whole repository, or a module you were not pointed at, decline and ask for the path.
