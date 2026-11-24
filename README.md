# @pipeworx/denmark-consultations

Danish public consultations (hoeringer): the draft bills and executive orders that every ministry and agency publish for comment before they are decided, with their deadlines, documents and, once the deadline has passed, the responses. It is the earliest public signal of upcoming Danish legislation.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2037+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `list_consultations(text?, authority?, area?, type?, year?, status?, deadline_from?, deadline_to?, limit?, skip?)` — consultations open now by default (soonest deadline first), or expired / archived / any, filtered by ministry or agency, subject area, type, year, free text and a deadline window. Returns the total matching count.
- `get_consultation(id)` — one consultation: description, authority, type, area, status, publication date, deadline and its remarks, archive and effective dates, the contact unit, and every document with group, size and link.
- `list_responses(id)` — the published response documents, the ministry's summary note, and the organisations and public authorities that submitted comments as named in that note. Says when the consultation is still open, when no responses are published, and when the note carries no list of respondents.
- `recent_consultations(days?, kind?, limit?)` — consultations opened (published) or closed (deadline passed) in the last N days.

## Auth

Keyless.

## Data sources

- Civilstyrelsen, Høringsportalen (the Danish government consultation portal). Public pages, no login. The portal has no API, so the pack reads its public pages and the ministries' published summary notes, one call at a time with an identifying User-Agent.

The portal publishes the responses to most consultations as one compiled document, not one file per respondent. The list of responding organisations is therefore read from the ministry's own summary note, and only where that note carries a list of those who submitted comments; otherwise `list_responses` says so and gives the number of responses the note states.

## Personal data

- Responding organisations and public authorities are named.
- A respondent whose name has the shape of a private person's name is counted in `private_individual_respondents` and never named. The test is deliberately conservative: it can withhold an organisation whose name looks like a person's, never the reverse.
- The ministry's contact person (name and e-mail) is not served; only the unit they sit in.
- Serving private individuals' names would need its own ruling.

## Notes

- Dates are ISO (YYYY-MM-DD); the portal writes DD-MM-YYYY.
- A deadline window is not a portal filter: the pack reads pages in deadline order and cuts, up to 12 pages per call.
- Text is Danish exactly as the portal publishes it.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "denmark-consultations": {
      "url": "https://gateway.pipeworx.io/denmark-consultations/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/denmark-consultations/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2037+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/list_consultations \
  -H 'Content-Type: application/json' \
  -d '{"status":"open","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/list_consultations`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-denmark-consultations` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/denmark-consultations/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "denmark-consultations": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-denmark-consultations"]
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
ask_pipeworx({ question: "your question about Denmark Consultations data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
