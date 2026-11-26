# popreach

See how far a K-pop act or release actually travels inside AI answer engines, without running your own citation tracking. popreach is a small forwarding client for the hosted KTRENZ MCP service, which publishes measured reach data for K-pop artists and releases: the tool list is answered from your own machine using captured definitions, and each call travels upstream, with the service's answer handed back exactly as it arrived.

All four of the service's captured tools are exposed. Find an artist by name first and keep the slug it returns, then read that artist's reach map across answer engines — which vendors cite them, whether citations come at answer time or from training, plus tracked releases and external demand from Naver and YouTube with 7-day momentum. For a wider view, list the top rising artists ranked by 7-day velocity blended from public search, upload, media and fan signals, or read a single release's measured AI-citation reach across answer engines.

These figures are the vendor's published measurements over public data about public artists and releases: descriptive reach metrics, not eligibility or risk verdicts on anyone.

## The tools

- **`search_kdata`** — Search KTRENZ K-pop / K-culture entities (artists) by `query`, a name or partial name. Returns matches with slugs and reach-map URLs; an optional `limit` (1–25, default 10) caps the results. Use a returned slug with the reach tools below.
- **`get_artist_reach`** — One artist's reach map for a `slug` from search: AI answer-engine citations (answer-time vs training), tracked releases, and external demand (Naver + YouTube) with 7-day momentum.
- **`get_trending`** — Top rising K-pop artists right now, ranked by 7-day velocity rather than standing popularity. An optional `limit` (1–30, default 15) sets how many come back. Rank is the signal.
- **`get_release_reach`** — One release's measured AI-citation reach across answer engines, for a release `slug` taken from an artist's releases (a comeback, MV, tour or chart announcement).

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/popreach mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "popreach": {
      "command": "npx",
      "args": ["-y", "@reneturcios/popreach", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/popreach
docker run --rm ghcr.io/cloudcorprecords/popreach search_kdata '{"query":"aespa"}'
```

## CLI

A single binary covers both the command line and the MCP server. Every tool is a subcommand whose one argument is a JSON string of that tool's inputs:

```bash
popreach search_kdata '{"query":"aespa","limit":10}'
popreach get_artist_reach '{"slug":"rescene"}'
popreach get_trending '{"limit":15}'
popreach get_release_reach '{"slug":"some-comeback"}'
popreach --version
```

Exit codes: `0` for a successful forwarded call, `1` when the result that comes back is an error, and `2` for usage mistakes like an unknown subcommand or a JSON argument that does not parse.

## Configuration

- `KTRENZ_MCP_URL` — Replaces the upstream endpoint. Defaults to the pinned hosted endpoint, `https://ktrenz.com/api/mcp`. (This override variable is a reconstruction introduced by this rebuild.)

The KTRENZ registry record lists no headers for its remote, so popreach never attaches an authentication header — every call goes upstream keyless.

When the upstream cannot answer — it returns a JSON-RPC error, answers with a non-2xx status, sends a body that cannot be read, or the request itself fails — the caller receives an error result whose text begins with `popreach error:`.

## How forwarding works

The first call of a process opens a session against the upstream: an `initialize` request (protocol version `2025-03-26`), a `notifications/initialized` notification, and then the `tools/call`. If the service issues an `Mcp-Session-Id` header during initialize, popreach repeats it on subsequent requests and keeps reusing that one session until the process ends. Replies are understood in both plain JSON and server-sent-events framing. Successful results are passed through untouched, `isError` flag and any extra fields included.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The suite compares the served tool list against the captured surface tool for tool, walks the session handshake and its reuse, proves no `Authorization` header is sent under any environment, and confirms that arguments travel upstream unchanged and results return unchanged for every one of the four tools.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
