# overmind

Sebastian's system for working with Claude Code agents across many projects.
Method and state live together here; `~/.claude` only holds symlinks into this repo.

## Layout

- `docs/`: the plan and its decisions. Start with `docs/01-documentacion.md`, then `02`, `03`, `04`.
- `agents/`: role definitions (`overmind`, `manager`, `reviewer`, `developer`). Symlinked from `~/.claude/agents/`.
- `skills/`: one folder per skill. Symlinked from `~/.claude/skills/`.
- `bin/`: deterministic scripts (`resume-project`). Symlinked from `~/bin/`.
- `portfolio/`: state. `projects.yaml` is the project registry, `todos.md` the quick-capture list.

## Rules for agents working in this repo

- Edits to `agents/` and `skills/` are edits to the method; record why in `docs/05-ard.md`.
- `portfolio/` changes are committed by the overmind as small commits.
- Each full sentence on its own line in Markdown.
- Never use the em dash.
