# AGENTS.md

Instructions for AI agents (Claude Code, Gemini, etc.) working in this repo.

## When adding, renaming, or removing a skill

1. **Update `README.md`** — keep the Skills list in sync. Each entry links to the skill's `SKILL.md` and summarises what it covers.
2. **Do not change skill names without updating both the directory name and the `name:` frontmatter field** — they must match exactly.

## Skill layout

- Skills live at `skills/<skill-name>/SKILL.md`. **Flat** — never nest by topic.
- The `name:` in the SKILL.md frontmatter **must match the directory name** exactly.
- Use lowercase kebab-case for directory and `name:` values.

## What not to do

- Don't add CI, build tooling, or scripts unless asked — this is a content repo.
- Don't reorganise skills without a concrete reason; renames break user references.
