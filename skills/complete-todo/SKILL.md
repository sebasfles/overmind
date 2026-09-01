---
name: complete-todo
description: Move a todo from portfolio/todos.md to portfolio/todos-done.md. overmind; Sebastian invokes it.
effort: low
argument-hint: "[TEXT_OR_NUMBER]"
disable-model-invocation: true
---

# complete-todo

## Purpose

Strike a todo: move the matching line from portfolio/todos.md to portfolio/todos-done.md with today's date, and commit.
overmind only; Sebastian invokes it.

Input: enough of the todo's text to match one line, or its position in the list as printed by `check-portfolio`.
Output: the line moved to `portfolio/todos-done.md` with the completion date, committed.

```
- {{added date}} {{text}} @{{project}}  (done {{YYYY-MM-DD}})
```

If the text matches more than one line, list them and ask which.
Confirm in one line: `done`.
Commit: `portfolio: complete todo`.

## Rules

- Never delete a todo; `todos-done.md` is the archive.
