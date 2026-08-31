---
updated: {{YYYY-MM-DD}}
source: setup
---

# Technical Requirements Document

Describes the whole application. One section per repo or app under "Components".

## Components

| Component | Kind | Path | Base branch | Stack |
|---|---|---|---|---|
| {{diy-platform}} | repo | `{{diy-platform/}}` | `{{develop}}` | {{NestJS, Prisma}} |
| {{backend}} | app | `{{./apps/backend}}` | (same repo) | {{...}} |

### {{Component}}

- Stack: {{language, runtime, frameworks, package manager}}
- Layout: {{top-level folders, one line each}}
- Install: `{{pnpm install --frozen-lockfile}}`
- Workspace files: {{`.env`, `.env.test`, other untracked files a fresh worktree needs}}
- API spec: {{how generated, where it lands, or none}}
- Data: {{engine, ORM or migrations, migrate command}}
- Delivery: {{environments, how deploys happen, IaC location}}

## Verification targets

`verify-task` reads this table literally. One row per component. Commands run one at a time, serial flags included.

| Target | Path | lint | typecheck | unit | e2e |
|---|---|---|---|---|---|
| {{diy-platform}} | `{{diy-platform/}}` | `{{cmd}}` | `{{cmd or n/a}}` | `{{cmd}} --runInBand` | `{{cmd or n/a}}` |

## Modules

Modules belong to the application and may span components.

| Module | Purpose | Components | Docs |
|---|---|---|---|
| {{billing}} | {{one line}} | {{diy-platform, web}} | [README](modules/{{billing}}/README.md) |

## Conventions

- {{pattern}}: {{one sentence}}. Example: `{{path}}`.
