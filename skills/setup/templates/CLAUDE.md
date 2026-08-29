# {{Project}}

{{One sentence: what this is.}}

## Where things are

- Product, technical and architecture docs live in `docs/`. Start with `docs/TRD.md` for stack and commands, `docs/PRD.md` for what the product does, `docs/ARD.md` for decisions and debt.
- Each module has its own folder in `docs/modules/{{module}}/` with `README.md`, `prd.md`, `trd.md`, `ard.md`, `database.md` and optionally `flows.md`.
- Tasks live in `docs/tasks/`; drafts in `docs/tasks/_drafts/` are not tracked.

## Read before working

1. This file.
2. `docs/TRD.md` and `docs/PRD.md`.
3. Only the module you are about to touch.

## Rules

- Base branch: `{{develop}}`.
- Run tests one at a time with `--runInBand`; see the Verification section of `docs/TRD.md`.
- {{project-specific rule kept from the previous CLAUDE.md, if any}}
