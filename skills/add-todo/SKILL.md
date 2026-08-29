---
name: add-todo
description: >-
  Quick capture: append one line to portfolio/todos.md under Open, optionally tagged with a project,
  and commit. Overmind only; Sebastian invokes it.
argument-hint: "[TEXT] [@project]"
disable-model-invocation: true
---

# add-todo

Input: free text; an optional `@{{project}}` anywhere in it tags the todo.
Output: one line appended under `## Open` in `portfolio/todos.md`, committed.

Format:

```
- [ ] {{YYYY-MM-DD}} {{text}} @{{project}}
```

Do not classify, rephrase or ask questions; capture and confirm in one line: `anotado`.
Commit: `portfolio: add todo`.

## Rules

- Never turn a todo into a task; Sebastian does that with the project's manager.
- Never reorder or edit existing todos.
