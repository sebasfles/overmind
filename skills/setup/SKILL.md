---
name: setup
description: Extract a project's documentation, wherever it is, into the docs/ convention, together with Sebastian. om-manager; Sebastian invokes it on any project joining the system.
effort: high
argument-hint: "[check | fill] [OVERVIEW_FILE] [DESIGN_FOLDER]"
disable-model-invocation: true
---

# setup

## Purpose

Bring a project into the documentation convention by extracting what is already known about it: existing docs wherever they live (READMEs, wikis, ADRs, comments, specs, commit history, in the root or in any component), the code itself, and Sebastian.
It does not expect the convention to exist; the convention is its output.
On a project that already follows it, it reconciles: refreshes stale files, creates missing ones, never deletes.
Idempotent, interactive, and parallel per module.

Input: optional mode (`check` only reports; `fill` reports and then fills, the default), and optionally an overview document and a design folder for the PRD.
Output: `docs/` in the convention (created from scratch or reconciled), `CLAUDE.md` short, a list of `[inferido]` items for Sebastian, one commit on the base branch.
The convention is the target shape, never a precondition: a project may arrive with docs anywhere, in any format, or with none.

The convention (see `docs/01-documentation.md` of the overmind repo):

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
- Whether the convention already exists, partially or not at all: which of its files are present, their `updated` date, and whether the code under each module changed after that date (`git log -1 --format=%cs -- {{module path}}` vs `updated`). Stale means code newer than docs. Absence is the normal case for a new project, not an error.
- `CLAUDE.md`: exists, and is it the short form (points to `docs/`, under 40 lines)?
- `.gitignore` has `docs/tasks/_drafts/` and, in single and mono, `.workspaces/`; in multirepo, the root `.gitignore` lists every code repo folder and `.workspaces/`.

## 1b. Multirepo folder without a root repo

If the root is a folder of repos with no `.git` of its own, the convention cannot hold there yet: `docs/` would be unversioned.
Offer Sebastian the option before anything else: make the folder a docs-only repo (`{{name}}-docs`, private, on his personal GitHub) whose `.gitignore` lists every code repo folder and `.workspaces/`.
With his yes: `git init`, write `.gitignore`, first commit `docs: init project root`, add the remote he gives.
With his no: stop; `setup` needs a versioned root. See `docs/05-layouts.md` of the overmind repo.

## 2. Discovery per component

Discovery is read-only and independent per component (repo or app), so it runs in parallel.
With more than one component, run it as a Workflow (load `workflow-authoring` first; this skill is the opt-in): `parallel` over components, concurrency 4, each step an `agent()` with the `om-setup-worker` brief in `discover` mode, `model: sonnet` (reading and inventorying does not need more), and this schema:

```
{ component, stack, base_branch, docs_found: [{path, covers, freshness}],
  layout: [{path, what}], candidate_modules: [{name, paths, purpose_guess}],
  debt_evidence: [{path, line, text}], verify_commands: {lint, typecheck, unit, e2e, install, workspace_files} }
```

Fallback without the Workflow tool: `Agent(om-setup-worker)` subagents, at most four at a time.
With a single component, do the discovery in this session.

Each worker reads, in its component, what people already wrote before reading code:

- `README*`, `CONTRIBUTING*`, `ARCHITECTURE*`, `CHANGELOG*`.
- `docs/`, `doc/`, `wiki/`, `adr/`, `decisions/`, `rfcs/` folders and anything `.md` outside `node_modules` and vendor folders.
- Long comments at the top of entrypoints and module roots; `TODO`, `FIXME`, `HACK` comments (they are debt evidence).
- API spec files, Postman collections, schema files, diagrams (`.mmd`, `.puml`, `.drawio`, images under docs).
- Commit messages and merged PR titles of the last months for decisions stated in words (`git log --merges --format=%s`).

Documentation can be anywhere and in any shape: a `NOTES.md` at the root, a Confluence export in `docs/legacy/`, a `docs/` in one repo and nothing in the others, a wiki checked in as a submodule.
Read all of it; do not skip a source because it is not where the convention would put it.
Then the code: structure, entrypoints, how the code already groups itself (NestJS modules, Rails engines, feature folders), manifests and scripts for the verification commands.

The main session merges the results into one inventory (path, what it covers, how current it looks) and one list of candidate modules.
Modules of the application often span components (`billing` in api, web and mobile): the merge is where they appear, by matching names, shared entities and call paths across the per-component candidates.
That merge is your work with Sebastian, never a worker's.
The inventory is the primary source for PRD intent and ARD reasons; code is the primary source for the TRD.
Nothing of what exists is deleted or moved; it is read, cited and, when a fact is confirmed, folded into the convention with its source noted.
Discovery is the first Workflow of `setup`; the module documentation in step 4 is the second.
They are separate because Sebastian's confirmation of the modules sits between them, and a Workflow cannot stop to ask.

## 3. Report and confirm modules

From the merged discovery, one screen to Sebastian:

```
Stack        NestJS 10, Prisma, pnpm          base: develop
Existing     README (2024), docs/adr/ (6 entries), swagger.json, 14 TODOs
Modules      7 detected: auth, billing, ...   (2 uncertain: shared, legacy)
docs/        missing                          (or: 4/7 modules documented, 2 stale)
CLAUDE.md    long (180 lines), not pointing to docs/
```

Then confirm the module list with him: name, purpose in one line, and which folders in which components belong to it.
Module boundaries are the one decision you must not guess.
In `check` mode, stop here.

## 4. Fill, in order, with Sebastian

`setup` is a working session, not a batch job.
Every document is written from three sources in this order: the inventory, the code, and Sebastian.
Before writing each document, ask him in one batch what neither the inventory nor the code can tell (for whom the product is, what is deliberately out, why a choice was made, what the environments are).
Do not ask what the inventory or the code already answers.

1. Modules, in parallel, as the second Workflow.
   The per-module documentation is N independent jobs with clean context: run them with the Workflow tool (load the `workflow-authoring` skill first).
   This skill instructing you to use Workflow is the opt-in; Sebastian does not need to say "ultracode".
   Script shape: `parallel` over the confirmed modules with concurrency 4, each step an `agent()` with the `om-setup-worker` brief in `document` mode and a schema `{module, files_written: [...], inferidos: [{file, line, text}]}`; return the union.
   If a module fails, fix the brief and resume the run with `resumeFromRunId`; finished modules come back from cache.
   If the Workflow tool is not available in the session, fall back to `Agent(om-setup-worker)` subagents, at most four at a time.
   Brief per module, in both cases:
   - module name, purpose, folders per component, the confirmed module list, the base branch, the inventory items that mention the module, and Sebastian's answers that concern it.
   - run `write-trd {{module}}`, `write-prd {{module}}`, `write-ard {{module}}`.
   - write `README.md` from `templates/module-README.md` and `database.md` from `templates/module-database.md`; `flows.md` from `templates/module-flows.md` only if the module has a flow worth a diagram.
   - return the `[inferido]` items it wrote, with file and line.
   The interactive parts of `setup` (merging the inventory, module confirmation, interviews) stay in this session; only the two fan-outs run as Workflows.
2. General TRD: run `write-trd general`. The module docs give it the Modules table; the detection gives it components, verification targets and workspace files. If any command comes out `unknown`, ask Sebastian.
3. General PRD: run `write-prd general` with the inventory's product documents and Sebastian's answers; it links the module `prd.md` files.
4. General ARD: run `write-ard general`. Optional in substance: on a new repo with no history, create it with the template header and an empty log, so `document-task` has where to append; do not invent entries. On an existing repo, record only decisions with evidence and rebuild the debt index.
5. `CLAUDE.md`: write or rewrite it from `templates/CLAUDE.md`, under 40 lines. If a long one exists, move anything that is not a rule into the TRD and keep the rules.
6. `.gitignore`: add `docs/tasks/_drafts/` and `.workspaces/` (multirepo: also every code repo folder), and create `docs/tasks/.gitkeep`.

On a partial repo, skip files that exist and are not stale; refresh stale ones; create missing ones.
Never delete documentation you did not write.

## 5. Inferences

Collect every `[inferido]` from the workers and from your own writes.
Present them grouped by file, one line each, and ask Sebastian to confirm, correct or delete.
Apply his answers: confirmed items lose the marker; deleted items are removed; corrected items are rewritten.
Items he does not answer keep the marker; they are visible to every future agent as uncertain.

## 6. Commit

Show the file list.
Ask: "Commit and push the documentation to {{base}}?"
Only then:

```
git add docs CLAUDE.md .gitignore
git commit -m "docs: setup documentation convention"
git push origin {{base}}
```

## Rules

- Never touch application code.
- Never guess module boundaries; Sebastian confirms them.
- Never write a document without first reading the existing documentation and asking Sebastian what neither it nor the code can answer.
- Never treat the absence of the convention as a problem; extracting it is the job.
- Never invent decisions or debt without evidence; mark inferences.
- Idempotent: running it twice in a row changes nothing the second time except `updated` on stale files.
- English in every file.
