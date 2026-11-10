# storedata

storedata is a small local bridge to the hosted HasData
Shopify MCP server, which returns public Shopify
storefront data as JSON. Point it at a storefront URL
and it lists that store's collections — id, title,
handle, description and image for each one — and it
pulls the store's products, optionally narrowed to a
single collection by handle. Both reads page with a
`limit` (up to 250) and a `page` number.

The bridge keeps no catalogue of its own. It answers
the tool list locally, exactly as captured from the
live endpoint, and forwards each tool call upstream,
returning whatever comes back untouched. The captured
surface is two read-only fetches over public
storefronts: there is no cart, checkout, order or
payment tool in it, and no store-owner contact data
is returned.

## Tools

- `hasdata_shopify_collections_getCollections` — list
  a storefront's collections. Required: `url` (the
  Shopify store URL). Optional: `limit` and `page`
  for pagination. Returns each collection's id,
  title, handle, description, image and timestamps.
  The returned handles feed the products tool when
  you want one category at a time.
- `hasdata_shopify_products_getProducts` — pull a
  storefront's products. Required: `url`. Optional:
  `collection` (a collection handle to filter by),
  plus `limit` and `page`. Returns product id, title,
  handle, vendor, product type, tags, body HTML,
  images, variants with prices, SKUs and inventory
  status, and timestamps.

## Configuration

- `HASDATA_SHOPIFY_MCP_URL` (optional) — override
  the upstream endpoint. Defaults to
  `https://mcp.hasdata.com/mcp?apis=shopify`. (The
  override hook is this rebuild's own labelled
  reconstruction.)
- `HASDATA_API_KEY` (optional) — when set, every
  upstream request carries an `x-api-key: <key>`
  header, for the caller's own HasData API key. When
  it is not set, no key header is sent. (The header
  name and the env name are pinned by the original's
  registry record; forwarding the key from this env
  var is a labelled reconstruction — the original
  surface was captured keyless.)

### Forwarding mechanics

On the first tool call the client opens a streamable
HTTP session upstream: a JSON-RPC `initialize` request
(protocolVersion `2025-03-26`, clientInfo name
`storedata`), then a `notifications/initialized`
notification, then the `tools/call` request carrying
the caller's arguments unchanged. When the upstream
answers `initialize` with an `Mcp-Session-Id` header,
that value rides on every later request of the
session, which is reused for the life of the process.
Replies may arrive as plain JSON or as server-sent
events; both are read.

Successful upstream results pass through verbatim,
including any `isError` flag the upstream sets.
Failures — a JSON-RPC error, a non-2xx status, an
unreadable response, or a request that cannot be
completed — come back as an `isError` result whose
text starts with `storedata error:` (this rendering
is this rebuild's own labelled reconstruction).

## CLI

Each tool is also a subcommand taking one JSON
argument string:

```bash
npx @reneturcios/storedata hasdata_shopify_collections_getCollections '{"url":"https://b2bdemoexperience.myshopify.com"}'
npx @reneturcios/storedata hasdata_shopify_products_getProducts '{"url":"https://b2bdemoexperience.myshopify.com","collection":"frontpage"}'
npx @reneturcios/storedata --version
```

Exit codes: `0` on success, `1` on an `isError`
result, `2` on a usage error.

## MCP server

```bash
npx -y @reneturcios/storedata mcp
```

MCP client config:

```json
{
  "mcpServers": {
    "storedata": {
      "command": "npx",
      "args": ["-y", "@reneturcios/storedata", "mcp"]
    }
  }
}
```

## Docker

```bash
docker run -i ghcr.io/cloudcorprecords/storedata
docker run --rm ghcr.io/cloudcorprecords/storedata --version
```

The container's default command is `mcp`, so the first
form is a stdio MCP server; any other arguments reach
the CLI normally.

## Development

```bash
npm install
npm run build
npm test
npm run dev
```

The test suite runs entirely against a stubbed
JSON-RPC upstream — no live network calls.

## License

MIT — see `LICENSE`.
