---
name: analyze-task
description: Read the task, ask the om-manager once, write Context & decisions. om-reviewer; runs automatically at session start and on retakes.
effort: xhigh
argument-hint: "[TASK_FOLDER]"
disable-model-invocation: false
---

# analyze-task

## Purpose

om-reviewer's first action. Reads the task folder, the module docs and the code the task will touch, batches its
doubts to the om-manager once, writes Context & decisions in the main checkout copy, and reports "consolidated".
Idempotent: if Context & decisions is already written it does not ask again. Also incorporates new retakes
when reiterated. om-reviewer only; runs automatically.

Input: the task folder path from your first message (`task: {{path}}`), an absolute path in the main checkout.
Output: `Context & decisions` written in that copy, and the message `consolidated` to the om-manager.
Then wait for `delegated, start`.

## 0. Which mode

- `Context & decisions` empty and no `retakes.md`: consolidation. Steps 1 to 5.
- `Context & decisions` written and no new retakes: relaunch after a lost session. Steps 1 and 2, then confirm to the om-manager `consolidated (context already on disk)` and stop.
- `retakes.md` has entries newer than your last round: reiteration. Steps 1, 2 and 6.

## 1. Read the task

`task.md` whole, `phase_N.md` if any, `replication.md` if any, `retakes.md` if any.
Note `type`, `modules`, `phases`, `depends_on`.

## 2. Read the project, in order

Stop as soon as you have enough.

1. `CLAUDE.md` of the project.
2. `docs/TRD.md`: stack, commands, base branch, how the API spec is generated. You need this to run `verify-task` later.
3. `docs/ARD.md`: decisions and debt that constrain the approach.
4. For each module in `modules`: `docs/modules/{{module}}/README.md`, then `trd.md`, `ard.md`, `database.md`, `flows.md` as the task requires.
5. The code the task will touch, enough to judge whether `Approach` holds and to spot what the plan missed.

## 3. Form your doubts

Look for, in this order:

- Ambiguity: two readings of Scope or Acceptance that lead to different code.
- Conflicts: `Approach` vs `ard.md` or `trd.md` of the module; `Database` vs the real schema.
- Gaps: acceptance criteria with no observable check; error paths not mentioned; migrations implied but not listed.
- Size: does this fit one PR Sebastian can review in one sitting? If clearly not, propose phases (rarely).
- Bugs: is `replication.md` executable as written, with real preconditions and evidence?

Do not ask what the docs or the code already answer.
Do not ask about implementation details you can decide yourself later.

## 4. One batched message to the om-manager

`SendMessage` to `om-{{project}}-manager`, once, with all your doubts grouped by topic, each with your recommendation.
Pointers, not content: cite `task.md#Acceptance 3` or `docs/modules/billing/ard.md`, do not paste them.
Wait for the answers.
If the answers open new doubts, one more batch is acceptable; a third is not, decide yourself and record it.

## 5. Write `Context & decisions`

In the main checkout copy of `task.md` (the path you were given), fill the section with:

- Decisions taken in the consolidation, each with its reason and who decided (Sebastian, om-manager, you).
- Adjustments to Scope, Acceptance or phases, marked as such, with why. Never change Goal.
- Constraints from `ard.md` and `trd.md` the om-developer must respect, cited by path.
- What you will check in `review-task` beyond the Pipeline, if anything specific.
- For bugs: any correction to `replication.md` preconditions or steps.

Keep it under 40 lines.
Then `SendMessage` to the om-manager: `consolidated`.
Do not touch any other file.
Do not commit; the om-manager does.

## 6. Reiteration

Read the new entries in `retakes.md` (worktree copy) and the PR comments they refer to.
Translate them into concrete findings for the om-developer: `file:line` or acceptance criterion, what Sebastian wants, what is expected now.
Append a dated `Reiteration {{n}}` block to `Context & decisions` in the worktree copy with those findings.
`SendMessage` to the om-developer: `retakes: {{k}} findings, see Context & decisions, round {{n}}`.
Then wait for `round {{n}} ready`.

## Rules

- Before delegation you write only `Context & decisions`, only in the main checkout copy.
- After delegation you write only in the worktree copy.
- Never write code, never change Goal, never talk to Sebastian directly.
