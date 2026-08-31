# ARD of the method

Log of decisions about how the agents work.
It stacks; it never gets rewritten.
The original design decisions are in `01` through `05`; here go the later changes and their reasoning.

## 2026-08-29: Roles with the `om-` prefix

- Decision: the agents are called `overmind`, `om-manager`, `om-reviewer`, `om-developer`, `om-setup-worker`; the sessions `om-{{project}}-manager`, `om-{{id}}-reviewer`, `om-{{id}}-developer[-phase-n]`.
- Alternatives rejected: `-agent` suffix; names without a prefix, always writing "the {{role}} agent" in prose.
- Reason: without a prefix, the three role names read as people in the prose of 27 skills and in the session lists.
- Debt created: none.
- Revisit when: never, unless another agent system shows up on the same machine with the same prefix.
- Files: agents/, skills/, docs/, scripts/resume-project.

## 2026-08-29: Standalone, not a plugin

- Decision: skills and agents live in this repo and are linked via symlinks from `~/.claude/`; it is not packaged as a plugin.
- Alternatives rejected: a plugin in a personal marketplace.
- Reason: one person, one machine; a plugin charges a prefix on every invocation and a reload step without giving anything a repo with symlinks doesn't already give.
- Debt created: the conversion to a plugin remains pending if a second machine or person shows up.
- Revisit when: two machines, two people, or name collisions with project plugins.
- Files: docs/04-operation.md.

## 2026-08-29: Project skills to maintain the method

- Decision: `update-method` (in this repo's `.claude/skills/`) applies cross-cutting changes, and `scripts/lint-method` checks the mechanical parts.
- Alternatives rejected: separate `update-skill` and `update-agent`; continuing to do the passes by hand.
- Reason: each change to the method touched between 20 and 44 files; without a procedure and a lint, something ends up half-done.
- Debt created: none.
- Revisit when: the lint starts giving false positives that cost more to maintain than what it catches.
- Files: .claude/skills/update-method/, scripts/lint-method, docs/method-ard.md.

## 2026-08-29: `setup` extracts, it doesn't wait for the convention; modules as Workflow

- Decision: `setup` treats the convention as output, never as a precondition: it inventories documentation wherever it is and in whatever format, cross-checks it against the code and with Sebastian, and produces `docs/`. Per-module documentation runs as a Workflow (parallel with a cap of 4, schema'd results, resumable), with `Agent(om-setup-worker)` as fallback.
- Alternatives rejected: requiring the `docs/` structure before running; making the whole skill a Workflow (the interactive parts can't run in the background); sticking with standalone subagents only.
- Reason: existing projects have scattered docs and none of them has the convention; and the per-module fan-out is the only work in the system with N independent jobs where resume and schema pay off.
- Debt created: the skill instructs writing the Workflow script at run time; it's worth saving a reference script in `skills/setup/references/` after the pilot.
- Revisit when: the pilot shows that the Workflow adds friction compared to subagents.
- Files: skills/setup/SKILL.md, docs/03-skills.md.

## 2026-08-29: `setup` with two fan-outs: discovery per component and documentation per module

- Decision: discovery runs in parallel per component (repo or app) with `om-setup-worker` in `discover` mode (read-only, schema'd output); the main session merges the results, proposes the folders → modules map and confirms it with Sebastian; then documentation runs in parallel per module with the same agent in `document` mode. These are two separate Workflows.
- Alternatives rejected: a single Workflow (it can't stop to ask questions); sequential discovery in the main session (in `hux` that's 10 repos, and it fills up the context before writing anything).
- Reason: the modules don't exist until discovery finishes and Sebastian confirms them; and the modules that cross components only show up in the merge, which is human work plus the main session, not a worker's.
- Debt created: none.
- Revisit when: the pilot on a multirepo shows that merging candidates needs more structure in the schema.
- Files: skills/setup/SKILL.md, agents/om-setup-worker.md, docs/03-skills.md.

## 2026-08-29: The om-manager forwards events to the overmind, not messages

- Decision: the om-manager forwards to the `overmind`, in one line and only if it's running, every message between sessions that changes a task's state or blocks it (consolidated, PRs ready, phase merged, retake sent, cleaned, errors). It does not forward the handoff of consolidation questions or content.
- Alternatives rejected: forwarding every message received (noise in the cockpit during consolidation); forwarding only "PR ready" (blockers stayed invisible until Sebastian entered the project).
- Reason: the cockpit exists to say where attention is needed; blockers are exactly that.
- Debt created: none.
- Revisit when: the event list grows and a structured format becomes worth it instead of one line.
- Files: agents/om-manager.md, agents/overmind.md.

## 2026-08-29: Three sessions in the overmind, typed events, single escalation

- Decision: the cockpit is split into `overmind` (state and projects), `om-events` (inbox, `om-events` agent on haiku) and `om-config` (method, `om-config` agent). The events the om-managers send are typed (`blocker`, `action`, `info`) and get archived in `portfolio/events/`, stacking with counters. The only message that goes up the whole chain is a blocker over permissions, credentials, environment or tools; om-reviewer and om-developer run with `bypassPermissions`. The three cockpit agents are project-scoped (`.claude/agents/`); the four project roles stay global.
- Alternatives rejected: a single overmind session with all three jobs (mixes context and makes noise with events); global cockpit agents (they would show up as delegable subagents in every om-manager); removing `update-method` now that `om-config` exists (the agent is who, the skill is the invocable procedure).
- Reason: every long-lived session needs a single reason to exist; and the separate inbox lets events be seen without interrupting the conversation with the cockpit.
- Debt created: `bypassPermissions` removes the classifier's safety net from om-reviewer and om-developer; the mitigation is workspace isolation and each agent's hard rules.
- Revisit when: an om-developer does something destructive outside its workspace, or the inbox needs more than three types.
- Files: .claude/agents/, agents/om-manager.md, agents/om-reviewer.md, agents/om-developer.md, bin/resume-overmind, scripts/lint-method, portfolio/events/, docs/04-operation.md, docs/03-skills.md, .claude/skills/update-method/SKILL.md, CLAUDE.md.

## 2026-08-29: Singleton sessions named after the agent; models and effort per role

- Decision: the cockpit sessions are named after their agent (`overmind`, `om-events`, `om-config`) because they're singletons; roles with multiple instances keep instance names (`om-diy-manager`, `om-0142-reviewer`). `overmind` drops to opus/medium (it aggregates and routes); `om-setup-worker` in discover mode runs on sonnet inside the Workflow; skills carry `effort` according to their nature: xhigh for analysis and review, high for planning and writing, medium for handoffs, low for the mechanical ones.
- Alternatives rejected: session names different from the agent for everything (`overmind-events`), a single effort for all skills.
- Reason: agent and session are different things and that's confusing; making them match where there's a single instance removes one name to remember. The model tracks the cost of the role's error and the effort tracks the reasoning the task calls for.
- Debt created: none.
- Revisit when: the pilot shows a mechanical skill that needs more reasoning, or an analysis one that doesn't make use of it.
- Files: .claude/agents/, skills/*/SKILL.md, docs/04-operation.md, bin/resume-overmind.

## 2026-08-29: Only resume-overmind goes into ~/bin

- Decision: `scripts/install` links only `resume-overmind` into `~/bin`. `resume-project` is run by the `overmind` session from the repo, `lint-method` by `om-config` through `update-method`, `install` once per machine.
- Alternatives rejected: linking every script (puts commands in the PATH that only agents run).
- Reason: each script has one operator; the PATH should hold only what Sebastian types.
- Debt created: none.
- Revisit when: Sebastian finds himself opening projects without the cockpit often enough to want `resume-project` in the PATH.
- Files: scripts/install, CLAUDE.md, README.md, docs/usage-guide.md, docs/04-operation.md.

## 2026-08-29: bin/ is what gets linked, scripts/ is internal

- Decision: `bin/` holds only commands Sebastian types and `scripts/install` links all of it into `~/bin`; `scripts/` holds `install`, `resume-project` and `lint-method`, run by agents or from the repo.
- Alternatives rejected: one `bin/` with a hand-kept list of what to link.
- Reason: one folder per operator; the install script needs no list.
- Debt created: none.
- Revisit when: never, unless a script needs both operators.
- Files: bin/, scripts/, scripts/install, CLAUDE.md, README.md, docs/.

## 2026-08-31: resume-project pins the om-manager window at index 1

- Decision: `scripts/resume-project` guarantees the `om-manager` window at tmux index 1 (Sebastian's `base-index` is 1): new sessions create it there, and existing sessions are repaired by inserting or moving it to 1 with `-b`, shifting other windows up; the script selects window 1 before attaching.
- Alternatives rejected: only creating the window on new sessions (an existing session with the om-manager window missing or displaced stayed wrong), swapping windows instead of shifting (moves an unrelated window to an arbitrary index).
- Reason: Sebastian navigates by number and expects the project's om-manager always on window 1; the same repair pattern already proved itself in `resume-overmind`.
- Debt created: the index is fixed at 1 and assumes `base-index 1`; with `base-index 0` the om-manager window still lands at 1 and index 0 stays occupied by another window.
- Revisit when: a project session needs a fixed window other than the om-manager, or the tmux config changes `base-index`.
- Files: scripts/resume-project, skills/resume-project/SKILL.md, docs/04-operation.md, docs/usage-guide.md, docs/06-pilot.md.

## 2026-08-31: The plan draft is always on disk; the om-manager session is disposable

- Decision: `plan-task` writes `docs/tasks/_drafts/{{title}}.md` from the first plan-shaped turn and updates it every turn, with no size condition; on "not now" after approval the draft is the persistence, retakable by `create-task` in any session. `agents/om-manager.md` gains the disposable-session rule: anything decided in conversation lands on disk in the same turn, and with heavy context the om-manager recommends recycling (`/clear`, or kill plus `resume-project`) instead of relying on autocompaction.
- Alternatives rejected: keeping the "when the conversation grows" condition (the om-manager's judgment of "grown" is exactly what a compaction can arrive before), persisting the whole conversation instead of the plan's current state (noise; the draft is what gets retaken), fighting context limits with compaction tuning (lossy and silent).
- Reason: every durable state already derives from disk (`check-task`, `check-work`, idempotent `analyze-task`); the only loss window was a plan not yet drafted. Closing it makes recycling the om-manager cost a `check-work` plus a look at `_drafts/`, so context pressure stops being a design concern.
- Debt created: one draft write per planning turn, negligible; drafts abandoned mid-plan stay in `_drafts/` until `check-work` surfaces them.
- Revisit when: the pilot shows draft updates adding friction to planning, or a recycled om-manager missing context that was neither in `_drafts/` nor in `docs/tasks/`.
- Files: skills/plan-task/SKILL.md, agents/om-manager.md, docs/02-orchestration.md, docs/usage-guide.md.

## 2026-08-31: Recycling is respawn-pane, never /clear; scripts/install owns the tmux binding

- Decision: recycling an agent session is `tmux respawn-pane -k` in its pane, bound to `prefix + R` by `scripts/install` (idempotent append to `~/.tmux.conf`, skipped if a `bind R` exists); tmux relaunches the pane's original command, so the fresh session keeps agent, name and cwd. `/clear` is banned from the om-manager's recycle advice because it starts a session without the name the protocol addresses. `resume-overmind` and `resume-project` stay as the repairers when the window or the tmux session is gone. Supersedes the `/clear` mention in the disposable-session entry of 2026-08-31.
- Alternatives rejected: `/clear` (loses the session name), `respawn-window -k` (kills sibling panes in the split cockpit window), renaming the old session to free the name (no supported command, and verified unnecessary: exited sessions are unreachable by SendMessage and names resolve only to live sessions), a separate install.md documenting the binding (a doc humans follow by hand drifts; the once-per-machine script is the single setup point).
- Reason: verified on 2026-08-31: `-n` always creates a new session, an exited session is not reachable by name, and with dead and live sessions sharing a name the message reaches only the live one; `respawn-pane` preserves per-pane original commands even in split windows.
- Debt created: `scripts/install` now writes to `~/.tmux.conf`, a file it does not own; the guard only checks for an existing `bind R`, not for the exact command.
- Revisit when: a machine uses a tmux prefix or binding scheme where `R` conflicts, or the pilot shows old transcripts accumulating enough to want cleanup in `clean-task`.
- Files: scripts/install, agents/om-manager.md, docs/usage-guide.md, docs/04-operation.md.

## 2026-08-31: English filenames and placeholders everywhere

- Decision: the design docs are renamed to English (`01-documentation.md`, `02-orchestration.md`, `04-operation.md`, `06-pilot.md`) and every Spanish placeholder is translated (`{{proyecto}}` to `{{project}}`, `{{nombre}}` to `{{name}}`, `{{ruta}}` to `{{path}}`, `{{rama}}` to `{{branch}}`, `{{rol}}` to `{{role}}`, `{{modulo}}` to `{{module}}`, `{{tus comentarios}}` to `{{your comments}}`, `{{slug-del-directorio}}` to `{{directory-slug}}`); references in past ARD entries were updated mechanically. Along the way a bad earlier rename was fixed: "package om-manager" back to "package manager", with a lint exception so "package manager" is not flagged as a bare role word.
- Alternatives rejected: keeping the Spanish filenames as legacy (the English-only rule already existed; filenames were the leftover), redirect stubs at the old paths (nothing external links to them).
- Reason: the repo's convention is English in every file; the doc filenames and a handful of placeholders predated the rule.
- Debt created: none.
- Revisit when: never.
- Files: docs/01-documentation.md, docs/02-orchestration.md, docs/04-operation.md, docs/06-pilot.md, docs/03-skills.md, docs/05-layouts.md, docs/usage-guide.md, docs/method-ard.md, README.md, CLAUDE.md references unchanged, .claude/skills/update-method/SKILL.md, skills/setup/SKILL.md, skills/write-trd/SKILL.md, skills/write-trd/templates/TRD.md, scripts/lint-method.
