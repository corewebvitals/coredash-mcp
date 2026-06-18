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

1. **OAuth (recommended).** Point your MCP client at the server URL and complete the browser sign-in when prompted — no API key required. The server follows the `WWW-Authenticate` discovery flow and publishes `/.well-known/oauth-authorization-server` and `/.well-known/oauth-protected-resource`.
2. **API key.** Best for CI or headless use. Generate one at [coredash.app/settings/api](https://coredash.app/settings/api) and pass it as `Authorization: Bearer cdk_YOUR_API_KEY`.

Anonymous calls succeed for `initialize` (so clients can introspect server capabilities) and fail for any tool call.

## Connecting

### Claude Code

Add the server, then authenticate in the browser:

```bash
claude mcp add --transport http coredash https://app.coredash.app/api/mcp
```

Run `/mcp` inside Claude Code, select **coredash**, and complete the OAuth sign-in.

To use an API key instead (e.g. in CI), pass it as a header:

```bash
claude mcp add --transport http coredash https://app.coredash.app/api/mcp \
  --header "Authorization: Bearer cdk_YOUR_API_KEY"
```

### Cursor

Add to `~/.cursor/mcp.json`, then click **Login** on the server in Cursor's MCP settings to complete OAuth:

```json
{
  "mcpServers": {
    "coredash": {
      "url": "https://app.coredash.app/api/mcp"
    }
  }
}
```

To use an API key instead, add a `"headers"` block with `"Authorization": "Bearer cdk_YOUR_API_KEY"`.

### VS Code

Add to `.vscode/mcp.json` in your workspace (or the user config via the **MCP: Open User Configuration** command). VS Code prompts you to sign in on first use:

```json
{
  "servers": {
    "coredash": {
      "type": "http",
      "url": "https://app.coredash.app/api/mcp"
    }
  }
}
```

To use an API key instead, add a `"headers"` block with `"Authorization": "Bearer cdk_YOUR_API_KEY"`.

### Windsurf

Same JSON shape as Cursor, in the Windsurf MCP settings.

### Any other MCP client

Point it at `https://app.coredash.app/api/mcp` with the `streamable-http` transport. Clients that support OAuth discover the flow via the `WWW-Authenticate` header; otherwise pass an `Authorization: Bearer cdk_YOUR_API_KEY` header.

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
- [Core Web Vitals Visualizer](https://chromewebstore.google.com/detail/core-web-vitals-visualize/mcffmgagphgpgkdclllnilokablhjcge). Chrome extension. Lets you see CWV metrics overlayed on any page.

## License

MIT. See [LICENSE](./LICENSE).
