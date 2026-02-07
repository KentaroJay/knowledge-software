# MCP Servers (Model Context Protocol)

Connect Claude Code to external tools, databases, APIs, and services.

## Quick Summary

- MCP servers extend Claude's capabilities by connecting to external services
- Supports HTTP, SSE, and stdio transports
- Configured via CLI (`claude mcp add`) or `.mcp.json` file
- Three scopes: local (personal), project (team-shared), user (cross-project)
- Popular integrations: GitHub, Sentry, Notion, databases, Slack

## Context

Out of the box, Claude Code can read/write files, run commands, and search the web. But what if you want it to query your database, check Sentry for errors, or create a Jira ticket? MCP servers bridge Claude to these external tools.

## How It Works

### Architecture

```
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐
│ Claude Code  │ ──▶ │  MCP Client  │ ──▶ │  MCP Server      │
│              │     │  (built-in)  │     │  (HTTP/stdio)    │
│              │     └──────────────┘     └────────┬─────────┘
│              │                                   │
│              │                          ┌────────▼─────────┐
│              │                          │  External Service│
│              │                          │  (GitHub, DB,    │
│              │                          │   Sentry, etc.)  │
└─────────────┘                          └──────────────────┘
```

### Transport Types

| Transport | How It Works | Best For |
|-----------|-------------|----------|
| **HTTP** | Remote server over HTTPS | Cloud services (GitHub, Sentry) |
| **SSE** | Server-Sent Events (deprecated) | Legacy servers |
| **stdio** | Local process via stdin/stdout | Local tools, databases |

## Adding MCP Servers

### Via CLI

```bash
# HTTP remote server
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# With authentication header
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"

# Local stdio server
claude mcp add --transport stdio database \
  -- npx -y postgres-mcp-server

# With environment variables
claude mcp add --transport stdio airtable \
  --env API_KEY=your_key \
  -- npx -y airtable-mcp-server
```

### Via .mcp.json (Team-Shared)

Create `.mcp.json` in your project root:

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "database": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "postgres-mcp-server"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

### Scopes

```bash
# Local (default) - just for you, this project
claude mcp add --scope local ...

# Project - shared via .mcp.json in git
claude mcp add --scope project ...

# User - all your projects
claude mcp add --scope user ...
```

| Scope | Config Location | Shared? |
|-------|----------------|---------|
| Local | `~/.claude.json` | No |
| Project | `.mcp.json` | Yes (git) |
| User | `~/.claude.json` | No |

## Real-World Examples

### 1. GitHub Integration

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```

What it enables:
- "Create a PR for my changes"
- "What are the open issues labeled `bug`?"
- "Review PR #42"

### 2. Database Access (Read-Only)

```json
{
  "mcpServers": {
    "analytics-db": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "postgres-mcp-server"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL:-postgresql://readonly:pass@localhost/analytics}"
      }
    }
  }
}
```

What it enables:
- "What were the top 10 errors last week?"
- "Show me the user growth trend"
- "Query the orders table for January"

### 3. Sentry Error Monitoring

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp \
  --header "Authorization: Bearer ${SENTRY_TOKEN}"
```

What it enables:
- "What's causing the most errors in production?"
- "Show me the stack trace for issue PROJ-123"
- "Find and fix the null pointer exception users are hitting"

## Managing MCP Servers

```bash
claude mcp list                    # List all configured servers
claude mcp get github              # Get details for a server
claude mcp remove github           # Remove a server
claude mcp reset-project-choices   # Reset OAuth/approval choices
```

Inside Claude Code:
```
/mcp                               # Interactive MCP management
```

## Environment Variables in .mcp.json

```json
{
  "mcpServers": {
    "api": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

- `${VAR}` - Use environment variable
- `${VAR:-default}` - Use default if not set

## OAuth Authentication

Some MCP servers support OAuth for browser-based login:

```bash
claude mcp add --transport http \
  --client-id your-client-id \
  --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

Then use `/mcp` inside Claude Code to complete the browser login flow.

## Key Takeaways

- MCP servers extend Claude to work with any external service
- Use HTTP transport for cloud services, stdio for local tools
- Share team configurations via `.mcp.json` in git
- Keep secrets in environment variables, never hardcode them
- Use `--scope project` for team-shared servers
- `/mcp` command manages servers interactively inside Claude Code

## Common Pitfalls

- **Hardcoded secrets** - Always use `${ENV_VAR}` in `.mcp.json`, never raw tokens
- **Missing dependencies** - stdio servers often need `npx` or `pip` packages installed
- **Wrong scope** - Project scope writes to `.mcp.json` which gets committed to git
- **Stale OAuth tokens** - Use `claude mcp reset-project-choices` if auth expires
- **Too many servers** - Each connected server adds startup latency; only add what you use

## Further Reading

- [MCP Documentation](https://docs.anthropic.com/en/docs/claude-code/mcp)
- [MCP Server Registry](https://github.com/modelcontextprotocol/servers)
- [Model Context Protocol Spec](https://spec.modelcontextprotocol.io/)

---

*Created: February 2026*
*Project Context: dev-learning-notes repository setup*
