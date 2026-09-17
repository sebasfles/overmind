---
name: check-prs
description: "Report the project's open PRs grouped by what they wait for, plus local PR reviews to clean, and related PRs. om-manager; when Sebastian asks."
effort: medium
disable-model-invocation: false
---

# check-prs

## Purpose

The PR board of the project: every open PR in the project's repos, grouped by what it is waiting for, the local PR reviews whose PR is already merged, and the PRs that belong to the same ticket.
Derived from `gh` and from disk; no session is asked anything.
om-manager only; runs when Sebastian asks how the PRs are.

Input: none.
Output: the grouped board, one line per PR, no explanations.

`ROOT` is the project root; `REPOS` the project's code repos (the repo itself in single and mono; the list in `docs/TRD.md` in multirepo).

## 1. Collect

Once: `ME = gh api user -q .login`.

Per repo:

```
gh -R {{owner/repo}} pr list --state open --limit 100 --json number,title,body,url,author,isDraft,headRefName,headRefOid,reviews,reviewRequests
```

A `gh` failure is reported as is for that repo (`{{repo}}: gh failed: {{first line}}`, plus the active account from `gh auth status`) and the rest continues; never treat it as "no PRs".

Local: every `{{ROOT}}/.workspaces/pr-{{n}}/` folder is a PR under local review; the repo is the name of the worktree inside it.
For each one whose `n` is not in the open list of its repo: `gh -R {{owner/repo}} pr view {{n}} --json state`.
Do not list Claude sessions; the workspace is the signal, and `clean-pr` finds the session by itself.

## 2. Derive

Per open PR, from `reviews` filtered to `author.login == ME` and sorted by `submittedAt`, take the last one: `MY_STATE` (`APPROVED`, `CHANGES_REQUESTED`, `COMMENTED`, none) and `MY_HEAD` (`commit.oid`).
`RE_REQUESTED` when `reviewRequests` names `ME`.
`LOCAL` when the workspace exists.
`BOT` when `author.login` is `app/dependabot` or `dependabot[bot]`.

| Group | Condition |
|---|---|
| `cleanable` | local workspace and the PR is `MERGED` or `CLOSED` |
| `reviewed locally, not posted` | open, `LOCAL`, no review by `ME` |
| `waiting on author` | `MY_STATE = CHANGES_REQUESTED`, `headRefOid == MY_HEAD`, not `RE_REQUESTED` |
| `back for re-review` | `MY_STATE = CHANGES_REQUESTED` and (`headRefOid != MY_HEAD` or `RE_REQUESTED`) |
| `approved, not merged` | `MY_STATE = APPROVED` |
| `not reviewed` | open, no review by `ME`, not `LOCAL`, not `BOT` |
| `dependabot` | `BOT`, whatever the state; listed here and nowhere else |

Related: from title, body and `headRefName` extract ticket keys (`[A-Z]{2,}-\d+`), issue numbers (`#\d+`, `closes #n`, `fixes #n`) and issue URLs.
Two PRs sharing a key are related, across repos too.

## 3. Report

One header per non-empty group in the order of the table, then one line per PR:

```
#{{n}} {{repo}}: {{title}} ({{author}}){{ (draft)}}
```

Then `Related`, one line per key: `{{key}}: #{{a}} #{{b}}`.
Nothing else: no sizes, no explanations, no advice.
If `cleanable` is not empty, end with one question: `clean them?` (`clean-pr` each).

## Rules

- Never ask a session anything; `gh` and disk only.
- Never list Claude sessions; workspaces are the local signal.
- A `gh` failure is reported, never silently read as "no PRs".
- One line per PR, no explanations.
