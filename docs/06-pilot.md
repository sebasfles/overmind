# Part 6: Pilot

Status: pending.
Project to be chosen (recommended: `auvral`, multirepo and personal).
Everything written in `01` through `05` is design; the pilot is where it gets corrected.
Each failure gets fixed in the skill or agent with `update-method` before moving on to the next step.

## Technical assumptions to validate

| Assumption | Where it's used | If it fails |
|---|---|---|
| `claude --agent {{role}}` loads agents from `~/.claude/agents/` (symlinks) and from the repo's `.claude/agents/` | all launches | move the cockpit ones to global |
| `claude --bg` + `claude attach` in a tmux pane gives the same experience as a direct session | consolidate, start-task, resume-overmind | alternative form documented in `02` |
| `SendMessage` between local sessions launched with `--bg` gets delivered and processed as a turn | all communication | direct sessions; if that fails too, an on-disk mailbox file |
| `--allow-dangerously-skip-permissions` with `--bg` leaves om-reviewer and om-developer without the classifier's blocks | consolidate, start-task | `permissionMode: auto` with an allowlist in settings |
| The Workflow tool is available inside an om-manager session and accepts `model` per step | setup (discovery and modules) | fallback `Agent(om-setup-worker)` already written |
| An `initialPrompt` that invokes a skill (`/check-work`, `/analyze-task`, `/check-portfolio`) runs on startup | om-manager, om-reviewer, overmind | send the first message by hand from whoever launches it |
| `wt.exe -w 0 new-tab ... wsl.exe --cd ... tmux attach` opens the tab in the current window | resume-project, resume-overmind | adjust Windows Terminal flags |

## Order

1. `scripts/install`: symlinks. Check that `claude --agent om-manager` starts in any directory and that `/plan-task` appears in `/help`.
2. `bin/resume-overmind`: the three sessions. Check that `overmind` opens `om-events` if it's missing.
3. `add-project {{project}}` from `overmind`: layout detection, `{{name}}-docs` repo if multirepo, registration.
4. `resume-project {{project}}`: tab, tmux, om-manager in window 1 with `check-work` saying "run setup first".
5. `setup`: discovery per component (Workflow), module confirmation, interview, per-module documentation (Workflow), TRD, PRD, ARD, `CLAUDE.md`, inferred items, commit. It's the hardest test.
6. One small, real task: `plan-task` → `create-task` → `consolidate-task` (this is where `--bg`, `attach`, `SendMessage` and the workspace bootstrap get validated) → `delegate-task` → `start-task` → one round → `publish-task` → merge → `clean-task`.
7. A second task in parallel with the first, to see two workspaces and two pairs at once.
8. A `reiterate-task` with a comment of yours on the PR.
9. If the project is multirepo, a task that touches two repos: two code PRs and the root PR with the summary.

## What to measure

- How many times you had to step in outside of `plan-task`, consolidation and merge.
- How many `[inferido]` from `setup` were wrong.
- How many om-reviewer findings were real and how many noise.
- Tokens per role and per task, to review models and effort.

## After the pilot

- Merge skills that were never invoked on their own (candidates: `create-task` into `plan-task`, `start-task` into `analyze-task` and `next-phase`).
- Save the Workflow script for `setup` in `skills/setup/references/`.
- Write `references/{{stack}}.md` for `execute-task` and `verify-task` with what the pilot's TRD declared.
- Create the final symlinks and delete `scripts/install` if it's no longer needed, or leave it for the second machine.
