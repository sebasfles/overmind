---
name: start-task
description: >-
  Launch the developer for the current task or phase in the right pane of the task's tmux window,
  with cwd in the worktree, and send it "context ready, start". Reviewer only; runs on
  "delegated, start" from the manager and inside next-phase.
argument-hint: "[TASK_FOLDER] [PHASE_N]"
disable-model-invocation: false
---

# start-task

Input: the task folder (worktree copy from here on) and, if the task has phases, the phase number.
Output: a running `task-{{id}}-developer` (or `task-{{id}}-developer-phase-{{n}}`) that has received `context ready, start`.

Set:

```
TASK      = {{id}}_{{title}}
WORKTREE  = your cwd
WINDOW    = task-{{id}}
DEV       = task-{{id}}-developer            or task-{{id}}-developer-phase-{{n}}
PROJECT   = the tmux session this window belongs to
TASK_DIR  = {{WORKTREE}}/docs/tasks/{{TASK}}
```

## 1. Preconditions

- The worktree branch is rebased on `origin/{{base}}` and contains `{{TASK_DIR}}` with `Context & decisions` written.
  If it is missing, the manager skipped the rebase; message the manager `task {{id}}: worktree has no task folder, rebase needed` and stop.
- No developer session for this task is running (`claude agents`, or a live pane in `{{WINDOW}}`).
  If one is, do not launch another; message it `context ready, start` and finish.

## 2. Launch

Primary form (`--bg` + `attach`):

```
cd {{WORKTREE}}
claude --bg --agent developer -n {{DEV}} "task: {{TASK_DIR}}/task.md phase: {{TASK_DIR}}/phase_{{n}}.md"
tmux split-window -h -t {{PROJECT}}:{{WINDOW}} -c {{WORKTREE}}
tmux send-keys -t {{PROJECT}}:{{WINDOW}}.1 "claude attach {{bg-id}}" Enter
```

Fallback form (direct session):

```
tmux split-window -h -t {{PROJECT}}:{{WINDOW}} -c {{WORKTREE}}
tmux send-keys -t {{PROJECT}}:{{WINDOW}}.1 "claude --agent developer -n {{DEV}} 'task: {{TASK_DIR}}/task.md phase: {{TASK_DIR}}/phase_{{n}}.md'" Enter
```

Omit the `phase:` part when the task has no phases.
Left pane stays yours; right pane is the developer's.

## 3. Hand over

Wait until `ListAgents` shows `{{DEV}}`.
`SendMessage` to `{{DEV}}`: `context ready, start`.
Then wait for `round 1 ready`.

## Rules

- One developer per task or phase, never two at once.
- Never start the developer before `delegated, start` has arrived.
- Never paste task content in the launch message; the developer reads the folder.
