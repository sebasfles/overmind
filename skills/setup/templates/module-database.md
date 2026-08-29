---
updated: {{YYYY-MM-DD}}
source: setup
---

# {{Module}}: database

Columns and types live in the schema ({{path to Prisma schema, migrations or models}}); this file says what the module owns and what it must keep true.

## Tables owned

| Table | Purpose |
|---|---|
| `{{table}}` | {{one line}} |

## Tables referenced

| Table | Owner module | Relationship |
|---|---|---|
| `{{table}}` | {{module}} | {{one line}} |

## Invariants kept in code

- {{rule the database does not enforce but the code relies on}}

## Migrations of note

- {{date}}: {{what changed and why}} [inferido if unknown]
