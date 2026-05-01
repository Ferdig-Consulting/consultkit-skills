# Stage 3 — Roll out and verify

You are at this stage if the gate code is merged to staging or production and the flag is at <100%.

**Goal of this stage:** ramp the flag safely, verify the exposure + success metrics, and unblock the cleanup child issue when at 100%.

## Pre-ramp verification (do once, in staging)

1. Confirm the PostHog flag exists in staging at the planned key. If missing, create it in the PostHog UI (or via PostHog API) **with default OFF**, then turn it on for staging traffic.
2. Hit the gated path in staging with the flag ON. Confirm in PostHog:
   - `$feature_flag_called` event fires with `$feature_flag = <key>` and `$feature_flag_response = true`.
   - The exposure metric (e.g. `summary_generated` with `summary_engine='v2'`) shows the new value.
3. Hit the gated path with the flag OFF. Confirm legacy behavior is unchanged.
4. If either check fails, ramp **back to 0** and reopen Stage 2.

## Production ramp plan

Default cadence (override per spec):

| Step | Audience | Wait | Watchdog |
|---|---|---|---|
| 1 | Internal users only | 24h | exposure metric appears at all |
| 2 | 10% rollout | 48h | error rate, p95 latency, kill-criteria from spec |
| 3 | 50% rollout | 48h | success metric trending the right direction |
| 4 | 100% rollout | n/a | unblock cleanup issue |

Post a Paperclip comment on the parent issue at each step:

```md
Ramp step <N>: <audience>. Posthog flag `<key>` set to <%>. Exposure event count last 24h: <X>. Error rate delta: <Y>. Holding <duration> before next step.
```

## Kill switch

If any kill criterion triggers:
1. Set the PostHog flag to 0% **immediately**.
2. Comment on the parent issue: `Kill triggered at <step>. Reason: <criterion>. Flag back to 0%.`
3. Reopen Stage 2 to fix the underlying issue, OR move to Stage 4 to remove the new branch entirely.

Do **not** wait for review to flip a kill switch. Comment after, not before.

## At 100%

When the flag has been at 100% for the spec's defined soak window (default: 7 days) with success criteria met:

1. Update the parent issue: `Flag at 100% since <date>. Cleanup target: <date>.`
2. Unblock the cleanup child issue:

   ```http
   PATCH /api/issues/{cleanupIssueId}
   { "status": "todo", "blockedByIssueIds": [] }
   ```

   Remember: `PATCH` replaces `blockedByIssueIds` wholesale — sending `[]` is intentional here.
3. Reassign the cleanup issue to the implementing engineer (or the agent named in the Flag Spec) and leave a comment: `Flag at 100%. Load feature-flag-lifecycle skill stage 4.`

## When this stage is done

- Flag at 100% (or killed, in which case proceed straight to cleanup).
- Cleanup child issue moved to `todo` and assigned.
- Parent issue updated with final ramp metrics.

**Next action:** the cleanup work belongs to the cleanup issue. Reload this skill in that heartbeat and switch to [`04-cleanup.md`](04-cleanup.md).
