---
name: request-pr-changes
description: "Submit the comments Sebastian selected from the review as a request-changes review on GitHub. om-pr-reviewer; only on his selection."
effort: low
argument-hint: "[PR_NUMBER] [SELECTION]"
disable-model-invocation: true
---

# request-pr-changes

## Purpose

Post the review this session wrote, filtered to what Sebastian selected, as a `request changes` review on GitHub.
The selection is his: all required comments by default, plus the optional ones he names, minus the required ones he drops.
om-pr-reviewer only; runs when Sebastian says which comments to send.

Input: the PR of this session and Sebastian's selection in the conversation (`$ARGUMENTS` or his words).
Output: a submitted `REQUEST_CHANGES` review with the selected inline comments and an overview body, and one `info` event.

## 1. Preconditions

`review-pr` ran in this session or its report exists in `{{REVIEWS}}/{{repo}}-pr-{{n}}.md` and `.json`; otherwise stop and say the PR was not reviewed.
`gh -R {{owner/repo}} pr view {{n}} --json headRefOid,state`: the PR is open and `headRefOid` equals the `Head` of the report.
If the head moved, stop and say so: the line numbers in the comments belong to a head that is gone; `review-pr` again first.

## 2. Selection

Default: every `Required before merge` comment, no `Optional` one.
Sebastian adjusts it in words: `all optional`, `optional 1 and 3`, `drop required 2`, `only the security one`.
Repeat the final selection in one line, numbered as in the overview, and take his confirmation as the go.
An empty selection is not a review: stop and say so.

## 3. Compose

From `{{REVIEWS}}/{{repo}}-pr-{{n}}.md` and `.json`:

- Body: the overview with only the selected lines under `Required before merge` and `Optional`; `Let pass`, `Summary` and `Since last review` are dropped, `Head`, `Verdict`, `Risk`, `Size` and `Intent` stay.
- Comments: the entries of the `.json` whose `path:line` match a selected line.

Write `{{WORKSPACE}}/review-request.json`:

```
jq -n --arg commit_id {{headRefOid}} --rawfile body {{WORKSPACE}}/review-body.md \
  '{commit_id: $commit_id, event: "REQUEST_CHANGES", body: $body, comments: input}' \
  {{WORKSPACE}}/review-selected.json > {{WORKSPACE}}/review-request.json
```

## 4. Submit

```
gh api repos/{{owner/repo}}/pulls/{{n}}/reviews -X POST --input {{WORKSPACE}}/review-request.json
```

The review is submitted, not pending: the decision was taken in the conversation.
`{{owner/repo}}` is the repo the diff belongs to; in multirepo that is the code repo's PR, never the root docs PR.
A `gh` failure is reported as is (`gh failed: {{first line}}`); a failed submission leaves nothing to clean up, GitHub rejects the whole review.

## 5. Report

One line to Sebastian: `PR #{{n}} changes requested, {{k}} comments`.
Then `om-events`, if it exists: `[info] {{project}}: PR #{{n}} changes requested, {{k}} comments`.
The worktree and the report copy stay: the re-review will compare against them.

## Rules

- Only the selected comments are posted; never add one Sebastian did not select, never rewrite one.
- Always `REQUEST_CHANGES`; approving is `approve-pr`, and it carries no comments.
- Never submit against a head that differs from the one reviewed.
- Never remove the worktree.
