# Example: style check for what the linter cannot express

```markdown
---
name: style
model: sonnet
paths:
  - apps/backend/src/**/*.ts
reference: docs/conventions/style.md
---

# style

Functions are named as verbs, classes and types as nouns, booleans with `is`, `has` or `can`, as the reference states.
Helpers used by more than one module live in `src/lib/`, not in the module that first needed them.
Errors thrown to callers are classes from `src/errors/`, never bare `Error` with a string.
A file exports one public symbol; private helpers are not exported.
Comments say what the code cannot: no comment restates the line below it.
Report as clean: anything the configured ESLint rules already enforce (import order, unused variables, formatting), generated files under `src/generated/`.
```

Why it works: every statement is something ESLint does not check, so the check adds coverage instead of duplicating `verify-task`; the last line stops it from re-reporting lint.
