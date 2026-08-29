---
name: add-project
description: >-
  Register a project in portfolio/projects.yaml, check it follows the documentation convention, and
  offer to run setup in its manager if it does not. Also pause-project and remove-project by argument.
  Overmind only.
argument-hint: "[PATH_OR_NAME] [pause | remove]"
disable-model-invocation: true
---

# add-project

Input: a repo path (to add), or a registered name with `pause` or `remove`.
Output: `portfolio/projects.yaml` updated and committed.

## Add

1. Resolve the path; it must be a git repo. `name` = folder name unless Sebastian gives one.
2. Detect `base_branch`: `docs/TRD.md` if present, else the remote default (`git symbolic-ref refs/remotes/origin/HEAD`), else ask.
3. `tmux` session name = `name`.
4. Append to `projects.yaml` with `status: active`.
5. Convention check, read-only: `CLAUDE.md` short and pointing to `docs/`? `docs/PRD.md`, `TRD.md`, `ARD.md` present? `docs/modules/` non-empty? `docs/tasks/` present?
   Report in one line per item.
   If anything is missing, say: `run setup from its manager: resume-project {{name}}`. Do not run `setup` yourself.
6. Commit: `portfolio: add project {{name}}`.

## Pause

Set `status: paused`. The project drops out of `check-portfolio` and `clean-portfolio` but stays registered.
Commit: `portfolio: pause {{name}}`.

## Remove

Delete the entry. Nothing on disk is touched.
Commit: `portfolio: remove {{name}}`.

## Rules

- Never run `setup` or any manager skill.
- Never modify the project.
