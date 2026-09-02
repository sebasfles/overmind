---
name: cache-ttl-behavior
description: Why task sessions sometimes show 5m cache TTL and pay re-caches; it is usage credits, not --bg or a setting
metadata:
  type: reference
---

Claude Code gives the main conversation (including `--bg` sessions launched by the method) a 1h prompt cache TTL while inside the subscription plan, and drops it to 5m once the 5-hour window is exhausted and usage credits are being billed.
Source: code.claude.com/docs/en/prompt-caching ("Cache lifetime") and code.claude.com/docs/en/costs ("Why usage climbs in a long session").

**Why it matters:** the om-reviewer idles more than 5 min per om-developer round (wall 43m vs API 15m on 2026-09-01), so on 5m TTL each round is a full ~150k re-cache; that was 55% of a $22 reviewer session, paid with real money because it happened on credits.

**How to apply:**

- Do not propose install checks or settings for this; nothing is misconfigured. Verified 2026-09-01 after the hypothesis that `--bg` sessions fell in the subagent bucket turned out wrong.
- `promptCacheTtl: "1h"` in `~/.claude/settings.json` only matters while on usage credits (keeps 1h at 2x write price). Sebastian's personal choice, not a method change.
- Cache write 5m = 1.25x base input, 1h = 2x; reads 0.1x (fable 0.025x, $0.25/M) on both. Fable base $10/$50, opus $5/$25 per M in/out.
- `/usage` reading rule: wall much greater than API plus misses means the session paid re-caches for waiting.
