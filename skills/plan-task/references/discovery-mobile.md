# Discovery track: mobile (React Native or native)

Use as a checklist. Ask only what the docs and code do not answer.

## Fit

- Which stack or navigator: auth, onboarding, main tabs. New stack or modal?
- Deep links or push notification entry points.

## Interface

- Components reused vs created; check existing ones first.
- Step-by-step flow from the user's perspective. Happy path, error paths, edge cases.
- Loading, success, error and empty states, and what the user can do in each.
- Figma link if it exists; the developer pulls design context from it.

## Data

- What the feature consumes and produces. Domain entity shape.
- Repository contract: existing or new.
- DTO to domain mapping when the API shape differs.
- Offline behavior and poor connectivity.
- New constants.

## Rules

- Input validation and where errors show.
- Conditional visibility or enablement.
- Security: sensitive data at rest, in transit, in logs.

## Integrations

- New API functions, headers, env vars.
- Authentication flow involved?

## Quality

- Accessibility: screen readers, tap targets, dynamic font scaling.
- iOS vs Android differences.
- Analytics events.
