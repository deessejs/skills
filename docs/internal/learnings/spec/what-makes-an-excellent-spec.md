---
title: What Makes an Excellent Implementation Spec
description: A guide to writing implementation specs that any developer can pick up and execute without context.
category: engineering
tags: [spec, implementation, planning, development]
author: deessejs
created: 2026-07-16
updated: 2026-07-16
---

# What Makes an Excellent Implementation Spec

A guide to writing specs that transform a "what needs to be done" into a "how it will be done" — in a way that any developer, without prior context, can understand and execute.

## TL;DR

An implementation spec is a development plan written for a developer who knows nothing about the project. It answers **what, why, how, and when it's done** — not just a list of steps. If a developer needs to ask clarifying questions after reading the spec, the spec has failed.

---

## Outline

- [What a Spec Is and Isn't](#what-a-spec-is-and-isnt)
- [The Three Phases of a Good Spec](#the-three-phases-of-a-good-spec)
- [What Belongs in a Spec](#what-belongs-in-a-spec)
- [The Definition of Done](#the-definition-of-done)
- [Spec vs Issue vs Code Review](#spec-vs-issue-vs-code-review)
- [Common Spec Failures](#common-spec-failures)

---

## What a Spec Is and Isn't

### A spec IS

- A **development plan** — concrete, ordered steps
- Written for **any developer** on the team
- A record of **decisions made** and why
- A contract between the triager and the implementer
- A place to **surface risks and unknowns** before implementation starts

### A spec IS NOT

- A **requirements document** — that's the issue's job
- A **design document** — for deep design work, use a separate design doc
- A **tutorial** — it assumes the developer knows the basics
- An **implementation log** — write notes elsewhere, not in the spec
- A **copy of the issue** — it adds interpretation and direction

---

## The Three Phases of a Good Spec

### Phase 1 — Research (Understand before planning)

Before writing a single step, understand the problem deeply.

**Read the issue thoroughly:**
- What is the actual problem being solved?
- Who benefits and how?
- What are the acceptance criteria?
- Are the acceptance criteria complete?

**Search for context:**
- Read existing code in the affected area
- Check `docs/internal/learnings/` for related past decisions
- Check for ADRs (Architecture Decision Records)
- Look at similar implementations in the codebase

**Web research when needed:**
- Search for how similar problems are solved in other projects
- Check documentation of libraries or frameworks involved
- Look for known pitfalls or breaking changes
- Find the best practices for the technology in question

**Challenge the issue:**
- Is the proposed solution the right approach?
- Are there simpler alternatives?
- What are the trade-offs?
- Is this scope creep or a genuine requirement?

> If the issue is unclear or the solution is wrong, go back to triage. A spec on a flawed plan is wasted effort.

### Phase 2 — Plan (Turn understanding into steps)

Now that the problem is understood, turn it into an ordered plan.

**Order steps logically:**
- What needs to be in place before what?
- What is the smallest slice that is still meaningful?
- What can be done in parallel?

**Define the scope:**
- What is IN this implementation?
- What is explicitly OUT?
- Where does this stop?

**Identify risks and mitigations:**
- What could go wrong?
- What is uncertain?
- What needs testing?

### Phase 3 — Write (Document for a stranger)

Write the spec as if you will be hit by a bus tomorrow and someone else needs to finish the job.

---

## What Belongs in a Spec

### TL;DR

One sentence. If someone reads nothing else, they should understand the end state after this PR merges.

### Context

Why does this need to be done? What problem does it solve? What is the current state and why is that a problem?

2-3 sentences maximum. The issue has more detail — this is the executive summary.

### Scope

**IN:**
- What this spec covers

**OUT:**
- What this spec explicitly does NOT cover
- What is deferred to a future issue

Being explicit about OUT prevents scope creep during implementation.

### Files to Touch

Every file that will be created, modified, or deleted. Not directories — specific files.

| File | Action | Why |
|------|--------|-----|
| `apps/web/src/auth/login.ts` | edit | Add rate limiting middleware |
| `packages/validators/src/index.ts` | edit | Export new validator |
| `apps/web/src/emails/welcome.ts` | create | New email template for sign-up flow |

### Step-by-Step Implementation

Each step should be:

- **Specific** — not "update auth", but "add rate limiting middleware to the login endpoint"
- **Ordered** — explain why this step comes before the next one
- **Complete** — a developer should not need to guess what to do

For each step:

1. **What** — what file or component is being touched
2. **Why** — why this approach, not another
3. **How** — the key implementation detail, not the full tutorial

### Acceptance Criteria

How do we know this is done? These should be:

- **Testable** — can be verified automatically or manually
- **Complete** — cover the happy path and important edge cases
- **From the issue** — lifted from the issue's acceptance criteria when present

### Definition of Done

When is this considered finished?

- Code is written
- Tests pass
- Documentation is updated (if needed)
- No regressions introduced
- The acceptance criteria are met

### Risks and Mitigations

What could go wrong, and what is the plan if it does?

| Risk | Likelihood | Impact | Mitigation |
|------|-------------|--------|------------|
| Breaking change to existing API | Low | High | Add migration guide |
| Performance regression on large datasets | Medium | Medium | Add load test |
| Edge case not covered by tests | High | Low | Add integration tests |

### Open Questions

What is unknown at spec time and needs to be answered during implementation?

- "Should the new rate limit be per-IP or per-user?"
- "What should the error message say when the limit is exceeded?"

These are not blockers — they are captured so the implementer makes a conscious decision rather than guessing.

### Alternatives Considered

What other approaches were considered and why were they rejected?

This is not always necessary, but when the chosen approach is non-obvious, documenting alternatives prevents future developers from re-asking "why didn't we do it this way?"

---

## The Definition of Done

An implementation is **done** when:

1. All acceptance criteria are met
2. The code passes build, typecheck, tests, and lint
3. The relevant documentation is updated
4. No regressions are introduced
5. The spec's scope boundaries were respected

If any of these is not true, the implementation is incomplete.

---

## Spec vs Issue vs Code Review

| | **Issue** | **Spec** | **Code Review** |
|---|---|---|---|
| **What** | What needs to be done and why | How it will be done | Was it done right? |
| **When** | Before triage | After triage, before implementation | After implementation |
| **Author** | Anyone | Developer or agent | Reviewer |
| **Audience** | Team + triager | Implementer | Team |
| **Content** | Problem, context, acceptance criteria | Plan, steps, risks, decisions | Feedback on the code |

---

## Common Spec Failures

### The "Read the Issue" spec

Lists the issue's acceptance criteria as steps without adding implementation detail. A developer still doesn't know how to start.

### The Over-Engineered Spec

Includes design discussions, trade-off essays, and alternative approaches that belong in ADRs, not in a spec meant to be executed.

### The Unordered List

Steps are listed but not ordered. The developer has to figure out the sequence.

### Missing the "Why"

Steps explain what to do but not why. When the developer hits an edge case, they don't know how to adapt.

### No Scope Boundaries

Everything looks like it could be in scope. Implementation drifts without clear boundaries.

### Ignoring the Existing Codebase

Spec assumes a greenfield approach when there's existing code to build on or work around. The implementer discovers mid-implementation that they need to refactor first.

### Skipping Web Research

For novel problems or unfamiliar technologies, the spec plans based on assumptions. The implementer discovers mid-implementation that there's a library for this, or that the proposed approach has known issues.

---

## Summary

| Phase | Key Question | Output |
|---|---|---|
| **Research** | Do we understand the problem and is our approach right? | Confirmed approach, open questions |
| **Plan** | What are the exact steps and in what order? | Ordered steps, scope boundaries, risks |
| **Write** | Can a stranger execute this without asking questions? | Complete spec |

A spec is done when a developer can read it, open their editor, and implement without needing to ask a single clarifying question.
