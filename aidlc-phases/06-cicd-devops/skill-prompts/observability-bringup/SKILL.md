---
name: observability-bringup
description: Use when a service needs its observability layer stood up or refreshed before it takes production traffic — generating the Service Health, SLO Tracking and Business KPI dashboards as committed code, deriving exactly one SLO per stated non-functional requirement with fast-burn and slow-burn alerts, wiring alert routing to the on-call system, and drafting a review-ready runbook for every alert that can fire so no alert reaches production without one. Triggers on "set up dashboards for this service", "we need SLOs before we go live", "generate burn-rate alerts from our NFRs", "every alert needs a runbook", "wire alert routing to pagerduty", "we have no visibility into this service". Uses only the metric names the service actually exposes and lists anything missing as a metric to add rather than guessing a name. Do NOT use for: choosing the observability backend (that is a recorded architecture decision), instrumenting the application with OpenTelemetry (that is application code and belongs to the implementation specialists), debugging or triaging a live incident, writing a service operational runbook, or wiring an alert to production before an SRE has signed off its runbook.
---

# Bring up observability for a service

You are guiding the team through the four observability artefacts, **in order, as one chain** — each step consumes the previous step's output. Running them as four separate sessions with metric names re-pasted between them is exactly where invented metric names enter.

The source of truth is the service's **actually exposed metrics** and the **recorded non-functional requirements**.

## Procedure

1. Read the exposed metrics and the stated non-functional requirements. **Refuse if no measurable target exists** for latency, availability or error rate.
2. Derive **one SLO per requirement** — SLI metric, target, rolling window. No collapsing two requirements into one SLO, no inventing a requirement to justify an SLO.
3. Generate the three dashboards as code for `{{OBS_BACKEND}}` **only** (never both backends), committed under `/observability/dashboards/`. The step-2 SLO targets populate the SLO Tracking dashboard.
4. Generate burn-rate alerts from the step-2 SLOs — fast burn (2% of budget in 1h → page) and slow burn (10% in 6h → ticket) — committed under `/observability/slos/`.
5. Wire routing to `{{ALERT_ROUTER}}`.
6. Draft one runbook per alert at `/docs/runbooks/alerts/<alert-name>.md`, where `<alert-name>` is the alert's name **exactly as configured in `{{OBS_BACKEND}}`, lowercased and kebab-cased**, and set each alert's runbook link to that path. The derivation rule is what makes "every alert links to a runbook" checkable by script instead of by eye. Mark unverifiable steps as to-verify. → **the observability gate: the dashboard and alert blocks pass.**
7. Test each alert: verify it fires and verify it resolves. An untested alert is an assumption.

## Refusal cases

- **Never invent a metric name.** Use only what the service exposes; list anything a panel needs but the service lacks under "metrics to add".
- **Never derive an SLO that no stated requirement backs, and never collapse two requirements into one SLO.**
- Requirements contain no measurable target → refuse. Vague inputs produce noisy alerts, which is worse than no alerts.
- **Never mark a runbook final.** Every generated runbook is a draft pending SRE review before its alert is wired.
- **The instrumentation block is not yours.** OpenTelemetry SDK integration, auto-instrumentation, structured logging and trace propagation are application code — say so rather than reporting that block as passing.
- Never create an alert that does not require human action. If it needs no action, it is a metric.
- Never write outside `/observability/` and `/docs/runbooks/alerts/`. Service operational runbooks live under `/docs/runbooks/services/`, are a different artifact class, and are not yours.
