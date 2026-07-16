# AGENTS.md

This repository is the **canonical aggregation point for Claude Code skills authored by the [`deessejs`](https://github.com/deessejs) GitHub organization**.

Individual skills live in their own source repositories under `github.com/deessejs/<skill-name>`. This repo mirrors them under `skills/` so a single checkout gives an agent (human or AI) access to the full ecosystem.

## Repository layout

```
.
├── AGENTS.md          # this file — instructions for agents working in this repo
├── skills/            # one subdirectory per skill, named after the skill
│   └── <skill-name>/
│       └── SKILL.md   # the skill definition (required)
│       └── ...        # any helper scripts, templates, references
└── .claude/           # local Claude Code config (agents, settings, memory)
```

Each skill folder under `skills/` mirrors the layout of its source repo. **Do not edit a mirrored copy directly** — changes must be made upstream in the source repo under `github.com/deessejs/<skill-name>` and re-synced here. See [Syncing a skill](#syncing-a-skill-from-its-source-repo).

## For agents (Claude Code or compatible)

When working in this repo:

1. **Skill discovery** — every folder under `skills/` containing a `SKILL.md` is a loadable skill. List `skills/` to enumerate what's available before assuming a task is not covered.
2. **Skill format** — each `SKILL.md` follows the Claude Code skill spec (YAML frontmatter with `name`, `description`, optional `metadata`; Markdown body with instructions). Do not invent a custom format.
3. **Tooling** — this repo's `.claude/agents/main/README.md` lists the local CLI helpers (e.g. `fresh` for web search/fetch). Prefer them over ad-hoc shell commands.
4. **Memory** — the main agent has persistent project memory at `.claude/agent-memory/main/`. Read it before making architectural assumptions; save non-obvious context (user role, feedback, project decisions, external references) when learned.
5. **Cross-skill work** — when a task spans multiple skills, treat each skill's `SKILL.md` as the source of truth for its contract. Do not duplicate skill logic; compose by invoking skills via the Skill tool.

## Conventions

- Skill folder names are **kebab-case** and match the skill's `name:` in frontmatter.
- One skill = one folder. Do not bundle unrelated skills in the same folder.
- Keep `SKILL.md` self-contained: an agent reading only that one file should be able to use the skill. Move large reference material to a `references/` subfolder if it bloats the skill.
- Update the upstream source repo first, then re-sync here. Never commit directly to a mirrored folder.

## Adding a new skill

1. Publish the skill in its own repo under `github.com/deessejs/<skill-name>` (with a valid `SKILL.md`).
2. Add a submodule or subtree in this repo under `skills/<skill-name>` pointing at the source repo, **or** mirror it as a folder if submodule is undesirable (document the choice in the skill's folder `README.md`).
3. Update the [index](#skill-index) below.

## Syncing a skill from its source repo

```bash
# example for a submodule-managed skill
git submodule update --remote skills/<skill-name>

# example for a subtree-managed skill
git subtree pull --prefix=skills/<skill-name> <remote-url> main --squash
```

Always run the skill's own test/validation hooks (if any) after a sync before committing.

## Skill index

> Filled in as skills are added. Format:
>
> - `skills/<name>` — <one-line description>

- `skills/create-issue` — create GitHub issues with org-level type, Priority, Effort fields via REST API
- `skills/triage` — triage a GitHub issue: analyze content, apply labels, post a structured review comment
- `skills/spec` — explore a status:ready issue, write an implementation spec for human review, push to branch
- `skills/implement` — implement a spec-reviewed issue, or apply requested changes from a PR review
- `skills/create-pr` — open a PR for an implemented issue: reads spec, opens PR, updates labels, posts comment
- `skills/review-pr` — review an open PR: check CI, validate against the spec, post approve/request-changes/comment review
