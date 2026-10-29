# US Code — federal statutes by citation

The text of the United States Code. Give it `18` and `1343`, get back the wire
fraud statute.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Auth

**None for retrieval.** `usc_section` and `usc_titles` are keyless.

`usc_search` uses the GovInfo API, which needs a data.gov key — Pipeworx fronts
one, or pass `_apiKey`. If you already have a citation, `usc_section` is keyless
and more direct.

## Why this pack exists when `govinfo` already reached the data

It's a reachability fix, not a data one. The US Code has always been fetchable
through the generic `govinfo` pack — but an agent asking *"what does the wire
fraud statute say"* will never call `search_packages({collections: 'USCODE'})`.
Tool selection is embedding similarity over descriptions, so a generic container
matches nothing in particular. Obvious entry points beat generic ones.

The keyless path is GovInfo's citation link service:

```
/link/uscode/18/1343  →  302 →  USCODE-2024-title18-partI-chap63-sec1343.htm
```

That matters more than it looks. The document id embeds part and chapter —
`partI-chap63` — which nobody knows from "18 U.S.C. § 1343". The link service is
the only path from the citation lawyers actually write to the document itself.

## Tools

| Tool | Key needed | Returns |
|---|---|---|
| `usc_section` | no | Full statutory text for a citation |
| `usc_titles` | no | The 54 titles and their subjects |
| `usc_search` | yes | Section headings matching a subject |

## Caveats worth passing on

- **The Code is published by annual edition**, so the text is current as of that
  edition and not as of today. A statute amended since is not reflected —
  check recent public laws for anything time-sensitive.
- **Section numbers are not contiguous.** Many are repealed or reserved, so a
  missing section is normal rather than an error.
- **Subsections are not separate documents.** Asking for `1343(a)` returns all
  of § 1343, and the response says so rather than silently widening the request.
- **`usc_search` returns headings, not section numbers.** GovInfo's search does
  not expose them, so a heading identifies a statute but cannot be cited from
  directly. The response says this instead of implying otherwise.

## Related

- `court-listener` — cases interpreting these statutes, with full opinion text
- `ecfr`, `federal-register` — regulations made under them
- `congress` — the bills that amend them

## Data source

https://www.govinfo.gov/ — United States Code, US Government Publishing Office.
Public domain.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "us-code": {
      "url": "https://gateway.pipeworx.io/us-code/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/us-code/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/usc_section \
  -H 'Content-Type: application/json' \
  -d '{"title":"18","section":"1343"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/usc_section`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-us-code` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/us-code/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "us-code": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-us-code"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Us Code data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
