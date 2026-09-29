# Part 3: Skills by role

Status: agreed on 2026-08-28.
Depends on: [01-documentation.md](01-documentation.md), [02-orchestration.md](02-orchestration.md).

## Principles

1. All skills live in `~/.claude/skills/` (global).
   The flow is Sebastian's convention, not the project's.
   Stack-specific detail goes in `references/{{stack}}.md` inside each skill and in the project's TRD.
2. Each role can only invoke its own skills.
   The agent definition in `~/.claude/agents/{{role}}.md` restricts which skills it has available.
   The om-developer cannot run `review-task`; the om-reviewer cannot run `execute-task`.
   It's by design, not by trust.
3. Sebastian drives the om-manager's skills.
   `plan-task`, `create-task`, `consolidate-task` and `delegate-task` are invocable by the om-manager so the task flow chains as one conversation: each runs on Sebastian's explicit yes to the question that offers it, never on the om-manager's initiative.
   `reiterate-task` and `clean-work` stay invocable only by the user.
   `clean-task`, `add-check`, `delegate-pr`, `delegate-plan` and `clean-pr` are invocable by the om-manager, but only on Sebastian's explicit ask, phrased however he likes; never on its own initiative.
   `check-task`, `check-work` and `check-prs` can be invoked by the om-manager when Sebastian asks about status.
   The overmind's skills (`resume-project`, `add-project`, `add-todo`, `complete-todo`, `clean-portfolio`) follow the same rule: the overmind invokes them on Sebastian's explicit ask, phrased however he likes, never on its own initiative.
4. The om-reviewer's, om-developer's and om-devops's skills run automatically.
   They are event-driven state machines.
   The machine lives in the agent's system prompt; nobody invokes the skills, the agent reacts.
5. One skill per procedure, not per round.
   There are no `re-*` variants.
   The input (state of the task folder, findings, retakes) determines the mode.
   Two skills that share 80% of the text drift out of sync over time.

## om-manager

| Skill | What it does |
|---|---|
| `setup` | Extracts a project's documentation, wherever it lives (READMEs, wikis, ADRs, comments, code, Sebastian), into the Part 1 convention. It doesn't wait for it as input: it's its output. Existing docs are claims: what the code can verify gets checked, and doc-vs-code contradictions are reported for Sebastian to arbitrate, never resolved silently. Two Workflows with `om-setup-worker`: discovery per component (read-only) and documentation per module, with module confirmation with you in between; then TRD, PRD and ARD (empty if there's no history), a short `CLAUDE.md`. Idempotent. |
| `write-prd` / `write-trd` / `write-ard` | Produce or update each document. `setup` uses them; `document-task` reuses them at the module level. |
| `add-check` | Adds one review check to `docs/checks/{{name}}.md`: frontmatter `model`, `paths`, `reference`, body of verifiable statements. Writes the reference document in `docs/conventions/` first when the rule is not documented anywhere; dry-runs the check on current code so Sebastian can calibrate it; commits `docs(checks): add {{name}}` on the base branch. The om-reviewer picks it up on the next round of every task. |
| `plan-task` | Planning conversation with Sebastian following the `docs/` reading path; decides `crew` (`crew: pair` or `crew: devops`) and `clearance` (`repo` or `full`) with him. Ends by offering `create-task`. Also run by an om-architect in its own window, which ends by sending the om-manager `draft ready: {{path}}`. |
| `create-task` | Creates the `docs/tasks/{{id}}_{{title}}/` folder in the root checkout with `task.md` (`type`, `crew`, `clearance`, `Goal`, `Scope`, `Acceptance`), `replication.md` if it's a bug, and, in the rare case of phases, one `phase_N.md` per phase. Doesn't commit; refuses `crew: devops` with phases. If the draft came from an om-architect, closes its session, window `plan-{{title}}` and scratch. Ends by offering `consolidate-task`. |
| `consolidate-task` | Syncs the base branch, creates a worktree and branch according to `type` from `origin/{{base}}`, opens the tmux window and launches only the om-reviewer (the om-devops when `crew: devops`) inside the worktree. Mediates its questions with Sebastian until `Context & decisions` is written in the root checkout copy. With Sebastian's approval it commits and pushes `docs(tasks): {{id}}_{{title}} planned` (plan plus decisions; the task's only docs commit). Asks whether to delegate now; if not, it stops the om-reviewer session (keeping the conversation) and closes the tmux window; the worktree and branch stay. |
| `delegate-task` | Checks `depends_on`. Reopens the om-reviewer (or om-devops) session if it was stopped (`claude attach` or `claude -r`); only launches a new one if it was lost. Runs `git fetch` and `git rebase origin/{{base}}` in the worktree so the branch receives the folder with the decisions. Sends "delegated, start". Doesn't launch om-developers. |
| `reiterate-task` | Notes Sebastian's comments on the PR, dated, in `retakes.md` and brings the crew back (om-reviewer, or om-devops) on the same branch, worktree and PR. |
| `check-task` | Derives a task's status: `planned` (no worktree), `consolidating` (worktree without `Context & decisions`), `consolidated` (worktree with `Context & decisions`, no commits or om-developer), `in_progress` (commits ahead of base, an om-developer session, or a delegated om-devops), `in_review` (PR open according to `gh`), `merged` (PR merged, worktree still exists), `done` (PR merged, no worktree). It doesn't query the sessions to ask them anything; it only checks whether they exist. A `gh` failure is reported as unknown, never as "no PRs". |
| `check-work` | `check-task` over all the project's tasks. It's Sebastian's dashboard. |
| `clean-task` | If the task's PR is merged into the base branch: `git pull` in the root checkout, stops and deletes every Claude session whose cwd is the workspace (derived from `claude agents --all --json`), then worktree, local and remote branch, tmux window. Doesn't commit anything; without a worktree the task is derived as `done`. |
| `clean-work` | Goes through all the worktrees, detects the merged ones and runs `clean-task` on each one, `clean-pr` on each local PR review whose PR is merged or closed, and closes each om-architect window whose draft is gone. |
| `delegate-plan` | Opens window `plan-{{title}}` and launches `om-{{title}}-architect` (agent `om-architect`) with `/plan-task {{description}}`; reopens it if it exists. On `draft ready: {{path}}` the om-manager offers `create-task` with that draft. |
| `delegate-pr` | Opens window `pr-{{n}}` in the project's tmux session and launches `om-pr-{{n}}-reviewer` (agent `om-pr-reviewer`) with `/review-pr {{n}} [{{m}}]`, `{{m}}` being a related PR passed as context only. If the session already exists, reopens it and sends `re-review, head {{sha}}`. The om-manager reads nothing of the PR and is never reported to. |
| `check-prs` | The PR board: every open PR of the project's repos grouped by what it waits for (`reviewed locally, not posted`, `waiting on author`, `back for re-review`, `approved, not merged`, `not reviewed`, `dependabot`), the local reviews whose PR is merged or closed (`cleanable`), and the PRs sharing a ticket key in title, body or branch. One table per group (PR, repo, title, author, created, updated). Derived from `gh` and the `.workspaces/pr-*` folders; never lists sessions. |
| `clean-pr` | If the PR is merged or closed: stops and removes the om-pr-reviewer session, removes the worktree, the workspace folder and the local branch `pr-{{n}}`, closes the window. `pr-reviews/` stays. |

## om-reviewer

| Skill | Event that triggers it | What it does |
|---|---|---|
| `analyze-task` | The session starts | Reads the task folder and the modules' `docs/`. If `Context & decisions` is empty, it does the back-and-forth with om-manager and Sebastian once and writes it in the root checkout copy (the only one written before delegation); it can adjust `Scope`, `Acceptance` and the phases. If it's already written (only happens when the original session was lost), it reads it and doesn't ask again. If there's a new `retakes.md`, it incorporates it. Notifies the om-manager "consolidated" and waits for "delegated, start". |
| `start-task` | Message from the om-manager "delegated, start" | Opens the right pane of the tmux window, launches `om-{{id}}-developer` (or `-developer-phase-1`) with cwd in the worktree and sends it "context ready, start". |
| `review-task` | Message "round N" from the om-developer | Launches the project checks (`docs/checks/*.md` matching the diff) as background `claude -p` processes, then runs the Pipeline over the worktree: intent, rebase, verification (audits `verify.log`; never runs `verify-task` itself), review, and the triage of the checks' output. If there are issues, it sends them to the om-developer with file:line, error and what was expected. If there are no issues, it sends "round N clean, document" and, on "docs ready", runs `publish-task`. |
| `publish-task` | `review-task` with no issues | Pushes each branch in the workspace, one PR per touched repo (plus the root one in multirepo), and the summary as the root PR's description with Intent, What changed (with links to each PR), Decisions (including merge order), Risk assessment and Pipeline per target. Writes no state; notifies the om-manager "PRs ready". |
| `next-phase` | Message from the om-manager "phase N merged, continue" | Kills the phase N om-developer session, creates the phase N+1 branch from `origin/{{base}}` in the worktree, writes phase N's `Result` in `phase_N.md` (it travels in the phase N+1 PR) and runs `start-task`. |

The om-reviewer never modifies code.
In tasks with phases it's the only one that lives through the whole task; the om-developers change per phase.
It can push: it shares the worktree with the om-developer and publishing isn't writing code.
It's the only publication point.

## om-developer

| Skill | Event that triggers it | What it does |
|---|---|---|
| `execute-task` | "context ready, start" from the om-reviewer, a message with findings, or "round N clean, document" | Implement mode if there's no task code yet; fix mode if there are findings or retakes; documentation mode on the clean signal. Rebase from `origin/{{base}}`, implements, `verify-task`, squashes into one commit, writes its closing note (what it did, what it left pending, decisions taken) in `task.md` or `phase_N.md`, notifies the om-reviewer "round N". |
| `document-task` | "round N clean, document" from the om-reviewer, once per task or phase | Updates `prd.md`, `trd.md`, `ard.md`, `database.md` and `flows.md` of the touched module with `updated` and `source` (Part 1), over the whole task's diff, when the code is final. Also the debt index of `docs/ARD.md`: rows for the debt it created, none for the debt it resolved. |

The om-developer never touches the remote.
Its reach outside the workspace is the task's `clearance`: none with `repo`, what the task names with `full`, each external action recorded in `om-developer notes`.
Its work ends in a local commit and a message to the om-reviewer.
It never talks to the om-manager or to Sebastian.

## om-devops

One session per `crew: devops` task (`om-{{id}}-devops`, alone in window `task-{{id}}`), instead of the pair.
It reuses the pair's skills in one loop that lives in its agent file; there is no `execute-task`, `review-task`, `start-task` or `next-phase` in its task.

| Skill | Event that triggers it | What it does |
|---|---|---|
| `analyze-task` | The session starts; `retakes updated` | Same as the om-reviewer's: consolidation once through the om-manager, `Context & decisions` with the clearance restated; on retakes, it applies them itself. |
| `verify-task` | Every round | Same as the om-developer's; nobody audits the log but Sebastian through the PR. |
| `document-task` | Once, when the work is final | Same as the om-developer's. |
| `publish-task` | After the docs commit | Same as the om-reviewer's; `Decisions` lists every external action applied, and the review and checks lines of the Pipeline read `n/a (crew: devops)`. |

## om-architect

A per-idea session (`om-{{title}}-architect`, window `plan-{{title}}`, cwd the root) the om-manager opens with `delegate-plan`; Sebastian plans with it there.
It runs `plan-task` and nothing else, may explore, prototype and query external systems with its scratch under `.workspaces/plan-{{title}}/`, and hands the draft over with `draft ready: {{path}}`.
It never creates tasks, never launches sessions and never changes an external system.

## Shared

| Skill | Who | What it does |
|---|---|---|
| `verify-task` | om-developer, or om-devops in a `crew: devops` task | Iterates the TRD's `Verification targets` (one per repo or app the task touches). Runs lint → typecheck → tests, one at a time and with `--runInBand`. For `type: docs` it doesn't run tests. One block per target in `verify.log`: what ran, when, the exit code of each step, the result and on which commit. Each command runs without a pipeline and appends its own `EXIT=$?` to a per-step artefact in `{{WORKSPACE}}/.verify/`; every log line is derived from that artefact. |

The `verify-task` log is the om-developer's claim that lint and tests ran after the last fix, and the `.verify/` artefacts are the evidence; the om-reviewer audits the log against them and never re-runs the commands.

## om-pr-reviewer

A per-PR session (`om-pr-{{n}}-reviewer`, window `pr-{{n}}`) the om-manager opens with `delegate-pr`; Sebastian talks to it in its window.
It never writes code, never runs anything, never talks to the om-manager; its only messages are events to `om-events`.

| Skill | Event that triggers it | What it does |
|---|---|---|
| `review-pr` | The launch prompt `/review-pr {{n}} [{{m}}]`, or `re-review, head {{sha}}` from the om-manager | Read-only review of a PR someone else wrote. Reads the description, the linked issue and the open threads, and the related PR `{{m}}` as context only; checks the PR branch out in a worktree under `.workspaces/pr-{{n}}/` and reads every changed file in full; runs nothing, since CI, lint and checks already show on the PR. Findings are typed comments (`issue`, `security`, `test`, `scope` block; `refactor`, `docs`, `question` do not by default; `suggestion` and `nit` never), few and well placed, at most three optional ones. Output in two layers: a short overview (verdict derived from the blocking comments, risk with the production-first rubric, size as counts, one line per comment split into required and optional, a closing summary with the counts) and one inline comment per finding with the explanation. Saved in the worktree and in the ignored `pr-reviews/`; posts nothing, offers nothing; sends `om-events` one `action` line. The worktree stays until `clean-pr`. When its previous report exists, it re-reviews: updates the worktree, marks each previous comment `addressed`, `still open` or `withdrawn`, and reviews the new code. |
| `approve-pr` | Sebastian says approve | `gh pr review --approve` with no body and no comments, on the same head the report reviewed. Sends `om-events` one `info` line. |
| `request-pr-changes` | Sebastian says which comments to send | Filters the saved overview and inline comments to his selection (every required one by default, plus the optional ones he names, minus the required ones he drops) and submits them as a `REQUEST_CHANGES` review on the same head. Sends `om-events` one `info` line. |

## State machines

### om-reviewer

```
starts ──> analyze-task ──> "consolidated" ──> waits "delegated, start"
delegated ──> start-task (launches om-developer) ──> waits
waits ──(round N)──> review-task ──(issues)──> sends findings ──> waits
                                 ──(no issues)──> "clean, document" ──> waits
waits ──(docs ready)──> publish-task ──> in_review ──> waits
in_review ──(phase N merged, phase N+1 exists)──> next-phase ──> start-task ──> waits
in_review ──(last phase merged, or no phases)──> ends
```

### om-developer

```
starts ──> waits for the om-reviewer's notice
notice ──> execute-task (implement) ──> "round 1" ──> waits
findings ──> execute-task (fix) ──> "round N" ──> waits
clean, document ──> document-task ──> "docs ready" ──> waits
```

## Name map

Final names, which replace the ones used provisionally in earlier conversations:

| Provisional | Final |
|---|---|
| `new-task` | `plan-task` + `create-task` |
| `implement-task` | `execute-task` |
| `retake-task` | `reiterate-task` |
| `summarize-task` | `publish-task` |
| `test-task` | `verify-task` |
| `reanalyze-task`, `reexecute-task` | removed; the mode is decided by the input |

## Relationship to current skills

| Current in `~/.claude/skills/` | Destination |
|---|---|
| `refine-us`, `us-to-tus`, `us-to-specs` | Absorbed into `plan-task` and `create-task`. |
| `implement-specs-{nestjs,nextjs,rails,react-native}` | Absorbed into `execute-task` with `references/{{stack}}.md`. |
| `pr-reviews` | Absorbed into `review-task` for the method's own tasks; `review-pr` covers PRs written by someone else (2026-09-14). |
| `address-pr-comments-nestjs` | Absorbed into `reiterate-task` + `execute-task` (fix mode). |
| `ds-write-prd`, `ds-write-trd` | Replaced by `write-prd`, `write-trd`, written from scratch. |

## Open items

- Write `~/.claude/agents/om-manager.md`, `om-reviewer.md` and `om-developer.md` with the list of allowed skills and each role's state machine.
- Define the exact format of `verify.log`.
- Define the format of the om-reviewer's findings message to the om-developer.

## Inventory (2026-08-29)

All written as drafts in `skills/`, pending pilot (see [06-pilot.md](06-pilot.md)).

| Role | Skills |
|---|---|
| om-manager | `setup`, `write-prd`, `write-trd`, `write-ard`, `add-check`, `plan-task`, `create-task`, `consolidate-task`, `delegate-task`, `reiterate-task`, `check-task`, `check-work`, `clean-task`, `clean-work`, `delegate-pr`, `delegate-plan`, `check-prs`, `clean-pr` |
| om-reviewer | `analyze-task`, `start-task`, `review-task`, `publish-task`, `next-phase` |
| om-developer | `execute-task`, `document-task`, `verify-task` |
| om-devops | `analyze-task`, `verify-task`, `document-task`, `publish-task` |
| om-architect | `plan-task` |
| overmind | `add-project` (with `pause` and `remove`), `resume-project`, `check-portfolio`, `clean-portfolio`, `add-todo`, `complete-todo` |
| om-pr-reviewer | `review-pr`, `approve-pr`, `request-pr-changes` |
| om-config (project skill in `.claude/skills/`) | `update-method` |

Global agents in `agents/`: `om-manager`, `om-reviewer`, `om-developer`, `om-devops`, `om-architect`, `om-pr-reviewer`, `om-setup-worker` (subagent of `setup` with `discover` and `document` modes).
Agents of this repo in `.claude/agents/`: `overmind`, `om-events`, `om-config`.

Content pending: `references/{{stack}}.md` for `execute-task` and `verify-task` (nestjs, nextjs, rails, react-native).
They get written when the first project of each stack goes through `setup`; the project's TRD is the source and the reference is only the fallback.
The previous skills in `~/.claude/skills/` were deleted by Sebastian on 2026-08-29 because they weren't being used.
