---
name: tmux-over-herdr
description: Decision (2026-09-09, reaffirmed 2026-10-19) to keep tmux as the session host under an om-session abstraction; herdr, t3code and a custom host rejected; chat UI comes as a client
metadata:
  type: project
---

tmux stays as the host of the method; herdr (2026-09-09) and t3code plus a custom desktop host (2026-10-19) were evaluated and rejected.

**Why:** the agents operate the multiplexer themselves (om-manager opens the task window and launches the om-reviewer, which launches the om-developer; clean-task closes the window; resume-* repair fixed indices), so the host must be programmable and stable.
herdr, t3code and Claude Desktop are all hosts that want to own the processes; two hosts cannot share a session, and Desktop does not see CLI-started sessions.
Both tools model one flat chat per agent; our unit is a crew of sessions messaging each other.
A custom host would reimplement permission dialogs, AskUserQuestion, plans and skills over the Agent SDK, which Anthropic already maintains in Desktop.

**How to apply:** do not propose replacing tmux; route every host need through `om-session` (see [[overmind-ui-direction]]) so the backend can change later without touching skills or the UI.
The gap herdr covered (which pane waits for input) is solved with Claude Code hooks; that trigger fired on 2026-10-19.
Revisit the host only if a Desktop or CLI feature lets an agent launch local sessions that Desktop can show.
