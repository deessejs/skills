---
name: research
description: Investigate a technical topic against primary sources — docs, forums, GitHub issues, best practices from other projects. Use when asked to "research", "investigate", "look into", or "find best practices for".
---

# `research` Skill

Investigate a technical topic by searching primary sources: official documentation, GitHub issues, Stack Overflow, forum discussions, and best practices from similar projects. Synthesize findings into a cited report.

## When to use

**Trigger phrases:** "research", "investigate", "look into", "find best practices for", "check how others do this", "what does the docs say about", "/research".

Use this skill when:
- Writing a spec for an unfamiliar technology or library
- Implementing something with non-obvious approaches
- Diagnosing a bug with no clear root cause
- Evaluating third-party libraries or tools
- The codebase doesn't have enough context to answer the question

## Prerequisites

Before searching, verify `fresh` is authenticated:

```bash
fresh auth status
```

**If "Token expired" or not logged in:** Stop. Inform the user that authentication is needed. Do not run `auth login` automatically.

## Workflow

```
1. Scope      — define what to research and what questions to answer
2. Sweep      — broad search to understand the landscape
3. Deep-dive  — fetch specific sources in parallel
4. Synthesize  — organize findings by theme, not by source
5. Report     — write findings with citations
```

---

## §1 — Scope

Before searching, define the research scope.

### What to capture

1. **The question** — what are you trying to learn?
2. **Key sub-questions** — specific angles to explore
3. **Context** — what have you already tried or looked at?
4. **Known unknowns** — what would change your decision?

### Output

```
## Research Scope

**Question:** {the main question}
**Context:** {what I know already}
**Sub-questions:**
- {specific angle 1}
- {specific angle 2}

**Would change my answer:** {condition or data point}
```

---

## §2 — Sweep (broad search)

Use `fresh search` to get an overview of the topic.

```bash
# Primary search — what the official docs say
fresh search -q "{technology} documentation {specific topic}" -l 5

# Community search — Stack Overflow, forums, GitHub discussions
fresh search -q "{technology} best practices {specific use case}" -l 5

# Issue search — problems others have hit
fresh search -q "{technology} github issue {problem}" -l 5
```

### What to look for

- Official documentation (docs.{technology}.io, {technology}js.com)
- Stack Overflow questions with accepted answers
- GitHub issues with official responses
- Blog posts from library authors
- RFCs or design discussions

### Collect URLs to deep-dive

Store URLs from results for §3. Focus on:
- **Primary sources** — official docs, library source, official blog posts
- **Canonical answers** — accepted Stack Overflow, highest-upvoted responses
- **Authoritative voices** — library maintainers, recognized experts

---

## §3 — Deep-dive

Fetch the most relevant sources in parallel.

```bash
# Official documentation
fresh fetch https://docs.{technology}.io/{path} -p "Extract: overview, usage examples, gotchas, API reference"

# Stack Overflow answers
fresh fetch https://stackoverflow.com/questions/{id} -p "Extract: the question, the accepted answer, any higher-voted alternatives"

# GitHub issues or discussions
fresh fetch https://github.com/{org}/{repo}/issues/{n} -p "Extract: problem description, official response, workaround if any"
```

### What to extract from each source

For each URL, extract:
- **Key facts** — what is definitively true
- **Patterns** — what works, what doesn't
- **Gotchas** — common pitfalls or edge cases
- **Alternatives considered** — what else was tried
- **Recommendations** — what the source suggests

### Record what didn't help too

Note sources that were irrelevant or outdated. This helps future readers avoid the same dead ends.

---

## §4 — Synthesize

Organize findings by **theme**, not by source. The goal is to answer the question, not to list what you read.

### Group by topic

```
## Findings

### Official Recommendation
{what the library author/docs recommend}

### Common Patterns
{what most projects seem to do}

### Edge Cases & Gotchas
{things that break or require special handling}

### Alternatives Considered
{other approaches and why they were rejected}
```

### Assess source quality

| Source | Weight |
|--------|--------|
| Official docs / library author | High |
| Accepted Stack Overflow answer | Medium-High |
| GitHub issue with official response | Medium-High |
| Community blog post | Medium |
| Forum / discussion without consensus | Low |

Weight findings accordingly. A single official doc outweighs ten forum opinions.

### Flag contradictions

If sources disagree, note the disagreement and which source is most authoritative.

---

## §5 — Report

Write the final report.

### Structure

```markdown
# Research: {topic}

**Date:** {YYYY-MM-DD}
**Question:** {the original question}
**Confidence:** High / Medium / Low

---

## TL;DR

{One sentence answer to the original question}

---

## Findings

{findings organized by theme from §4}

---

## Recommendations

{based on findings, what should we do?}

---

## Open Questions

{what couldn't be answered and needs a decision}

---

## Sources

- [{title}]({url}) — {brief note on relevance}
- ...
```

### Where to save

- For spec work: save as `docs/plans/{n}-research.md`
- For implement work: save alongside the implementation
- For general research: save as `docs/research/{topic}-{date}.md`

---

## Integration with Other Skills

### With `/spec`

Run research **before** writing the spec:

```
/spec #{n}  →  /research  →  write spec with citations
```

Add a "Research" section to the spec referencing the research doc.

### With `/implement`

If an implementation decision is uncertain:

```
/implement #{n}  →  /research {question}  →  resume implement with findings
```

### With `/diagnosing-bugs` (future)

Research is the investigation phase for bug diagnosis:
- Search for similar bugs reported
- Find official workarounds
- Check if it's a known issue with a fix

---

## Output Templates

Templates live in `templates/`. Copy and fill:

| Template | Use case | Output |
|----------|----------|--------|
| `templates/full-report.md` | Specs, complex decisions | `docs/plans/{n}-research.md` |
| `templates/quick-reference.md` | Fast implementation notes | `docs/plans/{n}-research-notes.md` |
| `templates/bug-research.md` | Bug diagnosis | `docs/bugs/{n}-research.md` |
| `templates/learning.md` | Reusable knowledge extraction | `docs/internal/learnings/{topic}/` |

### Usage

```bash
# Copy template to working location
cp templates/full-report.md docs/plans/{n}-research.md
```

Each template includes: question, confidence, TL;DR, findings, sources, recommendations, open questions.

---

## Learnings

Research findings that are **reusable knowledge** should be extracted into `docs/internal/learnings/`.

### When to write a learning

Write a learning when research reveals:
- A pattern that will apply to future work
- A gotcha or pitfall that caused problems
- A decision with trade-offs worth documenting
- A technique or approach worth remembering

### Learning structure

```markdown
---
title: {Descriptive title}
description: {One-line summary}
category: {engineering | product | ops}
tags: [{relevant-tags}]
author: {author}
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
---

# {Title}

## TL;DR

{One sentence summary of the learning}

---

## Context

{When does this apply? What problem does it solve?}

---

## What we learned

{Detailed finding}

---

## Application

{How to apply this learning in practice}

---

## Sources

- [{source}]({url})
```

### Where to save

```
docs/internal/learnings/
├── adr/                    # Architecture decisions
├── release/                # Release process insights
├── rfc/                    # RFC process insights
├── spec/                   # Spec writing insights
├── staging-pattern/        # Branching strategy insights
├── triage/                 # Issue triage insights
└── {topic}/               # Topic-specific learnings
    └── what-we-learned-about-{topic}.md
```

### Integration with research workflow

After completing research (§5), ask:

> "Does this research reveal anything worth adding to `docs/internal/learnings/`?"

If yes:
1. Extract the reusable insight
2. Write the learning doc
3. Add to the appropriate `learnings/` subfolder
4. Reference it in the research report's "Sources" section

---

## Error Handling

| Situation | Action |
|-----------|--------|
| `fresh auth status` expired | Inform the user that authentication is needed |
| No relevant results found | Broaden search terms; try alternative sources |
| Sources contradict each other | Note contradictions; prefer official sources |
| Documentation is outdated | Note in report; check GitHub issues for current state |
| Search returns spam | Ignore; focus on authoritative sources |

---

## Constraints

- **Cite all sources** — future readers need to verify and follow up
- **Weight by authority** — official docs > community opinions
- **Organize by theme** — not by source list
- **Flag uncertainty** — if confidence is low, say so
- **Save the output** — research is wasted if not documented
- **Scope creep** — research has a clear question; don't rabbit-hole
