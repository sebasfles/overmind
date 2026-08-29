---
name: consolidate-task
description: >-
  Take a created task from disk to consolidated. Creates its worktree and branch, opens its tmux window,
  launches the reviewer alone inside the worktree, relays the reviewer's questions to Sebastian until
  Context & decisions is written, then commits and pushes the task folder as planned with Sebastian's
  approval. Ends by asking whether to delegate now. Manager only; Sebastian invokes it.
argument-hint: "[TASK_FOLDER]"
disable-model-invocation: true
---

# consolidate-task

Input: `$ARGUMENTS[0]`, the absolute path of a task folder under `docs/tasks/` of the main checkout, uncommitted, with no worktree yet.
If the folder is missing or already has a worktree, stop and say so.

Output: worktree, branch, tmux window and a running reviewer; `Context & decisions` written in the main checkout copy; the task folder (plan plus decisions) committed and pushed as `planned`, the only docs commit of the task; and the question "¿delegar ahora?".

Read the frontmatter first: `id`, `title`, `type`, `phases`, `depends_on`.
Read `docs/TRD.md` for the base branch (fallback: the repository's default branch).
Set:

```
REPO      = main checkout root
TASK      = {{id}}_{{title}}
PREFIX    = feat | bugfix | docs | chore | refactor   (by type)
BRANCH    = {{PREFIX}}/{{TASK}}                or {{PREFIX}}/{{TASK}}-phase-1 if phases > 0
WORKTREE  = {{REPO}}/.claude/worktrees/{{TASK}}
WINDOW    = task-{{id}}
SESSION   = task-{{id}}-reviewer
PROJECT   = the tmux session you run in (same name as the project)
```

## 1. Fresh base

In the main checkout, on the base branch:

```
git pull --rebase origin {{base}}
```

The task folder is untracked at this point, so the pull is clean.
If the pull fails, stop and show the error; do not continue on a stale base.

## 2. Worktree and branch

```
git worktree add -b {{BRANCH}} {{WORKTREE}} origin/{{base}}
```

If the branch already exists on the remote, stop and ask Sebastian; do not reuse a branch you did not create.
The branch is born without the task folder; it receives it in `delegate-task`'s rebase, after the push below.

## 3. tmux window with the reviewer

Primary form (`--bg` + `attach`, to be validated in the pilot):

```
cd {{WORKTREE}}
claude --bg --agent reviewer -n {{SESSION}} "task: {{TASK_FOLDER}}"
tmux new-window -t {{PROJECT}} -n {{WINDOW}} -c {{WORKTREE}}
tmux send-keys -t {{PROJECT}}:{{WINDOW}} "claude attach {{bg-id}}" Enter
```

Fallback form (direct session):

```
tmux new-window -t {{PROJECT}} -n {{WINDOW}} -c {{WORKTREE}}
tmux send-keys -t {{PROJECT}}:{{WINDOW}} "claude --agent reviewer -n {{SESSION}} 'task: {{TASK_FOLDER}}'" Enter
```

Only the reviewer is launched here.
The developer is launched by the reviewer in `delegate-task`.
Record the reviewer's session id (from `claude --bg` output or `claude agents`) in your reply to Sebastian; `delegate-task` needs it if the session is stopped.

## 4. Relay the consolidation

The reviewer runs `analyze-task` on its own, reading the task folder at the main checkout path you gave it, and will `SendMessage` you its questions, batched.
It writes `Context & decisions` in that same main checkout copy; this is the only moment anyone writes there after `create-task`.
For each message:

1. Answer from `docs/` or the task folder yourself if the answer is already there.
2. Otherwise bring the questions to Sebastian, in his language, with your recommendation.
3. Send the answers back to `{{SESSION}}` as pointers and short text; never paste files.

Continue until the reviewer sends `consolidated`.
Do not start `delegate-task`, do not commit, and do not answer on Sebastian's behalf anything about Goal or Scope.

## 5. Commit and push `planned`

Show Sebastian, in a few lines, what `Context & decisions` says and any Scope or Acceptance adjustment the reviewer made.
Ask: "¿Pusheo la task a {{base}}?"
Only after his approval:

```
cd {{REPO}}
git pull --rebase origin {{base}}
git add docs/tasks/{{TASK}} .gitignore
git commit -m "docs(tasks): {{TASK}} planned"
git push origin {{base}}
```

This is the only docs commit the task will ever have; everything written afterwards travels in the task's PR.

If the push is rejected because the base branch is protected, say so and leave the commit local; Sebastian decides per project.
Never add anything outside `docs/tasks/{{TASK}}` and `.gitignore` to this commit.

## 6. Delegate now, or pause

Ask: "¿Delegar ahora?"

- Yes: invoke `delegate-task` with the task folder.
- No: pause without destroying anything.

```
claude stop {{bg-id}}                         # conversation kept; or Ctrl+C the direct session
tmux kill-window -t {{PROJECT}}:{{WINDOW}}
```

Worktree and branch stay.
The task shows as `consolidated` in `check-work`.
`delegate-task` will reopen the same session.

## Rules

- Never launch a developer.
- Never write inside the task folder; here only the reviewer writes, and only `Context & decisions`.
- Never commit before Sebastian approves.
- Never remove the worktree or the branch in this skill.
- One task per invocation.
