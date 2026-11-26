# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted KTRENZ MCP server, which publishes measured K-pop reach data for public artists and releases. All four captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `search_kdata`, `get_artist_reach`, `get_trending`, `get_release_reach` — artist search by name, one artist's measured reach across AI answer engines with tracked releases and external demand, top rising artists ranked by 7-day velocity, and one release's measured AI-citation reach.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `popreach error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `KTRENZ_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
