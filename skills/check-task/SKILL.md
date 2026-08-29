---
name: check-task
description: >-
  Derive the state of one task from disk, git and gh without asking any session: planned,
  consolidated, in_progress, in_review, merged or done, per phase when the task has phases.
  Manager only; may run on its own when Sebastian asks about a task.
argument-hint: "[TASK_ID_OR_FOLDER]"
disable-model-invocation: false
---

# check-task

Input: a task id (`0142`) or a task folder path.
Output: one line per task (or per phase), nothing else.

Read `task.md` frontmatter: `id`, `title`, `type`, `branch`, `phases`, `modules`, `depends_on`.
Set `WORKTREE = {{repo}}/.claude/worktrees/{{id}}_{{title}}`.

## Signals

Collect, without talking to any session:

| Signal | How |
|---|---|
| folder in base | the folder exists in the main checkout |
| worktree | `git worktree list` contains `WORKTREE` |
| context | `Context & decisions` in `task.md` is non-empty (main checkout copy if no worktree, else worktree copy) |
| ahead | `git -C WORKTREE rev-list --count origin/{{base}}..HEAD` > 0 |
| developer | `claude agents` (or tmux panes of `task-{{id}}`) shows `task-{{id}}-developer*` |
| pr | `gh pr list --head {{branch}} --state all --json number,state,mergedAt` |

For phases, evaluate `ahead` and `pr` per phase branch; the current phase is the first one without a merged PR.

## Derivation

First match wins:

| Condition | State |
|---|---|
| pr merged and no worktree | `done` |
| pr merged and worktree | `merged` (pending `clean-task`) |
| pr open | `in_review` |
| ahead or developer | `in_progress` |
| worktree and context | `consolidated` |
| worktree and no context | `consolidating` |
| folder only | `planned` |

## Output

```
0142 badge_wall  feature  in_review   PR #57   modules: billing   phase 2/3
```

One line.
If Sebastian asked "why", add at most three lines of evidence (which signals fired).
Never more.

## Rules

- Never message a session to ask its state; existence is the only thing you check.
- Never change anything.
