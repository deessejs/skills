---
name: project-skills-ecosystem
description: skills.sh is the open registry for Claude Code skills — the deessejs org repo mirrors skills from github.com/deessejs
metadata:
  type: project
---

## skills.sh — The Agent Skills Directory

**URL:** https://www.skills.sh

**What it is:** The open registry / directory for Agent Skills — reusable capability packages for AI agents (Claude Code, Cursor, Copilot, etc.). Think "npm for Claude Code skills."

**How it works:**
- Each skill = a `SKILL.md` file (YAML frontmatter + Markdown body)
- Install via: `npx skills add <skill-name>` → drops the SKILL.md into your project
- Claude Code loads it automatically on relevant tasks, or invoke via `/skill-name`
- Follows the **Open Agent Skills spec** — cross-agent compatible

**Key sources on the registry:**
| Source | Notable skills |
|---|---|
| anthropics/skills | pptx, pdf, docx, mcp-builder, canvas-design, webapp-testing |
| mattpocock/skills | grill-me, tdd, code-review, writing-great-skills, diagnosing-bugs |
| vercel-labs/agent-skills | vercel-react-best-practices, vercel-composition-patterns, deploy-to-vercel |
| microsoft/azure-skills | azure-kubernetes, azure-cost-optimization, azure-observability |
| heygen-com/hyperframes | ai-video-generation, website-to-video, hyperframes-animation |
| supabase/agent-skills | supabase-postgres-best-practices, supabase |

**Leaderboard (top 5 by installs):** find-skills (2.5M), frontend-design (669K), grill-me (567K), vercel-react-best-practices (555K), agent-browser (549K).

**Why this matters for ds-skills:** This repo mirrors skills from `github.com/deessejs`, not from skills.sh directly. skills.sh is a *reference* — it shows the broader ecosystem and may inform which skills deessejs publishes.

**How to apply:** When the user asks about skills, skills.sh is the canonical registry to search first. Use `fresh search` or `fresh fetch` on `https://www.skills.sh` to explore.
