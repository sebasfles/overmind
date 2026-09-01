## Intent

{{One paragraph: what the task set out to do and why, including what was agreed in consolidation.}}

## What changed

- {{behavior-level bullet}}
- {{...}} (max 10)

## Decisions

- {{decision taken by the om-reviewer after consolidation}}: {{reason}}
- Let pass: {{questionable thing}}: {{why it is acceptable}}

## Risk assessment

{{Low | Medium | High}}: {{one sentence}}. {{For Medium or High: what to watch after merge.}}

## Pipeline

- [x] intent: passed
- [x] rebase: passed
- [x] lint: passed
- [x] test: passed ({{n}} tests, `--runInBand`)
- [x] review: {{k}} issues found, fixed
  - `{{file}}:{{line}}`: {{defect}}. Fix: {{fix}}. Re-checked.
- [x] documentation: {{modules}} updated, ARD entries: {{n}}
- [x] replication: passed ({{bugs only}})
- [x] push: `{{sha}}`
