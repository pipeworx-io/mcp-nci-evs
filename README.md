# @pipeworx/nci-evs

NCI Thesaurus (NCIt) concepts and the other controlled vocabularies served by
the NCI Enterprise Vocabulary Services REST API — the C-codes that CDISC, GDC,
CTRP, SEER and the NCI trial systems all key on.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `evs_search_concepts(term, terminology?, type?, include?, page_size?, from_record?)`
  — name → code. Searches NCIt by default; also ICD-10, MedDRA, SNOMED CT,
  HGNC, GO and others served by the same API.
- `evs_concept(code, terminology?, include?)` — code → full record: preferred
  name, definitions with source, every synonym with its term type and source,
  semantic types, and cross-terminology maps to ICD-O-3 / ICD-10 / MedDRA.
- `evs_terminologies(latest_only?)` — which vocabularies are queryable and at
  which loaded version. A concept record is only interpretable against its
  version, so this is how you pin an answer.
- `evs_concept_relations(code, terminology?, kinds?)` — parents, children,
  roles, associations, inverse associations, descendants. Use it to expand a
  disease into its subtypes before querying trial or genomic data.

## Auth

Keyless.

## Data sources

- <https://api-evsrest.nci.nih.gov/api/v1/concept/{terminology}/search> — term search.
- <https://api-evsrest.nci.nih.gov/api/v1/concept/{terminology}/{code}> — one concept.
- <https://api-evsrest.nci.nih.gov/api/v1/concept/{terminology}/{code}/{parents|children|roles|associations|inverseAssociations|inverseRoles|descendants}> — relations.
- <https://api-evsrest.nci.nih.gov/api/v1/metadata/terminologies> — loaded terminologies.

## Traps

**The API version prefix is `/api/v1`, not bare `/`.** The paths in NCI's own
prose omit it; without it every call 404s with a Spring "No static resource"
body that looks like the endpoint was withdrawn.

**Relations are separate endpoints, not an `include`.** There is no
`/relationships` path — it 404s. Parents, children, roles and associations are
each their own sub-path. `maps` is the exception: it *is* an `include` value on
the concept endpoint, not a sub-path. `evs_concept_relations` fetches the
sub-paths in parallel and reports a per-kind count, so a kind that genuinely
has no rows is distinguishable from one that failed.

**`include=minimal` is the search default and omits definitions.** A search hit
carries only code / name / status until you ask for `summary` or `full`, which
is why a "the API has no definitions" conclusion is usually an include-level
problem.

**Version strings move.** `version` on a concept (e.g. NCIt `26.08e`) is the
monthly release. Two answers taken weeks apart can legitimately differ; quote
the version when the code's meaning matters.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "nci-evs": {
      "url": "https://gateway.pipeworx.io/nci-evs/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/nci-evs/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/evs_search_concepts \
  -H 'Content-Type: application/json' \
  -d '{"term":"melanoma","page_size":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/evs_search_concepts`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-nci-evs` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/nci-evs/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "nci-evs": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-nci-evs"]
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
ask_pipeworx({ question: "your question about Nci Evs data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
