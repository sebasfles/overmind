# Part 4: Daily operation, cockpit and portfolio

Status: partially agreed on 2026-08-28.
Sections marked "To be defined" have not been discussed yet.
Depends on: [02-orchestration.md](02-orchestration.md), [03-skills.md](03-skills.md).

## Cockpit

A single place where Sebastian sees all projects.
It's a tmux session `overmind` in the root of the `~/dev/personal/projects/overmind` repo, with three long-lived Claude sessions:

| Session | Agent | Where | For what |
|---|---|---|---|
| `overmind` | `overmind` | `overmind` window, left pane | cockpit: project status, todos, events; opens projects |
| `om-events` | `om-events` (haiku) | `overmind` window, right pane | inbox: receives events from the om-managers, archives them in `portfolio/events/`, prints the stacked counters |
| `om-config` | `om-config` | `config` window | maintain the method with `update-method` |

`bin/resume-overmind` opens the complete session; if `overmind` starts and `om-events` is not running, it opens or resumes it in the right pane.
The three agents live in the repo's `.claude/agents/` (project scope), because they only run here; the four roles that run inside projects live in `agents/` with a global symlink.

Routing rule: questions about projects, status or notifications go to `overmind`; changes to the method go to `om-config`.
If one is requested in the other's session, that session opens or resumes the correct one and forwards the request in one line.
The name is deliberately different from "om-manager" so it's never confused with a project's.

### Rule

The cockpit observes, aggregates, notifies and takes Sebastian to the right place.
It never plans or decides on a project.
Product, architecture and technical conversations stay with each project's om-manager, in its tmux session.

Reason: the om-manager's value comes from its deep context of a project.
An overmind with decision-making power would have shallow context of all of them, would decide worse and would add a hop where information gets lost.
Planning with the om-manager is the highest-value conversation in the flow and is not intermediated.

### What it does

- Aggregated status: `check-work` for each registered project, in a single table.
  One line per item, no explanations; detail is seen in the project's session or in the PR.
  It comes from disk (`docs/tasks/*.md`), git and `gh`; it doesn't ask the om-managers.
- Events: the om-managers notify it via `SendMessage` when something requires Sebastian (PR ready, task stuck in delegation).
- Cross-project cleanup: `clean-work` across all projects.
- Documentation drift: compares each module's docs `updated` field with the latest commits that touched that module.
  It's the check that was decided not to do with hooks; here it's a read, not a block.
- Maintaining the method: from `om-config`, with `update-method`.

Later, prioritization between projects ("what do I tackle today?") can live here, as a decision made by Sebastian informed by data, not by the agent.

`audit-portfolio` is not created: doc drift and accumulated clutter are columns of `check-portfolio`.
It would only be split out if the report becomes too long for the daily dashboard.

## Project registry

The cockpit doesn't guess by walking `~/dev`; it only considers registered projects.
File `portfolio/projects.yaml` in the `overmind` repo:

```yaml
projects:
  - name: diy
    root: /home/fless/dev/designli/projects/diy
    tmux: diy
    status: active        # active | paused
    repos:
      - name: diy-platform
        base_branch: develop
      - name: diy-infra
        base_branch: main
```

See [05-layouts.md](05-layouts.md) for `root` and `repos` in single repo, monorepo and multirepo.

## Overmind skills

| Skill | What it does |
|---|---|
| `add-project` | Registers a project (`root`, `repos`) in `projects.yaml`; in multirepo it converts the folder into a docs repo (`{{name}}-docs`). Checks the Part 1 convention and, if not met, offers to run `setup` from its om-manager. |
| `add-project {{name}} pause` / `add-project {{name}} remove` | Takes it off the dashboard without deleting anything. |
| `resume-project` | Opens a new Windows Terminal tab, in WSL, at the project's root, inside its tmux session, with the om-manager running. |
| `check-portfolio` | Per-project table: tasks in delegation, in progress, in PR waiting on Sebastian, idle executors, doc drift, accumulated clutter. |
| `clean-portfolio` | `clean-work` across all active projects. |
| `add-todo` | Quick capture of something not to forget, optionally linked to a project. |
| `complete-todo` | Crosses off a todo. |

### resume-project

It is a deterministic shell script (`scripts/resume-project`); the skill is a thin wrapper that calls it with the project name.
It is not linked into `~/bin`: the `overmind` session runs it from the repo.

Environment verified on 2026-08-28: Windows Terminal (`WT_SESSION` set), `wt.exe` invocable from WSL, `Ubuntu` distro, tmux 3.4.
Sebastian already works with one tmux session per project (`diy`, `auvral`, `drive-now`).

Behavior:

1. Reads `projects.yaml` to get `root` and `tmux`.
2. If the tmux session doesn't exist, creates it with cwd at `root` and the window `om-manager` running `claude --agent om-manager -n om-{{name}}-manager`.
3. Guarantees the `om-manager` window at index 1: if the session already exists without it, or with it at another index, the script inserts or moves it to 1 with `-b`, shifting other windows up.
4. Selects window 1, opens the tab in the current Windows Terminal window and attaches to the session.

```bash
tmux has-session -t "$name" 2>/dev/null || \
  tmux new-session -d -s "$name" -c "$path" -n om-manager "claude --agent om-manager -n om-${name}-manager"

# repair an existing session: om-manager always at index 1
tmux move-window -b -s "$name:om-manager" -t "$name:1"
tmux select-window -t "$name:1"

wt.exe -w 0 new-tab --title "$name" \
  wsl.exe -d Ubuntu --cd "$path" \
  -- tmux attach -t "$name"
```

`-w 0` opens the tab in the current Windows Terminal window.
Attaching to a session already attached from another tab is valid; both reflect it.

### Recycling sessions

Agent sessions are disposable: everything durable lives on disk, so a session with heavy context is replaced, not compacted.
The gesture is `prefix + R` in the pane (`respawn-pane -k`, installed into `~/.tmux.conf` by `scripts/install`): tmux relaunches the pane's original command, so the fresh session keeps the agent, the name and the cwd.
`/clear` is not used: it starts a session without the name the protocol addresses.
The old conversation stays as an inert transcript; names always resolve to the live session, verified on 2026-08-31.
When the window or the tmux session itself is gone, the repairers take over: `resume-overmind` for the cockpit, `resume-project` for a project.

## Todos

A todo is a quick capture: something Sebastian jots down so as not to forget it and continues with what he was doing.
It may or may not be related to a project.
It may end up being a task, a decision, a conversation with someone, or nothing.
It's not classified when jotted down; capture friction must be minimal.

Rules:

- A todo can optionally be linked to a project.
  `check-portfolio` shows it in that project's row; global ones go in a separate section.
- What the status already shows (for example a PR in `in_review`) doesn't need to be noted, but it's not forbidden either.
- When a todo turns into work for agents, Sebastian jumps to the project's om-manager, plans it with `plan-task` and closes it with `complete-todo`.

Skills: `add-todo` to jot it down and `complete-todo` to cross it off.
Listing doesn't need a skill: `check-portfolio` does it.

Storage: `portfolio/todos.md`, plain Markdown, readable and editable without the agent.

## A typical day

1. Sebastian opens the cockpit and asks for `check-portfolio`.
2. Sees what's waiting for him: PRs in `in_review`, stuck delegations, pending todos.
3. `resume-project {{name}}` takes him to that project's om-manager.
4. Talks with the om-manager: approves PRs, plans new tasks, delegates.
5. Returns to the cockpit or jumps to another project.

## To be defined

### Agent content

`agents/om-manager.md`, `om-reviewer.md`, `om-developer.md`, `overmind.md` and `om-setup-worker.md`, in the `overmind` repo, with symlinks from `~/.claude/agents/`.
Written as a draft on 2026-08-29; see `03-skills.md`.
Each with: role description, allowed skills, allowed tools, permission mode, state machine (for om-reviewer and om-developer), and rules for what it never does.

### `~/OPINIONS.md`: dropped

Decided on 2026-08-28: it's not created.
The opinions that matter to Sebastian's agents are narrow (code, architecture, process) and already have three better homes:

| Opinion type | Where it lives |
|---|---|
| Short rule, always applies | `~/.claude/CLAUDE.md` |
| A role's criteria | `~/.claude/agents/{{role}}.md` |
| A project's decision | project's `ARD.md` |

A fourth place would fall out of sync with the other three.
The line in the global `CLAUDE.md` that told it to read `~/OPINIONS.md` has already been removed.

This gets revisited only if the agents repeatedly make wrong judgment calls and the fix doesn't fit in one line of `CLAUDE.md` or belong to a role.
In that case the file is created with real content, not inferred.

### Pilot

Project where the end-to-end flow gets validated.
Must validate: `--bg` + `attach` in tmux panes, `SendMessage` delivery between sessions, `setup` on an existing repo, and a complete task through to merge.
Project to be chosen.
