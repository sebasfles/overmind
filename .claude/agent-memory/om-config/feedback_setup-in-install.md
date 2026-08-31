---
name: setup-in-install
description: Machine setup steps go into scripts/install idempotently, never into a manual doc like install.md
metadata:
  type: feedback
---

When a method change needs per-machine setup (tmux bindings, symlinks), implement it as an idempotent block in `scripts/install`, not as a document with manual steps.
**Why:** Sebastian proposed an `install.md` on 2026-08-31; we agreed a hand-followed doc drifts and gets forgotten, while `scripts/install` is already the once-per-machine entry point. He accepted this immediately.
**How to apply:** any future "add this to your shell/tmux/git config" lands as a guarded append in `scripts/install`, plus one mention in `docs/usage-guide.md` Install. Related: [[user-tmux-navigation]].
