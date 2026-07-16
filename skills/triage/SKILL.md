---
name: triage
description: Triage a GitHub issue — analyze content, search for duplicates, verify against codebase, apply labels, set org-level fields, post a structured review comment.
---

# `triage` Skill

Triage a GitHub issue: analyze content, search for duplicates, verify against the codebase, apply labels, set org-level fields, post a structured review comment.

## Source of truth

**Org-level fields** (Priority, Effort, Type) are the single source of truth for these values. **Labels do NOT represent Priority or Effort** in this workflow — use org-fields only.

## When to use

Use this skill when the user asks to triage or review an issue. Usually runs on newly opened issues.

**Trigger phrases:** "triage #N", "review this issue", "/triage #N".

## Prerequisites

The `gh` CLI must be authenticated:
```bash
gh auth status
```

## Step-by-step workflow

### Step 1 — Fetch issue details

```bash
gh issue view <issueNumber> --json title,body,labels,state,author
gh api "https://api.github.com/repos/<org>/<repo>/issues/<issueNumber>"
```

Read: `title`, `body`, `labels[]`, `state`, `author.login`.

Also fetch the org's issue fields to know what's available:

```bash
gh api "https://api.github.com/orgs/<org>/issue-fields"
```

Parse to build a map of `{ fieldName → { id, options: { optionName → id } } }`.

### Step 2 — Identify what's already present

From the `labels` array, note:
- Does it have an `area:*` label?
- Does it have a `status:*` label?

From the issue API response, note the existing org-field values (Priority, Effort, Type).

### Step 3 — Search for duplicates

```bash
# Search for open issues with similar titles
gh search issues --repo <org>/<repo> "<title keywords>" --state open --limit 10
```

If similar issues are found:

- Present them to the user
- Ask if the new issue is a duplicate of an existing one
- If yes → apply `duplicate` label and link to the existing issue (see closure template)
- If no → continue

### Step 4 — Verify against codebase

Read the codebase to confirm the issue is real, not based on incorrect assumptions.

**Bug report** (`bug` label present):
- Locate the relevant code mentioned in the issue
- Confirm the bug exists (or note if it cannot be reproduced)
- If the bug does not exist → flag in triage comment

**Feature request** (`enhancement` label present):
- Check if the requested feature or equivalent already exists in the codebase
- If the feature exists → note it in triage comment
- If partial → note what's missing

**Refactor request**:
- Locate the code referenced in the issue
- Confirm the problematic pattern exists
- If the code looks fine → flag it

**Task / chore**:
- Identify the files or areas mentioned
- Confirm the task is applicable and not already done

If the issue describes something that doesn't match reality → mark as `needs-info` with explanation. Do not apply `status:ready` on an unverified issue.

### Step 5 — Assess completeness

Based on the template type, check for required fields:

**Bug report** (`bug` label present):
- ✅ **Required:** `📝 Bug Description` filled?
- ✅ **Required:** `📋 Steps to Reproduce` filled?
- ✅ **Required:** `🎯 Area` selected?
- ℹ️ `❌ Actual Behavior` and `✅ Expected Behavior` — strongly recommended but not hard-required

**Feature request** (`enhancement` label present):
- ✅ **Required:** `📝 Feature Summary` filled?
- ✅ **Required:** `📖 Detailed Description` filled?
- ✅ **Required:** `🎯 Area` selected?

**Any other issue** (no template used):
- ✅ Is the body substantive (not just a title)?
- ✅ Is the intent clear enough to act on?

### Step 6 — Triage decision

Apply the decision tree:

```
1. Is it a duplicate of an existing issue?
   YES → label: duplicate + link to existing issue

2. Is it complete?
   NO  → label: needs-info
   YES → continue

3. Is it a valid task? (not question/invalid/wontfix)
   wontfix    → label: wontfix
   question   → label: question
   invalid    → label: invalid
   YES        → continue

4. Is it blocked by another issue? (mentioned in body or comments)
   YES → label: status:blocked + link blocking issue

5. Does the issue match reality? (codebase verified)
   NO  → label: needs-info with explanation
   YES → continue

6. Otherwise → status:ready
```

### Step 7 — Set org-fields and labels

#### Org-fields (Priority, Effort, Type)

If any org-field is missing or incorrect, set or correct it via PATCH:

```bash
MSYS_NO_PATHCONV=1 MSYS2_ARG_CONV_EXCL='*' \
  gh api -X PATCH "https://api.github.com/repos/<org>/<repo>/issues/<issueNumber>" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  --input - <<EOF
{
  "type": "Task",
  "issue_field_values": [
    {"field_id": <field_id>, "value": "<value>"}
  ]
}
EOF
```

- Use the **option name as a string** (e.g., `"Medium"`, `"High"`, `"Low"`) — NOT the numeric option ID.
- Both `type` and `issue_field_values` can be set in the same PATCH call.

#### Labels

```bash
gh issue edit <issueNumber> --add-label "label1,label2"
```

Labels to add if missing (do NOT remove existing labels):

**area:** (required — always add if absent)
- `area:auth`, `area:ui`, `area:web`, `area:app`, `area:docs`, `area:database`, `area:email`, `area:ci`, `area:build`, `area:deploy`
- Infer from body content if the dropdown was not used

**status:** (override template default only when justified)
- `status:ready` — complete + valid + unblocked + verified
- `status:needs-triage` — keep as-is, already set by template
- `needs-info` — incomplete or unverifiable, ask for more
- `status:in-progress` — someone is already working it
- `status:blocked` — depends on another issue

**Cross-cutting** (add when applicable)
- `breaking-change` — affects public API surface
- `github_actions` — related to CI/CD
- `dependencies` — dependency update request

### Step 8 — Post triage comment

```bash
gh issue comment create <issueNumber> --body "..."
```

Use the template below that matches your decision.

---

## Comment Templates

### `status: ready`

```
## Triage Review

**Type:** `{type}`
**Priority:** `{Priority}`
**Effort:** `{Effort}`
**Status:** `status:ready`
**Area:** `area:<...>`

**Codebase:** Verified — the issue matches the current state of the code.

**Decision:** All required information provided and verified. This issue is ready to be picked up.

---
*Triage by Tech Lead Agent*
```

### `needs-info`

```
## Triage Review

**Status:** `needs-info` — Additional information required

**Decision:** This issue cannot be triaged yet.

**Reason:** <incomplete / unverifiable / describes non-existent bug or feature>

**Missing fields:**
- <list each missing required field>

Please update the issue with the missing information so it can be properly triaged.

---
*Triage by Tech Lead Agent*
```

### `status: blocked`

```
## Triage Review

**Type:** `{type}`
**Status:** `status:blocked`

**Decision:** This issue cannot be acted on until the blocking issue is resolved.

**Blocking:** #<issue number> — <brief description>

Once the blocking issue is resolved, this can be moved to `status:ready`.

---
*Triage by Tech Lead Agent*
```

### `duplicate`

```
## Triage Review

**Status:** `duplicate`

**Decision:** This issue is a duplicate of #<existing issue number>.

<a brief explanation of why — similar root cause / same feature request / same bug>

Please continue the discussion on #<existing issue number> if needed.

---
*Triage by Tech Lead Agent*
```

### Closure labels (`wontfix`, `question`, `invalid`)

```
## Triage Review

**Status:** `<wontfix | question | invalid>`

**Decision:** <Brief explanation>

For questions, consider using GitHub Discussions instead of issues.

---
*Triage by Tech Lead Agent*
```

## Label Handling Rules

- **Always check existing labels** before adding — don't double-apply
- **Add missing labels only** — never remove user-added labels
- **Preserve user intent** — keep labels even if they seem slightly off
- **Keep `status:needs-triage`** — it's the template default; only change if justified
- **Infer `area:*` from body** when the template dropdown wasn't used
- **Verify before marking `status:ready`** — an issue that describes non-existent code should not be marked ready

## Error Handling

| Situation | Action |
|---|---|
| Org-field GET returns 404 | Report to user — org may not have Issue Fields enabled |
| Org-field value rejected | Verify option name matches the org-field definition; re-fetch if needed |
| Missing `area:*` label | Always add if absent; infer from issue body |
| Codebase check inconclusive | Default to `needs-info` |

## Constraints

- Prefix **every** `gh api` call with `MSYS_NO_PATHCONV=1 MSYS2_ARG_CONV_EXCL='*'` (Windows Git Bash path-rewrite bug).
- Use `--input -` for `issue_field_values` array — not `-f` or `-F`.
- **Priority** values: `Urgent`, `High`, `Medium`, `Low`. **Effort** values: `High`, `Medium`, `Low`. **Type** values: `Task`, `Bug`, `Feature`.
- Priority, Effort, Type are set via **org-fields** (API), NOT via labels.
- Never remove existing labels added by the user.
- Do not mark an issue `status:ready` unless the codebase has been verified.
