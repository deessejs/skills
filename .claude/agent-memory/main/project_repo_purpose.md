---
name: project-repo-purpose
description: This repo (ds-skills) is a meta-repo that aggregates Claude Code skills from the github.com/deessejs org — not the canonical source
metadata:
  type: project
---

This repo (`ds-skills`) is a **mirror/aggregation point** for Claude Code skills authored by the [deessejs](https://github.com/deessejs) GitHub organization. Each skill lives in its own source repo under `github.com/deessejs/<skill-name>` and is mirrored here under `skills/<skill-name>/`.

**Why:** individual skills are easier to version, owner, and release independently in separate repos; this repo gives a single checkout that exposes the full ecosystem to agents.

**Workflow (current):** The user gives me existing SKILL.md files → I store, improve, and enrich them in `skills/<name>/SKILL.md`. Publishing to github.com/deessejs/repo-per-skill is a future step, not immediate.

**How to apply:**
- When the user gives a SKILL.md: store it in `skills/<name>/SKILL.md`, apply Matt Pocock's quality standards (description format, <100 lines, triggers, examples), then present improvements.
- Skill discovery = list `skills/*/SKILL.md`. Don't assume a skill isn't here without listing first.
- Conventions documented in `AGENTS.md` (kebab-case folders, one skill per folder, self-contained `SKILL.md`).
