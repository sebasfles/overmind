# Discovery track: frontend (web)

Use as a checklist. Ask only what the docs and code do not answer.

## Fit

- New page, section of an existing page, modal or drawer? Protected route?
- How it relates to the layout system and navigation.

## Interface

- Components reused vs created; check the component library first.
- Step-by-step flow from the user's perspective. Happy path, error paths, edge cases.
- Loading, success, error and empty states.
- Figma link if it exists; the developer pulls design context from it.

## Data

- What the feature consumes and produces. Domain entity shape.
- State: new store or slice, or extend an existing one. Caching and its lifetime.
- Mapping when the API shape differs from what the UI needs.
- New constants: routes, limits, flags.

## Rules

- Input validation and where errors show: inline, toast, other.
- Conditional visibility or enablement by state, role or permission.

## Integrations

- New API functions, headers, env vars.
- Browser APIs: notifications, geolocation, files, clipboard.

## Quality

- Responsive behavior.
- Accessibility: keyboard, ARIA, screen readers.
- SEO if public: meta, SSR, structured data.
- Analytics events.
- Back and forward navigation.
