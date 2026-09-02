---
name: check-task
description: Derive one task's state from disk, git and gh. om-manager; when Sebastian asks about a task.
effort: low
argument-hint: "[TASK_ID_OR_FOLDER]"
disable-model-invocation: false
---

# check-task

## Purpose

Derive the state of one task from disk, git and gh without asking any session: planned, consolidating,
consolidated, in_progress, in_review, merged or done, across every repo the task touches, per phase when the
task has phases. om-manager only; may run on its own when Sebastian asks about a task.

Input: a task id (`0142`) or a task folder path.
Output: one line per task (or per phase), nothing else.

Read `task.md` frontmatter: `id`, `title`, `type`, `branch`, `phases`, `modules`, `repos`, `depends_on`.
`WORKSPACE = {{ROOT}}/.workspaces/{{id}}_{{title}}`.
Repos to inspect: `repos` plus the root in multirepo.

## Signals

| Signal | How |
|---|---|
| folder | task folder exists in the root checkout |
| workspace | `WORKSPACE` exists |
| context | `Context & decisions` non-empty (root workspace copy if the workspace exists, else root checkout copy) |
| ahead | any repo: `git -C {{WORKSPACE}}/{{name}} rev-list --count origin/{{base}}..HEAD` > 0 |
| om-developer | `claude agents --json` lists `om-{{id}}-developer*` with cwd under `WORKSPACE`, or panes of `task-{{id}}` show it |
| prs | per repo: `gh -R {{owner/repo}} pr list --head {{branch}} --state all --json number,state,mergedAt,url` |

If `gh` fails (not logged in, the active account has no access to the repo, network), `prs` is unknown, not empty: never derive `in_review`, `merged` or `done` from it.
Say so in the line (`PRs: gh failed: {{first line of the error}}`) and add one line with the active account from `gh auth status`; `gh auth switch --user {{account}}` fixes the wrong-account case.

With phases, evaluate `ahead` and `prs` per phase branch; the current phase is the first without all PRs merged.

## Derivation

First match wins:

| Condition | State |
|---|---|
| every expected PR merged (code repos touched, plus root in multirepo) and no workspace | `done` |
| every expected PR merged and workspace exists | `merged` (pending `clean-task`) |
| any PR open | `in_review` |
| ahead or om-developer | `in_progress` |
| workspace and context | `consolidated` |
| workspace and no context | `consolidating` |
| folder only | `planned` |

## Output

```
0142 badge_wall  feature  in_review   PRs: platform #57 open, infra #12 merged, docs #3 open   phase 2/3
```

One line.
If Sebastian asked "why", add at most three lines of evidence.

## Rules

- Never message a session; only check whether it exists.
- Never change anything.
- A `gh` failure is reported, never read as "no PRs".
