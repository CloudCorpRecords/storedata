# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted HasData Shopify MCP server, serving the two captured tools locally for tools/list and forwarding every tools/call upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when issued).
- Tools: `hasdata_shopify_collections_getCollections` and `hasdata_shopify_products_getProducts`.
- Upstream results pass through verbatim; upstream failures surface as `storedata error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). Upstream endpoint overridable via `HASDATA_SHOPIFY_MCP_URL`; when `HASDATA_API_KEY` is set it is sent as an `x-api-key` header.
