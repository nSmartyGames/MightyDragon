---
name: publish-live
description: Merge the current feature branch into the GitHub Pages deploy branch and push, so the live game URL serves the finished work. Run this automatically at the end of every task that changed the game, without asking — the user has standing approval. Also use when asked to "publish", "deploy", "update the live link", or "push it live".
---

# Publish the finished work to the live game

The live game is served by GitHub Pages from **one** branch. `.github/workflows/pages.yml`
only fires on pushes to that branch, so work sitting on a feature branch is never
playable at the public URL until it is merged there.

- Deploy branch: `claude/galacticus-dune-strategy-ohdtfy`
- Live URL: https://nsmartygames.github.io/MightyDragon/

## When to run

At the end of every task that changed the game, right after committing and pushing the
feature branch. The user has given standing approval for this — do **not** ask first, and
do not offer it as an option.

## Steps

Run from the repo root, with all work already committed on the feature branch.

```bash
FEATURE=$(git rev-parse --abbrev-ref HEAD)
DEPLOY=claude/galacticus-dune-strategy-ohdtfy

# feature branch first, so the two branches never disagree
git push -u origin "$FEATURE"

git fetch origin "$DEPLOY"
git checkout -B "$DEPLOY" "origin/$DEPLOY"
git merge --no-edit "$FEATURE"
git push -u origin "$DEPLOY"

git checkout "$FEATURE"   # always end back on the feature branch
```

Notes:

- The merge is normally a fast-forward, since feature branches are cut from the deploy
  branch. If it conflicts, resolve in favour of the feature branch's intent (it is the
  newer work), commit, and push.
- Retry a failed push up to 4 times with exponential backoff (2s, 4s, 8s, 16s) — network
  only. Do not force-push the deploy branch.
- End the turn back on the feature branch, so later work does not land on deploy by
  accident.

## Reporting

Tell the user the live URL and that Pages is rebuilding (the deploy workflow takes about
a minute). Do not claim the new version is live from having checked it — this sandbox's
proxy blocks `github.io`, so the deploy cannot be verified from here. Report what was
pushed, not what was observed.
