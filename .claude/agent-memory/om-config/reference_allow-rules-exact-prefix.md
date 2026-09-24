---
name: allow-rules-exact-prefix
description: The om-reviewer/om-developer launch (`claude --bg --allow-dangerously-skip-permissions`) is a built-in auto mode soft_deny "Create Unsafe Agents"; the bare `cd ws && claude --bg ...` form is necessary (a prefix sends it to the classifier) but not sufficient, only an `autoMode.allow` prose entry in ~/.claude/settings.json clears it durably
metadata:
  type: reference
---

Two layers gate a `claude --bg --allow-dangerously-skip-permissions --agent om-reviewer ...` launch from an om-manager in auto mode.

1. `permissions.allow` rules (`Bash(claude --bg *)` and friends) resolve before the classifier only when every segment of the compound command matches; a `export PATH=...;`, `VAR=` or shell function prefix sends the whole line to the classifier. `Bash(*)` is suspended in auto mode and helps nothing.
2. The classifier's default `soft_deny` "Create Unsafe Agents" blocks any agent loop started with `--dangerously-skip-permissions` / `--allow-dangerously-skip-permissions`. `permissions.allow` does not clear it; an `autoMode.allow` prose entry (with `"$defaults"`) in `~/.claude/settings.json` does, and so does Sebastian naming the exact launch in the conversation. Project `.claude/settings*.json` are not read for `autoMode`.

**Why:** 2026-09-20 auvral: prefixed launches denied, bare ones passed (layer 1). 2026-09-24 bseen: the bare form, identical to five launches that passed in my-napkin on 09-21 (2.1.278), was denied on 2.1.281 after settings.json changed the same morning; `claude auto-mode defaults --label 'Create Unsafe'` names the rule. Removing `Bash(*)` changed nothing; the `autoMode.allow` entry fixed it, but only in a session started after the edit (the same session was denied again with the entry present: rules are read at start and prior denials bias the classifier).

**How to apply:** when a manager reports "Create Unsafe Agents", read the exact command in its transcript (`~/.claude/projects/<proj>/*.jsonl`, tool_use Bash with `claude --bg`; a tool_result with `serverClassifierContext` means the classifier judged it, without it the allow rule resolved it). If it is bare: `scripts/install` says whether the `autoMode.allow` entry is present (Sebastian's act, ARD 2026-08-31 and 2026-09-24); if present, the manager predates it: `prefix + R`, never retry in the same session.
