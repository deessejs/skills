---
name: reference-mattpocock-skills
description: mattpocock/skills on GitHub (172K stars) — gold standard for Claude Code skills; philosophy, structure, and skill inventory
metadata:
  type: reference
---

## Source

**Repo:** `github.com/mattpocock/skills` — 172K stars, MIT
**skills.sh:** https://www.skills.sh/mattpocock/skills (567K installs)
**Install:** `npx skills@latest add mattpocock/skills` or Claude Code plugin (`/plugin install mattpocock-skills@mattpocock`)

## Philosophy

> "Skills are designed to be **small, easy to adapt, and composable**. They work with any model. Based on decades of engineering experience."

Matt Pocock built these to fix 4 failure modes he saw with coding agents:

1. **Misalignment** → `/grill-me` + `/grill-with-docs` (grilling sessions before building)
2. **Verbose output** → shared domain language via `CONTEXT.md` + ADRs
3. **Code doesn't work** → `/tdd` (red-green-refactor loop) + `/diagnosing-bugs`
4. **Ball of mud** → `/improve-codebase-architecture` + `/codebase-design`

## Key Concepts

- **User-invoked skills** — only reach when typed (e.g. `/grill-me`). Their job: orchestrate.
- **Model-invoked skills** — reached automatically by the agent when relevant. Hold reusable discipline.
- **Grilling session** — the core technique: agent interviews user relentlessly until every branch of the decision tree is resolved. Built into `/grill-me`, `/grill-with-docs`, `/grilling`.
- **Shared language** — `CONTEXT.md` file that defines project jargon. Reduces agent verbosity dramatically.
- **ADR** — Architecture Decision Records for documenting hard-to-explain decisions.

## Skill Inventory

### Engineering — User-invoked
| Skill | What it does |
|---|---|
| `/ask-matt` | Router — ask which skill fits your situation |
| `/grill-with-docs` | Grilling session that also builds domain model, updates `CONTEXT.md` + ADRs |
| `/triage` | Move issues through triage state machine using labels |
| `/improve-codebase-architecture` | Scan for deepening opportunities → visual HTML report → grill through picks |
| `/setup-matt-pocock-skills` | One-time per-repo config: issue tracker, triage labels, doc layout |
| `/to-spec` | Synthesize current conversation into a spec, publish to tracker |
| `/to-tickets` | Break spec/plan into tracer-bullet tickets with blocking edges |
| `/implement` | Build from spec/tickets, drives `/tdd` at seams, closes with `/code-review` |
| `/wayfinder` | Plan huge multi-session work as investigation tickets on tracker |

### Engineering — Model-invoked
| Skill | What it does |
|---|---|
| `/prototype` | Throwaway prototype to answer a design question |
| `/diagnosing-bugs` | Loop: reproduce → minimise → hypothesise → instrument → fix → regression-test |
| `/research` | Investigate against primary sources → cited Markdown in repo, background agent |
| `/tdd` | Red-green-refactor loop, one vertical slice at a time |
| `/domain-modeling` | Build/sharpen project domain model, stress-test with edge cases, update `CONTEXT.md` |
| `/codebase-design` | Design deep modules: behaviour behind small interface, clean seam, testable |
| `/code-review` | Two-axis: Standards (Fowler smells) + Spec (faithful implementation), parallel sub-agents |
| `/resolving-merge-conflicts` | Hunk-by-hunk merge, resolve by intent → finish operation, never `--abort` |

### Productivity — User-invoked
| Skill | What it does |
|---|---|
| `/grill-me` | Relentless interview about a plan/design until every branch is resolved |
| `/handoff` | Compact conversation into a handoff doc for another agent |
| `/teach` | Stateful multi-session teaching using current directory as workspace |
| `/writing-great-skills` | Reference for writing skills well (vocabulary, principles) |

### Productivity — Model-invoked
| Skill | What it does |
|---|---|
| `/grilling` | Reusable interview loop behind `/grill-me` and `/grill-with-docs` |

## write-a-skill Skill

Matt Pocock has a dedicated skill for authoring skills: `/write-a-skill`. Key rules:

- **SKILL.md structure:** YAML frontmatter (`name`, `description`) + Markdown body (Quick start, Workflows, Advanced features)
- **Description format:** max 1024 chars, third person, first sentence = what it does, second sentence = "Use when [specific triggers]"
- **File layout:** `skill-name/SKILL.md` (required) + `REFERENCE.md` / `EXAMPLES.md` / `scripts/` as needed
- **Split threshold:** SKILL.md > 100 lines → split into separate files
- **Scripts:** add when operation is deterministic, would be generated repeatedly, or needs explicit error handling
- **Review checklist:** description has triggers? SKILL.md under 100 lines? No time-sensitive info? Consistent terminology? Concrete examples?

## Why This Matters for deessejs

Matt Pocock's repo is the **gold standard reference** for the deessejs skills project. When authoring new deessejs skills:
- Follow his SKILL.md format (YAML frontmatter + Markdown)
- Use `write-a-skill` as a guide for structure
- His philosophy (small, composable, user/model-invoked split) is a proven pattern
- `/writing-great-skills` can be used to review drafts
