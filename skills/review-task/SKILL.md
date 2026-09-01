---
name: review-task
description: "Run the Pipeline on the om-developer's round and send findings or publish. om-reviewer; on every \"round N ready\"."
effort: xhigh
argument-hint: "[ROUND]"
disable-model-invocation: false
---

# review-task

## Purpose

Run the Pipeline on the om-developer's latest round in the worktree: intent, rebase, audit of verify.log,
code review, and for bugs the replication steps. Sends findings to the om-developer or, when clean, asks it
to document and then runs publish-task. om-reviewer only; runs on every `round {{N}} ready, commit {{sha}}`.

Input: the om-developer's message `round {{N}} ready, commit {{sha}}`.
Output: either one findings message to the om-developer, or `round {{N}} clean, document` followed by `publish-task`.

You review the worktree at `{{sha}}`.
You never modify code.
Stop at the first failing step; do not continue the Pipeline on a red step, report it.

## 1. Intent

Read the diff against `origin/{{base}}` (`git diff origin/{{base}}...HEAD --stat`, then the files).
Check against `task.md` Goal, Scope, Out of scope and `Context & decisions` (and the phase file if any):

- Everything in Scope is addressed.
- Nothing from Out of scope is touched.
- No unrelated refactors, renames or "improvements" outside the files the task had to touch.

A scope deviation is a finding, even if the extra work is good.

## 2. Rebase

`git merge-base --is-ancestor origin/{{base}} HEAD` after `git fetch origin`.
If false, finding: `rebase on origin/{{base}} required`.

## 3. Verify

Audit `verify.log`; never run lint, typecheck or tests yourself.
The om-developer's last block must be at `{{sha}}`, green on every step, and cover every target the diff touches; anything missing, stale or red is a finding on its own.

## 4. Review the code

Read every changed file in full, not only the hunks.
Judge in this order:

1. Correctness: logic errors, missing error paths, race conditions, wrong status codes, unvalidated input, broken invariants listed in `database.md`.
2. Security and data: authorization gaps, secrets, injection, data loss paths, migrations that are not reversible when the project expects them to be.
3. Acceptance: every criterion has a test that would fail without the change. Run one or two of the new tests in isolation if in doubt.
4. Architecture: adherence to the module's `trd.md` and `ard.md`, layer boundaries, existing patterns and conventions of the codebase.
5. Performance at the project's scale: N+1, unbounded queries, missing indexes named in `Database`.

Each finding: `{{file}}:{{line}}`, what is wrong, why it matters, what is expected.
Describe the defect in the code written for the task, not how the task should have been solved.
Style nits the linter does not catch: report only if they hide a real problem.
A code comment that restates what the code says is a finding; a comment is justified only by what the code cannot express.
When you let something questionable pass, note why; it goes to `Decisions` in the PR comment.

## 5. Bugs only

Run `replication.md` steps end-to-end as a user would, on `{{sha}}`.
The observed behavior must now match Expected.
Record date, commit and result under `om-reviewer verification` in `replication.md` (workspace copy).
A fix that does not make the steps pass is a finding.

## 6. Outcome

Findings: one `SendMessage` to the om-developer:

```
round {{N}} findings: {{k}}
1. {{file}}:{{line}}: {{what}} -> {{expected}}
2. ...
```

Then wait for `round {{N+1}} ready`.

No findings: `SendMessage` the om-developer `round {{N}} clean, document`, wait for `docs ready, commit {{sha}}`, then run `publish-task` on that commit; `publish-task` checks the docs before pushing.

## Rules

- Never fix anything yourself.
- Never soften a finding because the round count is high.
- Never run `verify-task` or any lint, typecheck or test command; audit `verify.log` every round, and treat a missing, stale or red block as a finding. The om-developer's log is the only evidence.
- Never publish before the om-developer's `docs ready` commit; documentation is checked by `publish-task`, not here.
- Never review before `round {{N}} ready, commit {{sha}}` arrives; never review a commit other than the one named.
