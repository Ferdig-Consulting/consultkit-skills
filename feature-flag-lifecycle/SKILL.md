---
name: feature-flag-lifecycle
description: Ship a feature flag with cleanup. Use when adding, ramping, or removing a PostHog feature flag in pcc-apps. Stages the workflow (plan → implement → rollout → cleanup) across separate references so each step loads only what is needed. Triggers on phrases like "add a feature flag", "gate this behind a flag", "ramp the flag", "clean up the flag", or when a Paperclip issue mentions a flag spec or flag removal.
---

# Feature Flag Lifecycle (Pilot)

ConsultKit ships behind PostHog feature flags. Every flag must have a planned cleanup or it becomes dead code. This skill stages the workflow across four references — load **only the one** for your current stage.

This skill follows the **progressive disclosure** pattern: the entry SKILL.md (this file) is intentionally short. Do not pre-load every reference. Identify the stage, then read **one** reference.

## Pick your stage

| If the situation is… | Load this reference | Output |
|---|---|---|
| Deciding whether/how to add a flag — name, default, audience, exposure metric, expiration date | `references/01-plan.md` | Flag Spec in the issue plan document |
| Spec is approved, you are about to write the gate code | `references/02-implement.md` | A small, well-tested diff that gates the new behavior |
| Code is merged, you are ramping in staging/prod and watching the exposure metric | `references/03-rollout.md` | Verified rollout, ramp comments on the issue, kill-switch confirmed |
| Flag is at 100% (or kill) and the experiment/launch is settled — time to remove | `references/04-cleanup.md` | Cleanup PR removing the flag and the losing branch |

If you cannot identify the stage from the issue context, **ask** before loading any reference. Do not guess.

### How to load a reference

- **You have pcc-apps checked out** (e.g. Software Engineer, CTO): use the Read tool on `.claude/skills/feature-flag-lifecycle/references/<file>.md`.
- **You don't have pcc-apps on disk**: WebFetch the raw file from the public mirror at `https://raw.githubusercontent.com/Ferdig-Consulting/consultkit-skills/main/feature-flag-lifecycle/references/<file>.md`. Paperclip's company-skill importer does not currently walk `references/`, so the skill body delivered through the agent prompt is just this SKILL.md — you must fetch references explicitly.

## Non-negotiables (read every stage)

1. **Every flag is born with a cleanup ticket.** When stage 1 plans a flag, it MUST create a child Paperclip issue titled `Clean up flag: <flag-key>` assigned to the implementing engineer with a `targetCleanupDate` no later than 30 days after planned 100% rollout. No flag spec is approved without it.
2. **Default OFF in production, ON in tests for new code paths.** This forces tests to exercise the new branch and keeps prod safe at boot.
3. **Use the shared helper, not raw `posthog.isFeatureEnabled` at call sites.** The helper is named in [`references/02-implement.md`](references/02-implement.md) and the lint rule is named in [`references/04-cleanup.md`](references/04-cleanup.md).
4. **Exposure event is captured exactly once per session per flag** (the helper does this). Never duplicate `$feature_flag_called` from app code.
5. **Flag key naming:** `<app>_<feature>_<verb>` — e.g. `clarify_summary_v2_enabled`, `referee_referrer_invite_v2`. Never reuse a key after archiving.

## Heartbeat hand-off

If your work for this heartbeat lands you partway through a stage, leave a Paperclip comment naming **the stage** and **the next reference to load**. The next heartbeat (or the next agent) re-enters the skill at the right place without rereading the upstream stages.

## Why this skill is staged

Loading every flag-lifecycle detail at the planning stage would burn context on cleanup commands the agent will not execute today. PostHog's progressive-disclosure technique (CON-1329 research, 2026-04-30) showed an ~80% v1-correct rate when reference material is staged behind a thin entry skill. This pilot tests that pattern internally before we apply it to other multi-step workflows.

Pilot tracker: [CON-1345](/CON/issues/CON-1345). Source plan: [CON-1328](/CON/issues/CON-1328#document-plan).
