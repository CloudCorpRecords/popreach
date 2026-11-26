# BUILD REPORT — popreach

Clean-room rebuild of `com.ktrenz/kdata` v1.0.0
(remote-only MCP server at `https://ktrenz.com/api/mcp`),
built 2026-10-28 as a forwarding client. Package: `@reneturcios/popreach`
0.1.0, registry name `io.github.CloudCorpRecords/popreach`.

## Surface built

**FULL surface: all 4 captured tools built**, none omitted.
**Flagged for re-triage before any publish wave** (per spec):
the surface carries the distinction recorded below, joining
awardsignal / recallpulse / safesignal / grantpulse /
sitedossier on the re-triage list.

| Tool | Required fields as served |
|------|---------------------------|
| `search_kdata` | `query` |
| `get_artist_reach` | `slug` |
| `get_trending` | *(none — no `required` key in the capture, served verbatim)* |
| `get_release_reach` | `slug` |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured. No
`inputSchema` carries a `$schema` marker or an
`additionalProperties` key — none was captured, none was
invented. Property shapes are served verbatim, including
`search_kdata`'s optional `limit` (integer, captured
description "Max results (1–25, default 10).") and
`get_trending`'s optional `limit` (integer, captured
description "How many (1–30, default 15).").
The tool literals were generated programmatically from
`harness/fixtures/ktrenz-tools-original.json` by the script
at `/tmp/laneb8-popreach-scratch/gen.py` (parse + re-serialise,
spliced into `src/forward.ts` between the TOOL_DEFS_GENERATED
markers) — never retyped — and the generator's round-trip check
confirmed the spliced block parses back deep-equal to the
fixture's 4 tool objects, in order. Tools are registered via
the MCP SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

DISTINCTION (recorded per spec): these are the vendor's
published measurements over public data about public artists
and releases — descriptive reach metrics (citations, demand,
velocity), not eligibility or risk verdicts; they gate
nothing, check no exclusion list, and concern no private
person. The surface is: search public artists by name; one
artist's measured reach across AI answer engines (which
vendors cite them, answer-time vs training) plus tracked
releases and external demand (Naver + YouTube) with 7-day
momentum; top rising artists ranked by 7-day velocity
blended from public search/upload/media/fan signals; and
one release's measured AI-citation reach.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
titles, descriptions, the `required` arrays (`query`; `slug`;
none for `get_trending`; `slug`), and the property shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://ktrenz.com/api/mcp`.
- Env override `KTRENZ_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `KTRENZ_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` /
  `KTRENZ_TOKEN` variables present.
- Error rendering `popreach error:` text for upstream failures
  — labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests, and no live call is
  claimed beyond the coordinator's tools/list capture.

## Deviations

**None.** No write-guard workaround was needed in this session:
the lockfile, `npm ci`, `tsc` build and test suite all ran
through the normal routes, and `dist/` is the direct output of
`tsc` on the validated sources.

## Lockfile

Generated clean from birth (Route 3, clean dir, no
`node_modules`, no pre-existing lockfile):
`npm install --package-lock-only --ignore-scripts --offline`
→ success. Parses as JSON; `name` == `@reneturcios/popreach`;
exactly **128** package entries; sha256
`69312b1e4baac0042cb9b91b6c011d033947667e34f155ac7e0f589cee7efa05`.
`npm ci --ignore-scripts --offline` then succeeded and the
lockfile sha256 was re-verified **unchanged** afterwards.

## server.json

- Description exactly: "K-pop reach data: artist search, AI answer-engine reach, trending velocity, release reach." — **90 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/popreach:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/popreach"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/popreach`
  and its smoke test asserts the server name `popreach`.

## Verification (run 2026-10-28, builder session)

1. `npm install --package-lock-only --ignore-scripts --offline`
   → success; lockfile parses, name `@reneturcios/popreach`,
   128 package entries (see Lockfile).
2. `npm ci --ignore-scripts --offline` → success; lockfile
   sha256 unchanged.
3. `tsc --noEmit` (direct `node_modules/.bin/tsc`) → exit 0.
4. `tsc` build → exit 0; `node dist/cli.js --version` → `0.1.0`.
5. Full suite (`tsx --test test/*.test.ts`, direct
   `node_modules/.bin/tsx`) → **65 tests, 65 pass, 0 fail**
   (registration-integrity suite deep-equals the served
   tools/list against the fixture's 4 tools, in order, at both
   the registry level and through an in-memory-transport MCP
   `tools/list`; plus per-tool schema detail assertions, session
   handling/reuse, env override, the no-Authorization-header
   assertions under every environment, verbatim argument
   forwarding for all 4 tools and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
6. Parity (harness, tools-file mode):
   `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/ktrenz-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/popreach/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/popreach/parity-out/`
   → **verdict PASS, parity 100.0** (4/4 identical, 0 missing,
   0 extra, no schema diffs). Output in `parity-out/`.
