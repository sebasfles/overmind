---
name: clean-task
description: "Remove everything of a merged task: sessions, workspace, branches, tmux window. om-manager; Sebastian invokes it."
effort: low
argument-hint: "[TASK_ID_OR_FOLDER]"
---

# clean-task

## Purpose

Clean up a task whose PRs are all merged: pull the root's base branch, stop and remove the om-reviewer and
om-developer sessions, remove every worktree of the workspace and the workspace folder, delete local and remote
branches in every repo, close the tmux window. No commit; without a workspace the task derives as done.
om-manager only; Sebastian invokes it.

Input: a task id or folder.
Output: nothing left of the task but its folder on the root's base branch and the merged PRs.

## 1. Preconditions

`check-task` must say `merged`: every expected PR merged, every phase if any.
Anything else: stop and say what is open.

## 2. Pull the root

In the root checkout, on its base branch: `git pull --rebase origin {{base}}`.
This brings the final task folder and module docs that traveled in the root PR.

## 3. Sessions

Killing the tmux window does not stop a `--bg` session: it keeps running detached and recreates `{{WORKSPACE}}/.claude/` after step 4.
Derive every session of the task from its cwd; nothing stores the ids:

```
claude agents --all --json | jq -r '.[] | select(.cwd | startswith("{{WORKSPACE}}")) | .id'
```

For each id: `claude stop {{id}}`, then `claude rm {{id}}`.
If the answer is "background service may be restarting", wait a few seconds and run the same command again.
Rerun the listing; continue to step 4 only when it returns nothing.

Direct form (sessions launched without `--bg`): `claude project purge {{WORKSPACE}}` after step 4.

## 4. Worktrees, workspace, branches

For each worktree in the workspace (code repos and root):

```
git -C {{ROOT}}/{{repo}} worktree remove {{WORKSPACE}}/{{name}}
git -C {{ROOT}}/{{repo}} branch -D {{branch}}                  # and phase branches
git -C {{ROOT}}/{{repo}} push origin --delete {{branch}}       # if the merge did not delete it
```

Then `rmdir {{WORKSPACE}}` (it must be empty; if not, list what is left and stop).

## 5. tmux

`tmux kill-window -t {{PROJECT}}:task-{{id}}`

## 6. Report

One line: `{{id}}_{{title}} cleaned`.
Then suggest recycling this om-manager session (`prefix + R` in this pane): the finished task's context is dead weight, and `check-work` rebuilds the board from disk.

## Rules

- No commit.
- Never delete a branch with commits not in its base.
- Never touch another task's workspace or window.
- Sessions first, files second: a live session recreates what step 4 removes.
- Absolute paths in every command.
