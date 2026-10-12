# Camofox update validation — 2026-10-12

The official `latest` and `1.18.1` tags resolved to the same AMD64/ARM64 manifest: `sha256:cff2c1432b609bde9c9d0b91e5b924dae0799d3c7bc9d468c0bc949325beaf2a`. The pulled ARM64 image's actual `package.json` reports `1.18.1`.

Tests ran locally on Apple silicon with Docker Desktop, using settings generated from the actual CA XML template, loopback-only host ports, isolated disposable appdata, and programmatically generated credentials in a 0700 directory/0600 file. No existing authenticated profile or personal cookie was used or imported.

Passed:

- REST health and unauthenticated-request rejection.
- Access Key rejected by the independently protected cookie-import endpoint.
- Bundled official MCP adapter starts and lists exactly 11 tools.
- Actual Camoufox tab creation, example.com navigation, DOM evaluation and accessibility snapshot with an actionable link ref.
- Own-tab cleanup confirmed by listing tabs afterward.
- A non-secret test-only localStorage marker restored after full container recreation with the same appdata mount.
- Optional noVNC mode repeats the MCP test successfully.
- `/vnc.html` is reachable with noVNC enabled.
- Native VNC listens on container `127.0.0.1:5900`; noVNC listens on container `0.0.0.0:6080`. Native VNC is not mapped to the host.
- Test containers and their Compose network removed.

The initial snapshot assertion incorrectly expected the document title in the accessibility body. Direct inspection showed the real example.com body has no corresponding heading; the assertion was corrected to its observed paragraph and link ref. No browser or upstream implementation was changed to make the check pass.

Not validated here: native AMD64 runtime, actual noVNC password login/interactive human input, personal-cookie import, or installation through Unraid's Docker Manager. Public template/icon/documentation URLs require verification after an approved publication. The recorded digest is test evidence; the installation template follows moving `latest`.
