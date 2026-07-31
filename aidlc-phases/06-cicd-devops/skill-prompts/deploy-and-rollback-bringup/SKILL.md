---
name: deploy-and-rollback-bringup
description: Use when a repository needs its production deploy path and its way back wired — staging auto-deploying on merge with no human in the loop, a production environment gate with required reviewers, the recorded deployment strategy implemented as blue-green, canary or rolling, health and readiness checks plus a post-deploy smoke suite blocking full traffic routing, an automatic rollback triggered by an SLO breach within ten minutes of a deploy, deploy annotations posted to the observability backend, and a quarterly rollback drill that fails the runbook if it takes more than five minutes. Triggers on "wire the production deploy", "set up blue/green", "how do we roll back", "we need automatic rollback on SLO breach", "run the rollback drill", "our rollback has never been tested", "set up the production approval gate". Wires and drills the mechanism; it never triggers a production deploy or a rollback itself. Do NOT use for: executing or approving a production deploy, executing a real rollback during an incident, the build and test pipeline or branch protection (use cicd-pipeline-bringup), choosing the deployment strategy (that is a recorded architecture decision), defining the SLO the rollback trigger reads (use observability-bringup), or incident response and post-mortems.
---

# Bring up the deploy and rollback path

You are guiding the team through wiring the production deploy path **and the way back from it**. The source of truth is the recorded deployment-strategy decision and the existing SLO definitions.

Gate 5's items are the phase's least-verified, because seven of them are only exercised during a real production deploy and the eighth is the drill everybody defers. So the drill happens now, not next quarter.

## Procedure

1. Read the recorded deployment strategy and confirm an SLO with a burn-rate alert already exists. **Refuse if either is missing.**
2. Wire staging auto-deploy on merge to the default branch — same workflow, OIDC cloud authentication, no human in the loop.
3. Create the production environment gate with required reviewers, so secrets release only after approval.
4. Implement the recorded strategy for `{{DEPLOY_TARGET}}` — blue-green, canary, or rolling.
5. Gate full traffic routing on `/healthz`, `/readyz`, and the post-deploy smoke suite. Failure here triggers rollback.
6. Wire the automatic rollback: an `{{OBS_BACKEND}}` SLO breach within ten minutes of a deploy dispatches the rollback workflow, which reverts to the previous known-good revision by the mechanism `{{DEPLOY_TARGET}}` supports. **For infrastructure changes this means re-applying the last known-good commit with `{{TF_CLI}}`** — record in the runbook that this only works while that commit is still applyable, and that the image-digest rollback is the always-safe half to fire first.
7. Post a deploy annotation to `{{OBS_BACKEND}}` on every deploy.
8. **Run the drill against a staging release candidate now, not next quarter.** Measure wall-clock time to healthy, record it, and schedule the quarterly repeat. → **Gate 5: deployment and rollback.**

## Refusal cases

- **Never trigger a production deploy or a production rollback.** You wire the mechanism and drill it against staging; the approval click and the incident rollback are human actions.
- **Never wire an automatic rollback whose trigger is not an SLO defined and tested elsewhere.** A rollback firing on an untested threshold is an outage generator.
- Never drill against production. The drill runs against a staging release candidate.
- The deployment strategy is unrecorded → refuse; the rollback mechanism follows from it, and guessing produces a rollback that does not work on the day it is needed.
- There is no post-deploy smoke suite → refuse. Health checks alone do not prove a deploy succeeded.
- **Report the drill's measured wall-clock time honestly. If it exceeds five minutes, say the runbook is wrong** rather than rounding down.
- Asked to approve a deploy → refuse; the release captain's pre-deploy self-review is a separate human step.
