# Part 2: Orchestration with om-manager, om-reviewer and om-developer

Status: agreed on 2026-08-28.
Depends on: [01-documentacion.md](01-documentacion.md).
The document [05-layouts.md](05-layouts.md) generalizes worktrees, docs and PRs to single repo, monorepo and multirepo; where this document says `.claude/worktrees/{{task}}/`, read `.workspaces/{{task}}/{{repo}}/`, and where it says "the worktree", read "the workspace".

## Goal

Sebastian does not run skills one by one, nor does he talk to whoever writes the code.
He talks to a single agent per project (the om-manager) and receives PRs ready to approve.
Each task is resolved by a pair of sessions (om-reviewer + om-developer) with their own context, scoped to that task.

## Three-level model

| Level | Session | Lifetime | Context it loads | Skills |
|---|---|---|---|---|
| om-manager | One per project | Long-lived, resumable | General `docs/`, conversation with Sebastian, task state | `setup`, `plan-task`, `create-task`, `consolidate-task`, `delegate-task`, `reiterate-task`, `check-*`, `clean-*` |
| om-reviewer | One per task | The task | Task file, module `docs/`, diff and PR | `analyze-task`, `start-task`, `review-task`, `publish-task`, `next-phase` |
| om-developer | One per task | The task | Task file, module `docs/`, code | `execute-task`, `document-task` |

The three contexts are deliberately disjoint.
The om-manager does not see code.
The om-reviewer does not write code.
The om-developer does not talk to Sebastian.

### om-manager

It is the only session Sebastian talks to.
Planning and product, architecture and technical decisions happen with it.
With `create-task` it creates the task file.
When creating it, it asks whether to run it now or leave it queued.
With `consolidate-task` it launches the om-reviewer and mediates its questions with Sebastian; with `delegate-task` it hands the task over to it.
It never launches om-developers: that is done by the om-reviewer.
It never enters a task's review cycle.

### om-reviewer

It receives the task from the om-manager with the instruction: "read the task, do you have any questions?".
Questions are discussed between om-reviewer, om-manager and Sebastian only once, during the consolidation phase.
What is decided in that conversation is written by the om-reviewer into the `Context & decisions` section of the task file before the om-developer starts.
After consolidation it is autonomous: it answers the om-developer's questions and makes decisions without escalating them to anyone.
Its decisions are documented in the `Decisions` section of the PR comment.
At the end of each om-developer cycle it runs `review-task`.
When `review-task` passes with no issues, it runs `publish-task`: push, PR and comment.
It never modifies code.
It is the only publishing point: pushing is not writing code.

### om-developer

It always works in its own worktree.
It runs `execute-task`, which includes `verify-task` and, at the end, `document-task`.
It raises its questions to the om-reviewer, never to the om-manager or Sebastian.
It never touches the remote: its work ends in a local commit and a "round N" message to the om-reviewer.
It squashes its work into a single commit before each round.
Review rounds happen in the worktree, before publishing; the PR is born with a single commit.
Only `reiterate-task` adds new commits to an existing PR.

## Roles as defined agents

Each role is a `.claude/agents/{{rol}}.md` file with frontmatter (name, description, allowed tools, model) and the system prompt in the body.
A full session with that role is launched using `claude --agent {{rol}}`.
The task is passed as a pointer: `claude --agent om-developer "task: docs/tasks/142.md"`.
The role lives version-controlled; the om-manager does not draft long prompts every time.
Each role has its own `--permission-mode`: the om-developer autonomous, the om-reviewer without code-writing tools.

These agents can live in `~/.claude/agents/` (global, applying to every project) because the flow is Sebastian's convention, not the project's.

## Session and tmux mechanics

Each project has one tmux session.
The om-manager runs in the first window.
Each running task is a `task-{{id}}` window with two panes: left om-reviewer, right om-developer (the current phase's, if there are phases).
The window is opened by the om-manager in `consolidate-task` with only the om-reviewer pane; the om-developer pane is opened by the om-reviewer when it delegates.

### Session identity

Each `claude --agent {{rol}}` is an independent process with its own session, history and context.
Two tasks in parallel are four processes that share nothing.
They are distinguished by three identifiers, all set by the om-manager when launching them:

| Identifier | Example | Use |
|---|---|---|
| Session name (`-n`) | `om-0142-reviewer`, `om-0142-developer`, or `om-0142-developer-phase-2` if there are phases | Address for `SendMessage`. |
| Working directory (worktree) | `{{root}}/.workspaces/0142_badge_wall/` | Isolates the code and groups the task's session history in its own folder. |
| Session ID | UUID | For `claude attach {{id}}` or `claude -r {{id}}`. |

The `task-{{id}}-{{rol}}` convention makes addressing deterministic: the om-reviewer knows who to talk to because it knows its own task id.

### Launch (main form, to validate in the pilot): `--bg` + `attach`

1. The om-manager launches both sessions in the background:
   First it creates the worktree and the branch; then, with cwd in the worktree, `claude --bg --agent om-reviewer -n om-{{id}}-reviewer "task: docs/tasks/{{id}}_{{title}}/"` and the equivalent with `--agent om-developer`.
   Both share the same worktree.
2. The om-manager creates the tmux window `task-{{id}}` with two panes and runs `claude attach {{id}}` in each one.
   `attach` opens the full interactive session in that pane: progress is visible live, half om-reviewer and half om-developer.
   With Ctrl+Z you return to the shell and the session keeps running.
3. The om-reviewer works in the same worktree as the om-developer because it needs to run lint and tests on the branch.
   They alternate; they never write at the same time because the om-reviewer does not write code.

The om-reviewer and om-developer agents run with `permissionMode: bypassPermissions`: they work isolated in their workspace and must not be held back by the classifier; with `--bg` they must be launched with `--allow-dangerously-skip-permissions`.
Advantages: `claude agents` lists the live sessions, `claude stop {{id}}` pauses, `claude rm {{id}}` deletes the session and its worktree, `claude attach {{id}}` reopens it.
The process survives if the pane is closed.

### Launch (alternative form, if `--bg` + `attach` does not convince)

In each pane, with cwd in the worktree, `claude --agent {{rol}} -n task-{{id}}-{{rol}} "task: docs/tasks/{{id}}_{{title}}/"` is run directly.
Functionally it is the same; management with `claude agents / stop / rm` is lost and the process dies with the pane.
The rest of the design does not change.

### Where histories live

Claude Code stores each session in `~/.claude/projects/{{slug-del-directorio}}/{{session-id}}.jsonl`, grouped by the directory it ran in.
Since om-reviewer and om-developer run inside the worktree, their sessions land in their own folder, for example `~/.claude/projects/-home-fless-dev-designli-projects-diy-diy-platform--claude-worktrees-0142_badge_wall/`.
That is why they do not show up in the main repo's `claude -r`: the picker filters by directory.

### Communication between sessions

`SendMessage` to the session by name.
The message enters the other session's queue and is processed on its next turn.
Messages are pointers (task folder, branch, PR number, round, phase), never diffs or content.
`tmux send-keys` is not used to communicate between sessions.

## Task folder

Path: `docs/tasks/{{id}}_{{title}}/`, for example `docs/tasks/0142_badge_wall/`.
`{{id}}` is sequential per project with four digits; `{{title}}` is short snake_case.
The folder is the task's source of truth.
Any new session can pick it back up by reading it; live sessions are an accelerator, not a dependency.

`docs/tasks/_drafts/{{title}}.md` holds plans still in conversation that are not yet a task; `plan-task` writes them when the conversation grows and `create-task` deletes them when it creates the folder.
`check-work` ignores `_drafts/`.

Decided on 2026-08-28: `docs/tasks/` is tracked by git; only `docs/tasks/_drafts/` goes in `.gitignore`.
A task's folder is written in only one place at any given time, and the handover point is the delegation:

- Before `delegate-task`, in the root checkout (base branch).
  There it is written by `create-task` (plan) and the om-reviewer during consolidation (`Context & decisions`).
- `consolidate-task` ends with the task's single docs commit, on the base branch and with Sebastian's approval: `docs(tasks): {{id}}_{{title}} planned`.
  It contains the plan and `Context & decisions`: exactly what Sebastian approved.
- `delegate-task` starts with `git rebase origin/{{base}}` in the worktree; the branch receives the complete folder.
- After `delegate-task`, only in the workspace copy: `om-developer notes`, `verify.log`, `Result`, `retakes.md`.
  They travel in the PR and reach the base branch with the merge.
  No one writes the root checkout copy again; it updates itself with `git pull`.
- `clean-task` does not commit anything.
- The root checkout always lives on the base branch; each om-manager skill syncs it with `origin/{{base}}` before acting.
  If a project forbids direct pushes to the base branch, the `planned` commit stays local and how to publish it is decided per project.

### task.md

```yaml
---
id: 0142              # sequential per project; the om-manager takes the highest one in docs/tasks/ and adds one
title: badge_wall
type: feature         # feature | bug | docs | chore | refactor
ticket: DIY-231       # optional: Jira, Linear, GitHub issue
branch: feat/0142_badge_wall      # empty if the task has phases; each phase has its own
modules: [billing, notifications]   # primary first; a task can touch several
phases: 0             # 0 if there are no phases
depends_on: []        # optional; only if there are stacked or parallel tasks that touch each other
updated: 2026-08-28
---
```

Body sections:

- `Goal`: what is meant to be achieved and why.
- `Scope`: what is in and what is not.
- `Acceptance`: verifiable criteria.
- `Context & decisions`: what was discussed during consolidation; written by the om-reviewer before an om-developer exists.
- `om-developer notes`: the om-developer's closing note per round: what it did, what it left pending.

### Derived state

The frontmatter has no status field; no commit exists to change status.
`check-task` deduces everything from disk, git and `gh`:

| Evidence | Status |
|---|---|
| Folder on the base branch, no worktree | `planned` |
| Worktree exists, `Context & decisions` empty | `consolidating` |
| Worktree exists, `Context & decisions` written, no commits ahead of `origin/{{base}}`, no om-developer session | `consolidated` (ready to delegate) |
| Branch with commits ahead of `origin/{{base}}`, or a `om-{{id}}-developer*` session exists | `in_progress` |
| `gh pr list --head {{rama}}` returns an open PR | `in_review` |
| PR merged and the worktree still exists | `merged` (pending `clean-task`) |
| PR merged and no worktree | `done` |

With phases, the same table applies per phase branch.
There is no `pr:` field; `gh` knows it from the branch.

### Types

The `type` determines the branch prefix and part of the behavior:

| type | Branch | Behavior |
|---|---|---|
| feature | `feat/{{id}}_{{title}}` | Normal flow. |
| bug | `bugfix/{{id}}_{{title}}` | The folder includes `replication.md`. The om-developer reproduces the bug by following it before touching code; if it does not reproduce, it stops and notifies the om-reviewer. The om-reviewer verifies the fix against the same steps. |
| docs | `docs/{{id}}_{{title}}` | `verify-task` does not run tests, only Markdown lint if it exists. |
| chore | `chore/{{id}}_{{title}}` | Normal flow. |
| refactor | `refactor/{{id}}_{{title}}` | Normal flow; `Acceptance` requires identical behavior. |

### Phases

Almost no task has phases; the default is a single PR.
Only an evidently large feature is split into phases within the same task, and Sebastian decides that.
Each `phase_N.md` has its own frontmatter (`branch`) and its own `Scope` and `Acceptance`.

Rules:

- One worktree per task, one branch per phase: `feat/0142_badge_wall-phase-1`, `-phase-2`.
  With a hyphen and not a `/` because git does not allow `feat/x` and `feat/x/phase-1` to coexist.
- One PR per phase.
- One om-reviewer for the whole task (`om-0142-reviewer`), which watches over the whole plan working.
- A new om-developer per phase (`om-0142-developer-phase-1`, `-phase-2`), with a clean context.
- Phase N+1 starts only once phase N's PR is merged.
  When Sebastian merges, he tells the om-manager, and the om-manager notifies the om-reviewer to continue.
  Stacked PRs are not used: they are faster but bring cascading rebases.
- When it finishes its work, the om-developer leaves a closing note in `phase_N.md` (what it did, what it left pending); the om-reviewer writes `Result` upon learning of the merge, in the workspace copy, and it travels in the next phase's PR.
  For the last phase there is no next PR: its result goes in the PR comment.
  This way a replacement om-reviewer can pick up from disk.
- When moving to phase N+1, the om-reviewer kills phase N's om-developer session and spins up a new one with no context.

### Worktree

`{{root}}/.workspaces/{{id}}_{{title}}/{{repo}}/`, one worktree per repo touched (see [05-layouts.md](05-layouts.md)).
It is created by the om-manager in `consolidate-task` with `git worktree add -b {{rama}} {{ruta}} origin/{{base}}` to control the branch name, and it launches the om-reviewer with that cwd.
The branch is born without the task folder; it receives it in `delegate-task`'s rebase, after the `planned` push.
It is created during consolidation and not during delegation because a Claude Code session has a fixed cwd and the om-reviewer needs to live inside the worktree to run lint and tests.
If Sebastian decides not to delegate yet, the worktree and the branch stay; only the om-reviewer's session is stopped (`claude stop`, which preserves the conversation) and the tmux window is closed.
`delegate-task` reopens the same session (`claude attach`, or `claude -r` if it was not `--bg`) with its context intact.
om-reviewer and om-developer share the worktree.

## Lifecycle of a task

1. Sebastian to the om-manager: "I want X".
2. `plan-task` (om-manager + Sebastian): reads `docs/` following Part 1's reading path and plans.
   Output: the closed idea, and `docs/tasks/_drafts/{{title}}.md` if the conversation grew.
3. `create-task` (om-manager + Sebastian): writes the `docs/tasks/{{id}}_{{title}}/` folder in the root checkout with `task.md`, `replication.md` if it is a bug, and `phase_N.md` if there are phases.
   Nothing is committed yet.
4. `consolidate-task` (om-manager): syncs the base branch, creates the worktree and branch according to `type` from `origin/{{base}}`, opens the tmux window `task-{{id}}` and launches `om-{{id}}-reviewer` inside the worktree, without an om-developer.
   The om-reviewer runs `analyze-task`: it reads the folder (in the root checkout) and the modules' docs, and shares its questions with the om-manager; the om-manager and Sebastian resolve them; the om-reviewer writes `Context & decisions` in the root checkout copy.
   When the om-reviewer signals "consolidated", the om-manager asks Sebastian for approval and commits and pushes `docs(tasks): {{id}}_{{title}} planned` on the base branch: plan plus decisions, the task's single docs commit.
   Then it asks: delegate now?
   If not: it stops the om-reviewer's session (preserving its conversation) and closes the tmux window. The worktree and branch stay. The task shows up as `consolidated` in `check-work`.
5. `delegate-task` (om-manager): checks `depends_on`.
   If the om-reviewer is stopped, it reopens the tmux window and the same session (`claude attach` or `claude -r`); only if the session was lost does it launch a new om-reviewer, whose `analyze-task` is idempotent.
   It runs `git fetch` and `git rebase origin/{{base}}` in the worktree: the branch receives the folder with the decisions.
   It sends "delegated, start" to the om-reviewer.
   From here everything written to the folder goes to the workspace copy, and the om-manager does not intervene until the om-reviewer publishes.
6. The om-reviewer launches `om-{{id}}-developer` (or `-developer-phase-1`) in the right pane, with cwd in the worktree, and sends it "context ready, start".
7. om-developer: `execute-task` → rebase from `origin/{{base}}` → (bug: reproduce with `replication.md`) → implements → `verify-task` → `document-task` → squashes into one commit → `om-developer notes` → `SendMessage` to the om-reviewer: `round 1 ready, commit {{sha}}`.
8. om-reviewer runs `review-task` (see Pipeline) on the worktree.
   If there are findings, it sends them to the om-developer; the om-developer fixes them, re-runs `verify-task`, squashes, signals "round 2".
   This repeats until there are no findings.
   With no findings, the om-reviewer runs `publish-task`: push, opens the PR, writes the comment, notifies the om-manager "PR #{{n}} ready".
9. om-manager notifies Sebastian, in one line, that the PR is ready.
10. Sebastian reads the PR comment and decides:
    - Approves and does the merge (he always does it, for now).
    - Or tells the om-manager `reiterate-task` with his comments.
11. With the merge, Sebastian asks the om-manager for `clean-task`; see "Cleanup".
    If the task has phases and it was not the last one, instead of cleaning up, the om-manager notifies the om-reviewer "phase N merged, continue"; the om-reviewer runs `next-phase` (kills the om-developer, creates phase N+1's branch, launches a new om-developer) and the cycle goes back to step 7.
    Cleanup happens when the last phase is merged.

### Cleanup of a merged task

With `--bg`:

```
claude rm {{id-reviewer}}                                  # session (and worktree when it is safe)
claude rm {{id-developer}}                                  # one per phase if there were any
git -C {{root}}/{{repo}} worktree remove {{root}}/.workspaces/0142_badge_wall/{{repo}}   # for each repo; then rmdir of the workspace
git branch -d feat/0142_badge_wall
tmux kill-window -t task-0142
```

With the alternative form (direct sessions):

```
claude project purge {{root}}/.workspaces/0142_badge_wall   # transcripts, tasks, history and config for that directory
git -C {{root}}/{{repo}} worktree remove {{root}}/.workspaces/0142_badge_wall/{{repo}}   # for each repo
git branch -d feat/0142_badge_wall
tmux kill-window -t task-0142
```

In both cases the om-manager does not commit anything: the final folder reached the base branch with the PR merge, and with no worktree the task is derived as `done`.
Before cleaning up, the om-manager runs `git pull` in the root checkout to bring in that final state.
Sebastian can also ask the om-manager to clean up all already-merged branches at once.

### reiterate-task

The om-manager notes Sebastian's comments, dated, in `retakes.md` in the workspace copy (the task is already delegated).
It brings the om-reviewer + om-developer pair back up on the same branch, worktree and PR.
If the pair is still alive, it notifies them; if not, it relaunches them.
The om-reviewer runs `analyze-task` incorporating the retakes (this is the only thing it asks about again) and the cycle continues from step 7.
`publish-task` updates the existing PR and adds the new round's commit.
`retakes.md` travels in the om-developer's next round commit and arrives with the PR.

### If something dies

The state is in the task file, in git and in the PR.
If the session exists: `claude attach {{id}}` or `claude -r`.
If not: a new pair is launched with the same pointer and it picks up from the file's state.

## review-task: the Pipeline

Orden: intent → rebase → lint → test → review → documentation → push.

Lint and test come before the deep review because a failing test changes what gets reviewed.

| Step | What the om-reviewer verifies |
|---|---|
| intent | That the code changes correspond to the task's `Goal` and `Scope`, with no deviations or extras. |
| rebase | That the om-developer rebased from `origin/{{base}}`. It requests it if not done. |
| lint | Re-runs lint on the final commit, via `verify-task`. |
| test | Re-runs typecheck and tests on the final commit, via `verify-task` (one by one, `--runInBand`). |
| review | Finds issues in the code created for the task. Documents the error and how it was fixed, not how the task was solved. |
| documentation | That the om-developer ran `document-task`: the module's docs updated with `updated` and `source`. |
| push | Done by the om-reviewer itself in `publish-task` once everything above has passed. |

Re-running lint and tests on the final commit is what makes irrelevant the order in which the om-developer ran them: it proves the last state is green.
The om-reviewer also checks in `docs/tasks/{{id}}.log` (the `verify-task` log) that lint and test ran after the last fix.

## publish-task: push, PR and comment

The om-reviewer does the push, opens the PR if it does not exist, and writes a single comment that it edits on each `reiterate-task` instead of stacking a new one.
It does not write any status; `in_review` is derived from the open PR.

```
## Intent
A paragraph: what the task wanted to achieve and what was agreed upon at delegation.

## What changed
Maximum 10 concise bullets.

## Decisions
Decisions the om-reviewer made autonomously during the task, with its reasoning.
This is what Sebastian reads to decide between approving and a retake.

## Risk assessment
Level (Low / Medium / High) and one sentence of justification.

## Pipeline
- [x] intent
- [x] rebase
- [x] lint
- [x] test
- [x] review: N issues → fixed (detail per issue: file:line, error, fix, re-check)
- [x] documentation
- [x] push
```

## Escalation

It only exists during the consolidation phase.
After that, the om-reviewer decides everything within the task's scope and documents it in the PR's `Decisions`.
The only way back is `reiterate-task` from Sebastian through the om-manager.
The only exception that escalates the whole chain (om-developer → om-reviewer → om-manager → `om-events`) is a block due to missing permissions, credentials, environment or tools, typed `blocker`; see [04-operacion.md](04-operacion.md).

## Parallelism

Several tasks in parallel in the same project: one tmux window and one worktree per task.
Conflicts between tasks are resolved in the rebase.
`depends_on` is used when there are stacked or parallel tasks touching the same thing; the om-manager does not launch a task until its dependencies are `done`.

## Skills

The full list of skills per role, their events and the state machines are in [03-skills.md](03-skills.md).

## Open items

- Validate in a pilot that `claude --bg` + `claude attach` inside tmux panes gives the same visual experience as a direct session, and that `SendMessage` between sessions launched this way is delivered reliably.
  If not, switch to the alternative form documented above.
- Define the files `~/.claude/agents/om-manager.md`, `om-reviewer.md` and `om-developer.md`.
- Decide the exact template for `docs/tasks/{{id}}_{{title}}/task.md` (sections above) within `create-task`.
