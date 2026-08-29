---
name: create-task
description: Write an approved plan as a task folder under docs/tasks/. Manager; runs after plan-task is approved.
argument-hint: "[PLAN_FILE]"
disable-model-invocation: false
---

# create-task

## Purpose

Write an approved plan to disk as a task folder under docs/tasks/{{id}}_{{title}}/ with task.md,
replication.md for bugs and one phase_N.md per phase, uncommitted. Then offer consolidate-task. Manager only;
normally invoked by plan-task after approval.

Input: the plan approved in this conversation by `plan-task`, or `$ARGUMENTS[0]`, a plan file (normally `docs/tasks/_drafts/{{title}}.md`).
If there is no approved plan in context and no file, stop and run `plan-task` first.

Output: `docs/tasks/{{id}}_{{title}}/` in the main checkout, uncommitted, and the question "¿la consolido ahora?".

## 1. Locate `docs/tasks/`

It lives in the main checkout of the repository, not in a worktree.
It is tracked by git. Before delegation a task folder is written here; after delegation only in the task's worktree copy.
Only the manager commits here, once per task, at the end of `consolidate-task`.
`docs/tasks/_drafts/` is not tracked.
If the folder does not exist, create it and add `docs/tasks/_drafts/` to `.gitignore`.

## 2. Assign id and title

- `id`: four digits, sequential per project. Read every folder name in `docs/tasks/`, take the highest id, add one. Start at `0001`.
- `title`: snake_case, two to four words, from the Goal. Example: `badge_wall`, `fix_login_redirect`.
- `repos`: from the plan's Approach; in single and mono always `["."]`. In multirepo, if the plan does not say, ask; never default to all.
- Folder: `docs/tasks/{{id}}_{{title}}/`. Example: `docs/tasks/0142_badge_wall/`.

## 3. Write `task.md`

Read `templates/task.md` and fill it from the plan.
Frontmatter:

```yaml
id: "0142"
title: badge_wall
type: feature            # feature | bug | docs | chore | refactor
branch: feat/0142_badge_wall     # empty when the task has phases
modules: [billing, notifications]   # primary first
repos: [diy-platform, diy-infra]     # repos the task touches; ["."] in single and mono
phases: 0                # number of phases, 0 if none
depends_on: []           # ids of tasks that must be done first
ticket:                  # external reference, optional
created: 2026-08-28
updated: 2026-08-28
```

Branch prefix by type: `feat/`, `bugfix/`, `docs/`, `chore/`, `refactor/`.

Body sections, in this order, from the plan: Goal, Scope, Out of scope, Acceptance, Approach, Database, Infra, Design, Risks, Depends on.
Then two empty sections the reviewer and the developer will own: `Context & decisions` and `Developer notes`.
Do not fill those two.

## 4. Write `replication.md` for bugs

Only when `type: bug`.
Read `templates/replication.md` and fill it from the plan's Replication section: preconditions, exact steps, expected, observed, environment, evidence.
The developer runs these steps before touching code; the reviewer verifies the fix against them.

## 5. Write phases, if any

For each phase, read `templates/phase.md` and write `phase_{{n}}.md`:

```yaml
phase: 1
branch: feat/0142_badge_wall-phase-1
updated: 2026-08-28
```

Body: Scope, Acceptance, then empty `Developer notes` and `Result`.
When there are phases, `task.md` keeps `branch` empty and `phases` set to the count.

## 6. Check overlaps

Read the frontmatter of every other task folder (ignore `_drafts/`) and derive its state as `check-task` does.
If any task not `done` touches one of the same modules, mention it and ask Sebastian whether it belongs in `depends_on`.

## 7. Delete the draft

If the plan came from `docs/tasks/_drafts/{{title}}.md`, delete that file now; the task folder is the source from here on.

## 8. Confirm and offer consolidation

Print the folder path and the frontmatter in a few lines.
Nothing is committed yet; that happens at the end of `consolidate-task`, with Sebastian's approval.
Ask: "¿La consolido con el reviewer ahora?"
If yes, invoke `consolidate-task` with the folder path.
If no, stop; the folder stays uncommitted in the main checkout and shows up in `check-work` as `planned`.
There is no status field; `check-task` derives every state.

## Rules

- Never write anything outside `docs/tasks/{{id}}_{{title}}/`, `docs/tasks/_drafts/` and `.gitignore`.
- Never commit; `consolidate-task` does.
- Never change an existing task folder; that is `reiterate-task` or the reviewer's job.
- Copy the plan faithfully; do not add, drop or reinterpret anything Sebastian approved.
- Files are in English.
