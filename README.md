# @pipeworx/nyc-animals

Dog licensing and dog-bite records for New York City — reported bite
incidents by borough, licensed-dog breed counts, and dog lookup by name.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

- `nyc_dog_bites(borough?, since?, limit?)` — reported dog bite incidents.
  If a `since` filter would return zero rows (the dataset is published in
  periodic batches and can lag), the tool automatically retries without the
  date bound and returns a `note` explaining the fallback rather than a
  silent empty result.
- `nyc_dog_licences_by_breed(breed?, zipcode?, top?)` — licensed-dog counts
  by breed, or the most common breeds overall.
- `nyc_search_animal(name, zipcode?, limit?)` — look up a licensed dog by
  name.

## Auth

Keyless. Pass your own Socrata app token via `_apiKey` for higher rate
limits.

## Data sources

- <https://data.cityofnewyork.us/resource/nu7n-tubp.json> — NYC Dog
  Licensing Dataset.
- <https://data.cityofnewyork.us/resource/rsgh-akpg.json> — DOHMH Dog Bite
  Data.

These are two distinct datasets, not a single intake/outcome shelter feed —
NYC does not publish a live shelter intake/outcome dataset the way Austin and
Sonoma County do, so this pack's tools match licensing + bite-report data
rather than the shelter shape. The bite dataset's live max date was
2025-12-31 as of 2026-09-04 (next batch not yet posted); `nyc_dog_bites`
handles that lag internally rather than erroring or returning empty for
"this year" queries early in a new year.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "nyc-animals": {
      "url": "https://gateway.pipeworx.io/nyc-animals/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/nyc-animals/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/nyc_dog_bites \
  -H 'Content-Type: application/json' \
  -d '{"borough":"Brooklyn"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/nyc_dog_bites`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "nyc-animals": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-nyc-animals"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-nyc-animals
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Nyc Animals data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
