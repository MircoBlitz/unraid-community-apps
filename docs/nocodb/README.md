# NocoDB on Unraid

This template uses the official [`nocodb/nocodb:latest` image](https://hub.docker.com/r/nocodb/nocodb). No custom image is built here.

NocoDB works with SQLite by default. PostgreSQL is recommended for ongoing use, but you must provide its database and user separately. The template does not install PostgreSQL.

## Install

After this template is published, install **NocoDB** through the Unraid Community Applications repository.

1. Keep the network mode set to **bridge**.
2. Map **Appdata** to persistent storage, normally `/mnt/user/appdata/nocodb`.
3. Choose an available host port. The container listens on port `8080`.
4. Generate a long random **Authentication Secret** with a password manager. Save it securely and keep it unchanged.
5. Choose the database configuration below before the first start.
6. Start the container and open its WebUI on your trusted LAN. Create the administrator account before allowing other users to reach it.

The default web address is `http://UNRAID-IP:8080/dashboard`. This template has not yet been validated on an actual Unraid installation; see [test results](test-results.md) for the checks performed.

## Database configuration

### SQLite

Leave **Database Connection** empty. NocoDB creates its SQLite database in the persistent appdata directory.

SQLite needs no separate database container and is the simplest way to try NocoDB or run a small installation. Do not store appdata only in the container's writable layer.

### PostgreSQL

Create an empty, dedicated database and its own login on a separately managed PostgreSQL server. An existing server can host it; a separate PostgreSQL container is not required for every application. Give that login ownership of the NocoDB database so NocoDB can create and migrate its schema.

Set **Database Connection** (`NC_DB`) using the upstream format:

```text
pg://HOST:5432?u=USER&p=PASSWORD&d=DATABASE
```

This is a format example, not a working credential. Replace the uppercase fields with your own values and URL-encode special characters in the query values. The connection contains credentials; do not post it in logs, screenshots, or support issues. Unraid's masked input hides it on screen but does not encrypt the saved template.

Use an address reachable from the NocoDB container. `localhost` refers to NocoDB itself, not to another container. For a database on a shared user-defined Docker network, use its container hostname; otherwise use the database server's reachable address and port. Keep PostgreSQL restricted to the required network and follow upstream guidance for TLS when using a remote database.

**Appdata remains required with PostgreSQL.** Local attachments and other application files are not all stored in the PostgreSQL database.

Changing `NC_DB` does not move an existing SQLite database into PostgreSQL. Plan and verify a separate migration instead of pointing an existing installation at an empty database.

## Access and credentials

- Keep direct HTTP access on a trusted LAN or VPN. For access from outside it, use an HTTPS reverse proxy and set **Public URL** to the externally reachable address.
- Create the initial administrator before exposing the service. Review signup and invitation settings for your intended users.
- Keep the authentication secret and database credentials stable, protected, and included in your recovery procedure.
- The optional **Connection Encryption Key** protects credentials for external data sources, not the `NC_DB` value in Unraid. Without it, upstream stores external-source credentials in plaintext in its database. Set it before adding external sources; later rotation must follow the [upstream secret CLI procedure](https://nocodb.com/docs/self-hosting/environment-variables).
- Anonymous usage telemetry is disabled by default in this template.

## Updates and backups

The image follows upstream `latest`, so ordinary Unraid image updates can advance it without a template version change. Read the [upstream release notes](https://github.com/nocodb/nocodb/releases) before updating.

Before an update, back up the saved configuration and secrets together with:

- **SQLite:** stop NocoDB and back up the entire appdata directory.
- **PostgreSQL:** stop NocoDB, take a consistent PostgreSQL backup, and back up appdata.

Database migrations can make an older image incompatible with the updated database. A rollback may require restoring the matching pre-update database and appdata, not just selecting an older image.

## Support and license

Report template or Unraid configuration problems in [this repository's issues](https://github.com/MircoBlitz/unraid-community-apps/issues). Report application defects to [NocoDB upstream](https://github.com/nocodb/nocodb/issues). Remove credentials and personal data from diagnostics.

NocoDB is distributed under its [Sustainable Use License](https://github.com/nocodb/nocodb/blob/develop/LICENSE.md), not this template repository's MIT License. Check the upstream terms for your intended use. The template references the icon hosted by the upstream project; no copy of that asset is included here.

## Sources

- [Official repository and Docker examples](https://github.com/nocodb/nocodb)
- [Self-hosting documentation](https://nocodb.com/docs/self-hosting)
- [Environment variables](https://nocodb.com/docs/self-hosting/environment-variables)
