# consultkit-skills

Public mirror of shareable Claude/Paperclip skill bundles for ConsultKit agents.

## Skills

- `feature-flag-lifecycle/` — plan → implement → rollout → cleanup workflow for PostHog feature flags in pcc-apps.

## Authoring standard (required for every skill in this repo)

Paperclip's company-skill importer ingests **only the top-level `SKILL.md`** for both `local_path` and `github` sources — `references/*.md` are silently dropped from the company-skill body delivered to agents. To make progressive-disclosure skills work for any agent (with or without `pcc-apps` checked out), every skill in this repo MUST follow this layout:

1. `SKILL.md` is short. List each stage/reference and tell the agent **how to load it** explicitly.
2. Every reference is reachable two ways:
   - `Read .claude/skills/<skill-name>/references/<file>.md` — for agents with `pcc-apps` on disk.
   - `WebFetch https://raw.githubusercontent.com/Ferdig-Consulting/consultkit-skills/main/<skill-name>/references/<file>.md` — for off-repo agents.
3. References live under `<skill-name>/references/`. Don't introduce nested subfolders unless required.
4. Skill content must be safe to publish — no secrets, no internal infra details, no business logic. Source of truth for unpublished drafts lives in `Ferdig-Consulting/consultkit-apps/.claude/skills/`.

Copy `_TEMPLATE/` when starting a new skill and replace the placeholders.

## Importing

```sh
curl -sS -X POST "$PAPERCLIP_API_URL/api/companies/$PAPERCLIP_COMPANY_ID/skills/import" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"source": "https://github.com/Ferdig-Consulting/consultkit-skills/tree/main/<skill-name>"}'
```

Refresh after a push:

```sh
curl -sS -X POST "$PAPERCLIP_API_URL/api/companies/$PAPERCLIP_COMPANY_ID/skills/<skill-id>/install-update" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```
