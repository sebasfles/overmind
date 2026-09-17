---
name: approve-pr
description: "Approve the reviewed PR on GitHub with no comments at all. om-pr-reviewer; only when Sebastian says approve."
effort: low
argument-hint: "[PR_NUMBER]"
disable-model-invocation: true
---

# approve-pr

## Purpose

Approve the PR this session reviewed, on GitHub, with nothing attached: no body, no inline comments, none of the findings.
The report Sebastian read stays in `pr-reviews/`; GitHub only gets the approval.
om-pr-reviewer only; runs when Sebastian says approve, phrased however he likes.

Input: the PR of this session (`$ARGUMENTS` or the one `review-pr` reviewed).
Output: an approving review on the PR of the repo the diff belongs to, and one `info` event.

## 1. Preconditions

`review-pr` ran in this session or its report exists in `{{REVIEWS}}/{{repo}}-pr-{{n}}.md`; otherwise stop and say the PR was not reviewed.
`gh -R {{owner/repo}} pr view {{n}} --json headRefOid,state`: the PR is open and `headRefOid` equals the `Head` of the report.
If the head moved, stop and say so: the code Sebastian approved is not the code on the PR; `review-pr` again first.

## 2. Approve

```
gh -R {{owner/repo}} pr review {{n}} --approve
```

No `--body`.
`{{owner/repo}}` is the repo the diff belongs to; in multirepo that is the code repo's PR, never the root docs PR.

## 3. Report

One line to Sebastian: `PR #{{n}} approved`.
Then `om-events`, if it exists: `[info] {{project}}: PR #{{n}} approved`.
The worktree and the report copy stay; `clean-pr` removes the worktree after the merge.

## Rules

- No body, no comments, no findings: the approval is the whole message.
- Never approve a head that differs from the one reviewed.
- Never remove the worktree.
