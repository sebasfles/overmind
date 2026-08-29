---
name: check-work
description: "The project's task board: every task, grouped by state. om-manager; at session start and when Sebastian asks how things are."
effort: low
disable-model-invocation: false
---

# check-work

## Purpose

The project's task board: check-task over every task folder, grouped by state, one line per task. Also lists
drafts and stale worktrees. om-manager only; runs at session start via initialPrompt and whenever Sebastian asks
how things are.

Input: none.
Output: a short board, one line per task, grouped by state in this order: `in_review`, `merged`, `in_progress`, `consolidated`, `consolidating`, `planned`, then `done` collapsed to a count.

## 1. Collect

- Every folder under `docs/tasks/` except `_drafts/`: run `check-task` on each.
- `docs/tasks/_drafts/*.md`: list by name as `draft`.
- `ls .workspaces/`: any workspace with no matching task folder is `orphan workspace`.
- `gh pr list --state open --json headRefName`: any open PR on a `feat/`, `bugfix/`, `docs/`, `chore/`, `refactor/` branch with no task folder is `untracked PR`.

## 2. Print

```
in_review    0142 badge_wall        PR #57   waiting for Sebastian
merged       0139 fix_login         PR #55   run clean-task
in_progress  0143 export_csv        round 2
consolidated 0144 invoice_pdf       ready to delegate
planned      0145 rate_limits
drafts       refund_flow
done         12
```

First what needs Sebastian, then what is moving, then what is waiting.
No explanations; Sebastian opens the task's tmux window or the PR for detail.

## 3. Stale signals

Append one line each only if present:

- orphan worktrees and untracked PRs.
- an `in_progress` task whose branch has no new commit in 24 hours: `idle`.
- a `merged` task not cleaned for more than a day.

## Rules

- Read-only.
- Never longer than one screen; if there are more than 20 tasks, collapse `planned` to a count too.
