---
name: delegate-task
description: "Hand a consolidated task to its om-reviewer with \"delegated, start\". om-manager; Sebastian invokes it, or consolidate-task chains into it."
effort: medium
argument-hint: "[TASK_FOLDER]"
disable-model-invocation: false
---

# delegate-task

## Purpose

Hand a consolidated task to its om-reviewer. Checks depends_on, makes sure the om-reviewer session is running
(reopening the stopped one with its context intact), rebases the root worktree so the workspace receives the
task folder with its decisions, and sends "delegated, start". The om-reviewer launches the om-developer. om-manager
only; Sebastian invokes it, or consolidate-task chains into it.

Input: `$ARGUMENTS[0]`, a consolidated task folder in the root checkout: committed as `planned`, workspace present, `Context & decisions` written.
If any is missing, stop: the task needs `consolidate-task` first.

Output: the om-reviewer running in its tmux window with `delegated, start` delivered.
From here the om-manager does not intervene until the om-reviewer reports `PRs ready`.

Set `TASK`, `WORKSPACE`, `WINDOW`, `SESSION`, `PROJECT`, `REPOS` as in `consolidate-task`.
`ROOT_WT` = the root's worktree inside the workspace: `{{WORKSPACE}}/{{root name}}` in multirepo; `{{WORKSPACE}}/{{repo name}}` in single and mono (the code worktree is the root's).

## 1. Dependencies

Derive the state of every id in `depends_on` as `check-task` does.
If any is not `done`, stop and say which task blocks this one.

## 2. om-reviewer session

| State | How you know | Action |
|---|---|---|
| Running | `claude agents` lists `{{SESSION}}` running, or a live pane in `{{WINDOW}}` | nothing |
| Stopped | `claude agents --all` lists it stopped, or its transcript exists under `~/.claude/projects/{{slug of WORKSPACE}}/` | `tmux new-window -t {{PROJECT}} -n {{WINDOW}} -c {{WORKSPACE}}`, then `claude attach {{id}}` (or `claude -r {{session-id}}`) in the pane; context intact |
| Lost | none of the above | launch a new om-reviewer as `consolidate-task` step 4 does; its `analyze-task` finds `Context & decisions` written and does not ask; wait for `consolidated` |

## 3. Bring the task folder into the workspace

```
git -C {{ROOT_WT}} fetch origin
git -C {{ROOT_WT}} rebase origin/{{root base}}
```

Now `{{ROOT_WT}}/docs/tasks/{{TASK}}/` exists with `Context & decisions`.
From here every write to the task folder and to `docs/modules/` goes to `{{ROOT_WT}}`, never to the root checkout.
If the rebase fails, stop and show the error.

## 4. Hand over

`SendMessage` to `{{SESSION}}`:

```
delegated, start. task: {{ROOT_WT}}/docs/tasks/{{TASK}}/
```

The om-reviewer runs `start-task`.

## 5. Report

One line to Sebastian: `{{TASK}} delegated; om-reviewer {{SESSION}} in window {{WINDOW}}`.
Then wait; do not poll.

## Rules

- Never launch an om-developer; never message the om-developer.
- Never write to the task folder; the rebase is the only change you make in the workspace.
- Never delegate a task whose `depends_on` is not all `done`.
- Absolute paths in every command.
