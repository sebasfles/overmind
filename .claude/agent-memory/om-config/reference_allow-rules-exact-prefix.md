---
name: allow-rules-exact-prefix
description: Since 2026-09-24 every om session runs in auto pinned with --permission-mode auto and no bypass flag; the frontmatter permissionMode never applied to --agent sessions, bypass flags trip the classifier's "Create Unsafe Agents", and mixed permission classes hold cross-session messages
metadata:
  type: reference
---

Facts the launch lines rest on, all verified in transcripts on 2026-09-24.

- An agent file's `permissionMode` does not apply to a `claude --agent` session; every om session since 2026-09-01 ran in `auto` while the frontmatter said `bypassPermissions`.
- `--allow-dangerously-skip-permissions` only adds bypass to the Shift+Tab cycle; either bypass flag on a launch is a built-in soft_deny ("Create Unsafe Agents") that the launcher's classifier judges even when `Bash(claude --bg *)` matches (seen on 2.1.281); only `autoMode.allow` prose clears it, and only in sessions started after the edit.
- The built-in starting mode depends on a feature-flag fetch: late flags start a `--bg` session in manual (0048 and bseen 0001 developers, the latter the first session on 2.1.282). `--permission-mode auto` on the line is precedence 1 and fixes it.
- Cross-session messages are delivered only between sessions of the same class (bypass vs everything else); a receiver cycled by hand into bypass holds every message from an auto sender.
- Allow rules still need the bare form: a `export`, `VAR=` or function prefix sends the whole command to the classifier.

**How to apply:** a manager reporting "Create Unsafe Agents" means a bypass flag is in its line (fix the line, no rules). Held messages between an om-reviewer and om-developer mean one of them left auto: check `permissionMode` in the transcript rows, Shift+Tab it back. Read the exact command and the modes in `~/.claude/projects/<proj>/*.jsonl` before touching settings; `serverClassifierContext` in a tool_result marks a classified call.
