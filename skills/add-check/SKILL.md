---
name: add-check
description: "Add a project review check (docs/checks/{{name}}.md) with its reference doc, dry-run it, commit. om-manager; on Sebastian's ask."
effort: high
argument-hint: "[NAME]"
disable-model-invocation: false
---

# add-check

## Purpose

Add one review check to the project: a small, single-purpose review the om-reviewer runs in the background on
every round with a cheap model (i18n coverage, a code style the linter cannot express, a module's patterns).
Makes sure the rule is written down in a project document first, writes `docs/checks/{{name}}.md` from the
template, dry-runs it against the current code to calibrate, and commits. om-manager; on Sebastian's ask.

Input: what Sebastian wants verified, in his words.
Output: `docs/checks/{{name}}.md`, its reference document if it did not exist, one commit on the base branch.

Checks are read-only single-rule checkers, not linters: they judge against a written rule, and their findings are triaged by the om-reviewer before reaching the om-developer.
A check without a reference document is not allowed; the checker would invent the rule.

## 1. Understand the check

Settle with Sebastian, in one batch of questions:

- `name`: kebab-case, the file name (`i18n`, `style`, `billing-patterns`).
- The rule: what is a finding and, as important, what is not (intentionally untranslated strings, generated files).
- `paths`: globs relative to the project root that select the files the check looks at; in multirepo prefix them with the repo folder, like `Verification targets`.
- `model`: `sonnet` unless Sebastian asks otherwise; `opus` only for rules that need to follow logic across files.
- `reference`: the document that defines the rule.

## 2. Reference document

Look for the rule in `docs/`: `TRD.md` Conventions, the module's `trd.md`, `docs/conventions/*.md`.
If it exists, confirm it says what Sebastian described; if it disagrees, fix the document with him before writing the check.
If it does not exist, write `docs/conventions/{{topic}}.md` from the code as `write-trd` writes conventions: evidence first, one example path per statement, `[inferido]` on anything not verified, frontmatter `updated` and `source: add-check`.
Show it to Sebastian and get his yes before continuing.

## 3. Write the check

Read `templates/check.md` and fill it.
Read the reference in `references/` that matches the kind of check (`i18n.md`, `style.md`, `patterns.md`) as a worked example; adapt, do not copy.
The body is a list of verifiable statements in the present tense, each one something the checker can point at with `file:line`.
End with what the checker must let pass.
Write it to `docs/checks/{{name}}.md` in the root checkout.

## 4. Dry run

Run the check once the way `review-task` will, over a bounded diff of files that match `paths`:

```
cd {{ROOT}} && git diff {{from}}...HEAD -- {{matching files}} | claude -p --model {{model}} --allowedTools Read,Grep,Glob -- "$(cat docs/checks/{{name}}.md) ... "
```

Pick `{{from}}` as the last merge into `{{base}}` and look at `git diff --stat` first; more than 30 files or 200 KB is not a review-sized diff, narrow it (fewer commits or a subset of files).
Then one positive test on a diff that must produce findings, typically the reversed diff from before the rule was applied (`git diff HEAD {{old commit}} -- {{files}}`).
Files that were moved come out of a reversed diff as whole-file deletions with no `+` lines and read as clean; for those compare blobs by path: `git diff {{old commit}}:{{old path}} HEAD:{{new path}}`.

Use the checker prompt from `review-task` step 0 literally, including the `--` before it (the frontmatter `---` would otherwise be parsed as an option).
Report the count and a sample of findings to Sebastian, one line each.
Many findings on existing code mean one of two things and Sebastian decides which: the rule is not followed today (record the debt in `docs/ARD.md`, the check stays), or the check is stricter than the rule (tighten the body and rerun).
Zero findings on code that should have some means the body is too vague; sharpen it.

## 5. Commit

With Sebastian's approval, one commit on the base branch: `docs(checks): add {{name}}`, with the reference document if it was created.
Tell Sebastian in one line that the check runs from the next round of every task; running om-reviewers pick it up on their next round since they glob `docs/checks/` from the workspace copy after their next rebase.

## Rules

- No reference document, no check.
- The check judges the diff, not the whole codebase; existing violations are debt for the ARD, not findings for the om-developer.
- One check, one rule; two rules are two files.
- Never run a check inside a task workspace; the dry run uses the root checkout.
- English; one sentence per line.
