# Camofox Browser on Unraid

This template runs the official [Camofox Browser](https://github.com/jo-inc/camofox-browser) image. It follows `ghcr.io/jo-inc/camofox-browser:latest`, rather than fixing an old application version. Version 1.18.1 was locally tested on 2026-10-12; see [test results](test-results.md).

## Install

Use [`templates/camofox-browser.xml`](../../templates/camofox-browser.xml) through Community Applications. The public template URL is:

```text
https://raw.githubusercontent.com/MircoBlitz/unraid-community-apps/main/templates/camofox-browser.xml
```

For a manual installation, save the XML as `/boot/config/plugins/dockerMan/templates-user/my-camofox-browser.xml`, then choose **Docker → Add Container → User Templates → Camofox-Browser**.

Before applying the template:

1. Generate two separate long random keys locally using a secret manager or cryptographic random generator. Never send them through chat.
2. Enter the normal REST/MCP key as **Access Key** (`CAMOFOX_ACCESS_KEY`).
3. Enter the separate cookie-import key as **Cookie API Key** (`CAMOFOX_API_KEY`). Give it only to clients allowed to import cookies.
4. Confirm the appdata directory, normally `/mnt/user/appdata/camofox-browser`.
5. Keep noVNC disabled unless an interactive desktop is needed.
6. Keep mapped ports restricted to a trusted LAN or VPN.

The unauthenticated `/health` endpoint checks service readiness, not every browser capability. Normal REST calls require bearer authentication. The template is non-privileged, uses 1 GiB shared memory and `no-new-privileges`, and disables crash telemetry.

## Connect an MCP client

Use the official standalone package **`@askjo/camofox-browser-mcp`**, without a fixed package version. It runs locally on each client and connects to the Unraid REST service. Follow the [upstream MCP guide](https://github.com/jo-inc/camofox-browser/blob/main/mcp/README.md) and your client's native MCP configuration and secret-handling features.

| Setting | Purpose |
|---|---|
| `CAMOFOX_BASE_URL` | `http://UNRAID-IP:9377` |
| `CAMOFOX_USER_ID` | Stable identity for this client, such as `workstation-pi` |
| `CAMOFOX_ACCESS_KEY` | Normal REST/MCP bearer authentication |
| `CAMOFOX_API_KEY` | Optional cookie-import authorization; omit unless needed |

Store credentials through the client's supported secure mechanism, with restrictive permissions. Do not commit credentials or place them in public/shared configuration. No custom client wrapper or portable assistant skill is supplied by this template repository; client behavior belongs to the official adapter and the client's documented native contract.

Different user IDs isolate persisted profiles. Deliberately shared IDs share browser state; avoid simultaneous human/agent changes in one tab. After manual navigation or input, refresh the current DOM or snapshot before reusing element references.

To validate a new connection, create one task-owned tab, inspect its actual DOM and snapshot, close it, and verify its absence. Do not close another user's tabs. A service-only health response is not a complete MCP/browser test.

## Optional noVNC

Set an eight-character random ASCII **noVNC Password**, then set **Enable noVNC** to `1`. Leave **noVNC Bind Address** at `0.0.0.0` inside Docker so port mapping works. Open:

```text
http://UNRAID-IP:6080/vnc.html
```

The REST key does not protect noVNC. Traffic is unencrypted by default, and classic VNC authentication uses only the first eight password characters. Keep it on the LAN/VPN. `View Only=1` disables human keyboard/mouse input.

As of 1.18.1, `VNC_BIND` controls noVNC only; native VNC stays on container loopback by default. This template does not expose native VNC port 5900. Do not change `VNC_RFB_BIND` to a broad address without a separately reviewed need and password.

## Persistence, updates and removal

Appdata can contain cookies, localStorage, uploaded files and traces. Protect its permissions and backups as credential-bearing data. Unraid's saved container configuration and flash backups can also contain the configured keys.

Before updating, close active sessions, stop the container, back up appdata and record the running image version/digest. Read upstream release notes, update the image, and verify MCP, browser startup, persisted profiles and any enabled noVNC service. A moving `latest` tag receives normal upstream releases; a digest recorded in test results is evidence, not an installation pin.

For rollback, use the previously recorded image temporarily and restore appdata if upstream changed its format. Removing the container should leave appdata intact; deleting appdata separately removes the stored sessions.

## Troubleshooting and support

- **Health works but tools fail:** verify the client's Access Key, URL and network route. Cookie import needs its own key.
- **Old element refs fail:** inspect the current DOM/snapshot after navigation or rerendering.
- **noVNC unreachable:** enable it, set the password, leave its container bind at `0.0.0.0`, and check port mapping and LAN/VPN reachability.
- **Profiles disappear:** verify the `/data` mount and profile directory; a working health endpoint alone does not establish persistence.
- **Browser crashes:** inspect resource limits and shared memory before changing the template's safety settings.

Report template/installation issues to [this repository's issue tracker](https://github.com/MircoBlitz/unraid-community-apps/issues), without credentials, cookies or authenticated profile data. Report reproducible upstream browser defects to [Camofox](https://github.com/jo-inc/camofox-browser/issues). For security-sensitive reports use GitHub's private reporting channel if it is enabled; do not assume it is enabled.

The fox icon is copied unchanged from upstream under its MIT license; see [third-party notices](../../THIRD_PARTY_NOTICES.md).
