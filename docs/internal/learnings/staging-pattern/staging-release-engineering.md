---
title: Staging Pattern for Release Engineering
description: A branching strategy where all changes flow through a staging branch before reaching main, with selective cherry-picks to control what goes in each release.
category: engineering
tags: [git, branching, staging, release-engineering, workflow, cherry-pick]
author: deessejs
created: 2026-07-16
updated: 2026-07-16
---

# Staging Pattern for Release Engineering

A branching strategy where all changes flow through a `staging` branch, but only **selected** changes go from staging to `main` for each release.

## TL;DR

Every change goes through `staging` for integration testing. But staging doesn't merge directly to main — you **select** which PRs go in each release. This gives you control over what ships when.

---

## Outline

- [The Pattern](#the-pattern)
- [Branch Hierarchy](#branch-hierarchy)
- [The Flow](#the-flow)
- [Branch Rules](#branch-rules)
- [Why Selective Releases](#why-selective-releases)
- [Common Mistakes](#common-mistakes)

---

## The Pattern

```
impl/123-add-auth    ── PR ──→ staging
impl/456-fix-bug     ── PR ──→ staging
impl/789-add-dashboard ── PR ──→ staging
                                        ↓
                              All merged to staging
                                        ↓
                              SELECT WHAT GOES TO MAIN
                                        ↓
                              release/v1.2.3 ── PR ──→ main
                                                      ↓
                                                    AUTO-RELEASE
```

### Key principles

1. **Never commit directly to `staging` or `main`**
2. **All work happens in feature branches**
3. **All feature branches PR into `staging`**
4. **Only selected PRs go to `main`** — cherry-pick, don't merge all of staging
5. **`main` is always in a deployable state**

---

## Branch Hierarchy

| Branch | Purpose | Lifetime | Who merges |
|---|---|---|---|
| `main` | Stable, release-ready | Permanent | Release PR |
| `staging` | Integration, all features meet here | Permanent | PR from feature branches |
| `impl/{n}-{slug}` | One feature/fix | Until merged | Developer/agent |
| `release/{date}` | Selected PRs for this release | Until merged | `/ship` skill |

### Optional branches

| Branch | Purpose | When to use |
|---|---|---|
| `hotfix/{n}-{slug}` | Critical production fix | When prod is down and can't wait |

---

## The Flow

### Normal flow

```
1. impl/123-add-auth is ready
2. Open PR → staging
3. Code review via /review-pr
4. Merge into staging

   Repeat for impl/456-fix-bug, impl/789-add-dashboard

5. When ready to ship:
   a. Run /ship
   b. Select which PRs go to main (cherry-pick)
   c. Release branch PR → main
6. Auto-release triggers on push to main
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
- **Versioned** — tagged/ changeset on every release
- **Rollback target** — if production breaks, rollback `main`

### `staging`

- **Integration point** — all feature branches merge here
- **Not always release-ready** — may have PRs that aren't meant for this release
- **Protected** — PR required to merge
- **CI runs on every PR** — each feature is tested before merging

### Release branches (`release/v{version}`)

- **Short-lived** — created for each release, deleted after merge
- **Selective** — cherry-picks only the PRs selected for this release
- **Contains changeset** — the version bump and changelog for this release
- **Named by version** — e.g., `release/v1.2.3`, not by date
- **One PR to main** — all selected PRs in one release branch

### Feature branches (`impl/{n}-{slug}`)

- **Scoped** — one issue, one branch
- **Short-lived** — days, not weeks
- **Deleted after merge** — branch is deleted from origin after merging into staging

### Hotfix branches (`hotfix/{n}-{slug}`)

- **Branches from `main`** — not from staging
- **Minimal** — fix only, no feature work
- **Tests on the fix** — CI must pass
- **PR → main** — bypasses staging for speed
- **Backport to staging** — cherry-pick or merge after

---

## Why Selective Releases

### Control what ships when

Not every PR in staging is ready for this release. Maybe a feature is behind a feature flag. Maybe a PR is waiting for documentation.

Selective releases let you:

- Ship features independently of each other
- Hold back incomplete work
- Ship a hotfix without shipping all of staging

### Staging = integration sandbox

Staging is where features meet and get tested together. It's not a release queue — it's a playground.

### Main = release target

Only what's explicitly selected goes to main. Main is always clean.

---

## Common Mistakes

### Merging all of staging to main

Staging is merged directly to main, shipping everything including PRs that aren't ready.

**Prevention:** Use `/ship` to cherry-pick only selected PRs.

### Long-lived release branches

Release branches live for days while waiting for PRs to be ready. They accumulate conflicts and become hard to merge.

**Prevention:** Keep release branches short (hours, not days). Merge early, even if incomplete.

### Forgetting to backport hotfixes

Hotfixes go to main but never make it back to staging.

**Prevention:** Always cherry-pick or merge the fix into staging after a hotfix.

### No release branch

PRs cherry-picked directly to main without a release branch, making rollback harder.

**Prevention:** Always create a release branch. It documents what went into the release.

### Staging becomes a dumping ground

PRs pile up in staging and nobody knows what's ready to ship.

**Prevention:** Review staging regularly. Use `/ship` to clear it out.

---

## Summary

| Branch | Role | Rule |
|---|---|---|
| `main` | Stable, deployable | Only receives merges from release branches |
| `staging` | Integration sandbox | All PRs merge here first |
| `impl/*` | Feature work | Short-lived, one issue per branch |
| `release/v{version}` | Selected PRs for release | Cherry-pick from staging, PR to main, named by version |
| `hotfix/*` | Critical fixes | Goes directly to main, backport to staging |

The pattern ensures that `main` is always clean, integration happens in staging, and you control exactly what ships in each release.
