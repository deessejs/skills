---
title: What Makes an Excellent ADR
description: A guide to Architecture Decision Records — when to write them, how to structure them, and how to use them in the development process.
category: engineering
tags: [adr, architecture, decision-record, documentation, trade-offs]
author: deessejs
created: 2026-07-16
updated: 2026-07-16
---

# What Makes an Excellent ADR

A guide to Architecture Decision Records — when to write them, how to structure them, and how they fit into the development process.

## TL;DR

An ADR records **why** a decision was made, not just **what** was decided. If future-you can read the ADR and understand the trade-offs that led to the choice, it's good. If it only records the outcome, it's documentation, not an ADR.

---

## Outline

- [What an ADR Is](#what-an-adr-is)
- [ADR vs RFC vs Spec](#adr-vs-rfc-vs-spec)
- [When to Write an ADR](#when-to-write-an-adr)
- [The Anatomy of an ADR](#the-anatomy-of-an-adr)
- [ADR Formats](#adr-formats)
- [The ADR Lifecycle](#the-adr-lifecycle)
- [Common ADR Mistakes](#common-adr-mistakes)
- [Tools and Workflows](#tools-and-workflows)

---

## What an ADR Is

An Architecture Decision Record (ADR) is a document that captures **one** significant architectural decision, including:

- The **context** — why this decision had to be made
- The **options considered** — what alternatives existed
- The **decision** — what was chosen
- The **consequences** — what this makes easier and harder
- The **status** — when it changes or gets superseded

ADRs are immutable logs. You don't edit an existing ADR. You create a new ADR that supersedes it.

---

## ADR vs RFC vs Spec

These three documents serve different purposes:

| | **ADR** | **RFC** | **Spec** |
|---|---|---|
| **Purpose** | Record a decision made | Propose a decision to be made | Plan an implementation |
| **When** | After a decision | Before committing to a path | After triage, before code |
| **Audience** | Future maintainers | Team members | The implementer |
| **Content** | Context, options, decision, consequences | Problem, options, trade-offs, recommendation | How to build it |
| **Output** | Immutable record | Draft → Accepted/Rejected | Draft → Approved |

### The relationship

```
Problem identified
       ↓
Is this a significant decision?
       ↓
Should we change this path? → RFC (decide the path)
       ↓
Path chosen → ADR (record the decision)
       ↓
How do we build it? → Spec (plan the implementation)
       ↓
Implementation
```

An RFC decides *which path*. An ADR records *that we chose this path and why*. A Spec plans *how to implement on that path*.

---

## When to Write an ADR

Write an ADR when a decision is **architecturally significant** — meaning it affects the shape of the system for months or years.

### Indicators that a decision is architecturally significant

- It would be painful to reverse
- Multiple teams or components are affected
- It constrains future decisions
- It's hard to understand without context
- You found yourself debating it in a meeting for more than 30 minutes
- Someone will ask "why did we do it this way?" in the future

### Examples that warrant an ADR

| Decision | Why an ADR |
|---|---|
| "We chose PostgreSQL over MongoDB" | Years of data migration pain |
| "We adopted GraphQL for the API" | API contract that consumers depend on |
| "We split into microservices" | Hard to undo, affects architecture |
| "We use feature flags for all features" | Patterns that persist for years |
| "We store auth tokens in Redis" | Security implications, hard to change |
| "We deprecated our REST API" | Breaking change for all consumers |

### Examples that don't need an ADR

| Decision | Why not an ADR |
|---|---|
| "We named the function `getUserById" | Reversible, low impact |
| "We put the button on the left" | Can be reverted trivially |
| "We used a library version 2.3.1 instead of 2.2.0" | Already documented in lock file |

---

## The Anatomy of an ADR

### Minimal ADR (Nygard format)

```markdown
# ADR-{number}: {Title}

## Status
Accepted | Deprecated | Superseded by ADR-{N}

## Context
{What is the issue? What constraints exist? What forces are in tension?}

## Decision
{What is the decision? Be explicit about what was chosen and what was rejected.}

## Consequences

### Positive
{What becomes easier?}

### Negative
{What becomes harder? What did we give up?}

### Neutral
{What changes without being clearly positive or negative?}
```

### Extended ADR (MADR format)

```markdown
# ADR-{number}: {Title}

## Status
Draft | Proposed | Accepted | Deprecated | Superseded by ADR-{N}

## Context
{What is the issue? Why does it matter now? What forces are in tension?
Constraints, stakeholders, team preferences, past decisions we're building on.}

## Alternatives Considered

### Option A: {Name}
Description, pros, cons.

### Option B: {Name}
Description, pros, cons.

### Option C: {Name}
Description, pros, cons.

## Decision
{What was decided. Be explicit: "We chose Option A because..."

## Consequences

### Positive
{What becomes easier, faster, safer.}

### Negative
{What becomes harder. What we gave up. Risks we accepted.}

### Risks
{What could go wrong? What do we accept?}

## Related Decisions
- ADR-{N}: {link}
- ADR-{M}: {link}

## Notes
{Optional: questions that came up during decision, open questions, future considerations.}
```

---

## ADR Formats

### 1. Nygard format (simple)

- Author: Michael Nygard
- Best for: Small teams, fast decisions, low ceremony
- Fields: Status, Context, Decision, Consequences
- Source: [Documenting Architecture Decisions](https://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions)

### 2. MADR format (structured)

- Author: MADR project
- Best for: Teams that want structured alternatives, trade-off analysis
- Fields: Extended context, alternatives, pros/cons, consequences, related decisions
- Source: [MADR](https://adr.github.io/madr/)

### 3. Y-statement format (concise)

- Author: Oliver Zimmermann
- Best for: Fast decisions, one-pagers, linking to longer docs
- Format: Because [forces], we decided [decision] to achieve [quality attribute]. Acceptable because [consequences].
- Source: [Sustainable Architectural Decisions](https://www.hsr.ch/fileadmin/Redaktion/Ueber_uns/Publikationen_Files/Zimmermann_Fuchshuber_Ragghianti_Schlagworte.pdf)

### 4. Business case format (stakeholder-focused)

- Best for: Decisions that need buy-in from non-technical stakeholders
- Fields: Problem, options, costs, benefits, risks, decision, timeline
- Source: [Tyree & Akerman (Capital One)](https://github.com/joelparkerhenderson/architecture-decision-record)

---

## The ADR Lifecycle

### Creating an ADR

```
1. Draft → Write the ADR with context, options, decision, consequences
2. Proposed → Share for review
3. Accepted → Merged to the decision log
4. Deprecated → Superseded by a new ADR
5. Superseded → Links to the new ADR that replaces it
```

### Managing the decision log

ADRs live in a decision log — a directory in the repo.

```
docs/
├── adr/
│   ├── 0001-use-postgresql.md
│   ├── 0002-adopt-graphql.md
│   └── 0003-deprecate-rest-api.md
└── rfcs/
    └── ...
```

Use sequential numbering (0001, 0002, ...) for ordering. Old ADRs stay — they are immutable logs.

### Updating an ADR

ADRs are immutable. To change a decision:

1. Create a new ADR
2. Mark the old ADR as "Superseded by ADR-{N}"
3. Link from old to new

Never edit the old ADR's content. Future readers need to understand what was known at decision time.

### ADR review

Not every ADR needs a review process. Small decisions can be self-documented. Significant decisions should be reviewed by stakeholders.

Ask: "Would changing this decision later cost weeks of migration work?"

---

## Common ADR Mistakes

### Recording outcomes, not rationale

**Bad:** "We chose PostgreSQL."
**Good:** "We chose PostgreSQL over MongoDB because our team has more SQL expertise, our data is relational by nature, and we value ACID compliance over flexible schemas. We accepted slower writes and less flexible indexing as trade-offs."

**Why it matters:** Outcome-only ADRs don't help future maintainers understand *why*. When the trade-offs change, they won't know to revisit the decision.

### Not recording alternatives considered

**Bad:** "We adopted GraphQL." (with no alternatives)
**Good:** "We considered REST, gRPC, and tRPC. GraphQL was chosen because..."

**Why it matters:** Future developers will re-litigate settled debates unless they understand why other options were rejected.

### Missing consequences

**Bad:** "We chose Kubernetes." (no consequences)
**Good:** "We chose Kubernetes. This makes deployment more complex (Negative). We chose it because we need auto-scaling and cloud portability (Positive). The team accepted a steeper learning curve."

**Why it matters:** Consequences frame the decision. Without them, you can't evaluate if the trade-offs are still worth it.

### ADRs that are too long

An ADR is not a design document. It's a decision record. If it's more than 2-3 pages, consider linking to a design doc and keeping the ADR concise.

### Not linking related decisions

**Bad:** "We use Redis for sessions" (no links)
**Good:** "We use Redis for sessions. See ADR-003 for the caching strategy."

**Why it matters:** Architecture is a graph of decisions. Linking them helps future readers trace dependencies.

### Forgetting to mark deprecated decisions

**Bad:** Old ADRs sit without a status. New developers assume they're current.
**Good:** All ADRs have a Status field. Deprecated ADRs link to the superseding ADR.

### Writing ADRs for everything

**Why it's a problem:** ADRs for naming conventions or library choices create noise. The decision log should signal "this was significant."

**Fix:** "If we'd have to migrate this in a week-long project, ADR it. If it's a 10-minute decision, skip it."

---

## Tools and Workflows

### Tools

| Tool | Purpose |
|---|---|
| [adr-tools](https://github.com/adr/adr-tools) | CLI for creating, accepting, superseding ADRs |
| [Decision Guardian](https://github.com/DecisharHQ/decision-guardian) | Surfaces relevant ADRs on PRs automatically |
| [adr.github.io](https://adr.github.io) | Collection of ADR formats and examples |
| [MADR](https://adr.github.io/madr/) | Structured template with alternatives considered |

### Integrating with CI/CD

Automate ADR discovery on PRs:

```bash
# Find relevant ADRs before a PR merges
grep -r "context\|consequences\|alternatives" docs/adr/
```

Some teams use fitness functions to verify architectural decisions are enforced in code (e.g., no direct DB access from the API layer enforces the ADR on service boundaries).

---

## Summary

| When to ADR | What to record | What to avoid |
|---|---|---|
| Significant, hard-to-reverse decisions | Context, options, decision, consequences | Library versions, naming, trivial choices |
| Multi-team decisions | Why each option was rejected | Long design docs (link instead) |
| Architecture-affecting choices | Status, alternatives, trade-offs | Outcome-only records |

An ADR is a gift to future-you. Write it as if you'll need to understand the decision in two years with no context beyond what's in the doc.
