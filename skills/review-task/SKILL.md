---
name: review-task
description: "Run the Pipeline on the developer's round and send findings or publish. Reviewer; on every \"round N ready\"."
argument-hint: "[ROUND]"
disable-model-invocation: false
---

# review-task

## Purpose

Run the Pipeline on the developer's latest round in the worktree: intent, rebase, verify-task on the final
commit, code review, documentation, and for bugs the replication steps. Sends findings to the developer or,
when clean, runs publish-task. Reviewer only; runs on every "round N ready".

Input: the developer's message `round {{N}} ready, commit {{sha}}`.
Output: either one findings message to the developer, or `publish-task`.

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

Run `verify-task` yourself on `{{sha}}`; it appends to `verify.log`.
Any red result is a finding with the failing command and the first relevant error lines.
Also open `verify.log` and check the developer's last entry is at `{{sha}}` and green; if the developer did not run it after its last change, that is a finding on its own.

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
When you let something questionable pass, note why; it goes to `Decisions` in the PR comment.

## 5. Documentation

For each module in `modules`:

- `docs/modules/{{module}}/*.md` touched by `document-task` have `updated` today and `source: {{id}}_{{title}}`.
- `trd.md` reflects new or changed endpoints; `database.md` reflects new tables, columns or invariants; `flows.md` if a complex flow changed.
- `ard.md` has an entry for every decision the developer took that `Approach` and `Context & decisions` did not already record.

Missing or stale docs are findings.

## 6. Bugs only

Run `replication.md` steps end-to-end as a user would, on `{{sha}}`.
The observed behavior must now match Expected.
Record date, commit and result under `Reviewer verification` in `replication.md` (worktree copy).
A fix that does not make the steps pass is a finding.

## 7. Outcome

Findings: one `SendMessage` to the developer:

```
round {{N}} findings: {{k}}
1. {{file}}:{{line}}: {{what}} -> {{expected}}
2. ...
```

Then wait for `round {{N+1}} ready`.

No findings: run `publish-task`.

## Rules

- Never fix anything yourself.
- Never soften a finding because the round count is high.
- Never trust the developer's report of lint, tests or docs; check.
- Never review before `round N ready` arrives; never review a commit other than the one named.
