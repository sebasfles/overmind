# Usage guide

How to operate the system day to day.
What you type, what you see, and what to do when something gets stuck.
Everything not covered here is in `01` through `06`.

## Requirements

- Claude Code with background sessions (`claude --bg`, `claude attach`, `claude agents`).
- tmux, `gh` authenticated with the account that can open PRs in each project, and Windows Terminal if you work in WSL.
- This repo cloned, and `~/bin` in the `PATH`.

## Install

```
scripts/install
```

Creates symlinks: `agents/*` to `~/.claude/agents/`, `skills/*` to `~/.claude/skills/`, and `resume-overmind` to `~/bin/`.
`resume-project` is not linked: the `overmind` session runs it for you when you ask to open a project.
It's idempotent; run it again after a `git pull` that adds skills.
The three cockpit agents are not symlinked: they only exist inside this repo.

## Starting the day

```
resume-overmind
```

This opens a tab with the tmux session `overmind`: `overmind` on the left, `om-events` on the right, and a `config` window with `om-config`.
`overmind` starts by itself with `/check-portfolio` and shows:

```
diy        in_review 2   in_progress 1   consolidated 1   planned 3   drift 2 modules   om-manager: up
auvral     planned 1                                                  drift 5 modules   om-manager: down

waiting for you
  diy    0142 badge_wall     PR #57
todos (open 2)
  - decide on Stripe vs Adyen for diy
notifications since last check
  [action] diy: task 0142 PRs ready
```

First what's waiting for you, then what's moving, then what's stuck.
If you want detail, you go into the project; the cockpit doesn't explain.

Things you do from `overmind`:

| You want | You type |
|---|---|
| to jot something down so you don't forget | `/add-todo call the bseen client @bseen` |
| to check off a todo | `/complete-todo call the client` |
| to open a project | `/resume-project diy` |
| to register a new project | `/add-project ~/dev/designli/projects/hux` |
| to pause or remove one from the board | `/add-project hux pause`, `/add-project hux remove` |
| to clean up what's merged across all projects | `/clean-portfolio` |

The `om-events` pane isn't used directly; it only shows incoming events and the counters: `blockers 0 | actions 1 | info 3`.

## Adding a project

1. In `overmind`: `/add-project {{ruta}}`.
   It detects whether it's a repo, a monorepo, or a folder of repos.
   If it's a folder of repos without git, it suggests turning it into a docs repo (`{{nombre}}-docs`) that ignores the clones; say yes and give it the remote.
2. `/resume-project {{nombre}}`: a new tab, the project's tmux session, `om-manager` in window 0.
   The first time it will say `no docs/tasks/ in this project; run setup first`.
3. In the `om-manager`: `/setup`.

`setup` is a working session with you, not a batch:

- It inventories what's already documented anywhere (READMEs, wikis, ADRs, comments, specs) and reads the code, in parallel by component.
- It shows you the stack, what it found, and a proposed set of modules with their folders.
  You confirm or correct the modules; it's the only decision it doesn't guess.
- It asks you in batches what neither the docs nor the code know: who the product is for, what's out of scope, why something was chosen.
- It documents each module in parallel, then the TRD, the PRD and the ARD, and writes a short `CLAUDE.md`.
- It shows you each `[inferido]` so you can confirm, correct, or delete it.
- With your ok, it commits to the base branch.

It takes as long as it takes to read you; in a large multirepo, plan for a long session.

## Working a task

Everything from the project's `om-manager`.

### 1. Plan

`/plan-task I want users to see their badges`

The om-manager reads `docs/` and asks you only what isn't there, in batches of up to five, always with a recommendation.
At the end it shows you the plan (Goal, Scope, Acceptance, Approach, Database, Risks) and asks "Create the task?".
If the conversation grows, it saves a draft in `docs/tasks/_drafts/` so nothing is lost.

### 2. Create

When you say yes, `create-task` writes `docs/tasks/{{id}}_{{title}}/` with `task.md` (and `replication.md` if it's a bug) and asks "Consolidate it with the om-reviewer now?".
Nothing is committed yet.

### 3. Consolidate

`consolidate-task` creates the workspace (`.workspaces/{{id}}_{{title}}/`, one worktree per repo), opens the `task-{{id}}` window with the `om-reviewer` on the left, and the om-reviewer reads the task and asks questions.
Its questions reach you through the om-manager; you answer there.
When it says `consolidated`, the om-manager shows you `Context & decisions` and asks "Push the task to develop?".
It's the task's only docs commit on the base branch.
Then: "Delegate now?".

- Yes: continue to step 4.
- No: the om-reviewer's session pauses (it keeps its memory) and the window closes; the task stays `consolidated`.

### 4. Delegate

`/delegate-task {{id}}` (or the earlier yes).
The om-manager reopens the om-reviewer if it was paused, sends it `delegated, start`, and the om-reviewer opens the right pane with the `om-developer`.
From here you don't intervene.
You can look at the `task-{{id}}` window whenever you want: the om-reviewer on the left, the om-developer on the right.

### 5. Wait

The om-developer implements, verifies, documents, and delivers rounds; the om-reviewer runs the Pipeline and returns findings until none are left.
Then it pushes, opens a PR per repo, and writes the summary comment.
You find out through the om-manager (one line) and through `om-events` (`[action] diy: task 0142 PRs ready`).

### 6. Review and merge

Open the root PR (in single-repo and monorepo setups, it's the only one).
The comment has everything you need to decide without reading the diff: Intent, What changed, Decisions (what the om-reviewer decided on its own), Risk, Pipeline.
If there are several repos, `What changed` links each PR and `Decisions` states the merge order.

- Agree: merge, always you.
- No: `/reiterate-task {{id}} {{tus comentarios}}` or `/reiterate-task {{id}} read the PR`.
  The om-reviewer turns your comments into findings and the cycle continues; the PR gets a commit per new round.

### 7. Clean up

After the merge: `/clean-task {{id}}`, or `/clean-work` for everything merged.
It deletes sessions, workspace, branches, and window.
There's no commit: the task's final state arrived with the PR.

## Tasks with phases

Almost no task has phases; only a feature that's clearly too big to fit in a reviewable PR.
Each phase is a PR; the next one starts when you merge the previous one and tell the om-manager (`phase 1 merged`).
The om-reviewer is the same throughout the task; the om-developer changes per phase, with a clean context.

## Reading state

`/check-work` in an om-manager, `/check-portfolio` in `overmind`.
Nobody stores state; it's inferred:

| You see | It means |
|---|---|
| `planned` | folder on the base branch, no workspace |
| `consolidating` | workspace exists, the om-reviewer is still asking questions |
| `consolidated` | ready to delegate |
| `in_progress` | there are commits or an om-developer working |
| `in_review` | a PR is open, waiting for you |
| `merged` | everything merged, `clean-task` still pending |
| `done` | no workspace, finished |

## When something gets stuck

- A `[blocker]` in `om-events` means an om-developer or om-reviewer couldn't continue due to missing permissions, credentials, environment, or tools, and neither the om-reviewer nor the om-manager could resolve it.
  It's the only thing that escalates the whole chain.
  Go into the project, look at the task's window, fix what's missing (an `.env`, a `gh` login), and tell the om-manager to continue.
- A session died: the state lives in the task's folder, in git, and in the PRs.
  `claude agents` lists live and stopped sessions; `claude attach {{id}}` reopens one with its context.
  If it's completely lost, `/delegate-task` launches a new om-reviewer that reads `Context & decisions` and continues.
- Orphan workspaces or branches: `/clean-work` reports them and asks before deleting anything it doesn't recognize.

## Changing how the agents work

In the overmind's `config` window, session `om-config`:

`/update-method rename retakes.md to feedback.md`

It applies the change in every agent, skill and document where it applies, runs `scripts/lint-method`, notes why in `docs/method-ard.md`, and commits.
If you ask for a method change in another session, it sends you here.

## What you will see in every project

```
CLAUDE.md                       short: stack, commands, "the docs live in docs/"
docs/PRD.md  TRD.md  ARD.md     product, technical, decisions, and debt
docs/modules/{{module}}/         README, prd, trd, ard, database, flows
docs/tasks/{{id}}_{{title}}/     task.md, replication.md, phase_N.md, retakes.md, verify.log
.workspaces/{{id}}_{{title}}/    one worktree per repo, ignored by git
```

Branches: `feat/0142_badge_wall`, `bugfix/0143_fix_login`, with `-phase-N` if there are phases.
Sessions: `om-diy-manager`, `om-0142-reviewer`, `om-0142-developer`.
