---
name: allow-rules-exact-prefix
description: Permission facts behind the launch lines; every om session bypassed since 2026-09-29 (--dangerously-skip-permissions), auto-era facts kept as history; mixed classes hold messages
metadata:
  type: reference
---

Since 2026-09-29 every launch line carries `--dangerously-skip-permissions`, from `resume-overmind` and `resume-project` down; the machine needs `skipDangerousModePermissionPrompt: true`.
Facts that still hold, verified in transcripts on 2026-09-24:

- An agent file's `permissionMode` does not apply to a `claude --agent` session; the flag on the launch line is the only source.
- Cross-session messages are delivered only between sessions of the same class (bypass vs everything else); a sender in another class has its messages held for Sebastian.

History, auto era (2026-09-24 to 2026-09-29):

- `--allow-dangerously-skip-permissions` only adds bypass to the Shift+Tab cycle; either bypass flag launched from an auto session is a built-in soft_deny ("Create Unsafe Agents") the classifier judges even with a matching allow rule; only `autoMode.allow` prose cleared it.
- The built-in starting mode depends on a feature-flag fetch; late flags start a `--bg` session in manual.
- Allow rules needed the bare form: an `export`, `VAR=` or function prefix sent the command to the classifier.

**How to apply:** held messages between two om sessions mean one was started without the bypass flag (by hand, or an old launch line): check `permissionMode` in the transcript rows under `~/.claude/projects/{{proj}}/*.jsonl` and relaunch it with the flag. Any classifier denial now means a session is not bypassed.
