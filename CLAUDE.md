# overmind

Sebastian's system for working with Claude Code agents across many projects.
Method and state live together here; `~/.claude` only holds symlinks into this repo.

## Layout

- `docs/`: the plan and its decisions. Start with `docs/01-documentacion.md`, then `02`, `03`, `04`.
- `agents/`: global roles that run inside projects (`om-manager`, `om-reviewer`, `om-developer`, `om-setup-worker`). Symlinked from `~/.claude/agents/`.
- `.claude/agents/`: roles that only run in this repo (`overmind`, `om-events`, `om-config`). Three long-lived sessions: `overmind` (cockpit, left pane), `om-events` (inbox, right pane), `om-config` (method, window `config`). `bin/resume-overmind` opens them.
- `skills/`: one folder per skill. Symlinked from `~/.claude/skills/`.
- `bin/`: deterministic scripts. `resume-project` is symlinked from `~/bin/`; `lint-method` checks this repo.
- `portfolio/`: state. `projects.yaml` is the project registry, `todos.md` the quick-capture list, `events/{blockers,actions,info}.md` the inbox.
- `.claude/skills/update-method/`: the project skill that changes the method consistently.

## Rules for agents working in this repo

- Edits to `agents/` and `skills/` are edits to the method: do them in the `om-config` session with `/update-method`, which lints with `bin/lint-method` and records why in `docs/method-ard.md`.
- Questions about projects, state or notifications belong in the `overmind` session.
- `portfolio/` changes are committed by the overmind as small commits.
- Each full sentence on its own line in Markdown.
- Never use the em dash.
