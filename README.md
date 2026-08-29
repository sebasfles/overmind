# overmind

A system for working with Claude Code agents across many software projects at once, without running skills by hand or losing context between sessions.

Sebastian talks to one long-lived `om-manager` per project.
Each task is executed by a pair of sessions that live for that task: an `om-reviewer` that never writes code and an `om-developer` that never pushes.
Above all projects sits an `overmind` cockpit that aggregates state and events but never decides anything about a project.

## Why

- Many projects at the same time: attention is the scarce resource, so the system batches what it needs from Sebastian and shows one board across everything.
- Agents should not rediscover a project from zero: every project carries a fixed documentation convention that agents read progressively and update as part of every task.
- Whoever reviews must not be whoever wrote: review happens in a separate session with a clean context, before anything is pushed.
- Disk is the source of truth, sessions are accelerators: any task can be resumed from its folder, git and its PRs, by a fresh session if needed.

## Roles

```
                    overmind  ─  om-events  ─  om-config          (cockpit, one tmux session)
                        │
          ┌─────────────┼─────────────┐
     om-manager    om-manager    om-manager                       (one per project, long-lived)
          │
   ┌──────┴──────┐
om-reviewer  om-developer                                         (one pair per task, in a workspace)
```

| Role | Talks to | Never |
|---|---|---|
| `overmind` | Sebastian | decides about a project |
| `om-events` | receives events from managers | replies |
| `om-config` | Sebastian, about the method | touches project state |
| `om-manager` | Sebastian, its reviewers | writes or reviews code |
| `om-reviewer` | its manager (once), its developer | writes code |
| `om-developer` | its reviewer | pushes, talks to anyone else |

## A task, end to end

`plan-task` (conversation) → `create-task` (folder on disk) → `consolidate-task` (workspace, reviewer alone, questions answered once, one docs commit) → `delegate-task` (reviewer launches the developer) → rounds of implement, verify, document, review → `publish-task` (one PR per repo, one summary comment) → Sebastian merges → `clean-task`.

State is never written: `planned`, `consolidated`, `in_progress`, `in_review`, `merged`, `done` are derived from the task folder, git and `gh`.

## Layouts

Single repo, monorepo and multirepo share one abstraction: a `root` (always a git repo, the owner of `docs/`), a list of `repos`, and a workspace per task at `root/.workspaces/{{task}}/` with one git worktree per repo touched.
See `docs/05-layouts.md`.

## Repository

```
agents/            global roles that run inside projects (om-manager, om-reviewer, om-developer, om-setup-worker)
.claude/agents/    cockpit roles that only run here (overmind, om-events, om-config)
skills/            28 skills, one folder each, with templates and references
.claude/skills/    update-method: how the method itself is changed
bin/               what Sebastian types, linked into ~/bin (resume-overmind)
scripts/           what agents and the repo run (install, resume-project, lint-method)
docs/              design (01 to 06), usage-guide.md, method-ard.md
portfolio/         state: project registry, todos, event inbox
```

## Getting started

1. `scripts/install`: links roles and skills into `~/.claude`, and `resume-overmind` into `~/bin`.
2. `bin/resume-overmind`: opens the cockpit.
3. In `overmind`: `/add-project {{path}}`, then `/resume-project {{name}}` and, in the project's `om-manager`, `/setup`.

The full walkthrough is in `docs/usage-guide.md`.

## Documents

| File | What |
|---|---|
| `docs/01-documentacion.md` | the documentation convention every project carries |
| `docs/02-orquestacion.md` | roles, sessions, task folder, cycle, pipeline, PR comment |
| `docs/03-skills.md` | every skill, by role, with its trigger |
| `docs/04-operacion.md` | the cockpit, registry, todos, events |
| `docs/05-layouts.md` | single repo, monorepo, multirepo |
| `docs/06-piloto.md` | what is still unvalidated and the order to validate it |
| `docs/usage-guide.md` | how to operate it day to day |
| `docs/method-ard.md` | why later decisions were made |

## Changing the method

In the `om-config` session: `/update-method {{change}}`.
It edits everywhere the change belongs, runs `scripts/lint-method`, records the reason in `docs/method-ard.md`, and commits.

## Status

Design complete; pilot pending.
Everything here is a draft until a real project has gone through `setup` and one task end to end.
