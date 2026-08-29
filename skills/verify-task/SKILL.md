---
name: verify-task
description: >-
  Run the project's lint, typecheck and tests, one command at a time and with --runInBand, following
  the commands declared in docs/TRD.md, and append the result to the task's verify.log with the commit
  it ran on. Shared by developer and reviewer. Never fixes anything.
disable-model-invocation: false
---

# verify-task

Input: the worktree at its current commit.
Output: green or red per step, and one appended block in `docs/tasks/{{id}}_{{title}}/verify.log` (worktree copy).

## 1. Commands

Read `docs/TRD.md`, section on verification, for the exact commands and their order.
Fallback when the TRD does not declare them: `references/{{stack}}.md` of this skill, chosen by the stack the TRD names.
If neither exists, stop and report `verify-task: no commands declared in TRD`; do not guess.

Skip tests for `type: docs`; run only Markdown lint if the project has one.

## 2. Run, one at a time

In this order, each command alone, waiting for it to finish before the next:

1. lint
2. typecheck (if the stack has one)
3. unit tests, with `--runInBand` or the stack's equivalent serial flag
4. integration or e2e tests, with `--runInBand`, if the TRD lists them for this project

Never run test suites in parallel; the machine runs several tasks at once and memory is the constraint.
Never pass `--watch`.
Capture exit code and the last relevant lines of output per step.

## 3. Log

Append to `verify.log`:

```
## {{ISO timestamp}} by {{developer | reviewer}} on {{sha}}
- lint: {{pass | fail}} ({{command}})
- typecheck: {{pass | fail | n/a}} ({{command}})
- unit: {{pass | fail}} ({{n}} tests, {{command}})
- e2e: {{pass | fail | n/a}} ({{command}})
{{first relevant error lines of the first failing step, if any}}
```

Never rewrite earlier blocks.
The reviewer reads the last block by the developer and expects it to be at the developer's final commit and fully green.

## 4. Report

Return the per-step result to the caller.
Red: stop at the first failing step and report it; the caller decides (developer fixes, reviewer files a finding).

## Rules

- Never modify code, tests or configuration.
- Never skip a step the TRD declares.
- Never parallelize.
