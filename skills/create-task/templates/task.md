---
id: "{{id}}"
title: {{title}}
type: {{feature | bug | docs | chore | refactor}}
branch: {{prefix}}/{{id}}_{{title}}
modules: [{{primary}}, {{other}}]
phases: 0
depends_on: []
ticket:
created: {{YYYY-MM-DD}}
updated: {{YYYY-MM-DD}}
---

# {{id}} {{Title in plain words}}

## Goal

{{One paragraph: what and why, from the user's perspective.}}

## Scope

- {{what is in}}

## Out of scope

- {{what is explicitly not in}}
- Deferred: {{adjacent ideas noted but not done}}

## Acceptance

1. {{observable, testable criterion}}
2. {{...}}

## Approach

- Module and layers touched: {{...}}
- Entities, endpoints, tables: {{...}}
- Decisions:
  - {{decision}}: chosen over {{alternative}} because {{reason}}.

## Database

{{Tables owned or referenced, migrations expected, invariants kept in code. Or "None".}}

## Infra

{{Resources, env vars, secrets, IAM, CI/CD changes, per environment. Or "None".}}

## Design

{{Figma links, or "None".}}

## Replication

{{Bugs only: see replication.md. Otherwise remove this section.}}

## Risks

- {{what could go wrong or is uncertain}}

## Depends on

{{ids, or "None".}}

## Context & decisions

{{Owned by the reviewer. Written during analyze-task, before the developer starts.}}

## Developer notes

{{Owned by the developer. What was done, what was left pending, per round.}}
