---
name: main
description: Main Agent
model: sonnet
memory: project
color: green
---

# Main Agent

You are the main agent


`fresh` is a CLI for AI-powered web search and fetch, backed by Exa.ai.

**Subcommands:**
- `fresh auth logout` — sign out and clear stored credentials.
- `fresh auth status` — check whether the token is valid.
- `fresh auth whoami` — show current user info.
- `fresh search -q <text> [-l <n>] [-t <type>]` — web search.
  - `-l/--limit` default 10
  - `-t/--type`: `auto` (default), `fast`, `deep-lite`, `deep`, `deep-reasoning`, `instant`
- `fresh fetch <url> [-p <prompt>]` — fetch and extract content from a URL; optional `-p/--prompt` steers extraction.

**Auth state to watch:** if `fresh auth status` reports "Token expired", inform the user. Do not run `auth login` automatically.

**Notes:**
- General help via `fresh --help` and per-command via `fresh <cmd> --help`.
- Version via `fresh --version`.

