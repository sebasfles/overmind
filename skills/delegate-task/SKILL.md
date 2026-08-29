---
name: delegate-task
description: >-
  Hand a consolidated task to its reviewer. Checks depends_on, makes sure the reviewer session is running
  (reopening the stopped one with its context intact), rebases the worktree branch so it receives the task
  folder, and sends "delegated, start". The reviewer launches the developer. Manager only; Sebastian
  invokes it, or consolidate-task chains into it.
argument-hint: "[TASK_FOLDER]"
disable-model-invocation: false
---

# delegate-task

Input: `$ARGUMENTS[0]`, the absolute path of a consolidated task folder in the main checkout: committed as `planned`, worktree present, `Context & decisions` written.
If any of those is missing, stop: the task needs `consolidate-task` first.

Output: the reviewer running in its tmux window with the message `delegated, start` delivered.
From here the manager does not intervene until the reviewer reports `PR #{{n}} ready`.

Read the frontmatter: `id`, `title`, `depends_on`, `phases`.
Set `TASK`, `WORKTREE`, `WINDOW`, `SESSION`, `PROJECT` as in `consolidate-task`.

## 1. Dependencies

For every id in `depends_on`, derive its state as `check-task` does.
If any is not `done`, stop and tell Sebastian which task blocks this one.
Do not delegate a blocked task.

## 2. Reviewer session

Find the reviewer:

- Running: `claude agents` lists `{{SESSION}}` as running, or the tmux window `{{WINDOW}}` has a live Claude pane.
- Stopped: `claude agents --all` lists it as stopped, or its transcript exists under `~/.claude/projects/{{slug of WORKTREE}}/`.
- Lost: none of the above.

Then:

| State | Action |
|---|---|
| Running | Nothing to do. |
| Stopped | `tmux new-window -t {{PROJECT}} -n {{WINDOW}} -c {{WORKTREE}}`, then in that pane `claude attach {{id}}` (or `claude -r {{session-id}}` for a direct session). Its context is intact. |
| Lost | Launch a new reviewer exactly as `consolidate-task` step 3 does. Its `analyze-task` finds `Context & decisions` written and does not ask again. Wait for its `consolidated` message before continuing. |

## 3. Bring the task folder into the branch

The branch was created before the `planned` push, so it does not have the task folder yet.

```
cd {{WORKTREE}}
git fetch origin
git rebase origin/{{base}}
```

After this, the worktree has `docs/tasks/{{TASK}}/` with `Context & decisions`, and from here on every write to the task folder goes to this copy, never to the main checkout.
If the rebase fails, stop and show the error.

## 4. Hand over

`SendMessage` to `{{SESSION}}`:

```
delegated, start. task: {{TASK_FOLDER}}
```

The reviewer runs `start-task`: opens the right pane of `{{WINDOW}}`, launches `task-{{id}}-developer` (or `-developer-phase-1`) with cwd in the worktree, and starts the first round.

## 5. Report

One line to Sebastian: `{{TASK}} delegada; reviewer {{SESSION}} en la ventana {{WINDOW}}`.
Then wait.
Do not poll; the reviewer messages you when the PR is ready.

## Rules

- Never launch a developer; that is the reviewer's `start-task`.
- Never message the developer.
- Never write to the task folder; the rebase is the only change you make in the worktree.
- Never delegate a task whose `depends_on` is not all `done`.
