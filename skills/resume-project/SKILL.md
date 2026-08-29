---
name: resume-project
description: Open a registered project in a new terminal tab with its manager. Overmind; Sebastian invokes it.
argument-hint: "[NAME]"
disable-model-invocation: true
---

# resume-project

## Purpose

Open a registered project in a new Windows Terminal tab, inside WSL, at the repo root, attached to its tmux
session, with its manager running in the first window. Thin wrapper over bin/resume-project. Overmind only.

Input: a registered project name.
Output: a new terminal tab attached to the project's tmux session, manager in window 0.

Run:

```
bin/resume-project {{name}}
```

The script reads `portfolio/projects.yaml`, creates the tmux session with a `manager` window running `claude --agent manager -n {{name}}-manager` if it does not exist, and opens the tab with `wt.exe -w 0 new-tab ... tmux attach -t {{name}}`.

Report one line: `{{name}} abierto` or the script's error verbatim.

## Rules

- If the name is not registered, say so and offer `add-project`.
- Never start the manager yourself; the script does it, once.
