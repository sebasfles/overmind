---
updated: {{YYYY-MM-DD}}
source: setup
---

# {{Module}}: flows

Only flows that deserve a diagram. A CRUD does not.

## {{Flow name}}

{{One sentence: when this flow runs.}}

```mermaid
stateDiagram-v2
  [*] --> {{State}}
  {{State}} --> {{Other}}: {{event}}
```

```mermaid
sequenceDiagram
  participant U as User
  participant A as {{Service}}
  U->>A: {{call}}
  A-->>U: {{result}}
```
