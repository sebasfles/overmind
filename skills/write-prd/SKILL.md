---
name: write-prd
description: Write or update the product requirements (general PRD.md or a module's prd.md). Used by setup and om-setup-worker.
effort: high
argument-hint: "[general | MODULE_NAME] [OVERVIEW_FILE] [DESIGN_FOLDER]"
disable-model-invocation: false
---

# write-prd

## Purpose

Write or update the Product Requirements Document: the general docs/PRD.md (what the product is, for whom, its
modules as capabilities) and per-module prd.md (what the module does for the user and why). No engineering
detail. Sources: an overview document, designs, the code's user-facing surface. Edits in place. Used by setup;
om-manager scope.

Input: `general` or a module name; optionally an overview document (`overview.md` or similar) and a folder of design images.
Output: `docs/PRD.md` or `docs/modules/{{module}}/prd.md`, written or updated in place, with `updated` and `source` frontmatter.

The PRD answers what and why, never how.
Its reader is a product person or a future agent that needs to understand intent before touching a module.
If a sentence mentions a table, an endpoint, a component or a library, it belongs in the TRD.

## Sources, in order of authority

1. An overview or product document Sebastian points to, if any.
2. Design files: read every image in the design folder; each screen tells you a capability and its states.
3. The user-facing surface of the code: routes and pages, screens and navigators, public endpoints and their names, copy and validation messages, feature flags. Read them as a user would experience them.
4. Existing README or docs.

When sources disagree, the overview wins for intent and the code wins for what exists today; note the gap under `Open questions`.

## General `docs/PRD.md`

Read `templates/PRD.md` and fill:

1. Product: one paragraph, what it is and for whom.
2. Users: the roles or personas the product serves, one line each.
3. Capabilities: one section per module, written as what the user can do, with a link to the module's `prd.md`. Order them the way a new user would meet them.
4. Cross-cutting rules: things true everywhere (auth model as the user sees it, permissions, notifications, i18n, accessibility commitments).
5. Not in the product: explicit exclusions found in the sources.
6. Open questions: gaps between intent and code, marked `[inferido]` where you had to guess.

## Module `docs/modules/{{module}}/prd.md`

Read `templates/module-prd.md` and fill:

1. Purpose: what the user achieves here and why it matters.
2. User flows: the main paths as numbered steps, from the user's point of view, including error and empty states as the user sees them.
3. Rules: business rules in plain language (limits, permissions, validations as the user experiences them).
4. Out of scope: what this module deliberately does not do.
5. Open questions.

## Rules

- No engineering detail; move it to the TRD.
- Mark inferences `[inferido]`; `setup` collects them.
- Edit in place; the PRD is current state, not history.
- Frontmatter `updated` and `source`.
- Keep the general PRD navigable: under 200 lines, with the detail in the module files.
- English.
