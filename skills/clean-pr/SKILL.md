---
name: clean-pr
description: "Remove everything of a merged or closed PR review: session, worktree, branch, tmux window. om-manager; Sebastian invokes it."
effort: low
argument-hint: "[PR_NUMBER]"
---

# clean-pr

## Purpose

Clean up a PR review whose PR is merged or closed: stop and remove the om-pr-reviewer session, remove the worktree and the workspace folder, delete the local branch, close the tmux window.
The report copy in `pr-reviews/` stays.
om-manager only; Sebastian invokes it, or `clean-work` does for every merged one.

Input: a PR number.
Output: nothing left of the review but its copy in `pr-reviews/`.

`ROOT` is the project root; `WORKSPACE = {{ROOT}}/.workspaces/pr-{{n}}`; `SESSION = om-pr-{{n}}-reviewer`; `WINDOW = pr-{{n}}`.

## 1. Preconditions

`gh -R {{owner/repo}} pr view {{n}} --json state` is `MERGED` or `CLOSED`; `{{owner/repo}}` is the repo of the worktree under `WORKSPACE`.
Anything else: stop and say the PR is still open.

## 2. Session

```
claude agents --all --json | jq -r '.[] | select(.name == "{{SESSION}}" or (.cwd | startswith("{{WORKSPACE}}"))) | .id'
```

For each id: `claude stop {{id}}`, then `claude rm {{id}}`.
If the answer is "background service may be restarting", wait a few seconds and run the same command again.
Rerun the listing; continue only when it returns nothing.

## 3. Worktree, workspace, branch

```
git -C {{clone}} worktree remove {{WORKSPACE}}/{{repo}}
git -C {{clone}} branch -D pr-{{n}}
```

`{{clone}}` is the repo itself in single and mono, and the code clone of the worktree in multirepo.
Then `rmdir {{WORKSPACE}}`; if anything else is left, list it and stop, it was not created by the method.
No remote branch to delete: `pr-{{n}}` only ever existed locally.

## 4. tmux

`tmux kill-window -t {{PROJECT}}:{{WINDOW}}` if it exists.

## 5. Report

One line: `PR #{{n}} cleaned`.

## Rules

- Never clean an open PR.
- Never touch `pr-reviews/`.
- Never touch another PR's or a task's workspace or window.
- Sessions first, files second: a live session recreates what step 3 removes.
- Absolute paths in every command.
