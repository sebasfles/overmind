---
name: token-economy
description: Sept 2026 token cost drive, what was cut, what was deliberately deferred, and the triggers to revisit
metadata:
  type: project
---

Sebastian ran a token cost reduction drive on 2026-09-01 ($171 spent, fable at 92% weekly).
Two rounds of cuts landed that day: the "token economy" ARD entry (d3e9bbc) and "om-reviewer never verifies; document-task once" (91e5ae3).

**Why:** 24h usage showed review-task + verify-task + document-task together near execute-task's cost, and 67% of usage at >150k context.

**How to apply:**

- Deliberately deferred, do not re-propose without their trigger firing:
  - Recycling the om-reviewer per phase in `next-phase`: only if a reviewer session crosses ~150k on a normal task WITH the 09-01 cuts active.
  - Restoring any independent verification gate: only if a merged PR turns red on CI or a lying `verify.log` is caught after publish.
- Sebastian recycles the om-manager manually with his own recycle-session routine, every task or two; no method change needed there, and `clean-task`'s suggestion coexists with it.
- 2026-09-01 later: om-reviewer moved to opus (05257e3) on two tasks of data ($22 and $29 reviewer sessions, 80% cache writes). om-manager stays fable on purpose: cheap session, highest leverage per token.
- 2026-09-08: Sebastian put the om-reviewer back on fable (ARD entry of that date). Reason not stated by him; the entry records it as his call.
- 2026-09-17: om-reviewer back on opus again (5d1721f), also his call without stated reason; the new om-pr-reviewer is opus too. Next trigger to revisit: a quota hit with a reviewer session as main consumer, re-measure first.
- 2026-09-17: per-skill `model:` rejected for good: a skill's model switches the session for one turn and prompt cache is per model, so every switch rewrites the context twice. Model is per session; `effort` is the per-skill dial. Do not re-propose.
- Rejected the same day: om-developer reading `ard.md` as an index (saves ~8k tokens/module but blinds the implementer; the developer is the cheap session anyway). Do not re-propose.
- Sebastian's working style here: cut based on usage data, then wait 2-3 days and re-measure before cutting more.
