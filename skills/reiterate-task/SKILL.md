---
name: reiterate-task
description: Send a published task back for another round with Sebastian's PR comments. om-manager; Sebastian invokes it.
effort: medium
argument-hint: "[TASK_ID_OR_FOLDER] [COMMENTS]"
disable-model-invocation: true
---

# reiterate-task

## Purpose

Send a published task back for another round with Sebastian's PR comments: record them dated in retakes.md of
the worktree copy, make sure om-reviewer and om-developer are running, and tell the om-reviewer "retakes updated".
om-manager only; Sebastian invokes it.

Input: a task id or folder, and Sebastian's comments (inline, or "read the PR" to pull review comments with `gh`).
Output: `retakes.md` updated in the worktree copy, om-reviewer notified.

## 1. Preconditions

`check-task` must say `in_review`.
If the PR is merged, this is not a reiteration but a new task; say so.

## 2. Collect the comments

Inline text from Sebastian, plus if asked: `gh pr view {{number}} --comments` and `gh api repos/{owner}/{repo}/pulls/{{number}}/comments` for inline review comments.
Do not interpret or filter them; the om-reviewer does.

## 3. Record

Append to `{{WORKTREE}}/docs/tasks/{{id}}_{{title}}/retakes.md` (create it if missing):

```
## Retake {{n}}: {{ISO date}}

Source: PR #{{number}}, {{inline | review comments}}

- {{comment, verbatim or lightly trimmed}}
- ...
```

Worktree copy only; the task is delegated.

## 4. Sessions

om-reviewer running: continue.
om-reviewer stopped: reopen its window and session (`claude attach` / `claude -r`) as `delegate-task` does.
om-developer missing: nothing to do; the om-reviewer relaunches it with `start-task` if needed.

## 5. Notify

`SendMessage` to `om-{{id}}-reviewer`: `retakes updated: retake {{n}}, PR #{{number}}`.
The om-reviewer runs `analyze-task` in reiteration mode, turns the retakes into findings, and the round loop continues.

## 6. Report

One line to Sebastian: `{{id}}_{{title}}: retake {{n}} sent to om-reviewer`.

## Rules

- Never rewrite or delete earlier retakes.
- Never talk to the om-developer.
- Never decide on Sebastian's comments; the om-reviewer does and documents it in the PR comment.
