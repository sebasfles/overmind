# Stack references

One file per stack, loaded only when `docs/TRD.md` names that stack.
Each file holds what the generic flow cannot know: where things live, how modules are laid out, testing conventions, migration commands, how the API spec is generated.

Planned:

- `nestjs.md`
- `nextjs.md`
- `rails.md`
- `react-native.md`

Written when the first project of each stack goes through `setup`; the project's `docs/TRD.md` is the source, the reference is the fallback for projects that lack a section.
