---
name: update-method
description: Apply a change to the method (skills, agents, docs) consistently, lint it, record why, commit. Project skill of the overmind repo; Sebastian invokes it.
effort: high
argument-hint: "[CHANGE]"
disable-model-invocation: true
---

# update-method

## Purpose

Every change to how the agents work touches several files at once: a rename crosses 30 files, a new rule belongs in one agent and three skills, a new skill needs its row in `docs/03-skills.md`.
This skill applies a change everywhere it belongs, checks the result with `scripts/lint-method`, records the reason in `docs/method-ard.md`, and commits.
It replaces doing that by hand and forgetting one file.

Where: in the `om-config` session (agent `om-config`), which holds the method's full context and is long-lived.
If invoked from any other session, do not apply the change there: open or resume `om-config` (window `config` of the overmind tmux session, `claude --agent om-config -n om-config`), forward the request in one line, and stop.

Input: `$ARGUMENTS`, the change in Sebastian's words (a rename, a new or changed rule, a new skill or agent, a removal).
Output: a lint-clean repo with one commit, and an ARD entry.

## 1. Understand the change

Classify it:

- Rename: an agent, a skill, a session name, a path, a term (`worktree` to `workspace`).
- Rule: something an agent must always or never do, or a step added to a skill.
- New skill or agent.
- Removal or merge of skills.
- Design: a decision that changes a document in `docs/`.

Read the documents that define the vocabulary before editing: `docs/02-orquestacion.md`, `docs/03-skills.md`, `docs/05-layouts.md`.
If the change contradicts a decision recorded there, say so to Sebastian and get his yes before continuing; the ARD entry will say `Supersedes`.

## 2. Find every place it touches

`grep -rn` over `agents/`, `skills/`, `docs/`, `bin/`, `CLAUDE.md` for every term involved, including Spanish and English forms, backticked and plain, singular and plural, in frontmatter, prose, tables, code blocks and session names.
List the files before editing; the list goes into the commit message.

## 3. Apply

- Renames: exact tokens first (frontmatter `name`, `--agent`, session names, paths), then prose; rerun the grep until it is empty.
- Rules: in the agent that owns the behavior (`## What you never do`, `## Rules`) and in every skill whose steps it changes; also the row in `docs/03-skills.md` if the skill's contract changed.
- New skill: folder `skills/{{name}}/SKILL.md` with frontmatter (`name` = folder, `description` under 170 chars saying what and when, `argument-hint` quoted, `disable-model-invocation`), body with `## Purpose`, Input/Output, numbered steps, `## Rules`; templates and references in subfolders; add it to the right agent's skill table and to `docs/03-skills.md`.
- New agent: `agents/{{name}}.md`; the name must be `overmind` or start with `om-`; add it to `scripts/lint-method`'s allowed list and to `docs/04-operacion.md`.
- Removal or merge: delete the folder, move what survives into the target skill, update every reference, add the pair to the name map in `docs/03-skills.md`.
- Design: edit the document that owns the decision and every skill or agent that implemented the old one.

Writing conventions: English in every file of the repo, including `docs/`; one sentence per line; no em dash; placeholders as `{{...}}`; roles always by agent name (`om-reviewer`), the person always `Sebastian`.

## 4. Lint

Run `scripts/lint-method`.
Fix every finding; do not commit with findings.

## 5. Record

Append to `docs/method-ard.md` an entry with the ARD format: date, decision in one line, alternatives, reason, debt, revisit when, and `Files: {{list}}`.
If the change reverses an earlier entry, mark `Supersedes`.

## 6. Commit

One commit, message in the form `method: {{change in one line}}`.
Never co-author.

## Rules

- Never change behavior silently: every change has an ARD entry.
- Never leave a term half renamed; the grep must come back empty.
- Never touch `portfolio/`; that is the overmind's state, not the method.
