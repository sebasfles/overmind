# overmind

Sebastian's system for working with Claude Code agents across many projects at once.

One `om-manager` per project to plan with; per task an `om-reviewer` that never writes code and an `om-developer` that never pushes; an `overmind` cockpit above all projects.
Documentation is extracted into a fixed convention so agents never rediscover a project from zero.

- Design: `docs/01` (documentation), `02` (orchestration), `03` (skills), `04` (operation), `05` (single, mono and multirepo layouts), `06` (pilot checklist), `method-ard.md` (later decisions).
- Method: `agents/` (global roles), `.claude/agents/` (cockpit roles), `skills/`, `bin/`.
- State: `portfolio/`.

Install: `bin/install` links roles, skills and scripts into `~/.claude` and `~/bin`.
Start: `bin/resume-overmind`.
Change the method: in the `om-config` session, `/update-method`.
