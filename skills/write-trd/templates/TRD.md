---
updated: {{YYYY-MM-DD}}
source: setup
---

# Technical Requirements Document

## Stack

- Language and runtime: {{...}}
- Frameworks: {{...}}
- Package manager: {{...}}

## Layout

| Path | What lives here |
|---|---|
| `{{path}}` | {{...}} |

## Modules

| Module | Purpose | Docs |
|---|---|---|
| {{name}} | {{one line}} | [README](modules/{{name}}/README.md) |

## Verification

Run one at a time, in this order. `verify-task` reads this section literally.

| Step | Command | Notes |
|---|---|---|
| lint | `{{cmd}}` | |
| typecheck | `{{cmd or n/a}}` | |
| unit | `{{cmd}} --runInBand` | |
| e2e | `{{cmd or n/a}} --runInBand` | |

## API spec

{{How it is generated and where it lands, or "none".}}

## Data

- Engine: {{...}}
- ORM or migrations: {{...}}
- Run migrations: `{{cmd}}`

## Environments and delivery

- Base branch: `{{develop}}`
- Environments: {{local, dev, staging, prod}}
- Deploy: {{how}}
- IaC: {{path or none}}

## Conventions

- {{pattern}}: {{one sentence}}. Example: `{{path}}`.
