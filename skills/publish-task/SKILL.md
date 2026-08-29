---
name: publish-task
description: >-
  Push the task branch, open the PR if it does not exist, and write or update its single summary
  comment (Intent, What changed, Decisions, Risk assessment, Pipeline). Notifies the manager.
  Reviewer only; runs when review-task finds no issues.
disable-model-invocation: false
---

# publish-task

Input: a clean `review-task` on `{{sha}}` in the worktree.
Output: branch pushed, PR open against the base branch, one summary comment written or updated, manager notified.

## 1. Push

```
git push -u origin {{branch}}      # first time
git push --force-with-lease         # later rounds after a reiteration, since rounds are squashed
```

Never `--force` without `--with-lease`.

## 2. PR

If `gh pr list --head {{branch}}` is empty:

```
gh pr create --base {{base}} --head {{branch}} --title "{{type}}({{modules}}): {{Title in plain words}}" --body ""
```

Title from `task.md`; for phases append ` (phase {{k}}/{{m}})`.
The PR body stays empty; the summary lives in one comment so it can be edited.

## 3. Summary comment

Read `templates/pr-comment.md` and fill it:

- Intent: one paragraph from Goal plus what was agreed in `Context & decisions`.
- What changed: at most 10 concise bullets from the diff, behavior first, files second.
- Decisions: what you decided on your own after consolidation, and what you let pass and why. This is what Sebastian reads to approve or reiterate.
- Risk assessment: Low, Medium or High, one sentence why. Medium or High must say what to watch after merge.
- Pipeline: one line per step with its result; for Review, each finding found and fixed across rounds as `file:line, defect, fix, re-checked`.

If a comment by you already exists on the PR (look for the `<!-- reviewer-summary -->` marker), edit it with `gh api` (`PATCH /repos/{owner}/{repo}/issues/comments/{id}`); never add a second one.
Otherwise `gh pr comment {{number}} --body-file`.

## 4. Notify

`SendMessage` to `{{project}}-manager`: `task {{id}}: PR #{{number}} ready (phase {{k}} of {{m}})`.
Do not message `overmind`; the manager does.
Do not write any state anywhere; `in_review` is derived from the open PR.
Then wait: for `retakes updated`, for `phase N merged, continue`, or for the manager to clean the task.

## Rules

- One comment per PR, edited in place.
- English, concise; Sebastian reads it to decide without opening the diff.
- Never merge; Sebastian merges.
