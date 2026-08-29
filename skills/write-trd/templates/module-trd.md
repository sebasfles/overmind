---
updated: {{YYYY-MM-DD}}
source: setup
---

# {{Module}}: technical

## Structure

| Path | What |
|---|---|
| `{{path}}` | {{...}} |

## Endpoints owned

| Method | Route | Purpose | Spec |
|---|---|---|---|
| {{GET}} | `{{/route}}` | {{one line}} | [spec]({{link}}) |

Jobs, listeners or scheduled work: {{one line each, or "none"}}.

## Depends on

- {{module or external service}}: {{why}}

## Depended on by

- {{module}}: {{how}}

## Configuration

- `{{ENV_VAR}}`: {{what it controls}}

## Testing

- Tests live in `{{path}}`.
- Module-specific run: `{{cmd or "see TRD"}}`.
