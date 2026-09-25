# @pipeworx/nci-evs

NCI Thesaurus (NCIt) concepts and the other controlled vocabularies served by
the NCI Enterprise Vocabulary Services REST API — the C-codes that CDISC, GDC,
CTRP, SEER and the NCI trial systems all key on.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/evs_search_concepts \
  -H 'Content-Type: application/json' \
  -d '{"term":"melanoma","page_size":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/evs_search_concepts`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

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

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-nci-evs
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

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
