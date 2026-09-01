---
name: scannable-outputs
description: Sebastian wants one concise line per point and depth collapsed behind opt-in (details blocks); dense paragraphs per bullet kill readability
metadata:
  type: feedback
---

When designing any output Sebastian reads (PR summaries, boards, reports), each point is one concise line; anything longer goes behind an opt-in layer (a collapsed `<details>` block, a link, a file).
**Why:** the first pilot PR summary (auvral, 2026-08-31) was correct but too dense; he rejected the format line by line: goal-level intent, one line per bullet, bare ✅ pipeline lines, detail hidden until opened.
**How to apply:** when writing or changing templates and report formats in the method, default to scannable-first with collapsed depth; visible values should be short enums (`Low | Medium | High`) not prose. Related: [[user-tmux-navigation]].
