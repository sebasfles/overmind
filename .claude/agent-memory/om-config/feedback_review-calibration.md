---
name: review-calibration
description: How Sebastian wants reviews and risk assessments calibrated, strict on defects, few optional comments, risk Low unless production users exist
metadata:
  type: feedback
---

Reviews must be strict where a defect costs and permissive everywhere else; risk assessments default to Low and only rise when real users can be hit.

**Why:** on 2026-09-14 Sebastian found the pilot's PR summaries too soft (Medium had become the default without asking whether the project was even in production) and, designing `review-pr`, insisted he does not want a wall of comments blocking a colleague: move fast but with care.
He also wants the author to see clearly which comments are required and which are optional, and no redundancy with what the PR already shows (CI, linter, checks).

**How to apply:** when a skill or agent produces findings or a risk value, check it against the rubric in `review-pr` step 5 and the few-comments rule in step 4; never add a step that re-runs or restates something GitHub already displays.
Related: [[token-economy]].
