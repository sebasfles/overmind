---
name: delegate-pr
description: "Open window pr-{{n}} with an om-pr-reviewer reviewing PR n, with an optional related PR as context. om-manager; on Sebastian's ask."
effort: low
argument-hint: "[PR_NUMBER] [RELATED_PR]"
disable-model-invocation: false
---

# delegate-pr

## Purpose

Hand the review of a PR someone else wrote to an om-pr-reviewer in its own tmux window.
The om-manager reads nothing of the PR: it opens the window, launches or reopens the session, and tells Sebastian where it is.
om-manager only; runs when Sebastian asks to review a PR, phrased however he likes.

Input: `$ARGUMENTS`: the PR number or URL, and optionally a related PR that the om-pr-reviewer reads as context and does not review.
Output: session `om-pr-{{n}}-reviewer` running `review-pr` in window `pr-{{n}}`.

`PROJECT` is the tmux session you run in; `ROOT` the project root; `WINDOW = pr-{{n}}`; `SESSION = om-pr-{{n}}-reviewer`.

## 1. Session

| State | How you know | Action |
|---|---|---|
| Running | `claude agents --json` lists `{{SESSION}}`, or a live pane in `{{WINDOW}}` | `SendMessage` to `{{SESSION}}`: `re-review, head {{headRefOid}}`; it runs `review-pr` in re-review mode |
| Stopped | `claude agents --all --json` lists `{{SESSION}}` (take its `id`) | `tmux new-window -t {{PROJECT}} -n {{WINDOW}} -c {{ROOT}}`, then `claude attach {{id}}` (or `claude -r {{id}}`) in the pane, then the same message; context intact |
| None | neither | step 2 |

`{{headRefOid}}` comes from `gh -R {{owner/repo}} pr view {{n}} --json headRefOid`.
The bare `claude agents` needs a TTY and fails from Bash; always pass `--json`.

## 2. Launch

```
tmux new-window -t {{PROJECT}} -n {{WINDOW}} -c {{ROOT}}
tmux send-keys -t {{PROJECT}}:{{WINDOW}} "claude --permission-mode auto --agent om-pr-reviewer -n {{SESSION}} '/review-pr {{n}} {{m}}'" Enter
```

`{{m}}` only when Sebastian gave a related PR.
Run both lines exactly as written, one Bash call each, nothing before `tmux` (no `export`, `VAR=` or shell function), or the command matches no allow rule and goes to the classifier.
`--permission-mode auto` pins the session's mode; never add a bypass flag.
Interactive form on purpose: Sebastian talks to this session in its window, and the om-manager never messages it after this.

## 3. Report

One line: `PR #{{n}} delegated; window {{WINDOW}}` (add `related #{{m}}` when given).
Then nothing: the om-pr-reviewer reports to Sebastian and to `om-events`, never to you.

## Rules

- Never read the PR, its diff or its description; you only open the window.
- Never message the om-pr-reviewer after the launch, except `re-review` when Sebastian asks again for the same PR.
- One window and one session per PR; a second ask for the same PR reuses them.
- Absolute paths in every command.
