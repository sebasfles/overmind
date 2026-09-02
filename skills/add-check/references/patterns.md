# Example: patterns check for a module's architecture

```markdown
---
name: billing-patterns
model: sonnet
paths:
  - apps/backend/src/billing/**/*.ts
reference: docs/modules/billing/trd.md
---

# billing-patterns

Controllers only validate input and call one service method; no business logic and no repository calls in controllers.
Services call repositories, never Prisma directly; the repository is the only file that imports `PrismaService`.
Every endpoint has one DTO class for its input under `dto/`, none reuses an entity type as input.
Money is handled with the `Money` value object from `src/lib/money.ts`, never as a number.
State transitions of an invoice go through `InvoiceStateMachine`; no direct assignment to `status`.
Report as clean: test files, migration files, and the mappers under `mappers/` which are allowed to touch Prisma types.
```

Why it works: it points at the module's own `trd.md`, so the rule and the check evolve together; the checker reads only the module's folder, which keeps it fast and sharp.
