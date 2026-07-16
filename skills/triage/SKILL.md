---
name: triage
description: Triage a GitHub issue — analyze content, apply labels, set org-level fields (Priority, Effort, Type), post a structured review comment. Use when asked to triage or review an issue.
---

# `triage` Skill

Triage a GitHub issue: analyze content, apply labels, set org-level fields, post a structured review comment.

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

### Step 3 — Assess completeness

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

### Step 4 — Triage decision

Apply the decision tree:

```
1. Is it complete?
   NO  → set needs-info
   YES → continue

2. Is it a valid task? (not dup/question/invalid/wontfix)
   duplicate  → label: duplicate
   wontfix    → label: wontfix
   question   → label: question
   invalid    → label: invalid
   YES        → continue

3. Is it blocked by another issue? (mentioned in body or comments)
   YES → label: status:blocked + link blocking issue

4. Otherwise → status:ready
```

### Step 5 — Set org-fields and labels

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
- `status:ready` — complete + valid + unblocked
- `status:needs-triage` — keep as-is, already set by template
- `needs-info` — incomplete, ask for more
- `status:in-progress` — someone is already working it
- `status:blocked` — depends on another issue

**Cross-cutting** (add when applicable)
- `breaking-change` — affects public API surface
- `github_actions` — related to CI/CD
- `dependencies` — dependency update request

### Step 6 — Post triage comment

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

**Decision:** All required information provided. This issue is ready to be picked up.

---
*Triage by Tech Lead Agent*
```

### `needs-info`

```
## Triage Review

**Status:** `needs-info` — Additional information required

**Decision:** This issue is missing required information and cannot be triaged yet.

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

### Closure labels (`duplicate`, `wontfix`, `question`, `invalid`)

```
## Triage Review

**Status:** `<duplicate | wontfix | question | invalid>`

**Decision:** <Brief explanation>

<If a related issue exists, link it here.>

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
- **When in doubt, ask for more info** (`needs-info`) rather than guessing

## Error Handling

| Situation | Action |
|---|---|
| Org-field GET returns 404 | Report to user — org may not have Issue Fields enabled |
| Org-field value rejected | Verify option name matches the org-field definition; re-fetch if needed |
| Missing `area:*` label | Always add if absent; infer from issue body |

## Constraints

- Prefix **every** `gh api` call with `MSYS_NO_PATHCONV=1 MSYS2_ARG_CONV_EXCL='*'` (Windows Git Bash path-rewrite bug).
- Use `--input -` for `issue_field_values` array — not `-f` or `-F`.
- Priority, Effort, Type are set via **org-fields** (API), NOT via labels.
- Never remove existing labels added by the user.
