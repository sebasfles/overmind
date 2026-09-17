---
name: publish-task
description: Push, open one PR per repo and write the summary as the root PR's description. om-reviewer; when review-task finds no issues.
effort: high
disable-model-invocation: false
---

# publish-task

## Purpose

Push every branch of the task workspace, open one PR per repo touched (plus the root docs repo in multirepo
projects), and write or update the summary as the root repo PR's description (Intent, What changed with
links to each code PR, Decisions, Risk assessment, Pipeline per target and per check). Notifies the om-manager. om-reviewer only;
runs when review-task finds no issues.

Input: a clean `review-task` plus the om-developer's `docs ready, commit {{sha}}`.
Output: all branches pushed, one PR per repo, the summary as the root PR's description, om-manager notified.

`ROOT_WT` as in `delegate-task`.
In single and mono the root PR is the code PR; there is only one.

## 1. Docs

The om-developer's last commit is the documentation commit; check it before pushing anything.
For each module in `modules`, plus any module the diff touched:

- `docs/modules/{{module}}/*.md` affected have `updated` today and `source: {{id}}_{{title}}`, or `om-developer notes` justifies in one line why nothing changed.
- The debt index of `docs/ARD.md` matches the module ARDs: a row for every new `Debt created`, none for an entry carrying `Resolved by`.
- `trd.md` reflects new or changed endpoints; `database.md` reflects new tables, columns or invariants; `flows.md` if a complex flow changed.
- `ard.md` has an entry for every decision `om-developer notes` records that `Approach` and `Context & decisions` did not.

Missing or stale docs: `SendMessage` the om-developer `docs findings: {{k}} ...`, stop, and rerun this step on the next `docs ready, commit {{sha}}`.

## 2. Push

For each worktree in the workspace with commits ahead of its base:

```
git -C {{WORKSPACE}}/{{name}} push -u origin {{branch}}          # first time
git -C {{WORKSPACE}}/{{name}} push --force-with-lease            # later rounds; rounds are squashed
```

Never `--force` without `--with-lease`.

## 3. PRs

Code repos first, root last (so the root summary can link them).
For each repo with a pushed branch, if `gh pr list --head {{branch}}` in that repo is empty:

```
gh -R {{owner/repo}} pr create --base {{base}} --head {{branch}} \
  --title "{{type}}({{modules}}): {{Title in plain words}}{{ (phase k/m)}}" \
  --body "{{body}}"
```

Body: for the root PR, a one-line placeholder (`Summary follows.`), replaced by the summary in step 4 once the code PR links exist; for code PRs in multirepo, one line: `Task PR: {{root PR url}}`.
Simplest order that satisfies this: create the root PR with the placeholder body, then code PRs with the link, then rewrite the root PR's description.

## 4. Summary, as the root PR's description

Read `templates/pr-summary.md` and fill it:

- Intent: two or three sentences at goal level, no implementation detail; name a consolidation agreement only if it changes how to read the PR.
- What changed: at most 10 bullets, one concise line each, no subclauses; in multirepo, group by repo and link each code PR.
- Decisions: one line per decision or let-pass with its reason; include the merge order between repos if any (for example `merge diy-infra first`).
- Risk assessment: exactly `✅ Low`, `⚠️ Medium` or `🔴 High`, judged with the rubric of `review-pr` step 5: `✅ Low` unless the base branch is in production and a defect would reach real users; Medium and High name the failure and what to watch after merge.
- Pipeline: one bare `✅` line per step, never GitHub task checkboxes, no inline extra info; the review line reads `{{k}} issues auto-fixed`; the checks line lists the check names that ran with their total findings fixed, or `n/a`; `documentation` and `push` are bare passed lines.
- Everything longer (commands, targets, findings as `file:line, defect, fix, re-checked`, shas, modules) goes only inside the collapsed `<details>` block; Sebastian opens it when he wants depth.

Write it as the PR's description: `gh -R {{owner/repo}} pr edit {{number}} --body-file {{file}}`.
On later rounds rewrite the whole description the same way; it is the single source, never add summary comments.

## 5. Notify

`SendMessage` to `om-{{project}}-manager`: `task {{id}}: PRs ready, summary at {{root PR url}} (phase {{k}} of {{m}})`.
Do not message `overmind`; the om-manager does.
Do not write any state; `in_review` is derived from the open PRs.
Then wait: for `retakes updated`, `phase N merged, continue`, or cleanup.

## Rules

- The summary is the root PR's description, rewritten in place; no summary comments.
- English, concise; Sebastian decides from it without opening the diffs.
- Never merge; Sebastian merges, code PRs in the order given, the root PR last.
