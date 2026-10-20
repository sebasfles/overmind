---
name: overmind-ui-direction
description: Sebastian's goal (2026-10-19) of a UI platform for the whole workflow, with the agreed order hooks -> om-session -> fleet -> UI and the rule that the UI is a client of om-session, never a host
metadata:
  type: project
---

Sebastian wants to work from a UI, not the terminal: a tab per project that glows when a thread of that project needs him, threads pinned inside the project (om-manager always fixed, chats do not reorder), an overmind tab to add and open projects and assign each project a host (local or an EC2 instance logged in with his same Claude account).

**Why:** in tmux he cannot tell which session finished or waits for him, and his machine suffers with many parallel tasks; a fleet of hosts with the same account doubles throughput.

**How to apply:** the agreed order is (1) attention hooks (Notification permission_prompt/idle_prompt/agent_needs_input, Stop, UserPromptSubmit) marking tmux windows and ringing BEL for the Windows Terminal tab, (2) `om-session start|send|stop|list|focus|wait` hiding every raw tmux verb from skills, with lint forbidding raw tmux outside it, (3) `host:` per project in projects.yaml, resume-project over ssh, Remote Control on overmind, om-events and remote om-managers for cross-machine SendMessage, (4) the UI as a separate project, a client of om-session and of disk (transcripts jsonl, projects.yaml, docs/tasks, hook marks), never owning processes.
Steps 1 to 3 are om-config work via update-method; step 4 is registered from overmind with add-project.
When Sebastian pushes for the UI, map each feature to the step that produces its data instead of resisting; he has decided on the UI.
