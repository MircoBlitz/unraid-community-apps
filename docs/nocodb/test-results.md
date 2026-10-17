# NocoDB test results

Tested on 2026-10-17. SQLite and PostgreSQL functional tests used Docker Desktop with a Linux/arm64 engine. A separate, narrower startup/HTTP check used an actual Unraid deployment.

## Image tested

- Image: `nocodb/nocodb:latest`
- Observed registry digest: `sha256:27d2fd14670db892ac721849d4f071edce47e386a1e74c921a0162644b9c213d`
- Local image architecture: `arm64`
- NocoDB package version in the image: `2026.09.1`
- Declared container port: `8080/tcp`
- The image does not define a Docker healthcheck. The running application provided `GET /api/v1/health`, which returned HTTP 200 with JSON containing `"message":"OK"`, a timestamp, and uptime.

`latest` is a mutable tag. The digest and version above describe the image pulled for this test only.

## SQLite default

The container was started with the template's default variable set and runtime options:

- a securely generated, non-empty `NC_AUTH_JWT_SECRET`
- `NC_DB=`
- `NC_PUBLIC_URL=`
- `NC_CONNECTION_ENCRYPT_KEY=`
- `NC_DISABLE_TELE=true`
- `/usr/app/data` mounted read-write
- container port 8080 published only on `127.0.0.1`
- Docker `--init` and `--stop-timeout=30`

Results:

- Empty `NC_DB`, `NC_PUBLIC_URL`, and `NC_CONNECTION_ENCRYPT_KEY` did not prevent startup.
- `/api/v1/health` returned HTTP 200.
- `/` returned HTTP 200, while `/dashboard` returned the application's HTTP 302 redirect.
- NocoDB created `/usr/app/data/noco.db` in the mounted appdata directory.
- SQLite reported `ok` from `PRAGMA integrity_check`.
- The database contained 179 tables, including 173 `nc_*` NocoDB tables.
- The same container became healthy after a Docker restart.
- A new container using the same appdata became healthy after the first container was removed. The SQLite file inode and schema fingerprint were unchanged, and the integrity check still returned `ok`.

A separate run omitted `NC_DB` entirely. NocoDB again started with SQLite, returned HTTP 200 from the health endpoint, created a valid database, and created 173 `nc_*` tables. This confirms both the documented fallback and the template's empty optional value behavior for the tested image.

## External PostgreSQL

A separate disposable `postgres:latest` container supplied the database over a dedicated Docker network. NocoDB received the connection through `NC_DB`; the other template variables and Docker runtime options matched the SQLite test.

Results:

- PostgreSQL became ready before NocoDB was started.
- NocoDB returned HTTP 200 from `/api/v1/health` with `"message":"OK"`.
- `/` returned HTTP 200, while `/dashboard` returned HTTP 302.
- PostgreSQL contained 178 public-schema tables, including 173 `nc_*` NocoDB tables.
- No SQLite `noco.db` file was created in PostgreSQL mode.

This validates the template's separate-PostgreSQL connection model. It does not install or manage PostgreSQL as part of the NocoDB container.

## Unraid deployment check

The template was staged on the owner's Unraid host. Its XML parsed successfully with Unraid's PHP XML parser, and its checksum matched the reviewed source at staging time.

Subsequent read-only checks on that host confirmed:

- The `NocoDB` container was running with `nocodb/nocodb:latest`.
- Container port `8080/tcp` was published on host port `8080`.
- `GET /api/v1/health` returned HTTP 200 with `"message":"OK"`.
- `GET /` returned HTTP 200.

The deployed database mode, credentials, browser onboarding, and persistence across restart/recreation were not inspected or tested on Unraid. PostgreSQL and lifecycle evidence above comes from the separate local tests. This check does not establish that Community Applications has indexed the newly published template.

## Test hygiene and limits

Random PostgreSQL credentials and authentication secrets were generated programmatically, kept in files with mode `0600` inside a mode `0700` directory, never printed, and removed after testing. All test containers, the dedicated network, the PostgreSQL volume, and disposable appdata were removed. No existing containers were changed.

Not tested:

- Unraid database operations, restart/recreation, or public Community Applications installation path
- explicit cross-architecture runtime coverage beyond the local arm64 tests
- first-user onboarding or browser interactions
- reverse-proxy behavior, upgrades, backup and restore, load, or long-running operation
