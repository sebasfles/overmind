---
name: check-portfolio
description: The board across all active projects, todos and notifications. Overmind; at session start and when Sebastian asks.
disable-model-invocation: false
---

# check-portfolio

## Purpose

The board across all active projects: for each, the tasks that wait for Sebastian, what is moving, what is
waiting, plus doc drift, leftovers and open todos. One line per item. Overmind only; runs on start via
initialPrompt and whenever Sebastian asks.

Input: none.
Output: one screen.

## 1. Per active project in `portfolio/projects.yaml`

Derive task states exactly as the manager's `check-task` does, but from here: read `{{path}}/docs/tasks/*/task.md` frontmatter, `git -C {{path}} worktree list`, commits ahead of the base per branch, `claude agents` for developer sessions, `gh -C {{path}} pr list --state all`.
Do not message any manager.

Also:

- Doc drift: modules whose code changed after their docs' `updated` (`git log -1 --format=%cs -- {{module path}}` newer than the frontmatter). Count only.
- Leftovers: orphan worktrees, `merged` tasks not cleaned.
- Manager alive: a `{{name}}-manager` session in `claude agents` or a tmux session `{{tmux}}`.

## 2. Print

Projects ordered by what needs Sebastian first.

```
diy        in_review 2   in_progress 1   consolidated 1   planned 3   drift 2 modules   manager: up
bseen      merged 1 (clean)   in_progress 2                          manager: down
auvral     planned 1                                                  drift 5 modules   manager: down

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
