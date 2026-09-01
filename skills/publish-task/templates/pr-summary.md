## Intent

{{Two or three sentences: what the task set out to achieve and why, at goal level. No implementation detail; name a consolidation agreement only if it changes how to read the PR.}}

## What changed

- {{one concise line, behavior-level, no subclauses}}
- {{...}} (max 10)

## Decisions

- {{decision taken after consolidation}}: {{why, one line}}
- Let pass: {{questionable thing}}: {{why acceptable, one line}}

## Risk assessment

{{✅ Low | ⚠️ Medium | 🔴 High}}: {{one sentence}}. {{Medium or High only: what to watch after merge.}}

## Pipeline

- ✅ intent
- ✅ rebase
- ✅ lint
- ✅ typecheck
- ✅ test
- ✅ review: {{k}} issues auto-fixed
- ✅ documentation
- ✅ replication {{bugs only; otherwise n/a}}
- ✅ push

<details>
<summary>Pipeline detail</summary>

- intent: {{scope check, one line}}
- rebase: {{base and head sha}}
- lint / typecheck / test: {{commands, targets, counts; one line per verification target}}
- review: {{file}}:{{line}}, {{defect}}, {{fix}}, re-checked (one line per finding, all rounds)
- documentation: {{modules updated}}, ARD entries: {{n}}
- push: {{sha per repo}}

</details>
