---
name: publish-task
description: >-
  Push every branch of the task workspace, open one PR per repo touched (plus the root docs repo in
  multirepo projects), and write or update the single summary comment on the root repo's PR (Intent,
  What changed with links to each code PR, Decisions, Risk assessment, Pipeline per target). Notifies
  the manager. Reviewer only; runs when review-task finds no issues.
disable-model-invocation: false
---

# publish-task

Input: a clean `review-task` on the workspace's current commits.
Output: all branches pushed, one PR per repo, one summary comment on the root PR, manager notified.

`ROOT_WT` as in `delegate-task`.
In single and mono the root PR is the code PR; there is only one.

## 1. Push

For each worktree in the workspace with commits ahead of its base:

```
git -C {{WORKSPACE}}/{{name}} push -u origin {{branch}}          # first time
git -C {{WORKSPACE}}/{{name}} push --force-with-lease            # later rounds; rounds are squashed
```

Never `--force` without `--with-lease`.

## 2. PRs

Code repos first, root last (so the root summary can link them).
For each repo with a pushed branch, if `gh pr list --head {{branch}}` in that repo is empty:

```
gh -R {{owner/repo}} pr create --base {{base}} --head {{branch}} \
  --title "{{type}}({{modules}}): {{Title in plain words}}{{ (phase k/m)}}" \
  --body "{{body}}"
```

Body: empty for the root PR (the summary lives in an editable comment); for code PRs in multirepo, one line: `Task PR: {{root PR url}}` (create the root PR first if needed, then the code PRs, then edit nothing else).
Simplest order that satisfies this: create the root PR with empty body, then code PRs with the link, then the summary comment.

## 3. Summary comment, on the root PR

Read `templates/pr-comment.md` and fill it:

- Intent: one paragraph from Goal plus what was agreed in `Context & decisions`.
- What changed: at most 10 concise bullets; in multirepo, group by repo and link each code PR.
- Decisions: what you decided on your own after consolidation, what you let pass and why, and the merge order between repos if any (for example `merge diy-infra first`).
- Risk assessment: Low, Medium or High, one sentence; Medium or High say what to watch after merge.
- Pipeline: one block per verification target, plus the review findings across rounds as `file:line, defect, fix, re-checked`.

If a comment with the `<!-- reviewer-summary -->` marker exists, edit it (`gh api PATCH /repos/{owner}/{repo}/issues/comments/{id}`); never add a second one.
Otherwise `gh pr comment {{number}} --body-file`.

## 4. Notify

`SendMessage` to `{{project}}-manager`: `task {{id}}: PRs ready, summary at {{root PR url}} (phase {{k}} of {{m}})`.
Do not message `overmind`; the manager does.
Do not write any state; `in_review` is derived from the open PRs.
Then wait: for `retakes updated`, `phase N merged, continue`, or cleanup.

## Rules

- One summary comment, on the root PR, edited in place.
- English, concise; Sebastian decides from it without opening the diffs.
- Never merge; Sebastian merges, code PRs in the order given, the root PR last.
