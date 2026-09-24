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

## 2026-09-01: om-reviewer runs on opus; om-manager stays on fable

- Decision: `agents/om-reviewer.md` moves from `model: fable` to `model: opus`; `om-manager` stays on fable, `om-developer` stays on opus. Skill efforts are unchanged (`analyze-task` and `review-task` at xhigh).
- Supersedes: 2026-09-01 "Token economy", the rejected alternative "downgrading om-manager or om-reviewer models", for the om-reviewer only.
- Alternatives rejected: om-manager to opus with om-reviewer on fable (the om-manager is the cheap session: few turns, waits for Sebastian, its large prefix is paid as cache reads, and fable reads cost half of opus reads; planning and `setup` are the highest-leverage output per token, so fable earns its price there and not in the volume role), both to opus (planning quality is where a wrong Approach costs a whole task), keeping both on fable (two tasks on 2026-09-01 measured the om-reviewer at $22 and $29 per task against $12 and $19 for the om-developer, with 78 to 80 percent of the om-reviewer's cost in cache writes at fable's 2x write price, and the 5-hour window hit every day).
- Reason: the om-reviewer is the volume role of the pipeline (7 to 10M tokens per task, and a full ~150k re-cache every time it idles past the cache TTL while waiting for a round), so its model price drives the daily quota; the "only unchecked gate" argument for fable stands but is mitigated by an opus om-developer, mandatory tests per acceptance criterion, `verify-task` on every round and Sebastian reading `Decisions`; no case is recorded of fable catching a finding opus would have missed.
- Debt created: the final gate before a PR now runs on the same tier as the code it reviews; a missed finding reaches Sebastian without a stronger model behind it.
- Revisit when: a merged PR breaks on something the Pipeline should have caught, or Sebastian spots in a PR a finding the om-reviewer missed; then the om-reviewer returns to fable with that case as evidence. Also if the fable weekly quota stops being the constraint.
- Files: agents/om-reviewer.md.

## 2026-09-01: Per-project review checks, run by the om-reviewer in the background

- Decision: a project can declare review checks in `docs/checks/{{name}}.md` (frontmatter `model`, `paths`, `reference`; body of verifiable statements and a closing let-pass line). `review-task` launches every check whose `paths` match the diff before step 1 as a background `claude -p` process on the declared model (diff on stdin, reference document in the prompt, read-only tools), collects the outputs after the code review, and triages every line against the code it read: confirmed lines join the round's findings as `[{{check}}]`, the rest go to `Decisions` as let-pass. New om-manager skill `add-check` writes a check on Sebastian's ask, writes the reference document under `docs/conventions/` first when the rule is not documented, dry-runs it on current code and commits. `publish-task` reports a `checks` line in the PR Pipeline.
- Alternatives rejected: running the checks in `verify-task` by the om-developer (mixes judgment with mechanical evidence, and the author of the code would triage its own false positives unaudited), subagents through the Agent tool inside the om-reviewer (not in its toolset, and the output lands in its context anyway; `claude -p` is equivalent and trivially parallel), a long-lived `om-checker` session per project (another session to manage and re-cache for stateless work), PostToolUse hooks on the om-developer (per edit instead of per round: noise and cost), a `Review checks` table in `TRD.md` (a prompt does not fit a row and the table would drift from the files; `docs/checks/` is the registry, like `modules/`).
- Reason: the general review on opus judges many dimensions at once and is weak at exhaustive coverage rules (every string translated, every endpoint with its DTO); a narrow single-rule checker on sonnet is better and cheaper at exactly that, and the om-reviewer already reads every changed file, so its triage of the checker's lines costs nothing extra. Sebastian wants these reviews per project and on demand, not baked into `review-task`.
- Debt created: the checker prompt lives inline in `review-task` step 0 and `add-check` step 4 asks to use it literally; if it grows, it moves to a shared file. Glob matching of `paths` against the diff is done by the om-reviewer by hand, not by a script. `.checks/` outputs in the workspace are deleted with the workspace and never travel in the PR.
- Revisit when: a check keeps producing false positives the om-reviewer must drop every round (tighten its body or delete it), when two projects copy the same check verbatim (promote it to a reference in `add-check`), or when the inline prompt drifts between `review-task` and `add-check` (extract it).
- Files: skills/add-check/SKILL.md, skills/add-check/templates/check.md, skills/add-check/references/{i18n,style,patterns}.md, skills/review-task/SKILL.md, skills/publish-task/SKILL.md, skills/publish-task/templates/pr-summary.md, agents/om-reviewer.md, agents/om-manager.md, docs/01-documentation.md, docs/02-orchestration.md, docs/03-skills.md, docs/method-ard.md.

## 2026-09-01: Checker command passes the prompt after `--`

- Decision: the `claude -p` command in `review-task` step 0 and the dry run in `add-check` step 4 put `--` before the positional prompt: `claude -p --model {{model}} --allowedTools Read,Grep,Glob -- "$(cat docs/checks/{{name}}.md) ..."`.
- Alternatives rejected: stripping the frontmatter from the check file before injecting it (extra plumbing in two places for the same effect, and the frontmatter is useful to the checker as a statement of its scope), piping the prompt on stdin (stdin already carries the diff), passing the prompt through a temp file (more state in the workspace for no gain).
- Reason: a check file starts with a `---` frontmatter line, so the expanded prompt begins with `---` and the CLI parses it as an unknown option and exits 1 (`error: unknown option '---'`). Found by the auvral om-manager while dry-running `docs/checks/i18n.md` through `add-check`; every check would have failed the same way at review time.
- Debt created: none.
- Revisit when: the checker prompt is extracted to a shared file (see the per-project review checks entry); the `--` moves with it.
- Files: skills/review-task/SKILL.md, skills/add-check/SKILL.md, docs/method-ard.md.

## 2026-09-02: Task sessions are derived from `claude agents --all --json`; clean-task stops them before touching files

- Decision: no skill stores a `--bg` session id. Whenever one is needed (`clean-task`, `delegate-task`, `next-phase`), it is read from `claude agents --all --json`, whose entries carry `id`, `name` and `cwd`: by `name` for one session, by `cwd` under the workspace for every session of a task. `clean-task` step 3 stops and removes every session whose cwd is the workspace, retries on "background service may be restarting", and only proceeds to remove worktrees when the listing is empty. Every skill that lists sessions from Bash uses `--json`; the bare `claude agents` needs a TTY and fails.
- Alternatives rejected: `consolidate-task` writing the id to `{{WORKSPACE}}/.session` or into the task folder (state that can drift from reality and one more file to keep consistent; the CLI already knows, and the method derives state rather than recording it), keeping the id in the om-manager's reply (lost on every recycle; this is how auvral 0006 ended with a detached om-reviewer recreating `.claude/` inside a removed workspace).
- Reason: on auvral 0006 the om-manager killed the tmux window believing it stopped the om-reviewer; `--bg` sessions survive the pane by design, so the cleanup has to address the process, and it can only do that with an id it can find after a recycle.
- Debt created: none; `scripts/install` reports a missing `jq` (and `gh`).
- Revisit when: `claude rm` learns to take a name or a cwd, or `claude agents` gains a filter flag.
- Files: skills/clean-task/SKILL.md, skills/consolidate-task/SKILL.md, skills/delegate-task/SKILL.md, skills/next-phase/SKILL.md, skills/start-task/SKILL.md, skills/check-task/SKILL.md, skills/check-portfolio/SKILL.md, skills/clean-work/SKILL.md, docs/02-orchestration.md, docs/03-skills.md.

## 2026-09-02: A `gh` failure is an unknown signal, never an empty one

- Decision: `check-task`, `check-work` and `check-portfolio` report a `gh` failure in the output (`gh failed: {{error}}`, with the active account from `gh auth status`) and never derive `in_review`, `merged` or `done` from it.
- Alternatives rejected: declaring the gh account per project in `TRD.md` and switching before every call (the method should not manage credentials; Sebastian switches once), retrying with `GH_TOKEN` (same problem, more secrets in prompts).
- Reason: on auvral the active gh account was `sflores-designli` instead of `auvral-development`; `gh pr list` failed and the board read the failure as "no PRs", showing merged tasks as in progress.
- Debt created: none.
- Revisit when: `gh` supports per-directory accounts natively; then the hint changes.
- Files: skills/check-task/SKILL.md, skills/check-work/SKILL.md, skills/check-portfolio/SKILL.md, docs/03-skills.md.

## 2026-09-02: add-check dry run is bounded and has a positive test

- Decision: `add-check` step 4 diffs from the last merge into the base, looks at `--stat` first and narrows anything over 30 files or 200 KB; then runs one positive test on a reversed diff from before the rule was applied, comparing blobs by path (`git diff {{old}}:{{old path}} HEAD:{{new path}}`) for files that were moved.
- Alternatives rejected: keeping the fixed `HEAD~20` window (on auvral it produced a 900 KB diff of a `[locale]` migration), skipping the positive test (zero findings on a clean diff proves nothing about the check).
- Reason: the dry run exists to calibrate the check; it needs one diff that should be clean and one that should not, both small enough to read the checker's output.
- Debt created: none.
- Revisit when: the checker prompt moves to a shared file; the dry run can then become a script.
- Files: skills/add-check/SKILL.md.

## 2026-09-02: om-manager writes memory with the absolute path only

- Decision: `agents/om-manager.md` Environment states that memory files are written to the absolute directory given in its memory instructions, never relatively, because its cwd may sit in a workspace.
- Alternatives rejected: ignoring `.claude/agent-memory/` in every project (hides the symptom; the write is still wrong), a rule in `execute-task` telling the om-developer not to commit foreign files (the om-developer cannot tell a foreign file from a task file).
- Reason: on auvral a memory write with cwd inside a worktree landed in the task branch and the om-developer committed it.
- Debt created: none.
- Revisit when: the harness resolves the memory directory itself regardless of cwd.
- Files: agents/om-manager.md.

## 2026-09-02: verify.log records the exit code of every step, captured without a pipeline

- Decision: `verify-task` step 2 prescribes the exact form for each step, `cd {{path}} && {{command}} > {{WORKSPACE}}/.verify/{{target}}-{{step}}.out 2>&1 ; echo "EXIT=$?"`, with no pipeline anywhere, and bans capturing status through `tee` or `${PIPESTATUS[0]}`. The log format carries `exit {{code}}` on every step line, `pass` means exit 0 and nothing else, and a step whose code could not be captured is `fail`. `review-task` step 3 and the om-reviewer's Pipeline treat a `pass` with an empty or missing code as the finding `verify.log: step {{name}} has no exit code`.
- Alternatives rejected: keeping "capture the exit code" as an instruction without a form (that gap is what let each om-developer improvise a `tee` pipeline), `set -o pipefail` (fixes the status but still hides which stage failed, and the om-developers were reading command output to judge steps anyway), running verification through a script in `scripts/` (the commands come from each project's TRD, not from this repo).
- Reason: the om-0005-reviewer on diy found that the improvised pattern logged `${PIPESTATUS[0]}`, which is empty in zsh (zsh spells it `$pipestatus` and indexes from 1) and where `$?` after a pipeline is the last stage's status. Every `EXIT=` line written so far proves nothing, and the om-reviewer audits that log as the only evidence that lint and tests ran, so the whole verification gate was resting on a value the shell never set. Verified in this repo's shell: after `false | cat`, zsh gives `${PIPESTATUS[0]}` empty and `$?` 0.
- Debt created: `verify.log` blocks written before this entry carry no usable exit codes; tasks in flight will show the finding on their next round, which is the intended outcome. Sebastian's shell is the assumption: the form uses only `$?`, which is portable, so a different shell does not reopen this.
- Revisit when: `verify-task` ever needs a pipeline (streaming output to the conversation), which it must not; or a project's runner exits 0 on failure, which is a finding about that project, not about the log.
- Files: skills/verify-task/SKILL.md, skills/review-task/SKILL.md, agents/om-reviewer.md, docs/02-orchestration.md, docs/03-skills.md.

## 2026-09-02: clean-task removes the method's scratch folders before rmdir

- Decision: `clean-task` step 4 removes `{{WORKSPACE}}/.checks`, `{{WORKSPACE}}/.verify` and `{{WORKSPACE}}/.claude` before `rmdir {{WORKSPACE}}`; anything else left is listed and stops the cleanup.
- Alternatives rejected: `rm -rf {{WORKSPACE}}` (removes whatever a session left there, including work nobody looked at), writing the scratch folders under `/tmp` (they belong to the task and are useful while it is open).
- Reason: `review-task` creates `.checks/` and `verify-task` now creates `.verify/`, so the plain `rmdir` would always find the workspace non-empty and stop; the skill would report a leftover it created itself.
- Debt created: none.
- Revisit when: another skill starts writing in the workspace root; it must be added to this list.
- Files: skills/clean-task/SKILL.md.

## 2026-09-02: A test earns its place if it can fail for a reason that matters

- Decision: `execute-task` and `review-task` gain a criterion for the tests a round adds, not only for the ones it lacks. A test is a finding when it asserts a literal the same change introduced, when it encodes a style or lint rule instead of behavior (that belongs in `docs/checks/` or the linter), or when it covers the trivial edge of a change while the real behavior stays unverified. The om-reviewer asks for its removal by `file:line`, records it in `Decisions`, and raises a second finding when the behavior it stood in for has no coverage.
- Alternatives rejected: a project check in `docs/checks/` for test quality (a checker sees the diff, not whether the behavior is covered elsewhere; this needs the judgment of the om-reviewer that read every file), a coverage threshold (it rewards exactly the padding this entry rejects), leaving it to the om-developer alone (it wrote the test, so it is the worst placed to judge whether it can fail).
- Reason: Sebastian's ask, from PR #506 on diy-platform (task 0015). A fix that removed two em dashes from copy shipped two `it.each` cases asserting that the two strings the same commit had just written contain no em dash. It pinned a house style rule from his global `CLAUDE.md` inside a unit test of one file, where it guards that file while suggesting the rule is enforced project-wide; it could not fail for any reason worth knowing; and it inflated the test count of a task whose real behavior change, the wizard's save slot, had no automated coverage at all.
- Debt created: the criterion is judgment, not a check, so it depends on the om-reviewer applying it; a test that merely duplicates an existing one is not covered by this wording.
- Revisit when: the om-reviewer starts asking to remove tests Sebastian wanted kept (the wording is too broad), or a second PR ships style-rule tests after this entry (the criterion is not reaching the om-developer and belongs earlier, in `analyze-task`).
- Files: skills/execute-task/SKILL.md, skills/review-task/SKILL.md, agents/om-developer.md, agents/om-reviewer.md, docs/02-orchestration.md.

## 2026-09-07: Launchers keep a failed claude's pane alive instead of losing the session

- Decision: `bin/resume-overmind` and `scripts/resume-project` append a guard to every `claude --agent ...` window command: `|| { rc=$?; echo "[script] claude exited with status $rc; fix the cause, then prefix + R relaunches it"; read -r line; }`. A claude that exits non-zero leaves its output and status on screen in a live pane; the script goes on creating the other windows; `prefix + R` (respawn-pane) re-runs the whole command string, claude included. A claude that exits 0 closes its window as before.
- Alternatives rejected: creating windows with a plain shell and `send-keys` for the claude command (a failed claude would be visible, but `prefix + R` would respawn a bare shell instead of the agent, breaking the recycle routine), `remain-on-exit on` per window (set after creation, so it races the failure it is meant to catch, and it also keeps healthy exits around), waiting for network or for the claude background service before launching (guesses at one cause; the guard shows the real one whichever it is).
- Reason: on 2026-09-07, right after a reboot, `resume-overmind` failed three times in a row (13:17:30 to 13:18:32) and Sebastian could only get in through `tmux-sessionizer` plus a manual `claude`. The `overmind` session it left behind had a single window, so the `claude --agent overmind -n overmind` launched by `new-session` had died at once; with the only window gone the session vanished and the next `tmux` command failed with a cryptic error, and claude's own message was lost with the window. Twelve minutes later the identical launch worked in an isolated tmux server (`-L omtest`), so the cause is a first-minute-after-boot transient (network or token refresh, or the first start of the background service on a fresh `/run/user/1000`); the guard captures it on the next occurrence. Reproduced both paths with a mocked claude: exit 7 leaves all three windows with the message on screen; exit 0 gives the previous layout.
- Debt created: the root cause of the boot-time failure is still unknown; the next reboot shows it in the pane. `resume-overmind` still places `config` at the next free index instead of a pinned one like `resume-project` does for the om-manager.
- Revisit when: the pane shows the cause (add a targeted check for it, or remove nothing: the guard stays regardless), or the boot failure recurs after that check.
- Files: bin/resume-overmind, scripts/resume-project, docs/method-ard.md.

## 2026-09-07: Launchers resolve the repo root through the symlink; resume-overmind pins its windows

- Decision: `bin/resume-overmind` and `scripts/resume-project` compute the repo root from `readlink -f "${BASH_SOURCE[0]}"`, not from `${BASH_SOURCE[0]}` itself. `resume-overmind` also pins window `overmind` at index 1 and `config` at index 2 with the same repair logic `resume-project` uses for the om-manager: an existing window elsewhere is moved, a missing one is inserted at its index, shifting whatever sits there.
- Supersedes: the "Reason" and "Debt created" of the 2026-09-07 entry "Launchers keep a failed claude's pane alive": the boot-time failure was not a transient. The guard from that entry stays; it is what exposed the real cause.
- Alternatives rejected: installing `bin/` as copies instead of symlinks (the copy drifts from the repo on every method commit), hardcoding the repo path in the script (breaks on a second machine and on a moved checkout), keeping `next_index` for `config` (on the live session it would have created `overmind` at index 2 or 3 and a second `om-config`, since the om-config window already existed under another name).
- Reason: the guard showed `--agent 'overmind' not found` with a list holding only the `~/.claude/agents` roles. `~/bin/resume-overmind` is a symlink into the repo (since 2026-08-31, "bin/ is what gets linked"), so `dirname "${BASH_SOURCE[0]}"` gave `~/bin` and the root resolved to `$HOME`; claude started there and could not see the project-scoped agents in `.claude/agents/`. The bug only fires when the session has to be created, which is why it looked boot-related: on any other day the session exists and the script launches nothing. Verified with `bash -c` through the symlink (`/home/fless` before, the repo after) and with a mocked claude on an isolated server (`-L omtest`) invoked through a symlink: fresh start, repair from a session holding only `config` at 1, and a no-op second run all leave `1:overmind` (two panes) and `2:config`, every claude started in the repo root.
- Debt created: `scripts/install` is not covered by `readlink -f`; it is run from the repo, never through a link. The move-to-taken-index branch of `ensure_window` is untested here (the sandbox kills seeded sessions); it is the same command `resume-project` runs.
- Revisit when: a launcher gains a fourth window (add it to the pinned list), or `install` starts being invoked through a link.
- Files: bin/resume-overmind, scripts/resume-project, docs/method-ard.md.

## 2026-09-08: om-reviewer returns to fable

- Decision: `agents/om-reviewer.md` moves back from `model: opus` to `model: fable`; `om-manager` stays on fable, `om-developer` stays on opus.
- Supersedes: 2026-09-01 "om-reviewer runs on opus; om-manager stays on fable".
- Alternatives rejected: keeping opus (the 2026-09-01 cost data still stands, but Sebastian chose the stronger gate), moving the om-developer to fable instead (the om-reviewer is the only independent gate before a PR, so the stronger model belongs there).
- Reason: Sebastian asked for the return on 2026-09-08 after a week with the om-reviewer on opus; the 2026-09-01 entry named its own return conditions (a finding the om-reviewer missed reaching a PR, or the fable quota no longer being the constraint) and this closes its debt of having the final gate on the same tier as the code it reviews.
- Debt created: the om-reviewer's cost per task goes back toward the 2026-09-01 measurements ($22 to $29 per task, 78 to 80 percent in cache writes) unless the later cuts (`verify-task` only by the om-developer, `document-task` once) changed that profile; nobody has re-measured.
- Revisit when: the fable weekly quota is hit again with the om-reviewer as the main consumer; re-measure a task before deciding.
- Files: agents/om-reviewer.md.

## 2026-09-10: A paid debt is recorded with `Resolved by`, and `document-task` keeps the index

- Decision: `write-ard` gains a closure form, `Resolved by: {{id}}_{{title}}, {{YYYY-MM-DD}}`, added under the `Debt created` of an existing entry, additive like `Superseded by` and with nothing deleted or reworded; the debt index of `docs/ARD.md` now lists only entries without it. `document-task` writes that line for the debt its own task paid, and in the same pass brings the index in line, adding rows for the debt it created and removing the rows it resolved. Its scope widens from `docs/modules/` to that one table, and to nothing else in `docs/ARD.md`; `publish-task` checks the table against the module ARDs before pushing.
- Alternatives rejected: supersession for paid debt (it is a different fact: the decision stands, only its cost is gone, and marking it superseded would falsify the trail); deleting the entry or its `Debt created` (the ARD is a log and the trail is its value); the om-manager rebuilding the index on a cadence (Sebastian chose the task that pays the debt over one more thing to remember); `document-task` writing to the root checkout instead of the root worktree (the change would sit uncommitted on the base branch, outside the PR and outside review).
- Reason: the auvral om-manager measured it on 2026-09-10: eleven module ARDs hold 261 entries with debt, the general index lists 121 of them, and among those is the account write surface that task `0005_account_write_surface` shipped weeks ago. The index billed for work already done and nobody could close it, because supersession was the only closure the method had.
- Debt created: two tasks documenting in parallel touch the same table and will conflict on merge; the conflict is line-level in a table, so it is left to git. The index is now maintained in two ways, rebuilt wholesale by `write-ard` and incrementally by `document-task`, so a missed `Resolved by` survives until the next `write-ard general`. Nothing verifies that a `Resolved by` matches real code; `publish-task` checks the table against the entries, not the entries against the diff.
- Revisit when: the conflicts on the table become routine (then the index moves to one row per module, or is generated), or a task is found claiming a `Resolved by` whose debt is still in the code.
- Files: skills/write-ard/SKILL.md, skills/write-ard/templates/ARD.md, skills/document-task/SKILL.md, skills/publish-task/SKILL.md, agents/om-developer.md, docs/01-documentation.md, docs/03-skills.md, docs/method-ard.md.

## 2026-09-14: `review-pr` reviews PRs written by someone else, read-only, in two layers

- Decision: a new global skill `review-pr`, invoked only by Sebastian, reviews a colleague's PR in any project, with or without the overmind convention. It checks the PR branch out in a worktree under `.workspaces/pr-{{n}}/` and reads every changed file in full; it runs nothing, not lint, tests, project checks or CI lookups, because the PR already shows those results. Findings are typed comments (`issue`, `security`, `test`, `scope` block by default; `refactor`, `docs`, `question` do not; `suggestion` and `nit` never), few and well placed: every blocking finding goes in, optional ones are capped at three, one comment per pattern. The output has two layers, a short overview (derived verdict, risk, one line per comment under required and optional) and one inline comment per finding carrying the explanation; both are saved in the worktree and in the ignored `pr-reviews/` at the project root, and on Sebastian's yes posted as a pending review on the PR of the repo the diff belongs to, which he edits and submits. The om-manager never runs it: on Sebastian's ask it opens window `pr-{{n}}` with a fresh session running `/review-pr {{n}}`.
- Alternatives rejected: running it inside the om-manager (it never reads diffs, and a full review fills its long-lived context); running verification and the project checks like `review-task` does (redundant with CI and the checks the PR already shows, and slow when dependencies have to be installed); posting the review directly (a colleague's PR deserves Sebastian's read first; a pending review is invisible to the author until he submits); three severity icons instead of typed comments (Sebastian wanted the type and the required/optional split visible to the author); a `.gitignore` edit for `pr-reviews/` (`.git/info/exclude` needs no commit in the project).
- Reason: Sebastian reviews PRs his colleagues write and wanted the method's review bar on them without its task machinery, moving fast but with care: strict where a defect costs, permissive elsewhere, never a wall of comments. `pr-reviews` had been absorbed into `review-task` on 2026-08-28, which covers only the method's own tasks.
- Debt created: the skill references `review-task` step 4 for the review order instead of restating it, so a change there changes both, which is the intent, but a reader of `review-pr` alone has to open a second skill. The om-manager launches a plain `claude` session, so `review-pr` is available with every other global skill in that window; the restriction is the `disable-model-invocation` flag and the skill's own scope. No `references/` per stack yet.
- Revisit when: a review misses something CI would have caught only because CI was not configured on that repo (then a `gh pr checks` glance returns as a one-line note), or Sebastian finds the three-optional cap too tight or too loose after a few reviews.
- Files: skills/review-pr/SKILL.md, skills/review-pr/templates/pr-review.md, agents/om-manager.md, docs/03-skills.md, docs/usage-guide.md, docs/method-ard.md.

## 2026-09-14: Risk assessment is production-first and defaults to Low

- Decision: the risk value in every review the method writes (`publish-task` summaries and `review-pr` overviews) follows one rubric, written in `review-pr` step 5 and referenced by `publish-task`. Production first: if the base branch does not deploy to something real users depend on (TRD `Delivery`, or Sebastian asked once and the answer written in the TRD), risk is `✅ Low` with `not in production yet` and nothing else is weighed. In production, `🔴 High` is a defect that corrupts or exposes data, breaks auth or payments, takes the service down or cannot be reverted; `⚠️ Medium` is a defect that breaks a flow real users depend on, revertible, on a path the tests do not cover; everything else is `✅ Low`. Medium and High must name the concrete failure, how it reaches a user and what to watch; Medium for size or uncertainty is forbidden.
- Alternatives rejected: leaving the value to judgment with the format alone (the pilot's summaries drifted to Medium for anything non-trivial); weighing diff size (a large, well-tested change to an unused flow is not risky); a separate risk for non-production environments (Sebastian: if it is not in production, the assessment has no meaning).
- Reason: Sebastian read the pilot's PR summaries and found the risk too soft to be useful: Medium had become the default, and none of them asked whether users existed yet.
- Debt created: projects whose TRD lacks a `Delivery` line will get asked once; the answer must reach the TRD or the question repeats. The rubric lives in `review-pr`, a skill the om-reviewer does not own, so the om-reviewer reads across skills to apply it.
- Revisit when: a Low-rated PR breaks production users (tighten the Medium criteria), or the rubric needs a home both skills own (move it to a shared reference).
- Files: skills/review-pr/SKILL.md, skills/publish-task/SKILL.md, docs/method-ard.md.

## 2026-09-14: A new skill is linked by `scripts/install`, and `update-method` says so

- Decision: `update-method` step 3, new skill, ends by running `scripts/install`; the symlinks under `~/.claude/skills/` are one per skill folder, so a new folder is invisible to every Claude session until it is linked and the session relaunched.
- Alternatives rejected: linking the whole `skills/` directory once (Sebastian keeps skills from other repos in `~/.claude/skills/`, see `jira-to-github-projects`); `update-method` writing the link itself (`install` already owns the linking and is idempotent).
- Reason: `review-pr` was committed on 2026-09-14 and announced as available; Sebastian's session did not find it because the link never existed.
- Debt created: none.
- Revisit when: `install` gains a way to link a single skill, or the skills move to a plugin.
- Files: .claude/skills/update-method/SKILL.md, docs/method-ard.md.

## 2026-09-17: om-reviewer back on opus

- Decision: `agents/om-reviewer.md` runs on `opus` again; the switch to `fable` of 2026-09-08 is reversed.
  Supersedes the 2026-09-08 entry.
- Alternatives rejected: none weighed; Sebastian's call, recorded as such.
- Reason: Sebastian's choice for the review sessions after a week on fable; no measurement attached.
- Debt created: none.
- Revisit when: fable or opus quota is hit with the om-reviewer as main consumer; measure before switching again.
- Files: agents/om-reviewer.md, docs/method-ard.md.

## 2026-09-17: `om-pr-reviewer` owns the review of PRs someone else wrote, with five skills around it

- Decision: a new global agent `om-pr-reviewer` (`opus`, `bypassPermissions`) runs the per-PR session `om-pr-{{n}}-reviewer` in window `pr-{{n}}`, launched by the om-manager's new `delegate-pr` (`/review-pr {{n}} [{{m}}]`, `{{m}}` a related PR read as context and never reviewed; an existing session is reopened and told `re-review, head {{sha}}` instead). `review-pr` changes contract: the overview gains `Head`, `Size` (additions, deletions, files, commits, counts only) and a closing `Summary` with required and optional counts; the verdict is `approve` or `request changes` (no `approve with comments`); it posts nothing and offers nothing, it sends `om-events` one `action` line, and the worktree stays until `clean-pr`. When its previous report exists it re-reviews: updates the worktree, marks each previous comment `addressed`, `still open` or `withdrawn`, reviews the new code. Two om-pr-reviewer skills post on Sebastian's word: `approve-pr` (`gh pr review --approve`, no body, no comments) and `request-pr-changes` (the saved overview and inline comments filtered to his selection, submitted as `REQUEST_CHANGES`, never pending); both refuse a head that moved since the report. Two om-manager skills complete it: `check-prs`, the PR board from `gh` and `.workspaces/pr-*` (groups `cleanable`, `reviewed locally, not posted`, `waiting on author`, `back for re-review`, `approved, not merged`, `not reviewed`, `dependabot`, plus PRs sharing a ticket key), and `clean-pr`, the `clean-task` of a merged or closed PR review; `clean-work` covers both. The om-pr-reviewer never reports to the om-manager; `om-events` accepts `PR #{{n}}` as id. All skills run on the model of the session that owns them; none sets `model`. The three om-pr-reviewer skills are model-invocable (`disable-model-invocation: false`): Sebastian's `approve` or his selection in the conversation is the ask, and `re-review` arrives as a message, so the agent must be able to run them itself.
- Alternatives rejected: per-skill `model` (`analyze-prs` on opus, `delegate-pr` and the posting skills on sonnet): a skill's `model` switches the session model for one turn, and prompt cache is per model, so every switch rewrites the whole context twice, at the price of the more expensive model on the way back; `context: fork` would avoid that but is overkill for skills that are one `gh` command. `approve with comments` as a third outcome (Sebastian: an approval carries nothing; optional comments go through `request-pr-changes` or not at all). A pending review Sebastian submits from GitHub (the decision is now taken in the conversation, so the review is submitted). Listing Claude sessions in `check-prs` (`claude agents --all --json` returns every session of the machine; the `pr-{{n}}` workspace is the local signal, and `clean-pr` finds the session itself). Removing the worktree at the end of `review-pr` (the re-review and `check-prs` need it). A `re-review-pr` skill (principle 5: mode derived from the report on disk). `analyze-prs` and `pr-approve`/`pr-request-changes` as names (verb first, and `check-*` is what the method calls a derived board).
- Reason: Sebastian reviews colleagues' PRs across projects and wanted the flow closed: a session that only reviews, a board of what each PR waits for, approve or request changes without leaving the window, and the re-review of the same PR against the same report. The plain `claude` session of 2026-09-14 had no owner, no events and no way to post the verdict.
- Debt created: `om-events` prints counts keyed on `task {{id}}` lines and `PR #{{n}}` lines alike, fine for counting, unparsed otherwise. `check-prs` reads `reviews[].commit.oid` from `gh`; if a repo's PR list is above 100 the `--limit` truncates silently. The interactive launch in `delegate-pr` dies with the window (`claude -r` brings it back; `check-prs` does not miss it because the workspace is the signal). No `references/` per stack yet.
- Revisit when: `check-prs` needs the review states of more than one person (a team board), or the om-pr-reviewer session crosses ~150k on a re-review (recycle per re-review with the report as the only context).
- Files: agents/om-pr-reviewer.md, agents/om-manager.md, .claude/agents/om-events.md, skills/review-pr/SKILL.md, skills/review-pr/templates/pr-review.md, skills/approve-pr/SKILL.md, skills/request-pr-changes/SKILL.md, skills/delegate-pr/SKILL.md, skills/check-prs/SKILL.md, skills/clean-pr/SKILL.md, skills/clean-work/SKILL.md, skills/publish-task/SKILL.md, scripts/lint-method, docs/02-orchestration.md, docs/03-skills.md, docs/04-operation.md, docs/usage-guide.md, docs/method-ard.md.

## 2026-09-17: `check-prs` prints tables with created and updated dates

- Decision: each group of the `check-prs` board is a Markdown table (`PR`, `Repo`, `Title`, `Author`, `Created`, `Updated`), rows sorted oldest `updatedAt` first, dates as day only; `Related` is a table too. No bullet lists in the board.
- Alternatives rejected: one line per PR with bullets (Sebastian: it does not read well); relative ages (`3d`) instead of dates (a date is stable in a transcript read hours later).
- Reason: the board is scanned, not read; a table aligns what has to be compared across PRs, and the two dates say which PRs are going stale.
- Debt created: none.
- Revisit when: the board needs more columns than fit a pane; then move detail behind the PR link.
- Files: skills/check-prs/SKILL.md, agents/om-manager.md, docs/03-skills.md, docs/method-ard.md.

## 2026-09-20: Launch lines run bare; a prefix sends them to the classifier

- Decision: `consolidate-task`, `start-task` and `delegate-pr` state that each launch line runs exactly as written, one Bash call, `cd {{abs}} && claude ...` or `tmux ...` with nothing before it; `om-manager` carries the same rule in Environment. No new allow rules, per project or global.
- Alternatives rejected: adding allow rules for `export *` or `Bash(* claude --bg *)` (a wildcard prefix rule is a blanket grant that defeats the classifier; an `export` rule still leaves `VAR=` and functions), copying the global rules into each project's `settings.local.json` (auvral had them since 2026-09-16 and they did nothing: rules match the command text, not the project), letting the om-manager retry until the classifier yields (deterministic block, and the classifier tags the following commands too).
- Reason: 2026-09-20 in auvral, `consolidate-task` for 0042 to 0044 was blocked four times as "Create Unsafe Agents"; every denied command carried `export PATH=$HOME/.nvm/...; R=...; WS=...` or a `launch(){}` wrapper, while my-napkin's om-manager launched the same command bare and never met the classifier. Allow rules skip the classifier only when every segment of the compound command matches a rule.
- Debt created: the environment the launched session needs (node version, PATH) is left to that session's shell profile; if a workspace needs an environment its profile does not provide, the method has no place to declare it yet.
- Revisit when: a launched om-reviewer or om-developer reports a missing tool that the om-manager's prefix was providing.
- Files: skills/consolidate-task/SKILL.md, skills/start-task/SKILL.md, skills/delegate-pr/SKILL.md, agents/om-manager.md, docs/02-orchestration.md, docs/06-pilot.md, docs/method-ard.md.

## 2026-09-21: verify.log lines derive from per-step artefacts that carry the shell-written exit code

- Decision: `verify-task` appends `EXIT=$?` to each step's artefact in `{{WORKSPACE}}/.verify/{{target}}-{{step}}.out`, with nothing between the command and the capture, and derives every `verify.log` line from that artefact's last line; a step without an artefact or `EXIT=` line is `fail`. `review-task` and om-reviewer check each step line against its artefact (same code, artefact newer than the round's commit) and report `verify.log: step {{name}} does not match .verify` otherwise.
- Alternatives rejected: having the om-reviewer re-run lint and tests (the 2026-08 decision against duplicate runs stands; memory is the constraint), committing `verify.log` from a script instead of the om-developer writing it (a script per stack that the method does not have yet; the artefact check gives the same guarantee with the existing steps), keeping the exit code only in the conversation (the om-reviewer cannot see it, which is the gap 0012 exposed).
- Reason: my-napkin task 0012, phase 3: the om-developer filled the `deploy` line of `verify.log` from the previous run instead of a command just run; it happened to be true. The log was hand-written and the om-reviewer's only evidence, with nothing on disk to contrast it against. A trailing `echo` or `tail` between the command and `$?` also turns a red run into `exit 0`.
- Debt created: the audit compares the artefact's mtime with the round's commit, a heuristic that a re-run after the commit satisfies without proving the code at the commit was the one verified; `.verify/` is per workspace, so a task with several phases keeps only the latest run per step.
- Revisit when: the method gets a per-stack `references/{{stack}}.md` for `verify-task` that can write the block itself, or an om-reviewer finds an artefact matching a log line that was still wrong.
- Files: skills/verify-task/SKILL.md, skills/review-task/SKILL.md, agents/om-reviewer.md, docs/02-orchestration.md, docs/03-skills.md, docs/method-ard.md.

## 2026-09-23: om-reviewer on fable, trial

- Decision: `agents/om-reviewer.md` runs on `fable`; the return to `opus` of 2026-09-17 is reversed.
  Supersedes the 2026-09-17 entry.
- Alternatives rejected: none weighed; Sebastian asked for a trial.
- Reason: Sebastian wants to try the review sessions on fable again; this is the third switch on this line (2026-09-01 opus, 2026-09-08 fable, 2026-09-17 opus), with no measurement attached to the last two.
- Debt created: the om-reviewer is the volume role, so the fable weekly quota is exposed again; without a measurement this trial will end the way the previous ones did.
- Revisit when: after the first two tasks reviewed on fable, compare cost per task and findings against the 2026-09-01 numbers ($22 and $29 on fable, $12 and $19 for the om-developer) and decide with data.
- Files: agents/om-reviewer.md, docs/method-ard.md.

## 2026-09-23: om-developer carries the serial test rule itself

- Decision: `agents/om-developer.md` gains a `What you never do` line: never two suites, targets or verification commands in parallel, and never tests without `--runInBand` or the stack's serial flag, even while iterating on one failing test outside `verify-task`.
- Alternatives rejected: leaving the rule only in `verify-task` (the skill is loaded when invoked; the om-developer runs tests by hand between rounds, and those runs never see the skill).
- Reason: the rule lived in `verify-task` and `docs/03-skills.md`, both scoped to the formal verification run; the ad hoc test runs during implementation are the ones that pile up across workspaces, and the agent file is the only text present in every turn.
- Debt created: the same rule now lives in two places; a change to the serial flags has to touch both.
- Revisit when: `verify-task` gets per-stack references that own the flags; then the agent line can point at them instead of naming `--runInBand`.
- Files: agents/om-developer.md, docs/method-ard.md.

## 2026-09-23: om-reviewer back on opus, same day

- Decision: `agents/om-reviewer.md` returns to `model: opus`; the fable trial of earlier today is closed before any task ran on it.
  Supersedes the 2026-09-23 "om-reviewer on fable, trial" entry.
- Alternatives rejected: keeping the trial for two tasks as that entry planned (Sebastian withdrew it before a task launched).
- Reason: Sebastian's call, the same day; the bare `opus` alias tracks the latest Opus release, so the review sessions get the newest Opus without an id change.
- Debt created: none new; the four switches on this line since 2026-09-01 still have no measurement attached to the last three.
- Revisit when: a next switch on this line comes with a measured task, cost and findings, so the decision stops flipping on preference.
- Files: agents/om-reviewer.md, docs/method-ard.md.

## 2026-09-24: The launch needs an autoMode.allow entry; allow rules alone no longer clear it

- Decision: `~/.claude/settings.json` needs, besides the `permissions.allow` rules, an `autoMode.allow` entry (after `"$defaults"`) naming the overmind launches (`claude --bg --allow-dangerously-skip-permissions --agent om-reviewer|om-developer|om-pr-reviewer` inside `.workspaces/`). `scripts/install` reports it when missing with the exact text and never writes it; `usage-guide.md` lists it in Requirements. `om-manager`, `consolidate-task`, `start-task` and `delegate-pr` say what a bare line denied as "Create Unsafe Agents" means (entry missing, or session older than the entry) and that the answer is to report, not retry or fall back. `docs/02-orchestration.md` and `docs/06-pilot.md` record both gates.
  Extends the 2026-09-20 entry: the bare form is still required, it is no longer sufficient.
- Alternatives rejected: removing `--allow-dangerously-skip-permissions` and running om-reviewer and om-developer in auto mode (the 2026-08 decision stands: they must not be held back by the classifier mid pipeline); a `permissions.allow` wildcard (auto mode suspends broad rules and the classifier judged the line anyway); Sebastian naming the launch in each conversation (explicit intent clears a soft_deny, but the protocol's "sí, consolidala" does not name the agent nor the flag, and four earlier approvals on that wording were the classifier's leniency, not the rule); filing the change only in memory (a new machine would hit it again).
- Reason: 2026-09-24 in bseen, the bare line that had passed in my-napkin, diy and auvral was denied by the classifier as "Create Unsafe Agents" in a session on 2.1.281 (binary installed 2026-09-23). Transcripts show the same line resolved by the allow rule in some sessions and judged by the classifier in others, approved four times on 2.1.278 and 2.1.280, denied on 2.1.281; removing a `Bash(*)` rule added the same morning changed nothing. `claude auto-mode defaults --label 'Create Unsafe'` names the rule as a built-in soft_deny that `permissions.allow` cannot clear and `autoMode.allow` can. With the entry, the retry from the same session was denied again (a session keeps the rules it started with, and three denials in its context bias the fourth); after `prefix + R` the launch passed.
- Debt created: one more manual requirement per machine and a wide exception in prose: any command matching that description skips the "Create Unsafe Agents" judgment in every auto mode session of the machine, not only om-managers. Whether 2.1.281 stopped honoring the `Bash(claude --bg *)` rule for this line or the classifier now runs on it regardless is not proven; the entry covers both.
- Revisit when: a Claude Code release states allow rules short-circuit the launch again, or offers a per-agent way to declare the exception; or when a launch is denied with the entry present in a fresh session, which means the server-side classifier stopped reading `autoMode.allow`.
- Files: scripts/install, docs/usage-guide.md, docs/02-orchestration.md, docs/06-pilot.md, agents/om-manager.md, skills/consolidate-task/SKILL.md, skills/start-task/SKILL.md, skills/delegate-pr/SKILL.md, docs/method-ard.md.

## 2026-09-24: om-manager gets the Artifact tool

- Decision: `Artifact` joins the `tools:` allowlist of `agents/om-manager.md`. `overmind` and `om-config` stay without it.
- Alternatives rejected: dropping the `tools:` list so the om-manager gets every tool (the allowlist is what keeps MCP servers and other tools out of a session that plans and delegates); adding it to every cockpit agent (only the om-manager produces documents Sebastian shares).
- Reason: on 2026-09-24 the diy om-manager could not publish the plan comparison Sebastian asked for: the tool errored as disabled, and a plain session started minutes later on the same version had it. The `tools:` list dates from 2026-08-29 and predates the tool; `plan-task` already says artifacts are published only on Sebastian's ask, so the behavior rule existed without the tool.
- Debt created: each new built-in tool has to be added to the allowlist by hand; there is no lint for a tool the skills mention and the agent lacks.
- Revisit when: a second tool goes missing the same way; then `scripts/lint-method` checks that every tool a skill names appears in its agent's `tools:`.
- Files: agents/om-manager.md, docs/method-ard.md.
