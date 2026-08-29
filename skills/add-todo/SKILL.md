---
name: add-todo
description: Append a quick-capture todo to portfolio/todos.md. overmind; Sebastian invokes it.
effort: low
argument-hint: "[TEXT] [@project]"
disable-model-invocation: true
---

# add-todo

## Purpose

Quick capture: append one line to portfolio/todos.md under Open, optionally tagged with a project, and commit.
overmind only; Sebastian invokes it.

Input: free text; an optional `@{{project}}` anywhere in it tags the todo.
Output: one line appended under `## Open` in `portfolio/todos.md`, committed.

Format:

```
- [ ] {{YYYY-MM-DD}} {{text}} @{{project}}
```

Do not classify, rephrase or ask questions; capture and confirm in one line: `noted`.
Commit: `portfolio: add todo`.

## Rules

- Never turn a todo into a task; Sebastian does that with the project's om-manager.
- Never reorder or edit existing todos.
