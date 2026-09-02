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
code review, the project's checks (small background checkers declared in docs/checks/), and for bugs the
replication steps. Sends findings to the om-developer or, when clean, asks it to document and then runs
publish-task. om-reviewer only; runs on every `round {{N}} ready, commit {{sha}}`.

Input: the om-developer's message `round {{N}} ready, commit {{sha}}`.
Output: either one findings message to the om-developer, or `round {{N}} clean, document` followed by `publish-task`.

You review the worktree at `{{sha}}`.
`WORKSPACE` and `ROOT_WT` as in `delegate-task`.
You never modify code.
Stop at the first failing step; do not continue the Pipeline on a red step, report it.
The project checks are the exception: they start before step 1 and are collected in step 5, so they run while you read.

## 0. Launch the project checks

Read every `{{ROOT_WT}}/docs/checks/*.md` (root worktree of the workspace).
List the files changed against `origin/{{base}}` per repo; a check applies when at least one changed file matches one of its `paths` globs (relative to the project root, repo folder first in multirepo).
For each applying check, launch it in the background, one process per check, output to `{{WORKSPACE}}/.checks/{{name}}.round-{{N}}.out`:

```
mkdir -p {{WORKSPACE}}/.checks
git -C {{WORKSPACE}}/{{repo}} diff origin/{{base}}...HEAD -- {{matching files}} \
  | (cd {{WORKSPACE}}/{{repo}} && claude -p --model {{model}} --allowedTools Read,Grep,Glob -- \
      "$(cat {{ROOT_WT}}/docs/checks/{{name}}.md)

Reference document:
$(cat {{ROOT_WT}}/{{reference}})

You are a read-only checker with a single rule, the one above.
The diff of this branch against its base is on stdin; the files are in the working directory if you need surrounding context.
Judge only the lines the diff adds or changes; pre-existing code is not yours to report.
Output exactly one of: the word clean, or one line per finding as {{file}}:{{line}}: {{what}} -> {{expected}}.
No prose, no summary, no fixes, no praise." \
      > {{WORKSPACE}}/.checks/{{name}}.round-{{N}}.out 2>&1 &)
```

The `--` before the prompt is required: the check file starts with a `---` frontmatter line and without it the CLI reads the prompt as an unknown option.
Do not wait for them; continue with step 1.
No `docs/checks/`, or no check whose `paths` match the diff: skip this step and step 5 reports `checks: n/a`.

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

## 5. Project checks

Wait for the processes of step 0 (`wait`), then read each `{{WORKSPACE}}/.checks/{{name}}.round-{{N}}.out`.
Triage every line against the code you read in step 4:

- Confirmed: it joins the round's findings, prefixed `[{{name}}]`.
- False positive or allowed by the check's own last line: drop it and record `Let pass: check {{name}}: {{what}}, {{why}}` for `Decisions`.
- Output that is neither `clean` nor finding lines (an error, prose): rerun that check once in the foreground; if it fails again, report `check {{name}} failed to run` in `Decisions` and continue.

Checkers are cheap and narrow; you are the judgment.
Never forward a checker's line you have not verified yourself.

## 6. Bugs only

Run `replication.md` steps end-to-end as a user would, on `{{sha}}`.
The observed behavior must now match Expected.
Record date, commit and result under `om-reviewer verification` in `replication.md` (workspace copy).
A fix that does not make the steps pass is a finding.

## 7. Outcome

Findings: one `SendMessage` to the om-developer:

```
round {{N}} findings: {{k}}
1. {{file}}:{{line}}: {{what}} -> {{expected}}
2. [{{check}}] {{file}}:{{line}}: {{what}} -> {{expected}}
3. ...
```

Then wait for `round {{N+1}} ready`.

No findings: `SendMessage` the om-developer `round {{N}} clean, document`, wait for `docs ready, commit {{sha}}`, then run `publish-task` on that commit; `publish-task` checks the docs before pushing.

## Rules

- Never fix anything yourself.
- Never soften a finding because the round count is high.
- Never run `verify-task` or any lint, typecheck or test command; audit `verify.log` every round, and treat a missing, stale or red block as a finding. The om-developer's log is the only evidence.
- Never publish before the om-developer's `docs ready` commit; documentation is checked by `publish-task`, not here.
- Never review before `round {{N}} ready, commit {{sha}}` arrives; never review a commit other than the one named.
- Never forward a check finding unverified, and never skip a check whose `paths` match the diff; checks run on every round, not only the first.
