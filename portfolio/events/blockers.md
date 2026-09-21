# blockers

Appended by the om-events session. Unchecked lines are open.

- [ ] 2026-09-18T03:27:52Z my-napkin general gh is authenticated as sflores-designli with no push/admin on sebasfles/my-napkin; publish-task (gh pr create) will fail for every task (0001, 0002). Needs gh auth login as sebasfles, or GH_TOKEN.
- [ ] 2026-09-18T05:00:00Z my-napkin tasks 0004 and 0005 cannot run e2e: app/.env.local pointing at dev is missing in .workspaces/0004_diagram_persistence/my-napkin/app/ and .workspaces/0005_password_auth/my-napkin/app/; 0004 also needs 0003's terraform applied on dev. Only Sebastian can place the files.
- [ ] 2026-09-20T00:00:20Z my-napkin 0016 om-developer refused "npx playwright test" against https://napkin.dev.sdfles.com by its permission layer ("Interfere With Workloads"); needs Sebastian: a Bash allow rule for playwright in the project settings, or the diagnosis moved to a local production build.
- [ ] 2026-09-20T00:00:30Z my-napkin general host out of memory for e2e: ~2 GB available of 7.8 GB, sessions across auvral, local-auctions, overmind, om-config and my-napkin hold the rest; a local Playwright run needs ~3 GB (0012 phase 3: 83 of 83 specs failed, next dev OOM-killed). Needs Sebastian: which sessions stand down, or fewer bg-spare daemons.
- [ ] 2026-09-20T00:00:31Z my-napkin 0012 phase 3 needs a 15-20 min window with no build or suite running on this machine; three e2e runs lost specs to other projects' builds (auvral 0042 twice), the suite needs ~1.8 GB headroom. Overmind must hold the other projects; om-manager cannot.
