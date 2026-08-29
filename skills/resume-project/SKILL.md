---
name: resume-project
description: Open a registered project in a new terminal tab with its om-manager. overmind; Sebastian invokes it.
effort: low
argument-hint: "[NAME]"
disable-model-invocation: true
---

# resume-project

## Purpose

Open a registered project in a new Windows Terminal tab, inside WSL, at the repo root, attached to its tmux
session, with its om-manager running in the first window. Thin wrapper over scripts/resume-project. overmind only.

Input: a registered project name.
Output: a new terminal tab attached to the project's tmux session, om-manager in window 0.

Run:

```
scripts/resume-project {{name}}
```

The script reads `portfolio/projects.yaml`, creates the tmux session with a `om-manager` window running `claude --agent om-manager -n om-{{name}}-manager` if it does not exist, and opens the tab with `wt.exe -w 0 new-tab ... tmux attach -t {{name}}`.

Report one line: `{{name}} opened` or the script's error verbatim.

## Rules

- If the name is not registered, say so and offer `add-project`.
- Never start the om-manager yourself; the script does it, once.
