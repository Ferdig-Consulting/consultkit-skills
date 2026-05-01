# consultkit-skills

Public mirror of shareable Claude/Paperclip skill bundles for ConsultKit agents. Imported by Paperclip via the GitHub source so progressive-disclosure references (`SKILL.md` + `references/*.md`) are picked up.

## Skills

- `feature-flag-lifecycle/` — plan → implement → rollout → cleanup workflow for PostHog feature flags in pcc-apps.

## Authoring

Skill bundles here MUST be safe to publish (no secrets, no internal infra details, no business logic). Source of truth lives in `Ferdig-Consulting/consultkit-apps/.claude/skills/`; copy here when ready to ship to the company library.

## Importing

```sh
curl -sS -X POST "$PAPERCLIP_API_URL/api/companies/$PAPERCLIP_COMPANY_ID/skills/import" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"source": "https://github.com/Ferdig-Consulting/consultkit-skills/tree/main/<skill-name>"}'
```
