---
name: commit-own-memory
description: Commit om-config agent memory as part of the method flow, without asking each time
metadata:
  type: feedback
---

Commit `.claude/agent-memory/om-config/` yourself, as `chore: om-config agent memory`, when you finish a change of method; do not leave it dirty and do not ask first.

**Why:** Sebastian approved this on 2026-09-02 after asking what the pending files were and whether they were worth committing. Two reasons hold: the repo already had three such commits, and a dirty `.claude/` shows up in the `git status` the `overmind` session sees on every `portfolio/` commit. The content earns its place because it is what the ARD does not hold: verified dead ends, rejected ideas and the reasoning behind them.

**How to apply:** at the end of `update-method`, after the method commit, in its own commit so the method history stays readable. Keep it to memory that a recycled om-config would otherwise re-derive or re-propose; anything that belongs in `docs/method-ard.md` goes there instead. See [[token-economy]] for the kind of content that qualifies.
