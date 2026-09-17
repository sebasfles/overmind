---
name: check-skills-disk-only
description: Board skills (check-*) derive from disk and gh, never from `claude agents --all --json`; Sebastian rejected session listings there as inefficient
metadata:
  type: feedback
---

`check-*` skills never list Claude sessions; the workspace folder (`.workspaces/{{x}}`) is the local signal, and only the `clean-*` skill that removes it may run `claude agents --all --json` to find the session by cwd or name.

**Why:** on 2026-09-17, designing `check-prs`, I proposed listing sessions to detect open PR reviews; Sebastian objected that `claude agents --all --json` returns every session of the machine, stopped ones included, and is wasteful for a board. Same day he cut a per-PR size line from the board output: boards carry only what drives an action.

**How to apply:** when a new board or state-derivation skill needs "is there a session for X", answer it with "is there a workspace for X" and let the cleanup skill deal with sessions. Keep board lines minimal: id, repo, title, author; no metrics unless they change what Sebastian does next.
