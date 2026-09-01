---
name: execute-task
description: "Implement the task or apply findings, verify, one commit per round; document once on the clean signal. om-developer; on \"context ready, start\" and each message."
effort: high
disable-model-invocation: false
---

# execute-task

## Purpose

Implement the task (or the current phase) in the worktree, one round at a time: reproduce bugs first, rebase,
implement with tests, verify-task, squash to one commit, write om-developer notes, report "round N
ready" to the om-reviewer. Also applies the om-reviewer's findings on later rounds, and runs document-task once,
on the om-reviewer's clean signal. om-developer only; runs on "context ready, start", on every findings message,
and on "round N clean, document".

Input: `context ready, start` from the om-reviewer (round 1), or a findings message `round N findings: k ...` (round N+1), or `retakes: k findings ...` (reiteration), or `round N clean, document` (documentation).
Output: one squashed commit on the task branch in the worktree, `om-developer notes` updated, and the message `round {{N}} ready, commit {{sha}}` (or `docs ready, commit {{sha}}`) to the om-reviewer.

You work only in your worktree.
You never push.

## 0. Mode

- Round 1: implement.
- Round N+1: apply every finding, all of them, then re-verify.
- Reiteration: apply the findings the om-reviewer derived from `retakes.md`.
- Documentation: on `round {{N}} clean, document`, run only step 10.

The first three modes share the steps; only step 5 changes.

## 1. Read the task

`task.md` in full: Goal, Scope, Out of scope, Acceptance, Approach, Database, Infra, Design, Risks, `Context & decisions`.
The phase file if any.
`replication.md` if `type: bug`.
`retakes.md` if it exists.
Your own previous `om-developer notes`.

## 2. Read the project, in order

Stop when you have enough.

1. `CLAUDE.md` of the project.
2. `docs/TRD.md`: stack, commands, conventions. Load the matching `references/{{stack}}.md` of this skill if it exists.
3. For each module in `modules`: `docs/modules/{{module}}/README.md`, `trd.md`, `ard.md`, `database.md`, `flows.md` as needed.
4. The code you will touch, and the tests around it, to follow existing patterns instead of inventing new ones.
5. If `Design` has Figma links, pull design context through the Figma MCP before writing UI.

## 3. Bugs, round 1 only

Run `replication.md` steps end-to-end, as a user would, on the current branch.
Record date, commit and result under `om-developer confirmation`.
Reproduced: continue.
Not reproduced: stop, write what you observed, and `SendMessage` the om-reviewer `round 0: not reproduced, {{one line}}`. Do not change code.

## 4. Rebase

```
git fetch origin
git rebase origin/{{base}}
```

Resolve conflicts if any; if a conflict touches code you do not understand, ask the om-reviewer before resolving.

## 5. Implement

Round 1: build what Scope says, the way `Approach` and `Context & decisions` say, following the module's `trd.md` and `ard.md`.
Later rounds: apply each finding exactly as stated; if you disagree with one, apply it anyway and say why in `om-developer notes`, or ask the om-reviewer before touching it. Never skip a finding silently.

Tests are part of implementation:

- Every acceptance criterion has at least one test that fails without your change and passes with it.
- Bugs: a regression test that encodes `replication.md`.
- Use the project's existing test layout and helpers; do not introduce a new test framework or pattern.

Write code comments only when absolutely necessary, which is almost never: a non-obvious invariant or an external workaround the code cannot express.
Never comment what the code already says; explanation belongs in the module docs and the ARD, not in comments.

Do not widen the scope.
Adjacent work you notice goes to `om-developer notes` as deferred, not into the code.
Leave the code better than you found it only inside the files you already had to touch.

## 6. Verify

Run `verify-task`.
Fix until everything is green.
It appends to `verify.log`; the om-reviewer audits that log and never re-runs it, so your last block must be green at your final commit and cover every target the diff touches.

## 7. One commit

Squash all work of this round, task folder changes included, into one commit on top of the previous round's commit:

```
git add -A
git commit -m "{{type}}({{modules}}): {{what}}, round {{N}}"
```

Round 1 has exactly one commit on top of `origin/{{base}}`.
Never amend a previous round's commit; the om-reviewer references them.

## 8. om-developer notes

In `task.md` (or the phase file), under `om-developer notes`, add a `Round {{N}}` block: what you did, what you left pending, what you deferred, which findings you disagreed with and why, and every decision you took that `Approach` and `Context & decisions` did not already record.
Then amend that into the round's commit (`git commit --amend --no-edit`), so notes and code travel together.
Decisions recorded here become ARD entries in step 10; module docs are not touched during rounds.

## 9. Report

`SendMessage` to `om-{{id}}-reviewer`: `round {{N}} ready, commit {{sha}}`.
Then wait.
Do nothing until a new message arrives.

## 10. Documentation, once, on the clean signal

Runs only when the om-reviewer sends `round {{N}} clean, document`; never before.

Run `document-task` over the whole task's (or phase's) diff against `origin/{{base}}`.
Every decision recorded in `om-developer notes` that `Approach` and `Context & decisions` did not already record becomes an ARD entry.
One commit on top of the last round's commit: `docs({{modules}}): {{id}} task docs`.
Then `SendMessage` to `om-{{id}}-reviewer`: `docs ready, commit {{sha}}`, and wait.
If the om-reviewer answers `docs findings: ...`, fix only the docs, commit on top, and report `docs ready` again.

## Questions

Ask the om-reviewer before writing code when two readings of Scope or a data shape lead to different implementations, or when the code contradicts `ard.md`.
One message, batched, with your recommendation.
Never ask what the docs or the code already answer.
Never ask the om-manager or Sebastian.

## Rules

- Never push, never open a PR, never touch the remote.
- Never touch the root checkout or other worktrees.
- Never skip `verify-task`, however small the change.
- Never run `document-task` before the om-reviewer's `round {{N}} clean, document`; docs are written once, when the code is final.
- Never change files in the task folder you do not own: `Context & decisions`, `Result`, `retakes.md`.
