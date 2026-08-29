---
name: document-task
description: Update the module docs affected by the round, with updated and source. om-developer; at the end of every execute-task round.
effort: medium
disable-model-invocation: false
---

# document-task

## Purpose

Update the module documentation affected by the current round: prd.md, trd.md, ard.md, database.md, flows.md
of each module touched, with updated and source frontmatter, and one ARD entry per decision not already
recorded. om-developer only; runs at the end of every execute-task round.

Input: the diff of the current round against `origin/{{base}}` and the task folder.
Output: `docs/modules/{{module}}/*.md` updated for every module the round touched, in the worktree.

Rule of the convention: do not document what the code already says; document why.
Derivable facts (columns, types, request and response shapes) are generated elsewhere and are not copied here.

## 1. Which modules

`modules` from `task.md`, plus any module whose code the diff touched that the plan did not list.
If the diff touches a module with no `docs/modules/{{module}}/` folder, create it from `templates/` and say so in `om-developer notes`; the plan missed it.

## 2. Per module, per file

Edit in place; these files describe the current state, not history.

| File | Update when | What |
|---|---|---|
| `README.md` | The module's purpose, boundaries or list of sub-features changed. | Keep it 20 to 40 lines with links to the other files. |
| `prd.md` | User-visible behavior changed. | What the module does for the user and why, without engineering detail. |
| `trd.md` | Endpoints, jobs, integrations, configuration or structure changed. | One line per endpoint the module owns, with purpose and a link to the generated API spec. No request or response schemas. |
| `database.md` | Tables, relationships or invariants changed. | Tables owned vs referenced; invariants kept in code, not in the schema. No columns or types. |
| `flows.md` | A complex flow was added or changed. | Mermaid state or sequence diagram. Only for flows that deserve one. |

Frontmatter of every file you touch:

```yaml
---
updated: {{today}}
source: {{id}}_{{title}}
---
```

## 3. ARD entries

`ard.md` is a log; append, never rewrite.
One entry per decision taken in this round that `task.md#Approach` or `Context & decisions` did not already record, and one per finding you disagreed with but applied.
Also one entry per piece of technical debt you knowingly created or deferred.

Read `templates/ard-entry.md` and fill it:

```
## {{today}}: {{decision in one line}}

- Decision: ...
- Alternatives rejected: ...
- Reason: ...
- Debt created: ... (or none)
- Revisit when: ... (a concrete trigger)
- Source: {{id}}_{{title}}
```

If a decision changes a global one in `docs/ARD.md`, do not edit `docs/ARD.md`; add the module entry and flag it in `om-developer notes` for the om-reviewer.

## 4. Check

Every module in the diff has its docs touched or a one-line justification in `om-developer notes` of why nothing changed.
`updated` and `source` are set on every file you edited.
Nothing in the docs restates code.

## Rules

- Only files under `docs/modules/`; the general `docs/PRD.md`, `TRD.md`, `ARD.md` are `setup`'s and the om-manager's.
- Never touch `docs/tasks/`; that is not documentation.
- Files are in English.
