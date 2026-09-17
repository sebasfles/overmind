## Review: #{{n}} {{title}}

Head: {{headRefOid}}
Verdict: {{✅ approve | 🔴 request changes}}
Risk: {{✅ Low | ⚠️ Medium | 🔴 High}}: {{one sentence}}. {{Medium or High only: what to watch after merge.}}
Size: +{{additions}} -{{deletions}}, {{files}} files, {{commits}} commits
Intent: {{one or two sentences: what the PR says it does, and whether the diff does that and only that; `depends on #{{m}}` when a related PR was given}}

### Since last review ({{re-review only}})

addressed {{a}}, still open {{s}}, withdrawn {{w}}, new {{x}}

- addressed: `{{file}}:{{line}}`: {{what the comment asked, one line}}
- withdrawn: `{{file}}:{{line}}`: {{why you were wrong, one line}}

### Required before merge ({{b}})

1. issue: `{{file}}:{{line}}`: {{what, one line}}
2. test: `{{file}}:{{line}}`: {{what, one line}}
3. refactor (blocking): `{{file}}:{{line}}`: {{what, one line}}

### Optional ({{o}})

- suggestion: `{{file}}:{{line}}`: {{what, one line}}
- nit: `{{file}}:{{line}}`: {{what, one line}}
- question: `{{file}}:{{line}}`: {{what you need to know, one line}}

### Let pass

- {{questionable thing}}: {{why acceptable, one line}}

### Summary

{{b}} required, {{o}} optional; +{{additions}} -{{deletions}}, {{files}} files, {{commits}} commits
