---
name: clean-task
description: >-
  Clean up a task whose PR (or last phase PR) is merged: pull the base branch, stop and remove the
  reviewer and developer sessions, remove the worktree, delete local and remote branches, close the
  tmux window. No commit; without a worktree the task derives as done. Manager only; Sebastian invokes it.
argument-hint: "[TASK_ID_OR_FOLDER]"
disable-model-invocation: true
---

# clean-task

Input: a task id or folder.
Output: nothing left of the task but its folder in the base branch and the merged PRs.

## 1. Preconditions

Run `check-task`.
The state must be `merged`; for phased tasks, every phase PR must be merged.
Anything else: stop and say what is still open.
Never clean a task with an open PR or unmerged commits.

## 2. Pull

In the main checkout, on the base branch:

```
git pull --rebase origin {{base}}
```

This brings the final task folder (notes, results, logs) that traveled in the PR.

## 3. Sessions

`--bg` form:

```
claude stop {{reviewer-id}} ; claude rm {{reviewer-id}}
claude stop {{developer-id}} ; claude rm {{developer-id}}     # one per phase if any remain
```

Direct form: `claude project purge {{WORKTREE}}` after step 4.

## 4. Worktree and branches

```
git worktree remove {{WORKTREE}}            # --force only if it holds nothing but the merged branch
git branch -D {{branch}}                    # and every phase branch
git push origin --delete {{branch}}         # if the PR merge did not already delete it
```

## 5. tmux

```
tmux kill-window -t {{PROJECT}}:task-{{id}}
```

## 6. Report

One line: `{{id}}_{{title}} cleaned`.
`check-task` now derives `done`.

## Rules

- No commit; the folder's final state arrived with the merge.
- Never delete a branch with commits not in the base branch.
- Never touch another task's worktree or window.
