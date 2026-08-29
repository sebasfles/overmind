# ARD of the method

Log of decisions about how the agents work.
It stacks; it never gets rewritten.
The original design decisions are in `01` through `05`; here go the later changes and their reasoning.

## 2026-08-29: Roles with the `om-` prefix

- Decision: the agents are called `overmind`, `om-manager`, `om-reviewer`, `om-developer`, `om-setup-worker`; the sessions `om-{{proyecto}}-manager`, `om-{{id}}-reviewer`, `om-{{id}}-developer[-phase-n]`.
- Alternatives rejected: `-agent` suffix; names without a prefix, always writing "the {{rol}} agent" in prose.
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
- Files: docs/04-operacion.md.

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
- Files: .claude/agents/, agents/om-manager.md, agents/om-reviewer.md, agents/om-developer.md, bin/resume-overmind, scripts/lint-method, portfolio/events/, docs/04-operacion.md, docs/03-skills.md, .claude/skills/update-method/SKILL.md, CLAUDE.md.

## 2026-08-29: Singleton sessions named after the agent; models and effort per role

- Decision: the cockpit sessions are named after their agent (`overmind`, `om-events`, `om-config`) because they're singletons; roles with multiple instances keep instance names (`om-diy-manager`, `om-0142-reviewer`). `overmind` drops to opus/medium (it aggregates and routes); `om-setup-worker` in discover mode runs on sonnet inside the Workflow; skills carry `effort` according to their nature: xhigh for analysis and review, high for planning and writing, medium for handoffs, low for the mechanical ones.
- Alternatives rejected: session names different from the agent for everything (`overmind-events`), a single effort for all skills.
- Reason: agent and session are different things and that's confusing; making them match where there's a single instance removes one name to remember. The model tracks the cost of the role's error and the effort tracks the reasoning the task calls for.
- Debt created: none.
- Revisit when: the pilot shows a mechanical skill that needs more reasoning, or an analysis one that doesn't make use of it.
- Files: .claude/agents/, skills/*/SKILL.md, docs/04-operacion.md, bin/resume-overmind.

## 2026-08-29: Only resume-overmind goes into ~/bin

- Decision: `scripts/install` links only `resume-overmind` into `~/bin`. `resume-project` is run by the `overmind` session from the repo, `lint-method` by `om-config` through `update-method`, `install` once per machine.
- Alternatives rejected: linking every script (puts commands in the PATH that only agents run).
- Reason: each script has one operator; the PATH should hold only what Sebastian types.
- Debt created: none.
- Revisit when: Sebastian finds himself opening projects without the cockpit often enough to want `resume-project` in the PATH.
- Files: scripts/install, CLAUDE.md, README.md, docs/usage-guide.md, docs/04-operacion.md.

## 2026-08-29: bin/ is what gets linked, scripts/ is internal

- Decision: `bin/` holds only commands Sebastian types and `scripts/install` links all of it into `~/bin`; `scripts/` holds `install`, `resume-project` and `lint-method`, run by agents or from the repo.
- Alternatives rejected: one `bin/` with a hand-kept list of what to link.
- Reason: one folder per operator; the install script needs no list.
- Debt created: none.
- Revisit when: never, unless a script needs both operators.
- Files: bin/, scripts/, scripts/install, CLAUDE.md, README.md, docs/.
