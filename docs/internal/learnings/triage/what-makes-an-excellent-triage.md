# What Makes an Excellent Issue Triage

A guide to understanding what separates a thorough, effective triage from a superficial one.

---

## The Definition of "Triaged"

An issue is **triaged** when all of the following are true:

1. **The issue is understood** — the problem or request is clear, not ambiguous
2. **The issue is verified** — the problem exists in the current state of the system
3. **The issue is categorized** — type, area, and any relevant cross-cutting labels are applied
4. **The issue is prioritized** — severity and effort are assessed and documented
5. **The issue is actionable** — the next step is clear and assigned

If any of these is missing, the issue is not triaged. It is only categorized at best.

---

## The Six Dimensions of an Excellent Triage

### 1. Verification — Does the issue describe reality?

Before anything else, verify that the issue matches the current state of the codebase or product.

- **Bug reports** — attempt to reproduce. If you cannot reproduce, the issue may be outdated, environment-specific, or incorrect.
- **Feature requests** — check if the feature already exists, partially or fully.
- **Refactor requests** — locate the code in question. Confirm the problem pattern exists.
- **Tasks** — verify the work is not already done or in progress.

Verification prevents wasted effort on issues that describe problems that don't exist or request features that already exist.

> A triage that only reads the issue text without checking the codebase is a partial triage.

### 2. Deduplication — Has this been reported before?

Search for similar open issues before treating the issue as new.

- Search by title keywords
- Search by the specific behavior or error message
- Check for recently closed issues that might have been reopened
- Check for issues in related repositories

If a duplicate is found, link to the original and close the new one. Do not let duplicate issues proliferate — they fragment discussion and dilute priority signals.

### 3. Completeness — Does the issue have what it needs to be acted on?

An incomplete issue cannot be triaged properly. Assess against the following criteria:

**For bug reports:**
- [ ] Clear description of what is broken
- [ ] Steps to reproduce (or a clear statement that they are unknown)
- [ ] Expected behavior vs actual behavior
- [ ] Environment or context (version, browser, OS)

**For feature requests:**
- [ ] Clear description of the desired outcome
- [ ] The problem it solves (not just the solution)
- [ ] Acceptance criteria or definition of done
- [ ] Context on why it matters

If incomplete, do not close the issue. Ask for the missing information and mark it as awaiting input.

### 4. Prioritization — How urgent and how hard?

Prioritization has two axes:

**Severity / Impact** — how badly does this affect users or the system?

- **Critical** — data loss, security breach, complete feature broken for all users
- **High** — major feature broken, workaround exists but is painful, blocks other work
- **Medium** — feature partially broken, non-critical bug, noticeable but not blocking
- **Low** — cosmetic issue, minor inconvenience, nice-to-have

**Effort / Complexity** — how much work to fix or implement?

- **High** — multi-week effort, requires architectural changes, touching many parts
- **Medium** — days to a week, bounded scope
- **Low** — hours to a day, isolated change

The combination of severity and effort determines priority. A high-severity, low-effort issue jumps to the top. A low-severity, high-effort issue gets deferred.

### 5. Categorization — Where does this belong?

Every issue should be labeled with enough context to be findable and assignable:

**Type label** — what kind of work is this?
- Bug, Feature, Chore, Docs, Refactor, Security

**Area label** — what part of the system is affected?
- auth, ui, api, database, email, ci, deploy, docs, etc.

**Cross-cutting labels** — does this affect multiple concerns?
- breaking-change, regression, dependencies, performance

The more precise the labels, the easier it is to find relevant issues and route them correctly.

### 6. Actionability — What happens next?

An issue is not triaged if nobody knows what to do with it next. After triaging:

- **Ready** — the issue is verified, complete, and ready to be picked up
- **Blocked** — the issue depends on something else; link to the blocking issue
- **Needs information** — the issue is incomplete; specify what is missing
- **Won't fix / Duplicate / Invalid** — close with explanation

Avoid leaving issues in a state where the next step is unclear.

---

## Regression — A Special Case

A regression is a bug introduced by a recent change. It requires special handling:

1. **Identify** — check git history for recent changes to the affected area
2. **Reproduce** — confirm the behavior worked before the change
3. **Tag** — label as regression; it signals higher urgency than a standard bug
4. **Escalate** — regressions in active development typically warrant direct notification to the team responsible
5. **Milestone** — assign to the release where the regression was introduced

Regressions should be treated with higher severity than equivalent non-regression bugs. The closer to a release, the more critical the response.

---

## Escalation — When to Notify Someone Directly

Not every issue should languish in the backlog. Some issues warrant immediate attention:

- **Security vulnerabilities** — notify the security team immediately, do not file a public issue
- **Data loss or corruption** — notify the engineering lead and data team immediately
- **Critical feature broken for all users** — notify the product manager and engineering lead
- **Regression in active development** — notify the contributor responsible for the change

Escalation is not a substitute for proper triage. Escalate after triaging, not instead of triaging.

---

## The Cost of Poor Triage

- **Incomplete triage** → engineers waste time asking clarifying questions, context-switching
- **Missing duplicates** → multiple people working on the same thing
- **No verification** → engineers implement features that already exist or fix bugs that don't
- **Unclear priority** → urgent work buried under less important issues
- **No escalation** → critical issues sitting idle while users are impacted
- **Vague categorization** → issues that can't be found or routed correctly

Triage is the filter between the stream of incoming issues and the focused work of the team. When the filter fails, the entire team suffers.

---

## Summary — The Triage Checklist

| Dimension | Question | If No |
|---|---|---|
| **Verification** | Does the issue match reality? | Flag as needs-info |
| **Deduplication** | Has this been reported? | Link and close as duplicate |
| **Completeness** | Does it have enough to act on? | Request missing information |
| **Prioritization** | How urgent and how hard? | Document severity + effort |
| **Categorization** | Is it labeled correctly? | Apply type + area + cross-cutting |
| **Actionability** | Is the next step clear? | Set status appropriately |
| **Regression?** | Is this a regression? | Tag, escalate, milestone |
| **Escalation?** | Does this need immediate attention? | Notify stakeholders directly |

An issue is triaged when all applicable questions have been answered.
