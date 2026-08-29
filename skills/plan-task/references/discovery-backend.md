# Discovery track: backend

Use as a checklist. Ask only what the docs and code do not answer.

## Fit

- New module, extension of an existing one, or a background job?
- Which bounded context owns it? Does it cross module boundaries?

## Contract

- Endpoints created or modified: method, route, purpose.
- Request and response shape at the level of fields, not types; the OpenAPI spec is generated from code.
- Status codes for success and each failure. Errors are HTTP errors, never 200 with an error body.
- Pagination, filtering, sorting if the endpoint lists things.

## Data

- Entities and their relationships. Tables owned vs referenced.
- Migrations expected. Soft delete, audit, versioning needs.
- Indexes for the expected volume.
- Invariants that live in code and not in the database.

## Rules

- Input validation (reject malformed early) vs business rules (domain invariants).
- Authorization: who can read, who can write.
- Idempotency for writes. Concurrency and race conditions.
- Rate limiting or abuse prevention if exposed.

## Integrations

- External services, webhooks, queues, events, other internal services.
- New env vars, secrets, configuration.
- Behavior when a dependency is down: retry, fallback, degrade.

## Operations

- Logging, monitoring, alerting.
- Data migration for existing records.
- Performance: caching, query cost.
