# CoreDash MCP Server

Real User Monitoring for Core Web Vitals. Query LCP, INP, and CLS field data from real visitors through the Model Context Protocol.

The MCP server is hosted at `https://app.coredash.app/api/mcp`. This repo holds the public connector docs and registry manifest. You do not need to install or run a server. You connect your MCP client directly to the hosted endpoint.

## What CoreDash is

CoreDash is a Real User Monitoring platform built around Core Web Vitals. It collects LCP, INP, and CLS from real visitors and lets you segment by page template, country, device, browser, OS, and 20 plus other dimensions. Data is stored in the EU. The product runs without consent banners. Paid plans start at $19 per month. More at [coredash.app](https://coredash.app).

## Tools exposed via MCP

| Tool | Purpose |
|---|---|
| `get_metrics` | Current performance scores at p75. Filter and group by any dimension. Use for "what is" questions. |
| `get_timeseries` | Metric over time with an `improving`, `stable`, or `regressing` summary. Use for regressions and trends. |
| `get_histogram` | Distribution shape of a single metric across roughly 40 buckets. Use to see whether traffic is bimodal, long tailed, or evenly slow. |

All three tools return rated buckets (`good`, `improve`, `poor`) matching the Core Web Vitals thresholds. Filter dimensions include device (`d`), country (`cc`), page path (`ff`), URL (`u`), browser, OS, and the LCP / INP / CLS attribution element selectors.

## Authentication

The server supports two methods:

1. **API key.** Generate one at [coredash.app/settings/api](https://coredash.app/settings/api). Pass it as `Authorization: Bearer cdk_YOUR_API_KEY`.
2. **OAuth.** Clients that follow the `WWW-Authenticate` discovery flow can authenticate without an API key. The server publishes `/.well-known/oauth-authorization-server` and `/.well-known/oauth-protected-resource`.

Anonymous calls succeed for `initialize` (so clients can introspect server capabilities) and fail for any tool call.

## Connecting

### Claude Code

```bash
claude mcp add --transport http coredash https://app.coredash.app/api/mcp \
  --header "Authorization: Bearer cdk_YOUR_API_KEY"
```

### Cursor

Add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "coredash": {
      "url": "https://app.coredash.app/api/mcp",
      "headers": {
        "Authorization": "Bearer cdk_YOUR_API_KEY"
      }
    }
  }
}
```

### VS Code

Add to your user `settings.json`:

```json
{
  "mcp.servers": {
    "coredash": {
      "type": "http",
      "url": "https://app.coredash.app/api/mcp",
      "headers": {
        "Authorization": "Bearer cdk_YOUR_API_KEY"
      }
    }
  }
}
```

### Windsurf

Same JSON shape as Cursor, in the Windsurf MCP settings.

### Any other MCP client

Point it at `https://app.coredash.app/api/mcp` with the `streamable-http` transport and the `Authorization` header above.

## Example queries

```
Why is LCP slow on mobile in Germany this week?
Has INP regressed on the product detail template in the last 24 hours?
Show me the LCP distribution on /checkout/* in the US.
```

## Registry listing

This server is published on the [Official MCP Registry](https://registry.modelcontextprotocol.io) as `io.github.corewebvitals/coredash`. From there it propagates to PulseMCP, Smithery, Glama, and Windsurf.

The registry manifest is in [`server.json`](./server.json). To publish a new version, edit the file and run `mcp-publisher publish`.

## Related projects

- [`cwv-superpowers`](https://github.com/corewebvitals/cwv-superpowers). Skills package for Claude Code, Cursor, and Gemini that drives this MCP server to diagnose and fix Core Web Vitals issues.
- [Core Web Vitals Visualizer](https://chromewebstore.google.com). Chrome extension. Lets you see CWV metrics overlayed on any page.

## License

MIT. See [LICENSE](./LICENSE).
