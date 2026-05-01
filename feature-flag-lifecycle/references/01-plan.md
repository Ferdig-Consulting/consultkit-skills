# Stage 1 — Plan the flag

You are at this stage if the issue asks you to add, propose, or design a feature flag and no flag key has been chosen yet.

**Goal of this stage:** produce an approved Flag Spec and a paired cleanup child issue. **Do not write gate code in this stage.**

## Flag Spec (write into the parent issue's `plan` document)

```md
## Flag Spec

- **Key:** `<app>_<feature>_<verb>` (lowercase, snake_case, no version inside the key — versions belong in the feature name itself)
- **App(s) affected:** referee | testify | clarify | dashboard | directory
- **Default in production:** OFF
- **Default in tests:** ON for new-branch tests, OFF for legacy-branch tests
- **Audience strategy:** internal-only → 10% → 50% → 100% (or describe the targeting cohort)
- **Exposure metric:** the PostHog event + property combination that proves the new branch is taking effect (e.g. `summary_generated` with `summary_engine='v2'`)
- **Success metric:** the user-facing metric this flag is supposed to move (e.g. summary completion rate +X%)
- **Kill criteria:** any condition under which we ramp back to 0 immediately (error rate, p95 latency, support reports)
- **Planned 100% date:** YYYY-MM-DD
- **Cleanup target date:** YYYY-MM-DD (≤ 30 days after 100%)
- **Cleanup issue:** [link to the child issue created below]
- **Owner:** Paperclip agent or human responsible for ramp + cleanup
```

Update the `plan` document via `PUT /api/issues/{issueId}/documents/plan` (see the `paperclip` skill for the exact payload). Send the latest `baseRevisionId` if the document already exists.

## Mandatory: create the cleanup child issue **now**

A flag without a scheduled cleanup is a bug. Create the cleanup issue in the same heartbeat that proposes the flag. Use `POST /api/companies/{companyId}/issues`:

```json
{
  "title": "Clean up flag: <flag-key>",
  "description": "Remove the `<flag-key>` PostHog flag and its losing branch. Spec: [<parent-identifier>](/CON/issues/<parent-identifier>#document-plan). Cleanup target: <YYYY-MM-DD>.",
  "parentId": "<parent-issue-id>",
  "goalId": "<parent-goal-id>",
  "priority": "medium",
  "status": "blocked",
  "blockedByIssueIds": ["<parent-issue-id>"],
  "assigneeAgentId": "<implementing-engineer-agent-id>"
}
```

The cleanup issue stays `blocked` until the flag reaches 100% — at which point Stage 3 unblocks it and reassigns to the cleanup owner.

## Approval gate

Flag specs that change pricing, billing, or legal/compliance behavior require Founder approval (see your CTO/CEO `AGENTS.md`). For everything else, request review from the PM or CTO via reassignment with `status: in_review` and a comment pointing at the updated `plan` document.

## When this stage is done

- Flag Spec section exists in the parent issue's `plan` document.
- Cleanup child issue exists, blocked by the parent, assigned to a real owner.
- Spec is reviewed/approved (or queued for review).

**Next action:** when implementation is approved, reload this skill and switch to [`02-implement.md`](02-implement.md). Leave a heartbeat comment that names the next reference so the implementer (often a different agent) starts at the right place.
