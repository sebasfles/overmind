---
name: consolidate-task
description: Create the task workspace, launch the om-reviewer and commit the task as planned. om-manager; Sebastian invokes it after create-task.
effort: high
argument-hint: "[TASK_FOLDER]"
disable-model-invocation: true
---

# consolidate-task

## Purpose

Take a created task from disk to consolidated. Creates the task workspace (one worktree per repo the task
touches, plus the root repo's worktree in multirepo projects), bootstraps each worktree, opens the tmux
window, launches the om-reviewer alone inside the workspace, relays the om-reviewer's questions to Sebastian until
Context & decisions is written, then commits and pushes the task folder as planned with Sebastian's approval.
Ends by asking whether to delegate now. om-manager only; Sebastian invokes it.

Input: `$ARGUMENTS[0]`, the absolute path of a task folder under `{{ROOT}}/docs/tasks/`, uncommitted, with no workspace yet.
If the folder is missing or a workspace already exists, stop and say so.

Output: workspace with bootstrapped worktrees, tmux window, running om-reviewer; `Context & decisions` written in the root checkout copy; the task folder (plan plus decisions) committed and pushed as `planned`, the only docs commit of the task on the root's base branch; and the question "¿delegar ahora?".

Read `task.md` frontmatter: `id`, `title`, `type`, `phases`, `depends_on`, `repos`.
Read `docs/TRD.md`, section Components, for the repos and each `base_branch`; the root is your cwd.
If the TRD has no Components table yet, use `git -C {{repo}} symbolic-ref refs/remotes/origin/HEAD` per repo and tell Sebastian the TRD should declare it.
Set:

```
ROOT       = project root (your cwd)
TASK       = {{id}}_{{title}}
PREFIX     = feat | bugfix | docs | chore | refactor   (by type)
BRANCH     = {{PREFIX}}/{{TASK}}                or {{PREFIX}}/{{TASK}}-phase-1 if phases > 0
WORKSPACE  = {{ROOT}}/.workspaces/{{TASK}}
REPOS      = task.md repos (in single and mono: ["."])
WINDOW     = task-{{id}}
SESSION    = om-{{id}}-reviewer
PROJECT    = the tmux session you run in (same name as the project)
```

## 1. Fresh bases

For each repo in `REPOS` (and the root in multirepo): `git -C {{repo}} fetch origin` and, in the root checkout on its base branch, `git pull --rebase origin {{base}}`.
The task folder is untracked, so the root pull is clean.
If a pull fails, stop and show the error.

## 2. Workspace

```
mkdir -p {{WORKSPACE}}
for repo in REPOS:
  git -C {{ROOT}}/{{repo}} worktree add -b {{BRANCH}} {{WORKSPACE}}/{{name}} origin/{{base of repo}}
if multirepo:
  git -C {{ROOT}} worktree add -b {{BRANCH}} {{WORKSPACE}}/{{root name}} origin/{{root base}}
```

`{{name}}` is the repo folder name; for `.` it is the root folder name.
If a branch already exists on a remote, stop and ask Sebastian; never reuse a branch you did not create.
Branches are born without the task folder (it is uncommitted); the root worktree receives it in `delegate-task`'s rebase.

## 3. Bootstrap each worktree

From the TRD section of that repo or app:

- Copy or symlink every file listed under `Workspace files` (typically `.env*`) from the main clone into the worktree.
- Run the install command (for example `pnpm install --frozen-lockfile`).

If the TRD has no such section, copy `.env*` files that exist in the main clone and run the stack's default install; then tell Sebastian the TRD should declare them.

## 4. tmux window with the om-reviewer

Primary form (`--bg` + `attach`, to be validated in the pilot):

```
cd {{WORKSPACE}}
claude --bg --allow-dangerously-skip-permissions --agent om-reviewer -n {{SESSION}} "task: {{TASK_FOLDER}} root: {{ROOT}} workspace: {{WORKSPACE}}"
tmux new-window -t {{PROJECT}} -n {{WINDOW}} -c {{WORKSPACE}}
tmux send-keys -t {{PROJECT}}:{{WINDOW}} "claude attach {{bg-id}}" Enter
```

Fallback form: same `tmux new-window`, then `claude --agent om-reviewer -n {{SESSION}} '...'` in the pane.

Only the om-reviewer is launched here.
Record the om-reviewer's session id in your reply; `delegate-task` needs it if the session is stopped.

## 5. Relay the consolidation

The om-reviewer runs `analyze-task`, reads the task folder at `{{TASK_FOLDER}}` (root checkout) and writes `Context & decisions` there.
For each message from it: answer from `docs/` yourself if the answer is there; otherwise bring the questions to Sebastian with your recommendation; send the answers back as pointers.
Continue until the om-reviewer sends `consolidated`.
Do not commit, do not delegate, do not answer anything about Goal or Scope on Sebastian's behalf.

## 6. Commit and push `planned`

Show Sebastian `Context & decisions` and any Scope or Acceptance adjustment in a few lines.
Ask: "¿Pusheo la task a {{root base}}?"
Only after approval, in the root checkout:

```
git pull --rebase origin {{root base}}
git add docs/tasks/{{TASK}} .gitignore
git commit -m "docs(tasks): {{TASK}} planned"
git push origin {{root base}}
```

Only docs commit the task will ever have on the base branch; everything afterwards travels in the task's PRs.
If the push is rejected by branch protection, say so and leave it local.

## 7. Delegate now, or pause

Ask: "¿Delegar ahora?"

- Yes: invoke `delegate-task`.
- No: `claude stop {{bg-id}}` (conversation kept) and `tmux kill-window -t {{PROJECT}}:{{WINDOW}}`. The workspace stays; the task shows as `consolidated`.

## Rules

- Never launch an om-developer.
- Never write inside the task folder; here only the om-reviewer writes, and only `Context & decisions`.
- Never commit before Sebastian approves.
- Never remove worktrees or branches in this skill.
- Absolute paths in every command.
