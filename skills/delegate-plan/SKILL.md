---
name: delegate-plan
description: "Open window plan-{{title}} with an om-architect that plans one idea with Sebastian and hands the draft back. om-manager; on Sebastian's ask."
effort: low
argument-hint: "[DESCRIPTION]"
disable-model-invocation: false
---

# delegate-plan

## Purpose

Hand the planning of one idea to an om-architect in its own tmux window, so a long or exploratory plan does not fill the om-manager's context.
The om-manager opens the window and says where it is; Sebastian plans there, and the om-architect sends back `draft ready: {{path}}`, which `create-task` consumes.
om-manager only; runs when Sebastian asks to plan something in its own window, phrased however he likes.

Input: `$ARGUMENTS`, Sebastian's description of the idea (free text or a ticket).
Output: session `om-{{title}}-architect` running `plan-task` in window `plan-{{title}}`.

`PROJECT` is the tmux session you run in; `ROOT` the project root (your cwd); `TITLE` a snake_case name of two to four words for the idea, the same the draft will carry; `WINDOW = plan-{{TITLE}}`; `SESSION = om-{{TITLE}}-architect`.

## 1. Session

| State | How you know | Action |
|---|---|---|
| Running | `claude agents --json` lists `{{SESSION}}`, or a live pane in `{{WINDOW}}` | tell Sebastian the window is already open; nothing else |
| Stopped | `claude agents --all --json` lists `{{SESSION}}` (take its `id`) | `tmux new-window -t {{PROJECT}} -n {{WINDOW}} -c {{ROOT}}`, then `claude attach {{id}}` (or `claude -r {{id}}`) in the pane; context intact |
| None | neither | step 2 |

The bare `claude agents` needs a TTY and fails from Bash; always pass `--json`.

## 2. Launch

```
tmux new-window -t {{PROJECT}} -n {{WINDOW}} -c {{ROOT}}
tmux send-keys -t {{PROJECT}}:{{WINDOW}} "claude --dangerously-skip-permissions --agent om-architect -n {{SESSION}} '/plan-task {{description}}'" Enter
```

Run both lines exactly as written, one Bash call each, nothing before `tmux` (no `export`, `VAR=` or shell function).
`--dangerously-skip-permissions` stays in the line: every session of the method runs bypassed, and a session in another permission class does not receive its messages.
Quote the description so the shell passes it whole; a long one is cut to its first sentence and the om-architect asks for the rest.
Interactive form on purpose: Sebastian talks to this session in its window.

## 3. Report

One line: `planning {{TITLE}} in window {{WINDOW}}`.
Then nothing until `draft ready: {{path}}` arrives; on it, tell Sebastian in one line and offer `create-task` with that draft.

## Rules

- Never plan the idea yourself once it is delegated; the draft is the om-architect's until it hands it over.
- One window and one session per idea; a second ask for the same idea reuses them.
- Never create the task on the om-architect's pointer alone; `create-task` runs on Sebastian's yes, as always.
- Absolute paths in every command.
