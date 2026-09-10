---
name: commit-scope
description: Never `git add -A`; stage only the files the change touched, because the tree often carries unrelated edits
metadata:
  type: feedback
---

Stage the exact files the change touched, one by one, never `git add -A` or `git add {{dir}}`.

**Why:** on 2026-09-10 an `update-method` commit swept in an uncommitted working-tree change to `agents/om-reviewer.md` (`model: fable` to `opus` plus a formatter table reflow) that nobody in the session had made, and that contradicted the ARD entry of 2026-09-08.
It took a second commit to undo, and it published a behavior change with no ARD entry, which is the one thing this session must never do.
The overmind repo is shared by three long-lived sessions plus every project's om-manager, so the working tree is almost never clean of other people's edits.

**How to apply:** in `update-method` step 6, stage the file list built in step 2 explicitly.
Before committing, run `git status --short` and account for every line; anything not on the list stays uncommitted.
See [[commit-own-memory]] for the one exception that is always yours to commit.
