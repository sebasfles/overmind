---
name: write-ard
description: >-
  Write or update the Architecture and Debt Record: the general docs/ARD.md (global decisions and the
  debt index) and per-module ard.md. It is a dated log: entries are appended, never rewritten.
  When reverse-engineering an existing repo, records decisions visible in the code and marks the
  reasoning as inferred. Used by setup; manager scope.
argument-hint: "[general | MODULE_NAME]"
disable-model-invocation: false
---

# write-ard

Input: `general`, or a module name.
Output: `docs/ARD.md` or `docs/modules/{{module}}/ard.md`, created or appended, never rewritten.

The ARD is the only document that grows as a log.
Its value is the path: why things are the way they are, what was rejected, what debt was accepted and when to revisit it.
It is the one thing an agent cannot rediscover from code.

## Entries

Every entry follows `templates/ard-entry.md`:

```
## {{YYYY-MM-DD}}: {{decision in one line}}

- Decision: ...
- Alternatives rejected: ...
- Reason: ...
- Debt created: ... (or none)
- Revisit when: ... (a concrete trigger)
- Source: setup | {{id}}_{{title}}
```

Later entries may supersede earlier ones; then the new entry says `Supersedes: {{date}}: {{title}}` and the old one gets a one-line `Superseded by ...` note at its top.
Nothing is deleted.

## General `docs/ARD.md`

Read `templates/ARD.md`.
Two parts:

1. Decisions: architecture-level choices visible in the code: framework, layering, persistence, auth, messaging, multi-tenancy, deployment model. One entry each.
2. Debt index: a table linking every `Debt created` across all module ARDs, with module, date, trigger. Rebuild the table on every run; it is the one derived part of the file.

## Module `docs/modules/{{module}}/ard.md`

Decisions scoped to the module: data model choices, sync vs async, caching, validation strategy, external provider, anything a developer would otherwise ask "why is it like this".

## Reverse engineering an existing repo

When there is no prior ARD, you are reading decisions out of code that nobody explained.
Rules:

- Record the decision as a fact (`Decision: payments go through Stripe via a provider interface`), that is visible.
- Mark reason and alternatives `[inferido]` unless a comment, commit message, README or PR states them. Cite the source when it exists.
- Do not invent debt. Record only debt with evidence: TODO and FIXME comments, skipped tests, disabled lint rules, hardcoded values with a comment, duplicated modules.
- Prefer fewer, solid entries over many speculative ones. Ten inferred reasons that Sebastian must review are worse than three he can confirm at a glance.

`setup` collects every `[inferido]` and shows them to Sebastian for confirmation.

## Rules

- Append only; never rewrite or reorder entries.
- One decision per entry, dated.
- Frontmatter `updated` and `source` on the file.
- English.
