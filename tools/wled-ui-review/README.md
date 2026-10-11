# Local Canvas review

## Functional dark UI

The production page is `/control`. These two development servers use only Node
built-ins and bind to loopback:

```sh
# Controller fixture; no hardware requests or writes.
node tools/wled-ui-review/device-server.mjs
# http://localhost:8096/control

# Actual device controls through an already established loopback tunnel.
# Pass that tunnel's local port explicitly.
node tools/wled-ui-review/live-server.mjs 18086
# http://localhost:8097/control
```

The live server serves the new UI assets from this checkout and forwards the
remaining HTTP/WebSocket requests to the supplied tunnel. Opening the page reads
state; using its controls changes the actual device. Native settings pages come
from the connected device. Neither server flashes firmware.

The fixture marks itself as simulated, supplies synthetic live pixels and
implements state/preset/playlist requests for development. Its settings pages
are form/navigation fixtures, not a complete firmware configuration emulator.
`/__sim` can inject offline responses, rejected commands or acknowledged no-ops.
See `docs/control-ui.md` for production scope and qualification.

## Earlier design prototype

Development-only Agentation setup for the static WLED prototype. It does not
modify the firmware build, contact the controller, or register global MCP servers.

```sh
cd tools/wled-ui-review
npm ci
npm start
```

Open http://localhost:8095/?variant=1. The prototype server binds to loopback on
8095; Agentation's HTTP server binds to loopback on 4747. The toolbar is injected
only by this server, and its bundle is generated in memory. The ordinary preview
on 8094 and the inline comparison have no React or Agentation dependency.
Server feedback lives in memory; the browser may retain its own review notes.

Use the official MCP client helper to read and resolve feedback:

```sh
node mcp-call.mjs agentation_get_all_pending '{}'
node mcp-call.mjs agentation_get_session '{"sessionId":"..."}'
node mcp-call.mjs agentation_acknowledge '{"annotationId":"..."}'
node mcp-call.mjs agentation_resolve '{"annotationId":"...","summary":"Verified change"}'
```

The helper launches the MCP transport with `--mcp-only`, connecting to the
existing HTTP service rather than starting a second service on 4747. Its `list`
command prints the server's actual tool schemas.

The self-driving review used the official
[Agentation skill](https://github.com/benjitaylor/agentation/blob/main/skills/agentation-self-driving/SKILL.md),
an isolated headed `agent-browser` session, annotations created through the
visible toolbar, MCP acknowledgement, source edits, browser verification, and
MCP resolution. Start an isolated visible browser with:

```sh
node_modules/.bin/agent-browser --session wled-canvas-review --headed \
  --executable-path '/Applications/Google Chrome.app/Contents/MacOS/Google Chrome' \
  open 'http://localhost:8095/?variant=1'
```

The October 10 review resolved four annotations, including Steven's request to
replace the visible scene caption with an info button. The captured session is
in `agentation-review.json`. The remaining designs stay available for comparison.
Production implementation and qualification remain separate, and main still
requires Steven's approval after that testing.

Canvas's effect thumbnails use the actual reference animations linked from
[WLED's effect documentation](https://kno.wled.ge/features/effects/). They keep
the published reference colors, generally Party, rather than simulating the
currently selected palette or speed. Solid uses the selected primary color.
The picker animates the hovered/focused effect; the selected effect also animates.
Reduced motion uses still frames. All assets are embedded, with no runtime
network requests for images. The upstream MIT notice is retained in
`WLED-Utils-LICENSE.txt` and the embedded data.

The 72 built-in palette previews are extracted from this checkout's
`palettes.cpp` and `JSON_palette_names`, following `/json/palx` and
`genPalPrevCss`. Color-derived palettes use the preview's primary color and
black secondary/tertiary defaults. Random Cycle shows a stable random sample.
Default follows the effect's default palette. Custom/device-specific palettes
require a reachable controller. The scene is still a local design simulation.

To rebuild the embedded reference data and Canvas preview code, use Python with
Pillow installed in a development environment:

```sh
python build-reference-previews.py
```

The builder pins the upstream animation commit, records original GIF checksums,
keeps the complete frame without cropping, and preserves elapsed loop time
while sampling every third frame. It updates `reference-previews.json` and the
self-contained gallery, leaving the other design definitions unchanged.
