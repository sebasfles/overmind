---
name: tmux-over-herdr
description: Decision (2026-09-09) to keep tmux as the multiplexer substrate and not adopt herdr; reasons and revisit triggers
metadata:
  type: project
---

tmux stays as the substrate of the method; herdr was evaluated on 2026-09-09 and rejected for now.

**Why:** herdr is built for a human watching N flat agents (state sidebar, desktop notification on blocked).
Our agents operate the multiplexer themselves (om-manager opens the task window and launches the om-reviewer, om-reviewer launches the om-developer, clean-task closes the window, resume-* repair fixed indices), which needs a programmable, stable substrate with per-command permission rules.
herdr is its own multiplexer (not a layer on tmux), pre-1.0 with a fast-changing API, WSL unconfirmed, and its Claude Code state detection is heuristic, risky with our `claude --bg` + `claude attach` pattern.
Sebastian's own framing: herdr fits "launch a session and let it run", it falls short for orchestration.

**How to apply:** do not propose herdr or other agent multiplexers as replacements.
The one gap herdr covers (which pane waits for input) can be solved with Claude Code hooks marking tmux windows; propose that via update-method if the pilot shows blocked agents going unnoticed.
Revisit if herdr reaches 1.0 with confirmed WSL support and a stable CLI, or if the pilot shows Sebastian missing blocked agents despite om-events.
