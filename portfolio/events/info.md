# info

Appended by the om-events session. Unchecked lines are open.

- [ ] 2026-09-02T20:52:36Z diy 0004 cleaned
- [ ] 2026-09-02T20:52:36Z diy 0005 consolidating
- [ ] 2026-09-02T20:52:36Z diy 0006 planned
- [ ] 2026-09-02T20:52:36Z diy 0007 planned
- [ ] 2026-09-02T21:09:09Z diy 0005 consolidated and delegated
- [ ] 2026-09-02T21:27:08Z diy 0006 consolidated and delegated
- [ ] 2026-09-02T21:31:47Z diy 0005 harness defect found by om-0005-reviewer, not a blocker: the verify-task skill logs `${PIPESTATUS[0]}`, which is empty in zsh ($pipestatus, 1-indexed), so every EXIT= line in verify.log across projects proves nothing; developers have been judging steps by reading output. Suggested chore on the overmind repo: capture `$?` per command (no pipelines) and require real exit codes in the log. Sebastian's call.
