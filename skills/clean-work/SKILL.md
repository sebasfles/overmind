---
name: clean-work
description: Run clean-task for every merged task and report leftovers. om-manager; Sebastian invokes it.
effort: low
disable-model-invocation: true
---

# clean-work

## Purpose

Run clean-task for every task in state merged, and report orphan worktrees and stale branches that clean-task
cannot claim. om-manager only; Sebastian invokes it.

Input: none.
Output: one line per task cleaned, then leftovers.

## 1. Find

Run `check-work`.
Take every task in `merged`.

## 2. Clean

For each, run `clean-task`.
Continue on failure; collect the error.

## 3. Leftovers

Report, without deleting:

- Orphan workspaces under `.workspaces/` with no task folder.
- Local branches with the task prefixes whose remote is gone (`git branch -vv | grep ': gone]'`).
- Stopped Claude sessions (`claude agents --all`) for tasks already `done`.

Ask Sebastian before removing any of them; they were not created by a task you can verify.

## Rules

- Only `merged` tasks are cleaned automatically.
- Never remove anything `clean-task` did not create.
