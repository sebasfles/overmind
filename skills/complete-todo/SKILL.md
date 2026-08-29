---
name: complete-todo
description: >-
  Strike a todo: move the matching line from Open to Done in portfolio/todos.md with today's date,
  and commit. Overmind only; Sebastian invokes it.
argument-hint: "[TEXT_OR_NUMBER]"
disable-model-invocation: true
---

# complete-todo

Input: enough of the todo's text to match one line, or its position in the Open list as printed by `check-portfolio`.
Output: the line moved to `## Done`, marked `[x]` with the completion date, committed.

```
- [x] {{added date}} {{text}} @{{project}}  (done {{YYYY-MM-DD}})
```

If the text matches more than one line, list them and ask which.
Confirm in one line: `tachado`.
Commit: `portfolio: complete todo`.

## Rules

- Never delete a todo; Done is the archive.
