---
title: What Makes an Excellent Release
description: A guide to understanding what a good release process should include and when to trigger it.
category: engineering
tags: [release, deployment, changelog, versioning]
author: deessejs
created: 2026-07-16
updated: 2026-07-16
---

# What Makes an Excellent Release

A guide to understanding what a good release process should include and when to trigger it.

## TL;DR

A release is the moment where a change goes live. An excellent release is one where the team knows what changed, users know what's new, and the system stays healthy. If anyone has to ask "what was in that release?" the release has failed.

---

## Outline

- [What a Release Is](#what-a-release-is)
- [The Release Checklist](#the-release-checklist)
- [Release Notes](#release-notes)
- [Versioning](#versioning)
- [Branch Cleanup](#branch-cleanup)
- [Post-Release Communication](#post-release-communication)
- [Rollback](#rollback)
- [Common Release Failures](#common-release-failures)

---

## What a Release Is

A release is the moment when changes accumulated in `main` are deployed to production (or packaged for distribution).

A release is **not**:
- A deployment — deployment is the technical act of pushing code live; a release is the communication and documentation around it
- A deployment pipeline — CI/CD handles the automation; the release process handles the human side

The release process answers:
- What changed?
- What do users need to know?
- Is anything at risk?

---

## The Release Checklist

Before completing a release, verify:

| Check | Why |
|---|---|
| **All staging PRs merged** | Nothing is waiting in staging |
| **CI green on main** | The code that will ship is tested |
| **Changeset present** | The version number will bump |
| **No regressions** | Production health is maintained |
| **Changelog generated** | Users know what changed |
| **Breaking changes communicated** | Users can migrate |
| **Release notes published** | Team and stakeholders know what's live |
| **Branches cleaned** | Old branches don't clutter the repo |

---

## Release Notes

Release notes answer: "what changed that I care about?"

### What to include

**For users:**
- New features (what it does, not how it works)
- Bug fixes (the problem that is now solved)
- Breaking changes (what they need to do differently)
- Known issues (what isn't fixed yet)

**For developers:**
- Technical changes (internal refactors, dependency updates)
- Migration steps for breaking changes
- Deprecations (what will be removed)

### What NOT to include

- Every commit (that's the git log)
- Internal discussions (that's not user-relevant)
- Boilerplate ("bug fixes and improvements")
- Vague promises ("various improvements")

### Tone

- **User-facing** — write for users, not developers
- **Active voice** — "We added X" not "X was added"
- **Specific** — "Users can now export to PDF" not "Export functionality improved"
- **Brief** — a paragraph per feature, a line per fix

### Format

```
## Release Notes — v1.2.3

### New Features
- **{Feature name}:** Brief description of what it does and why users want it.

### Bug Fixes
- Fixed **{issue}** — users no longer experience **{problem}**.

### Breaking Changes
- **{Change}:** Users must **{action}** before upgrading. [Migration guide link]

### Deprecations
- **{Feature}** is deprecated and will be removed in v2.0. Use **{alternative}** instead.

### Internal
- Upgraded dependencies
- Refactored **{component}**
```

---

## Versioning

Use semantic versioning (SemVer):

| Change | Version bump | Example |
|---|---|---|
| Bug fix | `patch` | v1.2.3 → v1.2.4 |
| New feature (backward-compatible) | `minor` | v1.2.3 → v1.3.0 |
| Breaking change | `major` | v1.2.3 → v2.0.0 |

### Changesets

If using Changesets:

- Every PR that changes the public API or ships user-visible changes must include a changeset
- The changeset file determines the version bump automatically
- Without a changeset, no version bump occurs

---

## Branch Cleanup

After a release, clean up:

| What | Action |
|---|---|
| **Merged feature branches** | Delete from origin |
| **Merged `staging`** | Already deleted on merge |
| **Stale branches (>30 days)** | Delete from origin |
| **Release tags** | Keep (they're the history) |

Do not delete branches that are still in use or that might be needed for hotfixes.

---

## Post-Release Communication

### Internal

Notify the team when a release is live:

- What version is deployed
- What changed
- Any flags to watch for

### External (users)

For significant releases, update:

- Changelog on the website
- Release notes in the product (if applicable)
- Social media / newsletter (for major releases)

### Breaking changes

Breaking changes require proactive communication:

- Notify before the release (in the PR or issue)
- Include migration steps
- Give users time to adapt (don't drop a breaking change on a Friday afternoon)

---

## Rollback

A rollback reverses a release. Know before you need it:

### When to rollback

| Trigger | Action |
|---|---|
| Critical bug affecting all users | Rollback immediately |
| Data corruption | Rollback + incident response |
| Security vulnerability | Rollback + security patch |
| Performance regression | Assess — may not need rollback |

### How to rollback

1. Revert the problematic PR(s)
2. Deploy the revert
3. Communicate the rollback
4. Investigate root cause
5. Ship a fix properly

### When NOT to rollback

| Situation | Action |
|---|---|
| Minor bug affecting few users | Fix in next release |
| UX regression | Fix in next release |
| Non-critical breaking change | Communicate + give users time |

---

## Common Release Failures

### Silent releases

Changes go live with no communication. Nobody knows what changed. Users discover bugs by accident.

**Prevention:** Always generate and publish release notes, even for small releases.

### Missing changelog

The team doesn't know what shipped. Someone has to dig through git log to figure it out.

**Prevention:** Automate changelog generation from commits and PRs.

### Breaking change surprise

A breaking change ships without warning. Users are caught off guard.

**Prevention:** Label breaking changes clearly. Communicate before the release, not after.

### Untested release

CI passed but something broke in production. Tests didn't cover the edge case.

**Prevention:** Add smoke tests. Monitor production after deploy.

### Branch bloat

Old branches pile up. Nobody knows which are stale.

**Prevention:** Delete merged branches as part of the release process. Set a policy (e.g., branches older than 30 days without activity).

---

## Summary

| Step | Action |
|---|---|
| **Before release** | Verify CI green, all PRs merged, changeset present |
| **During release** | Generate changelog, bump version |
| **After release** | Publish notes, notify team, clean branches |
| **Communication** | Users know what changed; team knows what's live |
| **Rollback** | Know when and how; don't rollback for minor issues |
