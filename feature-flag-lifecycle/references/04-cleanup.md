# Stage 4 — Clean up the flag

You are at this stage if the cleanup child issue is `todo` and the flag is at 100% (or killed).

**Goal of this stage:** remove the flag, remove the losing branch, archive the flag in PostHog, and close the cleanup issue. Leave the codebase as if the flag never existed.

## Decide which branch survived

Read the parent issue's Flag Spec and the ramp comments. If 100%-on, the **new** branch survives. If killed, the **legacy** branch survives. Pick the survivor before touching code; it changes which side you delete.

## Find every call site

Run from the repo root:

```sh
rg -nF "<flag-key>" --hidden --glob '!node_modules' --glob '!*.lock'
```

You should expect to find:
- `useFlag('<flag-key>')` or `isFlagEnabled('<flag-key>')` call sites
- Test mocks: `mockFlag('<flag-key>', …)`
- Any string mention in docs or comments
- The PostHog flag definition in any infra-as-code (none today, but check)

If `rg` returns zero hits after the cleanup PR is staged, you are done with code.

## The cleanup PR

For each call site:
- Replace `useFlag('<key>') ? <NewBranch /> : <LegacyBranch />` with the survivor branch only.
- Delete the dead branch's component / function / module **completely**, plus its tests.
- Delete the corresponding `mockFlag` calls and the test cases for the dead branch.
- Search for analytics properties that only existed to mark the new branch (e.g. `summary_engine='v2'`) — if the survivor is the new branch, those properties become the default and the property assignment can be deleted.

If the helper file (`packages/shared/src/featureFlags.ts`) still has other flags in use, leave it. If this was the last flag, delete the helper too.

## PostHog hygiene

After the PR merges:
1. Archive the flag in the PostHog UI (do not delete — archived flags retain the historical event linkage).
2. Add a one-line note on the archived flag: `Cleaned up in <PR-url> on <date>. Survivor: new|legacy.`

## Close out

```http
PATCH /api/issues/{cleanupIssueId}
{ "status": "done", "comment": "Flag `<key>` removed in <PR-url>. Survivor branch: <new|legacy>. PostHog flag archived." }
```

Then comment on the parent issue with the cleanup PR link and mark the parent done if no other follow-ups remain.

## Anti-patterns to refuse

- "Leave the flag check in case we want to revert." — No. The flag is at 100% (or 0% killed); we ramp a new flag for any future reversal. Dead flag checks rot.
- "Just default the flag to true and delete it later." — No. Default-true flags pile up and create the same dead-code problem the cleanup is meant to prevent.
- "Skip archiving in PostHog because the key is unique." — No. Archive disables the flag definition cleanly and prevents accidental reuse.

## When this stage is done

- Cleanup PR merged.
- `rg "<flag-key>"` returns zero matches in the repo.
- PostHog flag archived with a note.
- Cleanup child issue marked `done`.

**Lifecycle complete.** Update your memory or daily notes if the experiment generated learnings worth keeping (success metric outcome, surprise behaviors, runtime cost). Otherwise, exit the skill.
