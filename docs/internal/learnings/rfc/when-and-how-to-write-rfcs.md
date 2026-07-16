---
title: When and How to Write RFCs
description: A guide to Request for Comments — when to write one, how to structure it, and how to integrate it into the development process.
category: engineering
tags: [rfc, architecture, decision-making, adr, design-review]
author: deessejs
created: 2026-07-16
updated: 2026-07-16
---

# When and How to Write RFCs

A guide to Request for Comments — when to write one, how to structure it, and how to integrate it into the development process.

## TL;DR

Write an RFC when a decision is hard to reverse, affects multiple teams, or commits you to a path for months. A spec plans an implementation. An RFC decides whether to take the path at all.

---

## Outline

- [What an RFC Is](#what-an-rfc-is)
- [RFC vs Spec vs ADR](#rfc-vs-spec-vs-adr)
- [When to Write an RFC](#when-to-write-an-rfc)
- [When NOT to Write an RFC](#when-not-to-write-an-rfc)
- [RFC Structure](#rfc-structure)
- [The RFC Process](#the-rfc-process)
- [Common RFC Mistakes](#common-rfc-mistakes)

---

## What an RFC Is

An RFC is a document that proposes a significant change and requests feedback from the team before a decision is made.

RFC stands for **Request for Comments** — the key word is *comments*. It's a collaborative design process, not a one-person decree.

An RFC answers:

- What problem are we solving?
- What are the options?
- What are the trade-offs?
- What did we decide and why?

---

## RFC vs Spec vs ADR

These three documents serve different purposes:

| | **RFC** | **Spec** | **ADR** |
|---|---|---|---|
| **What** | Propose a significant decision | Plan an implementation | Record a decision made |
| **When** | Before implementation | After triage, before code | After decision |
| **Audience** | The team | The implementer | Future maintainers |
| **Content** | Problem, options, trade-offs, decision | How to build it | What was decided and why |
| **Status** | Draft → Accepted/Rejected | Draft → Approved | Proposed → Accepted |

### The relationship

```
Problem identified
       ↓
Should we change? → RFC (decide the path)
       ↓
Path chosen
       ↓
How do we build it? → Spec (plan the implementation)
       ↓
Implementation
       ↓
What did we decide? → ADR (record for the future)
```

---

## When to Write an RFC

Write an RFC when the decision is:

| Criterion | Why |
|---|---|
| **Hard to reverse** | Changing a database, auth system, or API contract later costs months |
| **Affects multiple teams** | A decision that constrains other teams needs their input |
| **Commits long-term** | Choosing a framework, architecture, or pattern that will live for years |
| **Ambiguous** | The right answer isn't obvious and trade-offs need discussion |
| **Irreversible without pain** | Migrating away later would require significant effort |

### Examples that warrant an RFC

| Situation | Why an RFC is needed |
|---|---|
| "Should we switch to a new database?" | Data migration is painful; affects every team |
| "Should we adopt GraphQL?" | API design is a long-term contract |
| "Should we split this service?" | Distributed systems are complex to untangle |
| "Should we add a new external dependency?" | License, security, and maintenance implications |
| "Should we change our branching model?" | Affects every developer daily |

---

## When NOT to Write an RFC

An RFC is not needed when:

| Situation | Why not an RFC |
|---|---|
| **The answer is obvious** | If there's one clear path, write a one-pager or skip to spec |
| **It's a small feature** | A spec covers it |
| **It's reversible** | If changing later is cheap, just try it |
| **There's no time** | RFCs without reviews are worthless docs |
| **The team is too small** | Just talk it through; RFCs shine when there are many stakeholders |

### When in doubt

If you're debating whether to write an RFC, ask:

> "If we choose wrong, how painful is it to change course?"

If the answer is "very painful," write an RFC. If "not that painful," try the simpler path.

---

## RFC Structure

### Minimal RFC (1-2 pages)

```markdown
# RFC: {Title}

## Status
Draft | Review | Accepted | Rejected

## Summary
One paragraph. What are we proposing and why now?

## Problem
What problem does this solve? Why does it matter?

## Options
What are the alternatives? (Usually 2-3, not a laundry list)

## Decision
Which option do we recommend and why?

## Consequences
What becomes easier? What becomes harder?

## Next Steps
What happens if accepted? What happens if rejected?
```

### Extended RFC (for complex decisions)

```markdown
# RFC: {Title}

## Status
Draft → [Author] → Review → Accepted/Rejected

## Summary
One paragraph.

## Motivation
What problem are we solving? What happens if we do nothing?

## Detailed Design

### Option A
Description, pros, cons.

### Option B
Description, pros, cons.

### Option C (if applicable)
Description, pros, cons.

## Decision
Which option is recommended and why?

## Alternatives Considered
Why the other options were rejected.

## Implementation
How would this be rolled out?

## Migration
How do we migrate existing data/users/systems?

## Unresolved Questions
Open questions that need resolution before acceptance.

## Consequences

### Positive
What becomes easier or better.

### Negative
What becomes harder or more complex.

### Risks
What could go wrong?

## Timeline
When would this land? What are the milestones?

## Checkpoints
Who needs to sign off?
```

---

## The RFC Process

### Step 1 — Draft

Write the RFC. Be concrete. Options should be real alternatives, not strawmen.

### Step 2 — Share for Review

Post the RFC and invite feedback:

```markdown
RFC: {title} — {link}

> {one-line summary}

Please comment by {date}. I'll propose a decision after the review period.
```

### Step 3 — Collect Feedback

Address comments. Update the RFC. Be explicit about disagreements.

### Step 4 — Make a Decision

Decide. Don't leave RFCs in limbo. If there's no consensus after reasonable time, the decider (tech lead, architect, or team) makes the call.

### Step 5 — Record the Decision

Once accepted:

- Update the status to Accepted
- Create an ADR documenting the decision
- Archive the RFC

If rejected:

- Update the status to Rejected
- Note why
- Close the discussion

---

## Common RFC Mistakes

### Writing RFCs for everything

RFCs are expensive. They require time to write, review, and maintain. Writing one for a simple feature wastes everyone's time.

**Fix:** Use the "pain to reverse" test. Only RFC if changing course is painful.

### Writing RFCs alone

A solo RFC misses the point. The collaboration is the value.

**Fix:** Discuss the problem verbally first. Write the RFC after the discussion clarifies the trade-offs.

### Leaving RFCs in Draft forever

A draft RFC is a draft, not a decision. If no one reviews it, it's a doc, not an RFC.

**Fix:** Set a deadline. If no review happens by the deadline, close or advance the RFC.

### Options without real trade-offs

A list of three options where one is obviously wrong isn't an RFC — it's a justification.

**Fix:** Make every option plausible. The goal is to pick the best of real alternatives.

### Not recording why

"I chose Option B" isn't enough. Future-you will wonder why Option A was rejected.

**Fix:** Write the decision rationale explicitly. Include what was traded off.

### Skipping the ADR

The RFC is a conversation. The ADR is the record. Without the ADR, the context fades.

**Fix:** Create an ADR when the RFC is accepted. Link them.

---

## Summary

| When to RFC | When to Spec | When to ADR |
|---|---|---|
| Hard to reverse decisions | Implementation planning | Decision recording |
| Affects multiple teams | One implementer | Any significant decision |
| Long-term commitments | Any feature | Architecture choices |

Write an RFC when changing course is painful. Write a spec when the path is clear. Write an ADR when the decision is made.
