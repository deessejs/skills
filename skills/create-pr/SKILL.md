---
name: create-pr
description: Open a PR for an implemented issue — reads the spec, opens PR, updates labels, posts comment. Run after /implement #{n}. Use when asked to create a PR, open a PR, or create a pull request.
---

# `create-pr` Skill

Open a PR, update issue labels, assign, and post a comment.

## When to use

**Trigger phrases:** "create-pr #N", "open pr #N", "/create-pr #N".

Run this **after** `/implement #{n}` — the branch must be pushed with the implementation.

## Workflow overview

```
0. Reset       — return to staging and pull latest
1. Fetch       — read the issue + the spec
2. Check       — gate A: branch on origin · gate B: no existing open PR
3. PR          — open the pull request to staging
4. Update      — label, assign, post comment
5. Changeset   — verify .changeset/*.md exists, add if missing
```

## §0 — Reset (always)

```bash
git checkout staging && git pull origin staging
```

## §1 — Fetch

Fetch all in parallel:

```bash
gh api "https://api.github.com/repos/<org>/<repo>/issues/{n}"
gh api --paginate "https://api.github.com/repos/<org>/<repo>/issues/{n}/comments"
gh api -H "X-GitHub-Api-Version: 2026-03-10" \
  "https://api.github.com/orgs/<org>/issue-fields"
gh api "https://api.github.com/repos/<org>/<repo>/contents/<spec-path>?ref=<impl-branch>" \
  --jq '.content' 2>/dev/null | base64 -d -
```

Also fetch the latest commit to confirm branch is up to date:

```bash
git fetch origin "<impl-branch>"
```

## §2 — Check

**Gate A — branch on origin**

```bash
git fetch origin "<impl-branch>" 2>/dev/null && echo "found" || echo "missing"
```

Refuse if missing:

> "Branch `<impl-branch>` is not on origin. Run `/implement #{n}` first."

**Gate B — no existing open PR**

```bash
gh api --paginate "https://api.github.com/repos/<org>/<repo>/pulls?state=open&base=staging&per_page=50" \
  --jq '.[] | select(.body | contains("#{n}")) | {number, html_url}'
```

If a PR exists:

> "A PR already exists for issue #{n}: {PR URL}. Do you want me to update it instead?"

If yes: proceed to §3 but use `--edit` instead of `--create`. Then go to §5 to verify the changeset file.

## §3 — Open PR

```bash
gh pr create \
  --base staging \
  --title "{issue title}" \
  --body "## Summary

{TL;DR from the spec}

## What changed

- {file}: {what changed}
- ...

## Changeset

<!-- changeset-type: patch | minor | major -->

A `.changeset/*.md` file should be present in this PR to document the change type and trigger the release workflow on merge to main.

## Verification

- [ ] Build passes
- [ ] Typecheck passes
- [ ] Tests pass
- [ ] Lint passes

## Related

Part of #{n}

---

🤖 Generated with [Claude Code](https://claude.com/claude-code)" \
  --label "{labels}" \
  --assignee <author>
```

Capture the returned URL.

## §4 — Update Issue

After PR is open, do all three in parallel:

**1. Remove `status:ready`**, add `status:in-progress`:

```bash
gh issue edit {n} --remove-label "status:ready" --add-label "status:in-progress"
```

**2. Assign to `<author>`:**

```bash
gh issue edit {n} --add-assignee <author>
```

**3. Post a comment:**

```markdown
<!-- triage-skill:v1 -->
## Implementation started

PR opened: {PR URL}

The spec was reviewed and approved. This PR targets `staging`. After CI green + approval, merge `staging → main` manually.

_Triage by @<author>._
```

## §5 — Verify changeset

Check if a `.changeset/*.md` file exists in the branch:

```bash
git fetch origin "<impl-branch>"
git ls-tree -r --name-only origin/<impl-branch> | grep "^.changeset/"
```

If **missing**, offer to create one. Adapt to your project's changeset tooling:

```bash
<pkg-manager> changeset add
```

Ask the user to describe the change and select `patch`, `minor`, or `major`. Commit and push:

```bash
git add .changeset/*.md
git commit -m "docs(changeset): document change

Co-Authored-By: Claude <noreply@anthropic.com>"
git push origin "<impl-branch>"
```

Tell the user:

> "Changeset `.changeset/*.md` added. This will trigger the release workflow when staging → main merges."

## Output

Confirm with a one-liner: PR number, title, URL, and next step.

> "PR #{n}: {title} — {PR URL}. Targets `staging`. After CI green + review approval, merge `staging → main` → release workflow triggers automatically."

## Error handling

| Situation | Action |
|---|---|
| Already on a branch | §0 resets to staging automatically |
| Branch not on origin | Refuse — Gate A |
| PR already exists | Tell user; offer to update instead |
| No changeset file found | Create one with `<pkg-manager> changeset add` before creating PR |
| PR creation fails | Check labels are valid, then retry |

## Constraints

- **Always return to `staging` first.**
- **Run after `/implement #{n}`** — the branch must exist and be pushed.
- **PR always targets `staging`.**
- **A `.changeset/*.md` file must be present** — the release workflow depends on it.
- Never push to `main`.
- Never merge a PR from this skill.
- Adapt `<org>/<repo>`, `<impl-branch>`, `<spec-path>`, `<author>`, and changeset commands to your project conventions.
