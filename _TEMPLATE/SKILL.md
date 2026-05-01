---
name: SKILL_NAME
description: ONE-LINE TRIGGER STATEMENT. Use when X. Triggers on phrases like "...", "..."
---

# Human-Readable Title

ONE-PARAGRAPH OVERVIEW. State the goal of the skill and why it's staged.

This skill follows the **progressive disclosure** pattern: this entry SKILL.md is intentionally short. Do not pre-load every reference. Identify the stage, then read **one** reference.

## Pick your stage

| If the situation is… | Load this reference | Output |
|---|---|---|
| FIRST STAGE DESCRIPTION | `references/01-NAME.md` | OUTPUT NAME |
| SECOND STAGE DESCRIPTION | `references/02-NAME.md` | OUTPUT NAME |

If you cannot identify the stage from the issue context, **ask** before loading any reference. Do not guess.

### How to load a reference

- **You have pcc-apps checked out** (e.g. Software Engineer, CTO): use the Read tool on `.claude/skills/SKILL_NAME/references/<file>.md`.
- **You don't have pcc-apps on disk**: WebFetch the raw file from the public mirror at `https://raw.githubusercontent.com/Ferdig-Consulting/consultkit-skills/main/SKILL_NAME/references/<file>.md`. Paperclip's company-skill importer does not currently walk `references/`, so the skill body delivered through the agent prompt is just this SKILL.md — you must fetch references explicitly.

## Non-negotiables (read every stage)

1. RULE ONE.
2. RULE TWO.

## Heartbeat hand-off

If your work for this heartbeat lands you partway through a stage, leave a Paperclip comment naming **the stage** and **the next reference to load**. The next heartbeat (or the next agent) re-enters the skill at the right place.
