---
name: write-trd
description: >-
  Write or update the Technical Requirements Document: the general docs/TRD.md (stack, layout, module
  list, verification commands, API spec generation, base branch) and per-module trd.md (structure,
  owned endpoints, integrations). Derives from the code; marks anything inferred. Edits in place.
  Used by setup; manager scope.
argument-hint: "[general | MODULE_NAME]"
disable-model-invocation: false
---

# write-trd

Input: `general`, or a module name.
Output: `docs/TRD.md` or `docs/modules/{{module}}/trd.md`, written or updated in place, with `updated` and `source` frontmatter.

The TRD describes how the system is built, as it is today.
It does not restate code: no request or response schemas, no column lists, no function signatures.
It says where things are, how they are organized, and how to operate them.

## General `docs/TRD.md`

The TRD describes the whole application; in monorepos and multirepos it has one component section per app or repo.
Read `templates/TRD.md` and fill every section from evidence:

1. Stack: languages, frameworks, package manager, runtime versions, from manifests (`package.json`, `Gemfile`, `pubspec.yaml`, `go.mod`, lockfiles, `.nvmrc`, `.ruby-version`).
2. Layout: top-level folders and what lives in each; monorepo packages or apps if any.
3. Modules: the list of modules with one line each and a link to `docs/modules/{{module}}/README.md`. A module is a bounded area of the domain the code already groups (a NestJS module, a Rails engine or namespace, a feature folder); do not invent boundaries the code does not have.
4. Verification targets: one row per component with path and the exact commands for lint, typecheck, unit and e2e, each with its serial flag. `verify-task` reads this table literally; write `unknown` rather than guessing.
   Workspace files and install command per component: what a fresh worktree needs before anything runs (`.env*`, generated files, `pnpm install`). `consolidate-task` reads this to bootstrap workspaces.
5. API spec: how the OpenAPI or GraphQL schema is generated and where it lands (`@nestjs/swagger`, rswag, `graphql-schema`), or `none`.
6. Data: database engine, ORM or migration tool, how migrations run.
7. Environments and delivery: base branch, environments, how deploys happen, IaC location if any.
8. Conventions: patterns the code follows consistently (layering, naming, error handling, testing layout), stated as observed, with one example path each.

## Module `docs/modules/{{module}}/trd.md`

Read `templates/module-trd.md` and fill it:

1. Structure: the folders and key files of the module, one line each, grouped by component when the module spans several repos or apps.
2. Endpoints owned: one line per endpoint or job, method, route, purpose, link to the generated spec. Nothing about payloads.
3. Depends on: other modules and external services it calls.
4. Depended on by: modules that call it, if discoverable.
5. Configuration: env vars and feature flags the module reads.
6. Testing: where its tests live and any module-specific way to run them.

## Rules

- Evidence first: every statement comes from a file you read; cite the path when it is not obvious.
- Mark inferences `[inferido]`; setup collects them for Sebastian to confirm or delete.
- Edit in place: the TRD is the current state, not a history. History is git.
- Frontmatter on every file: `updated: {{today}}`, `source: setup` (or the task id when called from a task).
- Keep the general TRD under 150 lines and a module TRD under 80; link, do not duplicate.
- English.
