---
name: setup
description: >-
  Create or reconcile the documentation convention in a project: detect stack, modules and the state of
  docs/, report the gap between ideal and actual, fill it (general TRD, then every module in parallel
  with setup-worker subagents, then general PRD and ARD), write the short CLAUDE.md, collect every
  [inferido] for Sebastian, and commit with his approval. Idempotent: works on an empty repo, an old one
  without docs, or a partial one. Manager only; Sebastian invokes it.
argument-hint: "[check | fill] [OVERVIEW_FILE] [DESIGN_FOLDER]"
disable-model-invocation: true
---

# setup

Input: optional mode (`check` only reports; `fill` reports and then fills, the default), and optionally an overview document and a design folder for the PRD.
Output: `docs/` matching the convention, `CLAUDE.md` short, a list of `[inferido]` items for Sebastian, one commit on the base branch.

The convention (see `docs/01-documentacion.md` of the overmind repo):

```
CLAUDE.md
docs/
  PRD.md  TRD.md  ARD.md
  modules/{{module}}/  README.md  prd.md  trd.md  ard.md  database.md  flows.md (optional)
  tasks/  (_drafts/ ignored)
```

Run in the project root (see `docs/05-layouts.md`): the repo itself for single and mono, the docs repo for multirepo.
Components are the repos (multirepo) or apps (monorepo) the TRD will describe; in a single repo there is one.

## 1. Detect

Without writing anything:

- Layout: single, mono or multirepo, from `.git` at the root and child repos or apps. Components and their paths.
- Stack per component, from manifests and lockfiles.
- Base branch, from the remote's default or the existing TRD.
- Candidate modules: the bounded areas the code already groups (NestJS modules, Rails engines or namespaces, feature folders). Modules belong to the application and may span components; propose the mapping folder(s) -> module across components, with a one-line guess of purpose.
- State of `docs/`: which files exist, their `updated` date, and for each module folder whether the code under it changed after that date (`git log -1 --format=%cs -- {{module path}}` vs `updated`). Stale means code newer than docs.
- `CLAUDE.md`: exists, and is it the short form (points to `docs/`, under 40 lines)?
- `.gitignore` has `docs/tasks/_drafts/` and, in single and mono, `.workspaces/`; in multirepo, the root `.gitignore` lists every code repo folder and `.workspaces/`.

## 2. Report the gap

One screen, to Sebastian:

```
Stack        NestJS 10, Prisma, pnpm          base: develop
Modules      7 detected: auth, billing, ...   (2 uncertain: shared, legacy)
docs/        missing                          (or: 4/7 modules documented, 2 stale)
CLAUDE.md    long (180 lines), not pointing to docs/
```

Then ask him to confirm the module list; module boundaries are the one decision you must not guess.
In `check` mode, stop here.

## 3. Fill, in order

Order matters because later documents index earlier ones.

1. General TRD: run `write-trd general`. It needs the confirmed components and modules, and produces the verification targets and workspace files per component. If anything comes out `unknown`, ask Sebastian now.
2. Modules, in parallel: one `setup-worker` subagent per module, each with a clean context and this brief:
   - module name and root path, the confirmed module list, the base branch.
   - run `write-trd {{module}}`, `write-prd {{module}}` (pass overview and designs if given), `write-ard {{module}}`.
   - write `README.md` from `templates/module-README.md` and `database.md` from `templates/module-database.md`; write `flows.md` from `templates/module-flows.md` only if the module has a flow worth a state or sequence diagram.
   - return the list of `[inferido]` items it wrote, with file and line.
   Run at most four workers at a time; the machine also runs other tasks.
3. General PRD: run `write-prd general`, now that module `prd.md` files exist to link.
4. General ARD: run `write-ard general`; it rebuilds the debt index from the module ARDs.
5. `CLAUDE.md`: write or rewrite it from `templates/CLAUDE.md`, under 40 lines. If a long one exists, move anything that is not a rule into the TRD and keep the rules.
6. `.gitignore`: add `docs/tasks/_drafts/` and `.workspaces/` (multirepo: also every code repo folder), and create `docs/tasks/.gitkeep`.

On a partial repo, skip files that exist and are not stale; refresh stale ones; create missing ones.
Never delete documentation you did not write.

## 4. Inferences

Collect every `[inferido]` from the workers and from your own writes.
Present them grouped by file, one line each, and ask Sebastian to confirm, correct or delete.
Apply his answers: confirmed items lose the marker; deleted items are removed; corrected items are rewritten.
Items he does not answer keep the marker; they are visible to every future agent as uncertain.

## 5. Commit

Show the file list.
Ask: "¿Commiteo y pusheo la documentación a {{base}}?"
Only then:

```
git add docs CLAUDE.md .gitignore
git commit -m "docs: setup documentation convention"
git push origin {{base}}
```

## Rules

- Never touch application code.
- Never guess module boundaries; Sebastian confirms them.
- Never invent decisions or debt without evidence; mark inferences.
- Idempotent: running it twice in a row changes nothing the second time except `updated` on stale files.
- English in every file.
