---
name: diagnosing-bugs
description: Diagnose a GitHub issue to find the root cause, document findings, and prepare the issue for triage. Use when asked to "diagnose #N", "investigate bug #N", or "find the root cause of #N".
---

# `diagnosing-bugs` Skill

Diagnose a bug: reproduce it, minimize the reproduction case, find the root cause, and document findings in the issue. Does NOT fix the bug — that comes after triage.

## When to use

**Trigger phrases:** "diagnose #N", "investigate bug #N", "find root cause of #N", "/diagnose #N".

Run on a GitHub issue that describes a bug. The issue should already exist — this skill diagnoses it, it does not create it.

**Prerequisites:**
- Issue exists in GitHub
- Issue has `type: bug` label or describes a bug
- `fresh` is authenticated (for `/research` if needed)

## Workflow

```
1. Fetch      — read the issue, get reproduction steps
2. Reproduce  — attempt to reproduce the bug
3. Minimize   — isolate the exact component/condition
4. Research   — if needed: find similar bugs, docs, workarounds
5. Hypothesize + Instrument — enumerate hypotheses, add logging
6. Root cause — identify the deep cause
7. Document   — update issue with findings
8. Learning   — extract reusable patterns
```

---

## §1 — Fetch

```bash
gh issue view {n} --json title,body,labels,state,comments
```

Extract:
- **Title** — what is broken
- **Body** — reproduction steps, expected vs actual behavior
- **Labels** — existing labels
- **Comments** — any additional context from reporter

Also check if there's already a diagnostic comment (skip if so).

---

## §2 — Reproduce

Attempt to reproduce the bug following the steps in the issue.

### If reproduction succeeds

Note:
- Environment (OS, version, browser)
- Steps followed exactly
- What happened vs what was expected

### If reproduction fails

Three possibilities:

| Situation | Action |
|-----------|--------|
| Bug is environment-specific | Note in issue: "Could not reproduce in [env]. May be environment-specific." |
| Bug was already fixed | Note in issue: "Bug could not be reproduced. May have been fixed in recent commits." |
| Steps are incomplete | Post comment asking for clarification |

Document reproduction status in issue.

---

## §3 — Minimize

Strip the reproduction to its essentials.

### Questions to answer

- Does it happen in staging, production, or both?
- Is it consistent or intermittent?
- What is the minimum data/condition needed?
- Which file/component is involved?
- When did it start? (recent change or longstanding?)

### Check git history

```bash
# If regression suspected
git log --oneline -10 -- {affected-file}
git log --oneline --since="2 weeks ago"
```

### Minimize strategies

| Bug type | Strategy |
|----------|----------|
| UI bug | Test in minimal page setup |
| API bug | Test with minimal payload |
| Data bug | Create minimal test dataset |
| Timing bug | Isolate the timing condition |

Document the minimum reproduction case.

---

## §4 — Research

If the bug is unclear or similar issues exist elsewhere, use `/research`.

### When to research

- Root cause is not obvious from code inspection
- Similar bugs reported elsewhere
- Library/dependency issue suspected
- No clear recent change caused the regression

### How to research

```bash
# Similar bugs in other projects
fresh search -q "{technology} {error-message} github issue" -l 5

# Official docs / known issues
fresh search -q "{technology} {feature} bug known issue" -l 5

# Fetch relevant sources
fresh fetch {url} -p "Extract: root cause, workaround, fix"
```

Use the bug-research template if needed:

```bash
cp templates/bug-research.md docs/bugs/{n}-research.md
```

### Integration with `/research`

```
/diagnose #N  →  /research  →  findings integrated into diagnostic
```

---

## §5 — Hypothesize + Instrument

Before looking at the code, enumerate all possible hypotheses.

### Enumerate hypotheses

List every possible cause:

```
Hypothesis 1: {possibility}
Hypothesis 2: {possibility}
Hypothesis 3: {possibility}
```

**Rule:** Always have at least 2 hypotheses. Never jump to the first obvious one.

### Instrument to verify

Add temporary logging to verify which hypothesis is correct.

```bash
# Add logging
git checkout -b diagnose/{n}-{slug}

# In the affected code, add:
logger.info('DEBUG: {context}', { relevant variables });
```

### Run the reproduction

```bash
# Deploy/test with instrumentation
{pkg-manager} dev
```

### Evaluate results

From the logs, eliminate incorrect hypotheses until only one remains.

---

## §6 — Root cause

Document the root cause clearly.

### Good root cause statement

**Format:** "The bug occurs because [cause], which causes [effect]."

**Example:**
"The bug occurs because `sendResetEmail()` sends the dynamic data field as `resetUrl`, but the SendGrid template expects `reset_link`. This causes SendGrid to ignore the reset link and fail delivery."

### Root cause vs. symptom

| | Root cause | Symptom |
|---|---|---|
| **What** | The actual failure point | What the user sees |
| **Example** | Wrong field name in code | Email not received |
| **Fix** | Change field name | N/A |

Always identify the root cause, not just the symptom.

---

## §7 — Document

Update the GitHub issue with findings.

### Add comment with diagnostic results

```bash
gh issue comment create {n} --body "$(cat <<'EOF'
## Diagnostic Results

**Reproduced:** {yes | no | partially}

**Environment:** {staging | production | both | {specific}}

**Root Cause:**

{clear explanation of the root cause}

**Affected Files:**
- `path/to/file` — {what's wrong}

**Minimum Reproduction:**
{steps to reproduce with minimal setup}

{If regression detected:}
**Regression:** Yes — introduced in commit `{hash}`

{If research findings:}
**Similar Issues:**
- [{title}]({url}) — {brief note}

---
*Diagnostic by Claude Code*
EOF
)"
```

### Update labels

Add appropriate labels:

```bash
# Area label if missing
gh issue edit {n} --add-label "area:{area}"

# Regression if applicable
gh issue edit {n} --add-label "regression"
```

### Link to research doc (if created)

If `/research` was used, link the research doc:

```bash
gh issue comment create {n} --body "Research findings: [docs/bugs/{n}-research.md](link)"
```

---

## §8 — Learning

After documenting, assess if this bug reveals a reusable pattern.

### When to write a learning

Create a learning doc if:
- This is a new category of bug we haven't seen before
- The fix reveals a pattern worth remembering
- The root cause is non-obvious
- The workaround teaches something

### How to extract

```bash
cp templates/learning.md docs/internal/learnings/{topic}/what-we-learned-about-{topic}.md
```

See [`skills/research/`](../research/) for the learning template and structure.

### Post learning link

After creating the learning, add to the issue:

```bash
gh issue comment create {n} --body "Learning extracted: [docs/internal/learnings/{topic}/](link)"
```

---

## Output

After completion, tell the user:

```
Bug #{n} diagnosed.

**Root cause:** {one-line summary}
**Affected:** {files/components}
{If regression:} **Regression:** introduced in {commit}

Next step: Run `/triage #{n}` to prioritize and mark as ready.
```

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Issue not found | Report: "Issue #{n} not found. Verify the number." |
| Issue is not a bug | Report: "Issue #{n} does not describe a bug. Use this skill only for bug reports." |
| Cannot reproduce | Document "Could not reproduce" in issue; note environment specifics |
| No root cause found | Document what's known; note what needs further investigation |
| `fresh auth status` expired | Inform user; skip research step or pause |

---

## Constraints

- **Never fix the bug directly.** This skill diagnoses only.
- **Always update the issue.** Diagnostic findings must be documented.
- **Never create branches for fixes.** Only for instrumentation during diagnosis.
- **Clean up instrumentation before finishing.** Remove debug logs, revert temporary changes.
- **Link to research if used.** Don't let research findings exist in isolation.
