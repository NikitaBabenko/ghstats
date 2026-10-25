# VibeNest mixed Compose pilot — not yet validated

This fork preserves ghstats v0.10.0 source, tests, MIT license, the original root
Dockerfile, and upstream workflows. The original Compose is preserved byte-for-byte
at `vibenest/upstream/compose.yml` for reference; its relative build paths describe
the original repository root, so do not deploy that archived copy in place.
For an intentional upstream local build, run from the repository root:
`docker compose --project-directory . -f vibenest/upstream/compose.yml up --build`.
This resolves its build context and `.env` against the original root; it is not
the VibeNest pilot's deployment command.
The selected root `compose.yml` adds a real HTTP frontend and uses the verified
upstream application image. Only `vibenest/frontend` is built by this recipe.
GitHub Actions are disabled in this fork's repository settings; upstream workflow
files remain intact. No image has been built or deployment accepted for this pilot.

[![Deploy to VibeNest](https://vibenest.net/deploy-to-vibenest.svg)](https://vibenest.net/deploy?repo=https://github.com/NikitaBabenko/ghstats)

## Guided configuration

1. Open the button and sign in or register. Verify that the setup flow still shows
   `https://github.com/NikitaBabenko/ghstats`, branch `main`. The link supplies only
   the repository, not credentials, storage, hardware, or admission approval.
2. Select Docker Compose from repository root, `/compose.yml`, with the public
   domain bound only to `frontend:8080`. An existing explicit Dockerfile override
   must be changed through normal project settings; adding Compose alone does not
   replace that override. The original Rust Dockerfile must not be selected.
3. Before first launch, verify the native Compose plan has a fresh, project-scoped
   named volume `ghstats-data` mounted only on backend `/app/data`. Do not add a
   second application-level volume, an external volume, or a host bind mount.
   Verify the actual mount and writability by `appuser:appgroup` after startup.
   SQLite is not a recognized Compose database engine in VibeNest's backup UI;
   neither a Storage-tab entry nor HTTP success proves persistence.
4. Configure the values below in the platform's environment settings. Keep secret
   values out of Git, screenshots, URLs, logs, and support messages.
5. Confirm hardware eligibility, source/security admission, build capability, and
   the intended public exposure before using the normal gated deployment action.
   This pilot has no runtime slot assigned yet. Do not launch this preparation as
   an acceptance test until its blockers have been resolved.

## Two useful services and separate identities

- `frontend` packages the real `nginx.conf` using COPY only. It runs as the base
  image's unprivileged UID 101, with one worker and PID/temp paths under `/tmp`.
  Every HTTP path, including `/health`, goes to the fixed `http://ghstats:8080`
  upstream, preserving request URI/query, Host, and `x-api-token`. There is no
  static success response, forward-proxy/CONNECT module, dynamic destination, or
  response cache. Only the frontend receives a public domain; neither service
  publishes host ports. No application secrets are passed to the frontend.
- `ghstats` runs the unchanged upstream image, not a binary compiled from the new
  frontend configuration. Its source is upstream commit
  `bfa5a13e9a520b695292f307ca774034eb20ceec` (v0.10.0). Registry-attached BuildKit
  provenance and upstream build run `32911443867` associate that source with the
  pinned image; independent signature verification has not been performed.

| Artifact | Immutable reference |
| --- | --- |
| ghstats index | `ghcr.io/vladkens/ghstats@sha256:23e05ff578d1e2b9ceb16228bae673d5dacfd34f5d8678407726d62a330682bf` |
| ghstats AMD64 manifest | `sha256:1913c91349150b6bc34f1275ff663d3d7efe1fc39590047dde0aaa56750c41d5` |
| ghstats AMD64 config / expected image ID | `sha256:d870727a958c47ff90ee5ff5ac3e6a5b02dc240b5ed1120d0adfa22b70fb143d` |
| NGINX base index | `nginx/nginx-unprivileged@sha256:ea8a305bb7bff1e622c8223a097bf5c7664d36b05371717e23dc69972e1924e6` |
| NGINX base AMD64 manifest | `sha256:1d88e6f7ea3aa1bb8e89e214e8ab4b0a659553d8a3f9ff7b40d0242119b00c57` |

The NGINX pin was resolved from the publisher's `1.30.6-alpine-slim` image. Its
AMD64 config has no ONBUILD triggers. The inherited entrypoint and its IPv6,
local-resolver, envsubst, and worker-autotune scripts were inspected as text from
digest-verified image layers, not executed. This recipe supplies no templates or
autotune switches; its full nginx.conf does not include the stock default server.
The base entrypoint/CMD are inherited unchanged. No RUN, downloaded executable,
custom entrypoint, or revision-label override is added.

The requested fork configuration commit and any platform-generated managed
configuration commit are distinct from the upstream application source commit.
VibeNest's Compose serving check verifies the built frontend's configuration
revision; it does not independently prove the prebuilt backend digest. Acceptance
must additionally compare both actual containers and the backend ImageId and
RepoDigests with the pinned manifest/config and provenance. The frontend's image
ID will differ from its base image after COPY. A new configuration requires fresh
source/owner/security admission; an earlier source observation is not reusable.

The proposed plan total is 256 MiB / 0.5 platform vCPU. Memory is explicitly split
into frontend 32 MiB and backend 224 MiB; runtime fit remains unmeasured. The fork
intentionally omits CPU caps: native managed preparation calculates the host CPU
budget using the assigned server's scale and divides it across both services.
On the reviewed platform revision, a scale of 0.5 yields a total host limit of
0.25 CPU, shared equally as 0.125 per service. These are expected normalized
values, not measured quotas. Acceptance must inspect both actual CPU quotas and
their sum, as well as memory and swap limits. Do not change server policy or grant
extra resources. Refusal is a stop condition, not permission to weaken a quota or
security gate. Managed preparation and initial domain binding may produce another
configuration commit and a second gated deployment.

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

- This pilot's assigned public repository is `NikitaBabenko/ghstats`. A restricted
  credential and masked project ENV have been prepared separately; their current
  validity must be checked before a launch. No secrets belong in this repository.
  An empty dashboard, fake token, or fixture database cannot substitute for a real
  synchronization.
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
- The preserved upstream Dockerfile builds both AMD64 and ARM64 with Rust/zig
  and RUN steps; the selected Compose does not execute it. COPY-only frontend
  packaging avoids those RUN steps, but registry access, extraction, build helpers,
  and Compose networking remain necessary. A known target-builder network incident
  has not been disproved by this static recipe. No build or runtime success is
  claimed; stop on an unexpected network failure or native gate refusal.
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
