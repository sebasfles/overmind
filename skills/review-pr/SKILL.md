---
name: review-pr
description: "Read-only code review of a PR someone else wrote: intent and code, nothing run. Reports to Sebastian; posts a pending GitHub review on his yes. Sebastian invokes it."
effort: xhigh
argument-hint: "[PR_NUMBER_OR_URL]"
disable-model-invocation: true
---

# review-pr

## Purpose

Review a pull request that neither Sebastian nor his agents wrote: a colleague's PR in any project, with or without the overmind documentation convention.
Reads the code with the same bar `review-task` applies to the om-developer's rounds, adapted to a PR that has no task folder: the PR description is the intent.
It runs nothing: CI, lint, tests and rules already show their result on the PR, and repeating them there is noise.
Ends with a short overview for Sebastian and, only on his yes, a pending GitHub review: the overview as its body and one inline comment per finding, where the explanation lives.

Where: a fresh Claude session in the project root, never inside a long-lived om-manager or om-reviewer session; a full review fills a context.
Input: `$ARGUMENTS`, a PR number or URL.
Output: `review.md` in the worktree and a copy in `{{ROOT}}/pr-reviews/`, the same content in the conversation, and on Sebastian's yes a pending review on the PR of the repo it belongs to.

`ROOT` is the project root (the repo, or the docs repo in multirepo).
`WORKSPACE = {{ROOT}}/.workspaces/pr-{{n}}`; the PR is read there, in a worktree of its own branch, never in the root checkout.
`REVIEWS = {{ROOT}}/pr-reviews/`, local and ignored: if `git -C {{ROOT}} check-ignore -q pr-reviews` fails, append `pr-reviews/` to `{{ROOT}}/.git/info/exclude`; it is never committed.
You never modify the PR's code, never push to its branch, and never run lint, typecheck, tests, checks or the application.
Stop only when a step cannot run (no `gh` access, no clone) and say why.

## 1. Fetch

```
gh -R {{owner/repo}} pr view {{n}} --json number,title,body,author,url,baseRefName,headRefName,headRefOid,additions,deletions,changedFiles,isDraft,reviews,comments
```

A `gh` failure is reported as is (`gh failed: {{first line}}`, plus the active account from `gh auth status`); never continue as if the PR had no comments.
Read the description and every linked issue or ticket it names; that is the intent you review against.
Read the existing review threads: what someone else already raised on the PR and the author already answered is not repeated, only referenced when it is still open and matters.
Ignore CI and check results: the PR already shows them, and whoever reads it already knows.

## 2. Worktree

```
git -C {{clone}} fetch origin {{base}} pull/{{n}}/head:pr-{{n}}
git -C {{clone}} worktree add {{WORKSPACE}}/{{repo}} pr-{{n}}
```

`{{clone}}` is the repo itself in single and mono, and the code clone the PR URL names in multirepo (`docs/05-layouts.md`).
The root checkout stays on its base branch; you never check the PR out there.
The worktree exists so you can read every changed file in full, with the code around it; nothing gets installed or executed in it.

## 3. Intent

Read `git diff origin/{{base}}...HEAD --stat`, then every changed file in full, not only the hunks.
Judge the diff against the description and the linked issue:

- Everything the description promises is in the diff.
- Nothing in the diff is outside what the description promises: unrelated refactors, renames, formatting sweeps, dependency bumps.
- The description tells the next reader what to look at; a PR whose description is empty or says less than its diff is a finding, because the next reader will have the same problem.

A scope deviation is a finding even when the extra work is good; the ask is to split it, not to drop it.
A PR above roughly 800 changed lines that could have been split gets one `scope` comment saying so; you still review all of it.

## 4. Review the code

Same order and same bar as `review-task` step 4: correctness, security and data, tests, architecture, performance.
Two additions for a PR from someone else:

- Documentation: if the project follows the overmind convention, the module docs the diff affects (`prd.md`, `trd.md`, `ard.md`, `database.md`, `flows.md`) must be updated; if it does not, whatever the repo treats as documentation of the touched behavior (README, API docs, comments the code cannot replace) must be.
- Conventions: judge against the patterns the codebase already has and the rules `CLAUDE.md`, `docs/conventions/` or the linter config write down; do not import Sebastian's personal conventions into a codebase that never adopted them, except the ones that hide defects.
  What the linter or a project check already flags on the PR is not repeated here.

Each finding is one comment: `{{type}}{{ (blocking | non-blocking)}}: {{file}}:{{line}}: {{what}} -> {{expected}}`, with why it matters when it is not obvious.

| Type | Use it for | Blocks by default |
|---|---|---|
| `issue` | wrong behavior, missing error path, broken invariant | yes |
| `security` | authorization gap, injection, secret, data loss, irreversible migration | yes |
| `test` | behavior the PR introduces without a test, or a test that cannot fail for a reason that matters; first ask whether the behavior is worth a test at all, a trivial mapping or framework behavior is not, and then it is no finding | yes |
| `scope` | work outside what the description promises; the ask is to split, not to drop | yes |
| `refactor` | code that works but breaks the codebase's layering or pattern | no |
| `docs` | documentation the change owed and did not update | no |
| `question` | something you cannot judge without the author's answer | no |
| `suggestion` | a possible improvement, optional; say it once, never argue it | never |
| `nit` | style, something you would have done differently; only when it hides nothing bigger and costs one line | never |

Write `(blocking)` or `(non-blocking)` only to override a default, and say why in the comment; `suggestion` and `nit` have no override, they are optional by definition.
Blocking comments are what the author must do before merge; non-blocking ones are optional, and the report says so in so many words.

Describe the defect in the code, never the author; no praise padding, no restating what the code already says.

Few comments, well placed; the goal is to merge fast and safely, not to stop the author with a wall of comments:

- Strict where a defect costs: every `issue`, `security`, `test` and `scope` finding goes in, without exception.
- Permissive elsewhere: at most three optional comments per PR, only the ones that teach something the author would want to know; the rest is dropped, not written.
- One comment per pattern, not per occurrence: name the pattern once with one `file:line` and say it repeats.
- A working solution that is not the one you would have written is not a finding.
- When in doubt about a non-blocking thing, let it pass and say why under `Let pass`.

## 5. Report

Two layers, because that is how they land on GitHub:

- Overview: `templates/pr-review.md`, short; verdict, risk, intent in one or two sentences, and one line per comment under `Required before merge` and `Optional`, so the author never has to guess which ones to act on.
- Inline comments: one per finding at `{{file}}:{{line}}`, starting with `[required]` or `[optional]` and the type, then what is wrong, why it matters and what is expected, as long as it needs and no longer.

Verdict is derived, never chosen: any blocking comment is `request changes`; only non-blocking ones is `approve with comments`; none is `approve`.
Risk is judged apart from the verdict, and it is about real users, not about the size of the diff:

1. Production first: read the TRD `Delivery` line (or ask Sebastian once, and write the answer in the TRD so nobody asks twice) to know whether the base branch deploys to something real users depend on.
   If it does not, risk is `✅ Low` and the sentence says `not in production yet`; nothing else is weighed.
2. `🔴 High`: a defect would corrupt or expose data, break auth or payments, take the service down, or could not be undone with a revert commit (migrations, messages sent, external side effects).
3. `⚠️ Medium`: a defect would break a flow real users depend on, a revert would fix it, and the tests do not cover that flow.
4. `✅ Low`: everything else, including features behind a flag, internal tooling, docs, UI adjustments and code the tests cover.

`✅ Low` is the default; Medium and High must name the concrete failure, how it reaches a user and what to watch after merge.
Never write Medium because the diff is large or you are unsure: a big, well-tested change to a flow nobody depends on yet is Low.
A PR can be `approve` and `🔴 High` at the same time: correct and dangerous.

Write the overview to `{{WORKSPACE}}/{{repo}}/review.md` (the worktree, never committed), the inline comments to `{{WORKSPACE}}/review-comments.json` (`path`, `line`, `side: RIGHT`, `body`), and copy both to `{{REVIEWS}}/{{repo}}-pr-{{n}}.md` and `.json`.
Show Sebastian the overview and the inline comments, in the language of the PR description, because that is what will be posted.
Then ask him one question: `post as a pending review on GitHub?`.
The review is written on the PR of the repo the diff belongs to, `{{owner/repo}}` from step 1; in multirepo that is the code repo's PR, never the root docs PR.

On yes:

```
gh api repos/{{owner/repo}}/pulls/{{n}}/reviews -X POST \
  -f commit_id={{headRefOid}} -F body=@{{WORKSPACE}}/{{repo}}/review.md \
  --input {{WORKSPACE}}/review-comments.json
```

No `event` field: the review stays pending, visible only to Sebastian, who edits, picks the verdict and submits from GitHub.
On no, or after posting: `git -C {{clone}} worktree remove {{WORKSPACE}}/{{repo}}` and `git -C {{clone}} branch -D pr-{{n}}`, unless Sebastian asks to keep them; the copy in `REVIEWS` stays.

## Rules

- Never modify, commit or push code on the PR's branch, and never check it out in the root checkout.
- Never run anything: no lint, typecheck, tests, checks, installs or the application; the PR already carries CI, and you only read code.
- Never repeat what the PR already shows: a red check, a linter line, a thread someone else already resolved.
- Never submit the review; a pending review on the PR of the repo it belongs to is the most you post, and only on Sebastian's yes.
- Never soften a blocking finding because the author is a colleague, and never phrase one about the author instead of the code; never flood the PR with optional ones either, three is the cap.
- Never leave the author guessing: every comment is marked required or optional, the overview stays short, the explanation lives in the inline comment.
