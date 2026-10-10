# Local Canvas review

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
