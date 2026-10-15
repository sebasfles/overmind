# blockers

Appended by the om-events session. Unchecked lines are open.
- 2026-10-14T19:46:24-05:00 my-napkin - MemAvailable 2.1 GB below the 3 GB Playwright floor; needs suites for 0025 then 0027 within the hour; asks to stand down idle sessions and bg-spares
- 2026-10-14T20:03:30-05:00 my-napkin task 0025 e2e held 15 min, MemAvailable 1.8 GB against a 3 GB floor, no builds running; needs Sebastian's decision on overmind's proposal (stop 6 bg-spare daemons and 6 idle local-auctions om-pr-reviewers). 0027 queued behind it.
- 2026-10-14T23:21:40-05:00 my-napkin AWS SSO session `fless` expired; every local e2e on this machine fails with CredentialsProviderError (0027 round 2 lost its suite). Sebastian must run `aws sso login --sso-session fless`; no agent can.
