---
name: plan-task
description: Plan a piece of work with Sebastian for this project. Reads the project docs following the reading route, asks only what the docs cannot answer, and produces an approved plan (goal, scope, acceptance, approach, phases). Ends by offering create-task. Manager only; Sebastian invokes it.
argument-hint: "[DESCRIPTION_OR_TICKET]"
disable-model-invocation: true
effort: high
---

# plan-task

Input: `$ARGUMENTS`, a free-text description from Sebastian, a ticket reference, or nothing (then ask what he wants to build).

Output: a plan approved by Sebastian, handed to `create-task` immediately after approval.
The only thing this skill may write is a draft under `docs/tasks/_drafts/`.

## 1. Read before asking

Follow the reading route and stop as soon as you have enough:

1. `CLAUDE.md`: stack, commands, where docs live, base branch.
2. `docs/PRD.md`, `docs/TRD.md`: what the product is, how it is built, list of modules, platform (backend, frontend, mobile, or several).
3. `docs/ARD.md`: global decisions and known debt that constrain this work.
4. Pick the modules the work belongs to; one is primary, the rest are touched.
5. Read only those modules: `docs/modules/{{module}}/README.md`, then `prd.md`, `trd.md`, `ard.md`, `database.md`, `flows.md` as needed.
6. `docs/tasks/`: anything planned, in progress or in review that overlaps; candidates for `depends_on`.

If `docs/` does not exist or is stale, stop and tell Sebastian to run `setup` first.
Do not plan against a codebase you have to rediscover from scratch.

## 2. Classify

Decide and state:

- `type`: `feature`, `bug`, `docs`, `chore` or `refactor`.
- `modules`: every module touched, primary first. A task may span several modules; do not force it into one.
- platform track(s): backend, frontend, mobile, infra. The TRD tells you. Infra is usually an extra track on top of a platform one.

For `type: bug`, the plan must include a replication section: exact steps to reproduce end-to-end as a user would, expected vs observed, environment, evidence (logs, screenshots, ids).
`create-task` writes it to `replication.md`; the developer must reproduce it before touching code and the reviewer verifies the fix against it.
For `type: refactor`, acceptance is "behavior identical", stated as what must not change.

## 3. Discover, in batches

Ask only what `docs/` and the code do not answer.
Group questions by topic and send one group at a time, at most five questions per group.
Lead each group with what you already concluded from the docs, so Sebastian only confirms or corrects.
Give a recommendation whenever there is a choice; never present a bare menu.

Load the track that applies from `references/` and use it as a checklist, not a script:

- `references/discovery-backend.md`
- `references/discovery-frontend.md`
- `references/discovery-mobile.md`
- `references/discovery-infra.md` (add it whenever the work touches IaC, environments, secrets, networking or CI/CD)

If the description or ticket links a Figma file, note the link for the plan; the developer will pull design context from it.

## 4. Draft the plan

Present it in this shape, in English, short:

```
Goal          one paragraph: what and why, from the user's perspective
Type / Modules
Scope         what is in
Out of scope  what is explicitly not in, and deferred ideas
Acceptance    numbered, observable, testable criteria
Approach      repos or apps touched, module and layers, entities, endpoints, tables, key decisions
              each decision with the alternative rejected and the reason (these become ARD entries)
Database      tables owned or referenced, migrations expected, invariants (or "none")
Infra         resources, env vars, secrets, IAM, CI/CD changes, per environment (or "none")
Design        Figma links, if any
Replication   bugs only: steps, expected vs observed, environment, evidence
Risks         what could go wrong, what is uncertain
Phases        almost never, see below
Depends on    other tasks that must be done first (or "none")
Ticket        external reference, if any
```

Do not invent anything the docs or Sebastian did not give you.
Do not widen the scope; if you see adjacent work worth doing, list it under Out of scope as deferred.

## 5. Phases

The default is no phases; almost every task is a single PR.
Propose phases only when it is evident that the work will not fit in one PR that Sebastian can review in one sitting, and let him decide.
Rules:

- Each phase is a vertical slice: a thin, complete path through every layer, demoable on its own.
- Phases are sequential; phase N+1 starts after phase N is merged.
- Prefer few phases; three is a lot.
- Each phase gets its own Scope and Acceptance; the task keeps the overall Goal.

Ask Sebastian whether the granularity feels right and whether the order is correct.

## 6. Keep a draft on disk when the conversation grows

A long planning conversation lives only in context and is lost if the session dies or compacts.
When the plan gets long, or the direction changes, write the current state to `docs/tasks/_drafts/{{title}}.md` and keep it updated.
It is a draft, not a task: it has no id, no status, and `check-work` ignores `_drafts/`.
`create-task` accepts it as input and deletes it once the task folder exists.
Do not publish artifacts on your own; if Sebastian wants one, he will ask.

## 7. Approve and hand off

Iterate until Sebastian approves the plan.
Then ask: "¿Creo la task?" and, if yes, invoke `create-task` with the approved plan in context.
If Sebastian says not now, keep the plan in the conversation and remind him it is not on disk yet.

## Rules

- Never write code, pseudo-code or file contents.
- Never write to disk except `docs/tasks/_drafts/`; task folders are `create-task`'s job.
- Speak to Sebastian in his language; the plan itself is in English.
- If Sebastian's answers reveal a general preference rather than a one-off, ask whether to record it in `~/.claude/CLAUDE.md`.
