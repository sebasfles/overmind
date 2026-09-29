---
name: clean-work
description: Run clean-task for every merged task and clean-pr for every merged PR review, then report leftovers. om-manager; Sebastian invokes it.
effort: low
disable-model-invocation: true
---

# clean-work

## Purpose

Run clean-task for every task in state merged, clean-pr for every local PR review whose PR is merged or closed,
close every om-architect planning window whose draft is gone, and report orphan worktrees and stale branches that neither can claim. om-manager only; Sebastian invokes it.

Input: none.
Output: one line per task or PR cleaned, then leftovers.

## 1. Find

Run `check-work`.
Take every task in `merged`.
Run `check-prs`.
Take every PR in `cleanable`.

## 2. Clean

For each task, run `clean-task`; for each PR, run `clean-pr`.
For each `.workspaces/plan-{{title}}/` whose draft `docs/tasks/_drafts/{{title}}.md` is gone, close it as `create-task` step 7 does: the om-architect's session, window `plan-{{title}}`, the folder.
Continue on failure; collect the error.

## 3. Leftovers

Report, without deleting:

- Orphan workspaces under `.workspaces/` with no task folder and no `pr-{{n}}` or `plan-{{title}}` name.
- Local branches with the task prefixes whose remote is gone (`git branch -vv | grep ': gone]'`).
- Stopped Claude sessions (`claude agents --all --json`) for tasks already `done`.

Ask Sebastian before removing any of them; they were not created by a task you can verify.

## Rules

- Only `merged` tasks, `cleanable` PR reviews and `plan-*` scratch without a draft are cleaned automatically.
- Never remove anything `clean-task` or `clean-pr` did not create.
