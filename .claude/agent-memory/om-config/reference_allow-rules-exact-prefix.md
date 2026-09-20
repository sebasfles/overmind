---
name: allow-rules-exact-prefix
description: Auto mode allow rules only skip the classifier when every segment of a compound Bash command matches a rule; prefixes like export PATH=, VAR= or shell functions send the whole launch to the classifier, which blocks --allow-dangerously-skip-permissions as "Create Unsafe Agents"
metadata:
  type: reference
---

A `claude --bg --allow-dangerously-skip-permissions --agent om-reviewer ...` launch passes only when the Bash command is the bare skill form: `cd /abs/ws && claude --bg ...`, nothing else in the command.

**Why:** 2026-09-20, auvral's om-manager prefixed the launch with `export PATH=$HOME/.nvm/...; R=...; WS=...` and once wrapped it in a `launch(){}` function; no allow rule matches those segments, so the classifier evaluated the full command and denied it as "Create Unsafe Agents". my-napkin's manager used the literal form and never hit the classifier. Follow-up commands (even a `grep` on settings) get the same tag because the classifier judges in context.

**How to apply:** when a manager reports the classifier blocking a launch, read the exact command text in its transcript before suspecting settings; the fix is the command form, not more rules. Per-project `settings.local.json` copies of the global rules do not help.
