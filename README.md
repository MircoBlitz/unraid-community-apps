# MircoBlitz Unraid Community Applications

Community Applications templates maintained by MircoBlitz.

This repository is the single Community Applications index. It hosts the Unraid documentation and template support for Camofox Browser and NocoDB, which use official upstream images. The Camofox MCP variants and God's Eye View use separate packaging repositories.

## Applications

| Application | Template | Documentation and support |
|---|---|---|
| Camofox Browser | [`templates/camofox-browser.xml`](templates/camofox-browser.xml) | [Installation and client setup](docs/camofox/README.md), [template support](https://github.com/MircoBlitz/unraid-community-apps/issues) |
| Camofox MCP | [`templates/camofox-mcp.xml`](templates/camofox-mcp.xml) | [Adapter packaging](https://github.com/MircoBlitz/camofox-mcp) |
| Camofox MCP Standalone | [`templates/camofox-mcp-standalone.xml`](templates/camofox-mcp-standalone.xml) | [Standalone packaging](https://github.com/MircoBlitz/camofox-mcp-standalone) |
| God's Eye View | [`templates/gods-eye-view.xml`](templates/gods-eye-view.xml) | [`MircoBlitz/gods-eye-view`](https://github.com/MircoBlitz/gods-eye-view) |
| NocoDB | [`templates/nocodb.xml`](templates/nocodb.xml) | [Installation and database setup](docs/nocodb/README.md), [template support](https://github.com/MircoBlitz/unraid-community-apps/issues) |

## Camofox MCP options

Both templates use the published GHCR `0.1.0` images for x86-64 (AMD64).

- **Camofox MCP** adds Streamable HTTP at `/mcp` to an existing Camofox Browser REST server. It maps host port `9378` to container port `8080`. Set `CAMOFOX_BASE_URL` to the Browser's container DNS name and REST port on a shared user-defined Docker network, or use the Unraid host IP and mapped REST port. Do not use `localhost`: it refers to the adapter itself. Set `CAMOFOX_ACCESS_KEY` to match the Browser key, and set `CAMOFOX_API_KEY` only if you need cookie imports.
- **Camofox MCP Standalone** runs the Browser and the adapter together. Host port `9377` serves both REST and `/mcp`; do not publish the internal backend ports. Set a REST access key even if you only plan to use MCP. noVNC is off by default. To use it, set `ENABLE_VNC=1`, set a separate `VNC_PASSWORD`, and map `6080` for `/vnc.html`. Do not publish native VNC port `5900`.

For either option, generate a separate `CAMOFOX_HTTP_API_KEY` with `openssl rand -hex 32` (64 lowercase hex characters). Configure the MCP client to send it as `Authorization: Bearer <key>` to `/mcp`. Do not reuse a Browser key. Keep the ports private and limited to trusted clients; do not expose them to the public internet. Use TLS for MCP whenever the endpoint is reachable beyond loopback.

### Standalone data and migration

A fresh standalone install mounts `/mnt/user/appdata/camofox-mcp-standalone` at `/data`. The image uses the same `/data` layout as the Browser template for profiles, cookies, uploads and traces.

Only select the old Browser appdata path if you are migrating. Stop the old container first, back up its appdata, then explicitly select its existing host path in the standalone template's **Data** field. Never let both containers use the same directory at once. Keep the backup private because it may contain login credentials. The shared layout is documented in the image configuration; migration has not been tested here.

## Support

Use this repository's issue tracker for Camofox and NocoDB template and installation issues, and for shared Community Applications metadata. Report defects in the official applications to [Camofox Browser](https://github.com/jo-inc/camofox-browser/issues) or [NocoDB](https://github.com/nocodb/nocodb/issues). Use the packaging repositories linked above for Camofox MCP image issues and for God's Eye View installation, runtime and image issues.

## License

The templates and repository metadata are available under the [MIT License](LICENSE). Each application and container image retains its own upstream license and notices. See [third-party notices](THIRD_PARTY_NOTICES.md) for the copied Camofox icon.
