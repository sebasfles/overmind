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

## 2026-08-31: setup treats existing documentation as claims, not facts

- Decision: in `setup`'s discovery, existing documentation is a set of claims: claims the code can verify (entities, endpoints, commands, flows) are checked against the code before being folded in, unverifiable claims (intent, reasons, audience) are folded in citing source and date, and doc-vs-code contradictions travel in the discover schema (`contradictions: [{doc_claim, code_evidence}]`), appear in the step 3 report, and are arbitrated by Sebastian, never resolved silently. The rule is echoed in `om-setup-worker`'s discover mode.
- Alternatives rejected: trusting docs by freshness alone (a recent doc can still be wrong), letting the worker resolve contradictions (it would silently pick a side with component-local context), verifying every claim including intent (intent is not verifiable against code; only facts are).
- Reason: the skill already ranked sources (code primary for the TRD, inventory for PRD intent and ARD reasons) but "when a fact is confirmed" never said how; a stale README could reach the PRD unchallenged because the inventory is its primary source.
- Debt created: discovery gets slower on projects with much prose documentation, since verifiable claims now require a grep against the code.
- Revisit when: the pilot shows contradiction lists too long to arbitrate one by one, or workers flagging cosmetic differences as contradictions.
- Files: skills/setup/SKILL.md, agents/om-setup-worker.md, docs/03-skills.md, docs/usage-guide.md.

## 2026-08-31: plan, create, consolidate and delegate are om-manager-invocable; Sebastian's yes is the trigger

- Decision: `plan-task`, `create-task`, `consolidate-task` and `delegate-task` drop `disable-model-invocation`, so the om-manager chains the task flow as one conversation: each skill runs when Sebastian asks for it or answers yes to the question that offers it, never on the om-manager's initiative, and the question is never skipped. `setup`, `reiterate-task`, `clean-task` and `clean-work` stay user-only. Supersedes the 03-skills point 3 rule that marked consolidate-task and delegate-task as invocable only by the user.
- Alternatives rejected: keeping the slash-only flow (Sebastian retypes a command the conversation already agreed on, which is the friction this removes), making every om-manager skill model-invocable (reiterate and clean are corrections and destructive cleanup, where the explicit command is the safety).
- Reason: the flow already advances through the om-manager's questions ("consolidate it now?", "delegate now?"); requiring a slash command after each yes duplicated the confirmation without adding safety.
- Debt created: the guardrail moves from the harness (disable-model-invocation) to the agent's rules; a sloppy om-manager could read enthusiasm as a yes.
- Revisit when: an om-manager runs one of the four without an explicit yes in the conversation.
- Files: skills/plan-task/SKILL.md, skills/create-task/SKILL.md, skills/consolidate-task/SKILL.md, skills/delegate-task/SKILL.md, agents/om-manager.md, docs/03-skills.md.

## 2026-08-31: The method requires user-level allow rules for agent-to-agent launching; install checks, never writes

- Decision: the method depends on `permissions.allow` rules in `~/.claude/settings.json` covering `claude --agent/--bg/attach/-r/--resume/stop/agents` and the tmux window and pane commands, because in auto mode the classifier blocks the om-manager's launch of om-reviewers mid `consolidate-task`. `scripts/install` verifies them and prints the missing ones with the exact JSON to paste; it never writes them, because granting permissions is Sebastian's act. `usage-guide.md` lists them in Requirements.
- Alternatives rejected: install writing the rules itself (a script silently widening permissions defeats the classifier's intent, and om-config's own classifier blocked exactly that today), per-project `.claude/settings.json` rules (repeated per project and missed on every new one), keeping `--allow-dangerously-skip-permissions` retries until the classifier yields (it does not; the block is deterministic).
- Reason: the pilot hit it live on 2026-08-31: auvral's `consolidate-task` finished the workspace but could not launch the om-reviewer in either form (`claude --bg`, `tmux send-keys`); explicit allow rules are the supported path around the classifier.
- Debt created: the required list lives in two places (Sebastian's settings and the install check) and must be kept in sync when a skill starts using a new launch command; `Bash(tmux send-keys *)` is a wide grant that can type into any pane.
- Revisit when: a skill adds a launch command the list does not cover, or an agent misuses `tmux send-keys` outside launching peers.
- Files: scripts/install, docs/usage-guide.md.

## 2026-08-31: The bypass-permissions disclaimer is a machine requirement; install checks, never accepts

- Decision: om-reviewer and om-developer running with `permissionMode: bypassPermissions` requires the bypass disclaimer accepted once per machine (`bypassPermissionsModeAccepted` in `~/.claude.json`); without it, `--bg` launches fail non-interactively and the sessions fall back to guarded modes whose classifiers block `SendMessage` between them. `scripts/install` checks the flag and prints the one-time command (`claude --dangerously-skip-permissions`, accept, exit); it never accepts for Sebastian, because the disclaimer is interactive by design. `usage-guide.md` lists it in Requirements.
- Alternatives rejected: install accepting the disclaimer by writing the flag (defeats an interactive safety acknowledgment), dropping `bypassPermissions` from om-reviewer and om-developer (the 2026-08-29 entry already weighed that; workspace isolation is the mitigation and the flow cannot stop for permission prompts), keeping the send-keys workaround the om-reviewer improvised (it types into panes instead of using the message protocol).
- Reason: the pilot hit it live on 2026-08-31 in auvral's `start-task`: the `--bg` launch failed with "bypass disclaimer not accepted" and the om-reviewer's classifier then blocked `SendMessage`, forcing a fragile send-keys workaround.
- Debt created: the check greps for a key name that Claude Code may rename across versions; the message says so and the functional symptom (failed `--bg` launch) remains the ground truth.
- Revisit when: the key name changes, or Claude Code offers a supported non-interactive way to verify the acceptance.
- Files: scripts/install, docs/usage-guide.md.

## 2026-08-31: The bypass acceptance check is functional, not a key grep

- Decision: `scripts/install` verifies the bypass-permissions acceptance by launching one throwaway `claude --bg --dangerously-skip-permissions` haiku session and cleaning it up (`claude stop` and `claude rm`), instead of grepping `bypassPermissionsModeAccepted` in `~/.claude.json`. Supersedes the check mechanism of the previous entry; the requirement itself stands.
- Alternatives rejected: the key grep (Sebastian accepted the disclaimer and the key exists in no state file this version writes; searched `bypass`, `dangerous`, `skip`, `accepted` across `~/.claude.json`, settings and policy files), asking Sebastian to test by hand each machine (the script exists to do that).
- Reason: the functional test is the ground truth the pilot exposed: the grep said MISSING on a machine where the `--bg` bypass launch demonstrably works.
- Debt created: each `scripts/install` run spends one haiku turn and a few seconds on the probe; run from inside an agent session the probe may be blocked by that session's classifier and report a false MISSING, so the script is for Sebastian's shell.
- Revisit when: Claude Code exposes a supported way to query the acceptance without launching a session.
- Files: scripts/install, docs/usage-guide.md.

## 2026-08-31: The summary is the root PR's description, not a comment

- Decision: `publish-task` writes the summary (Intent, What changed, Decisions, Risk assessment, Pipeline) as the root PR's description with `gh pr edit --body-file`, rewriting it in place on later rounds; the root PR is created with a one-line placeholder body, code PRs link to it, and no summary comment exists. The template renames to `templates/pr-summary.md` and drops the `<!-- reviewer-summary -->` marker, which existed only to find the comment.
- Alternatives rejected: keeping the editable comment (Sebastian reads PRs from the description; a body-empty PR with the substance in a comment reads backwards in every GitHub surface, including notifications and merge screens), summary in both places (two copies drift).
- Reason: Sebastian wants the PR description to carry the decision material; the description is what GitHub shows first and what the merge commit can inherit.
- Debt created: none; `gh pr edit` needs the same auth `gh pr comment` needed.
- Revisit when: a task needs per-round history visible in the PR, which the rewritten description no longer shows (the rounds remain in the commits and in `verify.log`).
- Files: skills/publish-task/SKILL.md, skills/publish-task/templates/pr-summary.md, agents/om-reviewer.md, agents/om-manager.md, docs/03-skills.md, docs/usage-guide.md, README.md, scripts/lint-method.

## 2026-08-31: PR summary format: scannable lines, detail collapsed

- Decision: the PR summary template writes Intent at goal level (two or three sentences, no implementation detail), one concise line per bullet in What changed and Decisions, and a Pipeline of bare `✅` lines (never GitHub task checkboxes) with the review line as `{{k}} issues auto-fixed` and `documentation` and `push` as bare passed lines; every longer detail (commands, targets, findings, shas, modules) lives only in a collapsed `<details>` block. Risk assessment keeps exactly `Low | Medium | High`. `lint-method` allows `<details>` and `<summary>` tags.
- Alternatives rejected: keeping the long inline format (the first real PR summary of the pilot was too long to read; density per line is what kills it), dropping the detail entirely (the findings and shas matter when something looks off; collapsing keeps them one click away).
- Reason: Sebastian reviewed auvral's first published summary on 2026-08-31 and asked for exactly this shape; the summary must let him decide in one screen, with depth opt-in.
- Debt created: none.
- Revisit when: a summary's collapsed detail is systematically ignored (drop it) or systematically opened (some line deserves promotion).
- Files: skills/publish-task/templates/pr-summary.md, skills/publish-task/SKILL.md, scripts/lint-method.

## 2026-08-31: Risk assessment carries a severity icon

- Decision: the Risk assessment value in the PR summary is written with its icon: `✅ Low`, `⚠️ Medium`, `🔴 High`.
- Alternatives rejected: text-only values (the icon makes severity readable at a glance, consistent with the ✅ pipeline lines).
- Reason: Sebastian asked for icon-coded severity after reviewing the pilot's first summary.
- Debt created: none.
- Revisit when: never, unless the severity scale itself changes.
- Files: skills/publish-task/templates/pr-summary.md, skills/publish-task/SKILL.md.

## 2026-08-31: SendMessage joins the required allow rules

- Decision: the required `permissions.allow` list gains the `SendMessage` tool; `scripts/install` checks it and `usage-guide.md` Requirements names it.
- Alternatives rejected: relying on bypass mode alone (the om-manager runs in auto mode by design, and om-reviewers born before the bypass acceptance stay guarded until recycled; both send protocol messages), per-target approvals (the protocol sends between ephemeral task sessions, so a one-off approval never generalizes).
- Reason: the pilot hit it on 2026-08-31 in auvral's task-0002: the om-reviewer's `context ready, start` to the om-developer was denied twice by the classifier; the existing rules only covered Bash commands and the protocol's messaging is a tool.
- Debt created: `SendMessage` is allowed globally for Sebastian's sessions, including sends to sessions outside the method.
- Revisit when: permission rules learn to scope SendMessage by target pattern (then scope it to `om-*` and `overmind`).
- Files: scripts/install, docs/usage-guide.md.

## 2026-08-31: om-events runs bypassed

- Decision: `om-events` moves from `permissionMode: acceptEdits` to `bypassPermissions`.
- Alternatives rejected: auto mode (unavailable on haiku, the model om-events runs on), keeping acceptEdits (it only auto-accepts file edits, so any Bash call, even the ISO timestamp, prompts in a pane nobody watches and the inbox hangs silently), upgrading to sonnet for auto mode (pays reasoning for a one-line append job).
- Reason: an unattended inbox cannot stop to ask; its writes are appends under `portfolio/events/` and its inputs come from Sebastian's own om-managers.
- Debt created: om-events holds Bash with no permission gate; the mitigation is its narrow rules (append one line, print counts, nothing else).
- Revisit when: an event's content ever makes om-events run something beyond filing and counting.
- Files: .claude/agents/om-events.md.

## 2026-09-01: Events and todos split open from done by file

- Decision: `portfolio/events/{blockers,actions,info}.md` and `portfolio/todos.md` hold only open items as plain lines without checkboxes; being in the file means open, and counting open items is counting lines. Resolving moves the line: events to `portfolio/events/done.md` as `- {{ISO timestamp}} [{{type}}] {{project}} {{id}} {{event}} (done {{YYYY-MM-DD}})` (moved by the `overmind` session, never by om-events), todos to `portfolio/todos-done.md` via `complete-todo`. The `## Open` and `## Done` sections of `todos.md` disappear.
- Alternatives rejected: checking lines off in place (hot files grow forever mixing open and resolved, and om-events must distinguish `[ ]` from `[x]` to count), one file per event or todo (one-line items; per-item files multiply creation, listing and cleanup with no reader gain).
- Reason: Sebastian wants the hot files scannable; reading a file should be reading only what is open, and om-events on haiku gets a dumber, more reliable job.
- Debt created: the existing `portfolio/` files still have the old format; migrating them (strip checkboxes, move `[x]` lines to the done files) is the `overmind` session's task, not the method's.
- Revisit when: `done.md` or `todos-done.md` grow enough to need rotation by year.
- Files: .claude/agents/om-events.md, .claude/agents/overmind.md, skills/add-todo/SKILL.md, skills/complete-todo/SKILL.md, docs/04-operation.md, docs/02-orchestration.md, CLAUDE.md.

## 2026-09-01: Token economy: verify once at the gate, read by sections, right-size models, recycle on clean

- Decision: six changes driven by the 2026-09-01 usage data ($171, 62% of usage above 150k context, 25% from om-setup-worker, fable at 81% weekly). (1) `review-task` audits `verify.log` on intermediate rounds (block at the round's sha, green, every touched target) and runs `verify-task` itself only as the publish gate, cutting verification from 2N runs per task to N+1. (2) `verify-task` returns only per-target status and the last 20 lines of failures; the log is the archive. (3) om-developer reads large files by sections with offset and limit. (4) `clean-task` ends suggesting recycling the om-manager. (5) `om-setup-worker` moves to sonnet for both modes, superseding the 2026-08-29 note that only discover ran on sonnet. (6) `start-task` launches the om-developer with `--model sonnet` for `docs` and `chore` tasks.
- Alternatives rejected: downgrading om-manager or om-reviewer models (planning and review are where an error costs a bad PR merged), dropping the om-reviewer's independent verification entirely (the publish gate keeps one untrusted-input check before any PR), compaction tuning (lossy; recycling plus disk-derived state is already the design).
- Reason: the pilot's first full task showed both task sessions near 300k context, verification output entering two contexts twice per round, and whole-file reads the harness itself flagged as 1.3m tokens readable for 377k.
- Debt created: a lying or broken `verify.log` on an intermediate round is caught one round later or at the gate, not immediately; sonnet setup-workers may produce docs needing one more correction pass in the inferidos review.
- Revisit when: the gate catches a red that an intermediate audit accepted (tighten the audit), or sonnet-written module docs need systematic rework (raise document mode back).
- Files: skills/review-task/SKILL.md, skills/verify-task/SKILL.md, skills/clean-task/SKILL.md, skills/start-task/SKILL.md, agents/om-reviewer.md, agents/om-developer.md, agents/om-setup-worker.md, docs/03-skills.md.

## 2026-09-01: Code comments only when the code cannot say it

- Decision: the om-developer writes code comments only when absolutely necessary, which is almost never: a non-obvious invariant or an external workaround the code cannot express. `execute-task` step 5 states the rule, om-developer's Judgment carries it, and `review-task` step 4 treats a comment that restates the code as a finding.
- Alternatives rejected: banning comments outright (an external workaround or a non-obvious invariant sometimes has no other home in the code), leaving it to model defaults (models over-comment; the pilot's diffs showed narration comments the code already said).
- Reason: Sebastian wants clean diffs; explanation already has designated homes in the module docs and the ARD through `document-task`, so comments in code duplicate what the method stores elsewhere.
- Debt created: none.
- Revisit when: a review round shows a real defect that a comment would have prevented and neither the docs nor the ARD could have held.
- Files: skills/execute-task/SKILL.md, agents/om-developer.md, skills/review-task/SKILL.md.

## 2026-09-01: The om-reviewer never verifies; documentation happens once, before publish

- Decision: two cuts to the round loop, decided by Sebastian on the 24h usage data. (1) The om-reviewer never runs `verify-task`, lint, typecheck or tests, on any round, publish gate included; it audits the om-developer's `verify.log` (block at the round's commit, green, every touched target) and a missing, stale or red block is a finding. `verify-task` becomes om-developer-only. (2) `document-task` runs once per task or phase instead of every round: on a clean Pipeline the om-reviewer sends `round {{N}} clean, document`, the om-developer documents the whole diff (decisions accumulated in `om-developer notes` become the ARD entries), commits `docs({{modules}}): {{id}} task docs` on top, and replies `docs ready, commit {{sha}}`; `publish-task` checks the docs (`updated`, `source`, ARD entries) as its first step, before pushing.
- Supersedes: 2026-09-01 "Token economy" items 1 and 2 in part (the publish-gate run of `verify-task` by the om-reviewer is removed) and the original per-round `document-task` design in `docs/01-documentation.md`.
- Alternatives rejected: keeping the publish-gate run (Sebastian judged the om-developer's log sufficient; the gate doubled the heaviest command output into a second context once per task), documenting on the last code round (the om-developer cannot know a round is last; only the om-reviewer's clean signal marks it), keeping the docs check in `review-task` (the docs commit does not exist yet when the Pipeline runs; the check belongs where the docs already exist, at publish).
- Reason: verification and documentation together were 6% of usage, close to `execute-task`'s 9%; doc reading and writing repeated N times per task and verification entered the om-reviewer's context once more at the gate, for work the om-developer had already done and logged.
- Debt created: no independent execution of lint or tests happens before a PR; a `verify.log` that lies (or a broken local toolchain) reaches Sebastian's review undetected. Docs arrive in one commit at the end, so a task stopped mid-flight has code without docs.
- Revisit when: a merged PR turns out red on CI or a lying `verify.log` is caught after publish (restore an independent gate), or end-of-task documentation is systematically thinner than the per-round version was.
- Files: skills/execute-task/SKILL.md, skills/review-task/SKILL.md, skills/document-task/SKILL.md, skills/verify-task/SKILL.md, skills/publish-task/SKILL.md, agents/om-developer.md, agents/om-reviewer.md, agents/om-manager.md, docs/01-documentation.md, docs/02-orchestration.md, docs/03-skills.md, docs/usage-guide.md.

## 2026-09-01: clean-task is model-invocable on Sebastian's ask

- Decision: `clean-task` drops `disable-model-invocation: true` (edited by Sebastian directly in the skill); the om-manager can invoke it when Sebastian asks in plain words, without needing the slash command. It still runs only on his explicit ask, never on the om-manager's initiative; `agents/om-manager.md` ("wait for his explicit ask") is unchanged and `docs/03-skills.md` point 3 now states the split.
- Alternatives rejected: keeping it slash-only (asking "clean up task 12" in conversation failed to load the skill, pure friction), making it autonomous after merge (destructive cleanup of sessions, branches and worktrees stays behind Sebastian's word).
- Reason: the flag blocked the natural way Sebastian asks for cleanup; the safety he wants is "only when I say so", not "only through one syntax".
- Debt created: none.
- Revisit when: an om-manager runs `clean-task` without an explicit ask; then the flag comes back.
- Files: skills/clean-task/SKILL.md, docs/03-skills.md.

## 2026-09-01: Model references are bare aliases, never versioned ids

- Decision: every model reference in the method (`model:` frontmatter in agents, `--model` flags in skills) uses the bare alias (`fable`, `opus`, `sonnet`, `haiku`), never a versioned id like `fable-5.1`. The alias resolves to the latest release on its own; a model upgrade (fable 5 to 5.1 today) requires no edit. Recorded as a writing convention in `update-method`.
- Alternatives rejected: pinning versioned ids (every release forces a repo-wide edit for zero behavior gain), pinning only the expensive models (same maintenance, split rule).
- Reason: Sebastian asked to update the skills for fable 5.1 and the grep showed nothing to update; making the convention explicit keeps it that way.
- Debt created: no way to hold a role on an older model if a release regresses; if that happens, the pin is the exception and gets its own ARD entry.
- Revisit when: a model release degrades a role's output and an explicit pin becomes necessary.
- Files: .claude/skills/update-method/SKILL.md.
