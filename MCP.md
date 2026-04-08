# Connecting AI Agents to Your Uniqorn APIs

Every endpoint you deploy on Uniqorn is automatically exposed as an MCP ([Model Context Protocol](https://modelcontextprotocol.io)) tool. No extra configuration, no annotations, no separate setup. Push your code, and AI agents can call it.

This means you can use AI to **build** an endpoint and then have AI agents **use** that endpoint — all on the same platform.

## How it works

When your Uniqorn instance starts, it registers all deployed endpoints as MCP tools. Each endpoint's HTTP method, URL, description, and parameter schema become a tool definition that any MCP-compatible AI client can discover and call.

For example, if you deploy an endpoint:
- **GET /api/users** with a `limit` parameter

An AI agent connected to your instance sees a tool called `GET /api/users`, knows it accepts a `limit` argument, and can call it directly.

As you deploy new endpoints or update existing ones, the MCP tool list updates automatically.

## Authentication

MCP access requires a Bearer token. You have two options:

1. **Generate an MCP token** from the Uniqorn management panel
2. **Use an existing API consumer key**

The token is passed via the `Authorization` header when connecting. All tool calls execute with the permissions of the authenticated user — the same security policies that apply to regular API calls apply to MCP calls.

## Connecting your AI client

Replace `YOUR_INSTANCE` and `YOUR_TOKEN` in all examples below.

---

### Claude Code

```bash
claude mcp add --transport http uniqorn https://YOUR_INSTANCE.uniqorn.dev/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"
```

### Claude Desktop

Edit `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "uniqorn": {
      "type": "http",
      "url": "https://YOUR_INSTANCE.uniqorn.dev/mcp",
      "headers": { "Authorization": "Bearer YOUR_TOKEN" }
    }
  }
}
```

### VS Code (GitHub Copilot)

Create `.vscode/mcp.json` in your project:

```json
{
  "servers": {
    "uniqorn": {
      "type": "http",
      "url": "https://YOUR_INSTANCE.uniqorn.dev/mcp",
      "headers": { "Authorization": "Bearer YOUR_TOKEN" }
    }
  }
}
```

### Cursor

Create `.cursor/mcp.json` in your project:

```json
{
  "mcpServers": {
    "uniqorn": {
      "type": "streamableHttp",
      "url": "https://YOUR_INSTANCE.uniqorn.dev/mcp",
      "headers": { "Authorization": "Bearer YOUR_TOKEN" }
    }
  }
}
```

### Windsurf

Edit `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "uniqorn": {
      "serverUrl": "https://YOUR_INSTANCE.uniqorn.dev/mcp",
      "headers": { "Authorization": "Bearer YOUR_TOKEN" }
    }
  }
}
```

### JetBrains (IntelliJ, WebStorm, etc.)

**Settings > Tools > AI Assistant > Model Context Protocol (MCP)** > Add server:

- URL: `https://YOUR_INSTANCE.uniqorn.dev/mcp`
- Header: `Authorization` = `Bearer YOUR_TOKEN`

### Amazon Q Developer

Edit `~/.aws/amazonq/mcp.json`:

```json
{
  "mcpServers": {
    "uniqorn": {
      "type": "http",
      "url": "https://YOUR_INSTANCE.uniqorn.dev/mcp",
      "headers": { "Authorization": "Bearer YOUR_TOKEN" }
    }
  }
}
```

### Cline

Open the MCP Servers panel in Cline and edit the config:

```json
{
  "mcpServers": {
    "uniqorn": {
      "type": "streamableHttp",
      "url": "https://YOUR_INSTANCE.uniqorn.dev/mcp",
      "headers": { "Authorization": "Bearer YOUR_TOKEN" }
    }
  }
}
```

### OpenClaw

Add to your `SOUL.md` MCP configuration:

```json
{
  "mcpServers": {
    "uniqorn": {
      "type": "streamableHttp",
      "url": "https://YOUR_INSTANCE.uniqorn.dev/mcp",
      "headers": { "Authorization": "Bearer YOUR_TOKEN" }
    }
  }
}
```

---

### Automation platforms

These platforms can connect to your Uniqorn MCP server, making your endpoints available as steps in automation workflows.

**Zapier** — Use the [MCP client action](https://zapier.com) to connect to `https://YOUR_INSTANCE.uniqorn.dev/mcp` with your Bearer token.

**n8n** — Add an **MCP Client** tool node pointing to `https://YOUR_INSTANCE.uniqorn.dev/mcp` with the `Authorization: Bearer YOUR_TOKEN` header.

**Make** — Use an HTTP module to send JSON-RPC requests to `https://YOUR_INSTANCE.uniqorn.dev/mcp` with the `Authorization: Bearer YOUR_TOKEN` header.

---

### Any other MCP client

Point any MCP-compatible client to:

```
POST https://YOUR_INSTANCE.uniqorn.dev/mcp
Authorization: Bearer YOUR_TOKEN
Content-Type: application/json
```

Uniqorn also supports the older SSE transport at `/sse` for clients that require it.

## What gets exposed

Every enabled endpoint in your workspaces becomes an MCP tool. The tool definition includes:

- **Name**: the HTTP method and path (e.g. `GET /api/users`)
- **Description**: from your endpoint's `summary()` and `description()`
- **Input schema**: generated from your `parameter()` definitions

Write good summaries and parameter descriptions — they help AI agents pick the right tool and pass the right arguments.

## Security considerations

- MCP access requires authentication. Anonymous requests are rejected.
- Tool calls run with the permissions of the token owner. Use scoped tokens with minimal privileges.
- The same sandbox that protects your endpoint code applies: no filesystem access, no reflection, no socket opening.
- Consider creating dedicated API consumer keys for AI agents rather than sharing personal tokens.
