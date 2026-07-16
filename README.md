# DeesseJS Skills

Canonical aggregation point for Claude Code skills authored by the [`deessejs`](https://github.com/deessejs) GitHub organization.

Each skill lives in its own source repository under `github.com/deessejs/<skill-name>`. This repo mirrors them here so a single checkout gives an agent (human or AI) access to the full ecosystem.

## Skills

| Skill | Description |
|---|---|
| [`skills/create-issue/`](skills/create-issue/) | Create GitHub issues with org-level type, Priority, Effort fields via REST API |
| [`skills/triage/`](skills/triage/) | Triage a GitHub issue: analyze content, apply labels, post a structured review comment |
| [`skills/spec/`](skills/spec/) | Explore a status:ready issue, write an implementation spec for human review, and push it to a branch |
| [`skills/implement/`](skills/implement/) | Implement a spec-reviewed issue, or apply requested changes from a PR review |

## Repository layout

```
.
├── README.md          # this file
├── AGENTS.md          # instructions for agents working in this repo
├── skills/            # one subdirectory per skill
│   └── <skill-name>/
│       └── SKILL.md   # the skill definition
└── .claude/           # local Claude Code config
```

## For agents

When working in this repo:

1. **Skill discovery** — every folder under `skills/` containing a `SKILL.md` is a loadable skill. List `skills/` to enumerate what's available before assuming a task is not covered.
2. **Skill format** — each `SKILL.md` follows the Claude Code skill spec (YAML frontmatter with `name`, `description`; Markdown body with instructions).
3. **Cross-skill work** — when a task spans multiple skills, treat each skill's `SKILL.md` as the source of truth for its contract. Compose by invoking skills via the Skill tool.
4. **Memory** — persistent agent memory is at `.claude/agent-memory/main/`. Read it before making architectural assumptions.

## Conventions

- Skill folder names are **kebab-case** and match the skill's `name:` in frontmatter.
- One skill = one folder. Do not bundle unrelated skills in the same folder.
- Keep `SKILL.md` self-contained: an agent reading only that one file should be able to use the skill.

## Contributing

Skills are authored in their own repos under `github.com/deessejs/<skill-name>`. Edit upstream first, then re-sync here.

See [`AGENTS.md`](AGENTS.md) for full contributing guidelines.
