# VibeNest mixed Compose pilot — guided traffic scenario verified

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
files remain intact. On 25 October 2026, the configured pilot passed real traffic
sync, public HTTPS/API/badges, and SQLite retention across a normal replacement.
This is a guided, already-configured project result; unattended fresh signup with
no environment values has not been validated.

The temporary public demo was stopped on **27 October 2026** through the owner
Stop and Release hardware flow. Its Free hardware returned to inventory for the
next pilot. The named SQLite volume and eight project ENV entries were retained;
this guide no longer offers a live demo.
The pilot token expired at **20:00 UTC on 25 October 2026** (00:00 on 26 October
in Asia/Tbilisi); continued live synchronization after expiry is not promised.
Use your own repository-scoped credential for your deployment.

The runtime-tested fork commit is `161628afaa67019ff89304a48a36c297adf3614f`.
Successful deployments 5494 and 5495 served managed commits `432b78ba7fa723abfa0af124dfb0649c5a93d2d1`
and `84d02543269adccec114a66c91f3f6c92aa06ffc`. The latter has the tested fork
commit as its parent and changes only `compose.yml` for native configuration.
The 26 October Compose/defaults/template changes have only been statically
reviewed against platform revision `26261cb5fcaec1b05781560ca5ccda529e38979d`.
They have not been deployed. That static preparation did not change the demo or
its ENV; the later 27 October retirement only used the native Stop/Release flow.

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
4. Supply your own `GITHUB_TOKEN` for the selected public repository. Ordinary
   settings below have safe pilot defaults; VibeNest's existing Compose reconcile
   reads `.env.example` and persists missing defaults and the generated internal
   secret. Review these settings, particularly the repository filter, rather than
   manually inventing all eight values. Keep secrets out of Git, screenshots,
   URLs, logs, and support messages. Local Compose does not generate secrets:
   copy the template to an uncommitted `.env` and set both credentials yourself.
   For this pilot, an operator disabled the provider's
   `inject_build_args_to_dockerfile` setting before the verified rebuild. Current
   VibeNest owner UI cannot set this option. This operator-assisted step is part
   of the tested setup; the button alone does not provide it. Project ENV still
   reaches both services at runtime, so do not assume per-service isolation.
5. Confirm hardware eligibility, source/security admission, build capability, and
   the intended public exposure before using the normal gated deployment action.
   After a prior Stop, the button flow required an explicit VibeNest Start
   Application. For Compose, that action performed a fresh gated redeploy.
   Do not bypass a refused gate or use a direct provider Start.

## Two useful services and separate identities

- `frontend` packages the real `nginx.conf` using COPY only. It runs as the base
  image's unprivileged UID 101, with one worker and PID/temp paths under `/tmp`.
  Every HTTP path, including `/health`, goes to the fixed `http://ghstats:8080`
  upstream, preserving request URI/query, Host, and `x-api-token`. There is no
  static success response, forward-proxy/CONNECT module, dynamic destination, or
  response cache. Only the frontend receives a public domain; neither service
  publishes host ports. The Compose recipe declares no frontend application ENV;
  however, the provider injects project secrets into frontend runtime ENV.
  Initially they also reached image build history. After the operator-assisted
  setting and normal rebuild, both token values were absent from the new frontend
  image history and its image ENV. Old images were not retroactively cleaned.
  Runtime fanout remains; no per-service secret isolation is claimed.
- `ghstats` runs the unchanged upstream image, not a binary compiled from the new
  frontend configuration. Its source is upstream commit
  `bfa5a13e9a520b695292f307ca774034eb20ceec` (v0.10.0). Registry-attached BuildKit
  provenance and upstream build run `32911443867` associate that source with the
  pinned image; independent signature verification has not been performed.

| Artifact | Immutable reference |
| --- | --- |
| ghstats index | `ghcr.io/vladkens/ghstats@sha256:23e05ff578d1e2b9ceb16228bae673d5dacfd34f5d8678407726d62a330682bf` |
| ghstats AMD64 manifest | `sha256:1913c91349150b6bc34f1275ff663d3d7efe1fc39590047dde0aaa56750c41d5` |
| ghstats AMD64 config | `sha256:d870727a958c47ff90ee5ff5ac3e6a5b02dc240b5ed1120d0adfa22b70fb143d` |
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
must additionally compare both actual containers and resolve the backend's image
index, platform manifest, and config against the pins and provenance. Docker's
reported ImageId can identify an OCI index with the containerd image store, so it
must not simply be equated to the config digest. The frontend's image
ID will differ from its base image after COPY. A new configuration requires fresh
source/owner/security admission; an earlier source observation is not reusable.

Memory is explicitly split into frontend 32 MiB and backend 224 MiB. Both services
were healthy without restarts or OOM at the first observation; sustained fit is
not yet proven. At the assigned server's scale of 0.5, Free's 0.5 platform vCPU
provides a 0.25 host-CPU ceiling. Automatic 0.125-per-service quotas in the earlier
revision became actual cgroup quotas of `13000 100000` each (0.26 total).
The current recipe therefore explicitly requests `cpus: '0.12'` on each service:
0.24 host CPU total, below that ceiling, not a claim to the full plan CPU amount.
Actual cgroups confirmed `12000 100000` on each service, with 32 + 224 MiB memory
limits and no observed OOM or restarts. A different server scale requires a fresh
native budget check. Acceptance must inspect actual CPU quotas and
their sum, as well as memory and swap limits. Do not change server policy or grant
extra resources. Refusal is a stop condition, not permission to weaken a quota or
security gate. Managed preparation and initial domain binding may produce another
configuration commit and a second gated deployment.

| Application variable | Resolution when absent / empty | Owner input |
| --- | --- | --- |
| `HOST` | Compose default and template value `0.0.0.0` | None |
| `PORT` | Compose default and template value `8080` | None; frontend expects 8080 |
| `DB_PATH` | Compose default and template value `/app/data/ghstats.db` | None; inside the persistent volume |
| `GHS_FILTER` | Compose default and template value `NikitaBabenko/ghstats` | Change to your approved public repo if different; no wildcard default |
| `GHS_INCLUDE_PRIVATE` | Compose default and template value `false` | None |
| `GITHUB_TOKEN` | Blank template; required Compose expression; never generated | Your repository-limited GitHub credential |
| `GHS_API_TOKEN` | Existing nonempty value wins; otherwise use `GHS_AUTH_SECRET`, generated and stored by VibeNest from its blank template entry | No new internal secret input on the reviewed platform path; local Compose users must supply one |
| `GHS_CRON_SCHEDULE` | Compose default and template value `0 59 * * * *` (hourly, six fields) | None; startup also attempts synchronization |

`GHS_AUTH_SECRET` is an adapter name, not an additional ghstats application
setting. The platform classifies it as `GeneratedSecret`; `GHS_API_TOKEN` is
`UserOnly` because generic token names are intentionally not generated.
The external-provider rule also makes `GITHUB_TOKEN` `UserOnly`.
Reconciliation generates 24 random bytes as 48 hex characters, stores the value
as sensitive, and preserves an existing value on later runs. Nothing is generated
by this repository, and no real or shared sample secret is committed.
The adapter applies only to this fork's root `compose.yml`; direct upstream
`docker run` and the archived upstream Compose require `GHS_API_TOKEN` itself.
The [nested Compose fallback](https://docs.docker.com/reference/compose-file/interpolation/)
preserves a legacy nonempty `GHS_API_TOKEN` rather than silently rotating it.
If neither internal value is available, Compose fails instead of enabling an
unauthenticated API. An explicitly empty or invalid existing platform variable
needs correction; the reconciler does not promise to overwrite existing rows.

In the reviewed platform source, Compose environment extraction keeps YAML scalar
values intact, and the native resource overlay copies scalar values unchanged.
This recipe has no `env_file` for the missing-file rewrite to replace.
The reconciler obtains defaults from the template, not from a partial parse of
the nested expression. These are source-review findings; the provider's own
parsing and final container environment still need a runtime check.

Static review establishes this configuration contract, not end-to-end onboarding.
The remaining owner-only gap is the operator-assisted provider setting above:
the reviewed platform source exposes no owner control for
`inject_build_args_to_dockerfile`, and ENV sync still sends project-wide key/value
pairs. Defaults and secret generation do not remove build-history or runtime
fanout concerns. A fresh button run must verify missing-token guidance, generated
secret persistence, nested interpolation, ordinary gate success and the provider
setting before this revision can be called runtime-verified. For another user's
repository, the filter must also be changed to that intended public repository.

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

## Verified scope and remaining limitations

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
  and Compose networking remain necessary. The earlier COPY-only frontend build
  succeeded; that does not establish that a Rust RUN build would succeed.
- The real README button retained the repository through Register and existing
  account Sign in, then reused a manually configured Compose project. Anonymous
  preview still labeled it Dockerfile. Clean zero-configuration onboarding is
  not proven. The first native deployment hit `snapshot_forbidden` during domain
  continuation. A platform fix and a new normal gated deployment subsequently
  established public access; no source-access or security gate was bypassed.
  After the pilot was stopped, the repeat badge flow required explicit native
  Start. Normal Deploy Now then proved replacement persistence on the same volume.
- Real synchronization stored 14 daily traffic rows for the assigned public
  repository (zero traffic is legitimate). Stargazer backfill returned 403, so
  this is not proof that every metric works. Keep credential permissions narrow.
- Project ENV still reaches the frontend runtime despite backend-only Compose
  references. Current owner settings expose no runtime, build-time, or per-service
  ENV controls. The operator-assisted build-argument mitigation was verified on
  both new builds; it does not provide runtime secret isolation.
- Actual CPU/RAM limits, public HTTPS/API/badges, and replacement persistence
  passed. Twenty paced public requests all returned 200 in 7.11 seconds (median
  102.9 ms, maximum 135.5 ms), with no restarts or OOM. RAM after that small check
  was 6.23 MiB; the sum of individual lifetime memory peaks was 16.99 MiB, an upper
  bound rather than a simultaneous peak. This is not sustained-load evidence.
- Observed disk use was below the plan: 36,864 bytes in the volume, 65,536 bytes
  of container writable layers, and approximately 54.8 MiB of conservatively
  summed virtual image sizes. Hard 4 GB storage-quota enforcement was not proved.
- `/health` verifies database writability, not a successful GitHub sync. Per-repo
  API failures can be logged while the overall job records success, so a green
  updated badge is not sufficient acceptance evidence.

## Acceptance evidence and repeat procedure

Both public verification rounds passed: dashboard and repository details,
views/clones SVG badges, HTMX tables and health returned 200; `/api/repos` returned
401 without a key or with a wrong key, and 200 with the correct key for exactly
the assigned repository. Browser navigation and period selection also worked.
Real GitHub synchronization stored 14 daily traffic rows (zero traffic), with
the final observed sync taking 2.525 seconds. Star-history backfill remained 403.

The replacement used new frontend and backend containers with the same named
volume, database inode, creation time and 32,768-byte SQLite file. The canonical
repository and traffic-row hash matched before and after, retaining one repository
and 14 daily rows. No fixtures or SQL writes were used to manufacture this result.

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
delete, or change the volume path to make the check pass. After the review window,
the pilot was retired on 27 October using owner Stop and Release hardware.
The provider app remained stopped with no containers, and the same SQLite volume,
file identity and content hash survived both actions. Release cleared the project's
active provider reference while retaining the provider app and recorded binding;
future reattachment/recovery has not been tested. No data volume was deleted.
Other temporary tests should use the native lifecycle for cleanup.

Long-term history requires continued collection before GitHub's 14-day traffic
retention expires. A short pilot proves synchronization and replacement persistence,
not months of retained production history or permanent Free hosting.
