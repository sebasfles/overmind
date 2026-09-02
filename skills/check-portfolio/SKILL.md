---
name: check-portfolio
description: The board across all active projects, todos and notifications. overmind; at session start and when Sebastian asks.
effort: low
disable-model-invocation: false
---

# check-portfolio

## Purpose

The board across all active projects: for each, the tasks that wait for Sebastian, what is moving, what is
waiting, plus doc drift, leftovers and open todos. One line per item. overmind only; runs on start via
initialPrompt and whenever Sebastian asks.

Input: none.
Output: one screen.

## 1. Per active project in `portfolio/projects.yaml`

Derive task states exactly as the om-manager's `check-task` does, but from here: read `{{path}}/docs/tasks/*/task.md` frontmatter, `git -C {{path}} worktree list`, commits ahead of the base per branch, `claude agents --json` for om-developer sessions, `gh -C {{path}} pr list --state all` (a `gh` failure prints as `gh failed`, never as zero PRs).
Do not message any om-manager.

Also:

- Doc drift: modules whose code changed after their docs' `updated` (`git log -1 --format=%cs -- {{module path}}` newer than the frontmatter). Count only.
- Leftovers: orphan worktrees, `merged` tasks not cleaned.
- om-manager alive: a `om-{{name}}-manager` session in `claude agents --json` or a tmux session `{{tmux}}`.

## 2. Print

Projects ordered by what needs Sebastian first.

```
diy        in_review 2   in_progress 1   consolidated 1   planned 3   drift 2 modules   om-manager: up
bseen      merged 1 (clean)   in_progress 2                          om-manager: down
auvral     planned 1                                                  drift 5 modules   om-manager: down

waiting for you
  diy    0142 badge_wall     PR #57
  diy    0139 fix_login      PR #55
  bseen  0021 export         merged, run clean-work

todos (open 4)
  - decide on Stripe vs Adyen for diy
  - call the bseen client about scope
  ...

notifications since last check
  diy: PR #57 for task 0142 is ready
```

Paused projects: one line at the end with the count.

## 3. Route

If Sebastian picks a project, answer `resume-project {{name}}` and run it if he confirms.

## Rules

- Read-only.
- Never longer than one screen; collapse `planned` and `done` to counts.
- Never open a task file beyond its frontmatter.
