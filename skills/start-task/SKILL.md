---
name: start-task
description: "Launch the om-developer for the current task or phase in the workspace. om-reviewer; on \"delegated, start\" and inside next-phase."
effort: low
argument-hint: "[TASK_FOLDER] [PHASE_N]"
disable-model-invocation: false
---

# start-task

## Purpose

Launch the om-developer for the current task or phase in the right pane of the task's tmux window, with cwd in
the worktree, and send it "context ready, start". om-reviewer only; runs on "delegated, start" from the om-manager
and inside next-phase.

Input: the task folder (root workspace copy inside the workspace) and, if the task has phases, the phase number.
Output: a running `om-{{id}}-developer` (or `om-{{id}}-developer-phase-{{n}}`) that has received `context ready, start`.

Set:

```
TASK      = {{id}}_{{title}}
WORKSPACE = your cwd ({{ROOT}}/.workspaces/{{TASK}})
ROOT_WT   = the root's worktree inside it (the code worktree in single and mono)
WINDOW    = task-{{id}}
DEV       = om-{{id}}-developer            or om-{{id}}-developer-phase-{{n}}
PROJECT   = the tmux session this window belongs to
TASK_DIR  = {{ROOT_WT}}/docs/tasks/{{TASK}}
```

## 1. Preconditions

- `{{TASK_DIR}}` exists with `Context & decisions` written (the root worktree was rebased by `delegate-task`).
  If it is missing, the om-manager skipped the rebase; message the om-manager `task {{id}}: worktree has no task folder, rebase needed` and stop.
- No om-developer session for this task is running (`claude agents --json`, or a live pane in `{{WINDOW}}`).
  If one is, do not launch another; message it `context ready, start` and finish.

## 2. Launch

Model by task type: read `type` from `task.md`; for `docs` and `chore` add `--model sonnet` to the `claude` launch command; `feature`, `bug` and `refactor` keep the agent's default.

Primary form (`--bg` + `attach`):

```
cd {{WORKSPACE}}
claude --bg --allow-dangerously-skip-permissions --agent om-developer -n {{DEV}} "task: {{TASK_DIR}}/task.md phase: {{TASK_DIR}}/phase_{{n}}.md workspace: {{WORKSPACE}}"
tmux split-window -h -t {{PROJECT}}:{{WINDOW}} -c {{WORKSPACE}}
tmux send-keys -t {{PROJECT}}:{{WINDOW}}.1 "claude attach {{bg-id}}" Enter
```

Fallback form (direct session):

```
tmux split-window -h -t {{PROJECT}}:{{WINDOW}} -c {{WORKSPACE}}
tmux send-keys -t {{PROJECT}}:{{WINDOW}}.1 "claude --agent om-developer -n {{DEV}} 'task: {{TASK_DIR}}/task.md phase: {{TASK_DIR}}/phase_{{n}}.md'" Enter
```

Run each launch line exactly as written, absolute paths filled in, one Bash call per line, `cd` and `claude` joined only by `&&`; nothing before `cd` or `tmux` (no `export`, `VAR=` or shell function), or the command matches no allow rule and goes to the classifier.
Omit the `phase:` part when the task has no phases.
Left pane stays yours; right pane is the om-developer's.

## 3. Hand over

Wait until `ListAgents` shows `{{DEV}}`.
`SendMessage` to `{{DEV}}`: `context ready, start`.
Then wait for `round 1 ready`.

## Rules

- One om-developer per task or phase, never two at once.
- Never start the om-developer before `delegated, start` has arrived.
- Never paste task content in the launch message; the om-developer reads the folder.
