# VibeNest configuration — preparation only

This fork preserves ghstats v0.10.0 source, tests, MIT license, Dockerfile, Compose
file, and upstream workflows. The VibeNest additions are documentation only.
GitHub Actions are disabled in this fork's repository settings; upstream workflow
files remain intact. No image has been built or deployment accepted for this pilot.

[![Deploy to VibeNest](https://vibenest.net/deploy-to-vibenest.svg)](https://vibenest.net/deploy?repo=https://github.com/NikitaBabenko/ghstats)

## Guided configuration

1. Open the button and sign in or register. Verify that the setup flow still shows
   `https://github.com/NikitaBabenko/ghstats`, branch `main`. The link supplies only
   the repository, not credentials, storage, hardware, or admission approval.
2. Configure a single Dockerfile service from repository root, using `/Dockerfile`
   and container port `8080`. The included `compose.yml` is a local-build example,
   not a validated VibeNest deployment recipe. Do not select it inadvertently.
3. Before any launch, configure a persistent volume at `/app/data` through the
   platform's normal storage flow. Verify it is mounted and writable by the image's
   `appuser:appgroup`; neither directory creation nor a successful HTTP response
   proves persistent storage. Rust detection may not suggest the volume automatically.
4. Configure the values below in the platform's environment settings. Keep secret
   values out of Git, screenshots, URLs, logs, and support messages.
5. Confirm hardware eligibility, source/security admission, build capability, and
   the intended public exposure before using the normal gated deployment action.
   This pilot has no runtime slot assigned yet. Do not launch this preparation as
   an acceptance test until its blockers have been resolved.

| Variable | Intended value / requirement |
| --- | --- |
| `HOST` | `0.0.0.0` |
| `PORT` | `8080` |
| `DB_PATH` | `/app/data/ghstats.db` |
| `GHS_FILTER` | Exactly `OWNER/ONE_PUBLIC_REPO`, assigned explicitly for this pilot; replace the placeholder before any launch. No `*` or owner wildcard. |
| `GHS_INCLUDE_PRIVATE` | `false` |
| `GITHUB_TOKEN` | A separately approved, repository-limited GitHub credential. Never reuse the owner's general CLI/PAT or a platform credential. |
| `GHS_API_TOKEN` | A separate strong secret for server-side JSON API clients; never the GitHub credential. |
| `GHS_CRON_SCHEDULE` | `0 59 * * * *` (six fields; hourly). Startup also attempts synchronization. |

Alternatively, upstream supports `GITHUB_TOKEN_FILE`, which takes precedence over
`GITHUB_TOKEN` and is reread for each request. Installation tokens require an
existing external rotation mechanism and a supported read-only secret mount.
VibeNest token-file mounting and rotation have not been validated by this pilot;
the application does not refresh installation tokens itself.

## Credential and exposure requirements

GitHub's traffic endpoints require repository **Administration: read** for a
fine-grained token; use only the explicitly assigned public repository and
**Metadata: read**. The public pull-request listing is documented as available
without additional permissions; the authenticated endpoint documents
**Pull requests: read**. Validate the chosen token's actual behavior before
considering any additional grant. Do not add write permissions as a fallback.
The classic `public_repo` recipe in the upstream README grants broader access
and is not the credential recipe for this pilot.

Sources: [traffic permissions](https://docs.github.com/en/rest/metrics/traffic),
[pull-request permissions](https://docs.github.com/en/rest/pulls/pulls#list-pull-requests),
[stargazer permissions and access restrictions](https://docs.github.com/en/rest/activity/starring#list-stargazers).
Stargazer access has additional owner/collaborator restrictions; successful
public repository reads do not prove that traffic or star history will work.

The HTML dashboard, repository traffic details, badges, and `/health` are public.
`GHS_API_TOKEN` protects only `/api/repos` via the `x-api-token` header. Publishing
a public repository's otherwise restricted traffic/referrer data is an explicit
disclosure choice. Use only data approved for public display. Do not place the
API secret in browser-side scripts. The UI loads JavaScript and CSS from
`unpkg.com`; those scripts execute on the dashboard origin.

## Current limitations and launch blockers

- No repository has yet been assigned for real traffic collection and no suitable
  restricted credential has been identified. An empty dashboard, fake token, or
  fixture database cannot substitute for a real synchronization.
- `GHS_FILTER` filters after `/user/repos` or `/installation/repositories` discovery.
  It is not a credential boundary. The hourly job also makes a read-only version
  check to `https://api.github.com/repos/vladkens/ghstats/releases/latest`. This
  documented GitHub-only request is separate from the one-repository statistics
  scope; it sends the GitHub authorization header to GitHub and no database data.
  It does not write to upstream. No token is included in the URL or response UI.
- Use a fresh dedicated database containing only approved public data. Reusing
  a database after narrowing `GHS_FILTER` is not a privacy boundary: star backfill
  reads existing unsynced rows without that filter, and HTMX referrer/path partial
  handlers return before the repository-page filter check. Never reuse an unknown
  or private-data volume for this public pilot.
- The Dockerfile builds both AMD64 and ARM64 with Rust/zig and executes `RUN`
  steps. It is not a lightweight build. Builder resources, network support,
  Dockerfile feature support, and immutable artifact/source identity must be
  checked separately. No alternate prebuilt image is verified here.
- Runtime Free limits of 256 MB RAM, 0.5 vCPU, and 4 GB storage remain untested.
  A small image is not evidence of runtime memory fit or build feasibility.
- `/health` verifies database writability, not a successful GitHub sync. Per-repo
  API failures can be logged while the overall job records success, so a green
  updated badge is not sufficient acceptance evidence.

## Acceptance after prerequisites are met

Follow the button through authentication with the repository preserved, then use
the normal gated deployment on explicitly assigned Free hardware. Confirm the
running source/image identity and effective limits. Observe startup and a complete
real sync, including peak/idle RAM, CPU, disk, restarts, and OOM state.

For the one assigned repository, verify the dashboard and
`/badge/OWNER/ONE_PUBLIC_REPO/views.svg`; verify that `/api/repos` rejects missing
and incorrect tokens and returns exactly the intended repo with the correct token.
Record real traffic response evidence (zero traffic is legitimate), stored dates,
and actual rows independently of the health and updated badges. Keep synthetic
tests labeled separately.

Capture a baseline, perform a permitted same-provider container replacement through
the platform, and verify the same mounted database retains history. Never detach,
delete, or change the volume path to make the check pass. Stop and clean up the
pilot through the native lifecycle after acceptance or a terminal failure.

Long-term history requires continued collection before GitHub's 14-day traffic
retention expires. A short pilot proves synchronization and replacement persistence,
not months of retained production history or permanent Free hosting.
