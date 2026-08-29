---
name: add-project
description: >-
  Register a project in portfolio/projects.yaml with its root and repos (single repo, monorepo or a
  multirepo folder), make the root a git repo when it is a multirepo folder, check the documentation
  convention, and offer to run setup in its manager. Also pause-project and remove-project by argument.
  Overmind only.
argument-hint: "[PATH_OR_NAME] [pause | remove]"
disable-model-invocation: true
---

# add-project

Input: a path (to add), or a registered name with `pause` or `remove`.
Output: `portfolio/projects.yaml` updated and committed.

## Add

1. Detect the layout of the path:
   - `.git` at the path: single repo or monorepo. `repos: [{name: ".", base_branch}]`.
   - No `.git` at the path but child folders with `.git`: multirepo. `repos:` one entry per child repo, `base_branch` from each remote's default (`git symbolic-ref refs/remotes/origin/HEAD`) or ask.
2. `name` = folder name unless Sebastian gives one; `root` = the path; `tmux` = `name`.
3. Multirepo without `.git` at the root: propose to make the root a docs repo and, with Sebastian's yes:
   ```
   git -C {{root}} init
   printf '%s/\n' {{each repo folder}} .workspaces > {{root}}/.gitignore
   git -C {{root}} add .gitignore && git -C {{root}} commit -m "docs: init project root"
   ```
   Ask for the remote (suggest `{{name}}-docs` on his personal GitHub) and add it if given.
4. Single or mono: make sure `.workspaces/` is in the repo's `.gitignore` (add it and tell Sebastian it needs a commit; do not commit in his repo).
5. Append to `projects.yaml`:
   ```yaml
   - name: diy
     root: /home/fless/dev/designli/projects/diy
     tmux: diy
     status: active
     repos:
       - name: diy-platform
         base_branch: develop
       - name: diy-infra
         base_branch: main
   ```
6. Convention check, read-only: `CLAUDE.md` short and pointing to `docs/`? `docs/PRD.md`, `TRD.md`, `ARD.md`? `docs/modules/` non-empty? `docs/tasks/`?
   One line per item; if anything is missing: `run setup from its manager: resume-project {{name}}`.
7. Commit: `portfolio: add project {{name}}`.

## Pause

`status: paused`; drops out of `check-portfolio` and `clean-portfolio`. Commit `portfolio: pause {{name}}`.

## Remove

Delete the entry; nothing on disk is touched. Commit `portfolio: remove {{name}}`.

## Rules

- Never run `setup` or any manager skill.
- Never commit inside a code repo.
