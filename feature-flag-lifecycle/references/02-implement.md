# Stage 2 — Implement the gate

You are at this stage if the Flag Spec is in the parent issue's `plan` document and you are about to write code.

**Goal of this stage:** a small, well-tested diff that gates the new branch behind the flag and captures one exposure event per session.

## The shared helper (use this — never call PostHog directly at the call site)

The shared helper lives in `packages/shared/src/featureFlags.ts` (create it the first time this skill ships a flag — see the bootstrap snippet at the end of this file). Call sites use:

```ts
import { useFlag } from '@consultkit/shared/featureFlags';

const summaryV2 = useFlag('clarify_summary_v2_enabled');
return summaryV2 ? <SummaryV2 /> : <SummaryV1 />;
```

Server-side / outreach-runner:

```ts
import { isFlagEnabled } from '@consultkit/shared/featureFlags';

if (await isFlagEnabled('outreach_quiet_hours_v2', { workspaceId })) {
  …
}
```

The helper:
1. Calls PostHog under the hood (`posthog.isFeatureEnabled` in browser, `posthog.getFeatureFlag` server-side).
2. Captures `$feature_flag_called` exactly once per (session, flagKey) — never duplicate this from app code.
3. Defaults to **false** if PostHog is offline. Never assume the new branch on failure.

## Implementation rules

- **Diff size:** keep the gate diff < ~150 lines where possible. If you need a bigger refactor, do that in a *separate* PR before introducing the flag.
- **Both branches must be exercised by tests.** Add at least one test per branch. The legacy branch keeps its existing tests; the new branch needs new tests asserting the new behavior. Use the pattern below to flip the flag in tests.
- **No flag checks deep in libraries.** Gate at the highest practical level — usually the route handler, page component, or feature entry point. Deep flag checks make cleanup painful.
- **No nested flag gates.** A single flag controls a single decision point.
- **Capture the exposure metric.** If the spec names `summary_engine='v2'` as the exposure metric, ensure the new branch sets that property when it emits its analytics events.

## Test pattern

```ts
import { mockFlag } from '@consultkit/shared/featureFlags/testing';

describe('summary v2 gate', () => {
  it('renders v2 when flag is on', () => {
    mockFlag('clarify_summary_v2_enabled', true);
    …
  });

  it('renders v1 when flag is off', () => {
    mockFlag('clarify_summary_v2_enabled', false);
    …
  });
});
```

## Bootstrap snippet (only the first flag in the codebase needs this)

If `packages/shared/src/featureFlags.ts` does not exist yet, create it as part of this PR. Minimal shape:

```ts
import { posthog } from 'posthog-js';

const exposureFired = new Set<string>();

export function useFlag(key: string): boolean {
  const enabled = posthog?.isFeatureEnabled?.(key) ?? false;
  if (!exposureFired.has(key)) {
    exposureFired.add(key);
    posthog?.capture?.('$feature_flag_called', { $feature_flag: key, $feature_flag_response: enabled });
  }
  return Boolean(enabled);
}

export async function isFlagEnabled(
  key: string,
  ctx: { workspaceId?: string } = {},
): Promise<boolean> {
  // server-side path — wire up via posthog-node in the runner package
  return false;
}
```

Add the test helper at `packages/shared/src/featureFlags/testing.ts` with `mockFlag(key, value)` overriding the runtime function. Document both in the package README.

## When this stage is done

- New branch is gated behind the helper at one well-chosen point.
- Tests cover both branches.
- PR description references the Flag Spec section of the parent issue's `plan` document.
- Exposure metric is firing (verify locally with PostHog devtools or the test event recorder).

**Next action:** once the PR is merged to staging, reload this skill and switch to [`03-rollout.md`](03-rollout.md). Update the parent issue with the merge commit + flag-on test result, and re-comment naming the next reference.
