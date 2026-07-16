---
title: Staging Pattern for Release Engineering
description: A branching strategy where all changes flow through a staging branch before reaching main, enabling integration testing and controlled releases.
category: engineering
tags: [git, branching, staging, release-engineering, workflow]
author: deessejs
created: 2026-07-16
updated: 2026-07-16
---

# Staging Pattern for Release Engineering

A branching strategy where all changes flow through a `staging` branch before reaching `main`, enabling integration testing and controlled releases.

## TL;DR

Every change goes through `staging` before reaching `main`. `staging` is the integration branch where features meet. `main` is always release-ready.

---

## Outline

- [The Pattern](#the-pattern)
- [Branch Hierarchy](#branch-hierarchy)
- [The Flow](#the-flow)
- [Branch Rules](#branch-rules)
- [Why This Pattern](#why-this-pattern)
- [Common Mistakes](#common-mistakes)

---

## The Pattern

```
main          ←─────────────── stable, always deployable
  ↑
staging       ←─────────────── integration branch, all PRs merge here first
  ↑
  ├── impl/123-add-auth
  ├── impl/456-fix-bug
  └── impl/789-add-dashboard
```

### Key principles

1. **Never commit directly to `staging` or `main`**
2. **All work happens in feature branches**
3. **All feature branches PR into `staging`**
4. **`staging` is merged into `main` when ready to release**
5. **`main` is always in a deployable state**

---

## Branch Hierarchy

| Branch | Purpose | Lifetime | Who merges |
|---|---|---|---|
| `main` | Stable, release-ready | Permanent | Release process |
| `staging` | Integration, all features meet here | Permanent | PR from feature branches |
| `impl/{n}-{slug}` | One feature/fix | Until merged | Developer/agent |
| `hotfix/{n}-{slug}` | Critical production fix | Until merged | Developer/agent |

### Optional branches

| Branch | Purpose | When to use |
|---|---|---|
| `dev` | Development integration | When `staging` is too stable for bleeding edge |
| `release/v{version}` | Release preparation | For release-specific fixes |

---

## The Flow

### Normal flow

```
1. impl/123-add-auth is ready
2. Open PR → staging
3. Code review
4. Merge into staging
5. Repeat for impl/456-fix-bug, etc.
6. When staging is ready to release:
   a. Merge staging → main
   b. Tag version
   c. Deploy
```

### Hotfix flow

```
1. Critical bug in production
2. Branch from main: hotfix/789-fix-auth
3. Fix, test, PR → main (bypass staging)
4. Deploy immediately
5. Cherry-pick or merge fix into staging
```

---

## Branch Rules

### `main`

- **Always deployable** — CI passes, tests pass
- **Protected** — no direct commits, PR required
- **Versioned** — tagged on every release
- **Rollback target** — if production breaks, rollback `main`

### `staging`

- **Integration point** — all feature branches merge here
- **Not always deployable** — may have conflicting changes being resolved
- **Protected** — PR required to merge
- **CI runs on every PR** — each feature is tested before merging

### Feature branches (`impl/{n}-{slug}`)

- **Scoped** — one issue, one branch
- **Short-lived** — days, not weeks
- **Clean history** — squash or rebase before merge
- **Deleted after merge** — branch is deleted from origin after merging into staging

### Hotfix branches (`hotfix/{n}-{slug}`)

- **Branches from `main`** — not from staging
- **Minimal** — fix only, no feature work
- **Tests on the fix** — CI must pass
- **PR → main** — bypasses staging for speed
- **Backport to staging** — cherry-pick or merge after

---

## Why This Pattern

### Integration testing happens before release

Features are integrated in `staging` before going to `main`. This catches interaction bugs between features.

### `main` stays clean

`main` only receives merges from `staging`, never from feature branches directly. If `main` has a problem, rolling back is straightforward.

### Parallel development

Multiple features can be developed in parallel, merged into `staging` when ready, and released together or separately.

### Hotfixes are fast

Critical bugs can go directly to `main` without waiting for the full `staging` integration cycle.

---

## Common Mistakes

### Committing directly to `staging`

Developers bypass the PR process and commit directly to `staging`. This breaks the integration flow and makes `staging` unstable.

**Prevention:** Branch protection + CODEOWNERS that require PRs.

### Long-lived feature branches

Branches live for weeks and accumulate conflicts. Resolving them becomes a project in itself.

**Prevention:** Keep branches short (days, not weeks). Merge early, even if incomplete.

### Forgetting to delete merged branches

Branches pile up and nobody knows what's merged and what's not.

**Prevention:** Automate deletion on merge. GitHub can delete head branches automatically.

### Skipping `staging`

Features go directly from feature branch to `main`. This bypasses integration testing.

**Prevention:** Branch protection rules that require `staging` as the base for all PRs to `main`.

### Not tagging releases

No record of what changed in each release.

**Prevention:** Tag `main` on every release. Use automated versioning (e.g., Changesets).

---

## Summary

| Branch | Role | Rule |
|---|---|---|
| `main` | Stable, deployable | Only receives merges from `staging` |
| `staging` | Integration | All PRs merge here first |
| `impl/*` | Feature work | Short-lived, one issue per branch |
| `hotfix/*` | Critical fixes | Goes directly to `main`, backport to `staging` |

The pattern ensures that `main` is always clean, integration happens in `staging`, and hotfixes can bypass the normal flow when needed.
