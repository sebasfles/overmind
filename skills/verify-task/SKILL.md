---
name: verify-task
description: Run lint, typecheck and tests per verification target, serially, and log to verify.log. om-developer before every round; om-reviewer at the publish gate.
effort: low
disable-model-invocation: false
---

# verify-task

## Purpose

Run lint, typecheck and tests for every verification target of the task (one per repo or app, declared in
docs/TRD.md), one command at a time and with --runInBand, and append one block per target to the task's
verify.log with the commit it ran on. Shared by om-developer and om-reviewer. Never fixes anything.

Input: the workspace at its current commits.
Output: green or red per target and step, and one appended block per target in `{{ROOT_WT}}/docs/tasks/{{TASK}}/verify.log`.

## 1. Targets

Read `docs/TRD.md` (root workspace copy), section `Verification targets`.
Each target has a name, a path relative to the workspace (`{{repo}}/` or `{{repo}}/apps/backend/`), and commands for lint, typecheck, unit and e2e with their serial flags.
Verify only the targets whose repo is in the task's `repos`, plus any target whose path the diff touched.
If the TRD declares no targets, fall back to `references/{{stack}}.md` of this skill by the stack the TRD names; if neither exists, stop and report `verify-task: no verification targets in TRD`. Do not guess.

Skip tests for `type: docs`; run only Markdown lint if the project has one.

## 2. Run, one at a time

Per target, in this order, each command alone, absolute path with `cd {{WORKSPACE}}/{{target path}} &&`:

1. lint
2. typecheck (if declared)
3. unit tests, `--runInBand` or the stack's serial flag
4. integration or e2e tests, `--runInBand`, if declared

Never run two targets or two suites in parallel; several tasks share the machine and memory is the constraint.
Never pass `--watch`.
Capture the exit code and the last relevant lines per step.

## 3. Log

Append per target:

```
## {{ISO timestamp}} by {{om-developer | om-reviewer}} target {{name}} on {{sha of that repo}}
- lint: {{pass | fail}} ({{command}})
- typecheck: {{pass | fail | n/a}} ({{command}})
- unit: {{pass | fail}} ({{n}} tests, {{command}})
- e2e: {{pass | fail | n/a}} ({{command}})
{{first relevant error lines of the first failing step, if any}}
```

Never rewrite earlier blocks.

## 4. Report

Return only the per-target, per-step status and, for failures, the last 20 lines of the first failing step.
The full output lives in `verify.log`, never in the conversation.
Red: stop at the first failing step of that target, report it, continue with the next target only if the caller asked for a full run.

## Rules

- Never modify code, tests or configuration.
- Never skip a declared step.
- Never parallelize.
- The log is the archive: never paste full command output into the conversation or a message.
