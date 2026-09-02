---
name: next-phase
description: "Close phase N and start phase N+1 with a fresh om-developer. om-reviewer; on \"phase N merged, continue\"."
effort: medium
argument-hint: "[PHASE_N]"
disable-model-invocation: false
---

# next-phase

## Purpose

Move a phased task to its next phase after the om-manager reports the current phase merged: write Result, stop
the current om-developer, create the next phase branch from origin/base in the same worktree, and run start-task.
om-reviewer only; runs on "phase N merged, continue".

Input: the om-manager's message `phase {{N}} merged, continue`.
Output: the worktree on `{{prefix}}/{{id}}_{{title}}-phase-{{N+1}}`, a fresh om-developer started on it.

## 1. Confirm the merge

`gh pr view --json state,mergedAt` for phase N's PR must say merged.
If not, message the om-manager `phase {{N}} PR is not merged` and stop.

## 2. Stop the om-developer of phase N

```
claude stop {{dev-id}} && claude rm {{dev-id}}     # --bg form
```

`{{dev-id}}` is the `id` of the entry named `{{DEV}}` in `claude agents --all --json`; retry if the answer is "background service may be restarting".

Direct form: close its pane (`tmux kill-pane -t {{PROJECT}}:{{WINDOW}}.1`).
One om-developer per phase; the next one starts with a clean context.

## 3. Switch the worktree to the next phase

```
git fetch origin
git checkout -b {{prefix}}/{{id}}_{{title}}-phase-{{N+1}} origin/{{base}}
```

`origin/{{base}}` already contains phase N's code and the task folder as it was at that merge.

## 4. Write `Result` of phase N

In `docs/tasks/{{id}}_{{title}}/phase_{{N}}.md` (workspace copy, now on the new branch), fill `Result`: outcome, deviations from the plan, debt created, anything phase N+1 must know.
It travels in phase N+1's PR.
If N+1 is the last phase, remember its own `Result` will have no PR to travel in: put it in that PR's comment instead.

## 5. Start phase N+1

Re-read `Context & decisions` and `phase_{{N+1}}.md` Scope and Acceptance.
Run `start-task` with phase `{{N+1}}`.

## Rules

- Never start phase N+1 before phase N's PR is merged.
- Never reuse the previous om-developer.
- Never rebase phase N's branch onto anything; it is done.
