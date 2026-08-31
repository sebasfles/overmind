---
name: user-tmux-navigation
description: Sebastian navigates tmux by window number (base-index 1) and expects fixed positions for key windows
metadata:
  type: user
---

Sebastian navigates tmux windows by number and his config sets `base-index 1` with `renumber-windows off`.
He expects the important window of a session at a fixed index: the project's om-manager always at window 1 (decided 2026-08-31, see the ARD entry "resume-project pins the om-manager window at index 1").
**How to apply:** when designing or changing session scripts (`resume-project`, `resume-overmind`), prefer deterministic window indices and repair logic over "append at the next free index".
