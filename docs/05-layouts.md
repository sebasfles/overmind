# Part 5: Project layouts (single repo, monorepo, multirepo)

Status: agreed on 2026-08-29.
Depends on: [02-orchestration.md](02-orchestration.md), [03-skills.md](03-skills.md), [04-operation.md](04-operation.md).
Where this document contradicts the previous ones, this one governs.

## Objective

One single flow for the three layouts Sebastian has:

- Single repo: one repo, one app.
- Monorepo: one repo with several apps inside (`/backend`, `/frontend`, `/infra`, or turborepo).
- Multirepo: a project folder with several sibling repos (`diy/diy-platform`, `diy/diy-infra`, ...).

All three are described with the same abstraction, and single and mono are cases with single-element lists.

## Glossary

- Root checkout: the clone of the root where om-manager runs, always on its base branch.
- Workspace: `{{root}}/.workspaces/{{task}}/`, cwd of om-reviewer and om-developer, with one worktree per repo touched.
- Root worktree: the root's worktree inside the workspace; in single and mono it is the same worktree as the code.
- Root checkout copy: the task folder in the root checkout; it is written only before `delegate-task`.
- Workspace copy: the task folder in the workspace's root worktree; it is written only after `delegate-task`.

## Abstraction

A project has `root`, `repos` and workspaces.

| Concept | Single | Monorepo | Multirepo |
|---|---|---|---|
| `root`: cwd of om-manager, owner of `docs/`, always a git repo | the repo | the repo | the project folder, converted into a docs git repo with the clones ignored |
| `repos`: code repos | `[.]` | `[.]` | `[diy-platform, diy-infra, ...]` |
| Workspace for a task | `.workspaces/{{task}}/{{repo}}/` | same | `.workspaces/{{task}}/{{repo}}/` per repo touched, plus `.workspaces/{{task}}/{{root}}/` |
| `docs/` and `docs/tasks/` | in the root | in the root | in the root |
| `verify-task` targets | 1 | N, one per app, declared in the TRD | N, one per repo, each with its own TRD section |
| Code PRs | 1 | 1 | one per repo touched |
| Task PR (carries the summary) | the same code PR | the same | the root repo's PR |

## The root repo in multirepo

The project folder is initialized as a git repo that tracks only `CLAUDE.md`, `docs/` and `.gitignore`.
Code clones and `.workspaces/` go in `.gitignore`; git does not look inside ignored paths, so the nested repos do not cause problems.
Personal private remote, named `{{project}}-docs` (for example `fless/diy-docs`); the local folder keeps being called `{{project}}`.
`add-project` creates it when it detects a multirepo without `.git` at the root, asking for the remote.

```
diy/                      # diy-docs repo
  .gitignore              # diy-platform/ diy-infra/ diy-pocs/ .workspaces/
  CLAUDE.md
  docs/
  diy-platform/           # clone, ignored
  diy-infra/              # clone, ignored
  .workspaces/            # ignored
```

In single and mono this does not apply: the code repo already is the root.

## Workspaces

`{{root}}/.workspaces/{{id}}_{{title}}/` is the cwd of the task's om-reviewer and om-developer.
It contains one worktree per repo touched, all on the same branch `{{prefix}}/{{id}}_{{title}}`, and in multirepo also the root's worktree.
It replaces `{{repo}}/.claude/worktrees/`.

- They are created with `git -C {{repo}} worktree add -b {{branch}} {{root}}/.workspaces/{{task}}/{{repo}} origin/{{base}}`.
  A worktree shares `.git` with its clone; it is not a copy.
- In single and mono the workspace stays inside the repo; `.workspaces/` goes into the repo's `.gitignore`.
- Bootstrap per worktree, in `consolidate-task`: copy or link the unversioned files the repo needs (`.env*` and whatever the TRD lists under `Workspace files`) and run the TRD's install command.
  Without this, the first `verify-task` fails for reasons unrelated to the task.
- Always absolute paths: the shell does not preserve `cd` between commands.
- When cleaning up: `git -C {{repo}} worktree remove {{path}}` for each one, and delete the workspace folder.

## Documentation

`docs/` belongs to the root and describes the whole application, not a single repo.
Modules belong to the application: `docs/modules/billing/trd.md` has one section per repo or app that participates.
The general TRD has one section per repo or app: stack, base branch, `Verification targets`, `Workspace files`, installation.

The writing rule for the task folder does not change, it just gets read against the root:

- Before `delegate-task`: in the root checkout.
- `consolidate-task` ends with the commit `docs(tasks): {{task}} planned` on the root's base branch, with Sebastian's approval.
- `delegate-task` rebases **the root's worktree** onto `origin/{{base}}`; in single and mono that worktree is the code's worktree.
- After that: in the root's worktree inside the workspace. It travels in the root's PR.

## Verification

`verify-task` iterates over the TRD's `Verification targets`: each one with a name, path (relative to the workspace) and lint, typecheck, unit and e2e commands with their serial flag.
Single: one target.
Monorepo: one per app.
Multirepo: one per repo touched.
One block per target in `verify.log`.

## Publishing

`publish-task` opens one PR per repo touched, plus the root's PR in multirepo.
The summary (Intent, What changed, Decisions, Risk, Pipeline per target) lives in **the root repo's PR**: in single and mono it is the same code PR; in multirepo it is the `{{project}}-docs` PR.
Code PRs in multirepo carry a one-line body with the link to the root's PR.
`What changed` links to each code PR.
If there is a merge order between repos (infra before platform), it goes in `Decisions`; Sebastian merges in that order and merges the root's last.
There is no concept of a primary repo.

## Derived status

Same as in `02`, evaluated over all the task's PRs:

- `in_review`: some PR is open.
- `merged`: all merged, including the root's, and the workspace still exists.
- `done`: all merged and no workspace.

## Registry (`portfolio/projects.yaml`)

```yaml
projects:
  - name: diy
    root: /home/fless/dev/designli/projects/diy
    tmux: diy
    status: active
    repos:
      - name: diy-platform
        base_branch: develop
      - name: diy-infra
        base_branch: main
  - name: backend-school
    root: /home/fless/dev/designli/projects/backend-school
    tmux: backend-school
    status: active
    repos:
      - name: .
        base_branch: main
```

`task.md` gains `repos:` with the repos it touches (default: all of the list when there is only one; mandatory to choose in multirepo).

## Changes to what was written

- `02`: task folder, worktree, cycle and cleanup are read against root and workspaces; `.claude/worktrees/` disappears.
- `03`: `verify-task` per targets; `publish-task` one PR per repo with the summary in the root.
- `04`: registry with `root` and `repos`; `resume-project` opens the root.
- Skills: `add-project`, `resume-project`, `setup`, `write-trd`, `create-task`, `consolidate-task`, `delegate-task`, `start-task`, `verify-task`, `publish-task`, `check-task`, `clean-task`.
- Agents: absolute-path rule in `om-manager`, `om-reviewer`, `om-developer`; workspace cwd in `om-reviewer` and `om-developer`.
