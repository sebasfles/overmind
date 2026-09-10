---
name: document-task
description: Update the module docs affected by the task, with updated and source. om-developer; once per task or phase, on the om-reviewer's clean signal.
effort: medium
disable-model-invocation: false
---

# document-task

## Purpose

Update the module documentation affected by the task (or phase): prd.md, trd.md, ard.md, database.md, flows.md
of each module touched, with updated and source frontmatter, and one ARD entry per decision not already
recorded. om-developer only; runs once, on `round {{N}} clean, document`, when the code is final and before publish.

Input: the diff of the task (or phase) against `origin/{{base}}`, the task folder and `om-developer notes`.
Output: `docs/modules/{{module}}/*.md` updated for every module the task touched, and the `Debt index` of `docs/ARD.md` current, both in the root worktree.

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
One entry per decision recorded in `om-developer notes` that `task.md#Approach` or `Context & decisions` did not already record, and one per finding you disagreed with but applied.
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

If a decision changes a global one in `docs/ARD.md`, do not edit its `Decisions`; add the module entry and flag it in `om-developer notes` for the om-reviewer.

If the task paid a debt an earlier entry recorded, that entry is not superseded and not rewritten: add one line under its `Debt created`.

```
- Resolved by: {{id}}_{{title}}, {{YYYY-MM-DD}}
```

Only for debt this task actually removed, with the code to show it.
The entry normally lives in a module of the diff; if it does not, leave it and say so in `om-developer notes`.

## 4. Debt index

`docs/ARD.md` has one derived part, the `Debt index`, and this task just changed it in two ways: the debt its entries created, and the debt it resolved.
Bring the table in line with the module ARDs, in the root worktree, so it travels in the same PR as the entries it indexes:

- One row per `Debt created` you wrote in step 3, with module, date, the debt in one line and its trigger.
- Remove the row of every entry that now carries `Resolved by`.
- Touch nothing else in the file, and bump its `updated`.

The rest of `docs/ARD.md` stays `setup`'s and the om-manager's.

## 5. Check

Every module in the diff has its docs touched or a one-line justification in `om-developer notes` of why nothing changed.
`updated` and `source` are set on every file you edited.
Nothing in the docs restates code.

## Rules

- Only files under `docs/modules/`, plus the `Debt index` of `docs/ARD.md`; the rest of that file, `docs/PRD.md` and `docs/TRD.md` are `setup`'s and the om-manager's.
- Never touch `docs/tasks/`; that is not documentation.
- Files are in English.
