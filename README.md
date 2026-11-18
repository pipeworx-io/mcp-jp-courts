# @pipeworx/jp-courts

Japanese court decisions from the Courts of Japan (裁判所) judgments database:
Supreme Court, high court, lower court, administrative, labour and Intellectual
Property High Court decisions, with the court's own case name, holding and
summary where it publishes them, and the judgment text.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2014+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `jp_court_search(query, must_include?, court?, date_from?, date_to?, limit?, offset?)` --
  Japanese full-text search. Each record gives the case number (for example
  平成30(受)1961), case name, decision date (ISO and as printed), court, the court's
  detail lines for the decision, the record `id` and `get_id`, and the link to the
  full text. `court` is one of `all` (default), `supreme`, `high`, `lower`,
  `administrative`, `labor`, `ip`. Pages are 30 records; `next_offset` gives the next
  page and `total` the full count.
- `jp_court_get(id, max_chars?, offset?)` -- one decision by `get_id` from the
  search: every labelled field on the record, the holding (判示事項) and summary
  (裁判要旨) where given, and the judgment text read from the court's PDF in slices.

## Auth

Keyless.

## Data sources

- Courts of Japan (裁判所), judgments database. Read live on each call; nothing is
  stored or copied. `source` names the publisher and dataset.

## Behaviour worth knowing

- The database publishes only some decisions, as the court itself says. No hit means
  not published here, not that no such case exists.
- Words in `query` separated by spaces match decisions containing ANY of them.
  `must_include` adds words that must also appear. A search matching more than 2,000
  decisions is refused by the court, counted on `query` before the court and date
  filters apply; the tool returns `found: false` with
  `empty_reason: "too_many_results"` rather than a partial list. Add specific words to
  `query` or `must_include`.
- `date_from` / `date_to` are ISO dates from 1926-12-25 (start of the Showa era).
- Results are newest first.
- Judgment text comes from the PDF. A PDF with no readable text layer returns
  `text: null` with `empty_reason: "no_text_layer"`; characters the PDF could not map
  appear as the replacement character and are counted in `unreadable_characters`.
- Parties appear as the court published them; nothing is added.
- An unknown id returns `found: false` with `empty_reason: "unknown_id"`. A result
  or detail page whose markup has changed is refused, never read as zero results.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "jp-courts": {
      "url": "https://gateway.pipeworx.io/jp-courts/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/jp-courts/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2014+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/jp_court_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"ストックオプション","court":"supreme"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/jp_court_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-jp-courts` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/jp-courts/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "jp-courts": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-jp-courts"]
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
ask_pipeworx({ question: "your question about Jp Courts data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
