# 11. MCP Consumption

> How Claude Code discovers, connects to, and uses MCP (Model Context Protocol) servers.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Claude Code consumes MCP servers to extend its capabilities. MCP servers expose:

- **Tools** - Invoked via natural language
- **Resources** - Referenced with `@server:protocol://path`
- **Prompts** - Available as `/mcp__server__prompt` commands

---

## Discovery and Connection

### Server Discovery

Claude Code discovers MCP servers from:

| Source | Location |
|--------|----------|
| User config | `~/.claude/settings.json` |
| Project config | `.mcp.json` |
| Plugin config | Plugin's `.mcp.json` or `plugin.json` |
| CLI | `claude mcp add` command |

### Connection Process

1. Server configuration loaded from settings
2. Transport connection established (STDIO, HTTP, or SSE)
3. Capabilities negotiated (tools, resources, prompts)
4. Server becomes available for invocation

### Server Status

Check server status with:
- `/mcp` command in Claude Code
- `claude mcp list` in CLI

---

## Capability Types

### Tools

MCP tools are invoked through natural language:

```
User: Query the database for active users
Claude: [Invokes mcp__database__query tool]
```

| Aspect | Details |
|--------|---------|
| Discovery | Automatic from server |
| Invocation | Natural language |
| Naming | `mcp__<server>__<tool>` |

### Resources

Resources provide context via `@` syntax:

```
@myserver:file:///path/to/document
```

| Aspect | Details |
|--------|---------|
| Syntax | `@server:protocol://path` |
| Purpose | Attach context to messages |
| Discovery | Server-defined |

### Prompts

MCP prompts become slash commands:

```
/mcp__github__create_issue title description
```

| Aspect | Details |
|--------|---------|
| Format | `/mcp__<server>__<prompt>` |
| Arguments | Space-separated |
| Discovery | Server-defined |

---

## Scope Hierarchy

### Configuration Precedence

| Priority | Scope | Purpose |
|----------|-------|---------|
| Highest | Local | Personal, experimental |
| Medium | Project | Team-shared via git |
| Lowest | User | Cross-project tools |

### Scope Selection

| Requirement | Details |
|-------------|---------|
| SHOULD | Use project scope for team tools |
| SHOULD | Use local scope for sensitive credentials |
| SHOULD | Use user scope for personal utilities |

---

## Authentication

### OAuth 2.0 Flow

1. User invokes `/mcp` command
2. Selects server requiring auth
3. OAuth flow initiated
4. Token stored securely
5. Auto-refresh on expiration

### Token Management

| Action | Method |
|--------|--------|
| Authenticate | `/mcp` menu |
| Refresh | Automatic |
| Revoke | `/mcp` menu |

### Manual Tokens

Configure via environment variables:

```json
{
  "mcpServers": {
    "api": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  }
}
```

---

## Token Limits

### Output Limits

| Threshold | Value | Action |
|-----------|-------|--------|
| Warning | 10,000 tokens | Warning displayed |
| Maximum | 25,000 tokens | Output truncated |

### Configuration

```bash
export MAX_MCP_OUTPUT_TOKENS=50000
```

| Requirement | Details |
|-------------|---------|
| SHOULD | Monitor output sizes for large data servers |
| MAY | Increase limit for data-intensive operations |
| SHOULD | Stream or paginate large payloads and summarize before returning to the main conversation |
| SHOULD | Fail fast with actionable errors when a tool would exceed limits, rather than dumping partial data |

**Chunking playbook:** when a server needs to return thousands of lines, page the results, ask Claude whether more chunks are necessary, and log the retrieval range so users know how much data was consumed. Keep raw payloads in MCP resources and send lightweight summaries plus resource references back to the orchestrator.

---

## Best Practices

### Server Selection

| Requirement | Details |
|-------------|---------|
| SHOULD | Prefer STDIO for local tools (lower latency) |
| SHOULD | Prefer HTTP for remote services (scalable) |
| MUST NOT | Use SSE for new implementations (deprecated) |

### Security

| Requirement | Details |
|-------------|---------|
| MUST | Trust servers before installation |
| MUST | Review server capabilities |
| SHOULD | Use project scope for team-audited servers |
| MUST NOT | Install untrusted servers |

### Performance

| Requirement | Details |
|-------------|---------|
| SHOULD | Monitor token usage |
| SHOULD | Use local servers when possible |
| MAY | Set timeout for slow servers |

---

## Common Commands

### CLI Commands

| Command | Purpose |
|---------|---------|
| `claude mcp list` | List all servers |
| `claude mcp add` | Add new server |
| `claude mcp get <name>` | View server details |
| `claude mcp remove <name>` | Remove server |

### In-Session Commands

| Command | Purpose |
|---------|---------|
| `/mcp` | Access MCP menu |
| `/mcp__server__prompt` | Invoke MCP prompt |

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| Server not connecting | Check transport configuration |
| Auth failing | Revoke and re-authenticate via `/mcp` |
| Timeout | Set `MCP_TIMEOUT` environment variable |
| Windows npx failing | Use `cmd /c` wrapper |

### Windows Configuration

```json
{
  "mcpServers": {
    "server": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "server-package"]
    }
  }
}
```

### Timeout Configuration

```bash
export MCP_TIMEOUT=30000  # 30 seconds
```

---

## Plugin MCP Servers

### Auto-Start Behavior

| Aspect | Details |
|--------|---------|
| Startup | Automatic when plugin enabled |
| Restart | Required after config changes |
| Identification | Marked in `/mcp` output |

### Plugin vs User Servers

| Aspect | Plugin | User |
|--------|--------|------|
| Configuration | In plugin | Manual CLI |
| Startup | Automatic | Manual |
| Updates | With plugin | Independent |

---

## Integration Patterns

### Database Integration

```
User: Query the database for users created this week
Claude: I'll use the database MCP server to query for recent users.
[Invokes mcp__postgres__query]
```

### File System Integration

```
User: List all config files in the project
Claude: Let me check the file system.
[Invokes mcp__filesystem__list_directory]
```

### API Integration

```
User: Create a GitHub issue for this bug
Claude: I'll create an issue using the GitHub MCP server.
[Invokes mcp__github__create_issue]
```

---

## Server Capabilities

### Capability Discovery

Claude discovers capabilities from server metadata:

```json
{
  "capabilities": {
    "tools": true,
    "resources": true,
    "prompts": true
  }
}
```

### Resource Availability

| Requirement | Details |
|-------------|---------|
| Depends on | Server implementation |
| Discovery | Server-defined |
| Availability | May vary by server |

---

## Enterprise Considerations

### Managed Servers

Enterprise can deploy centralized MCP configurations:

```json
{
  "mcpServers": {
    "corporate-api": {
      "type": "http",
      "url": "https://internal.corp.com/mcp"
    }
  }
}
```

### Access Control

| Requirement | Details |
|-------------|---------|
| MAY | Allowlist approved servers |
| MAY | Denylist prohibited servers |
| SHOULD | Audit server usage |

---

## Related Documents

- [06-mcp-servers.md](./06-mcp-servers.md) - MCP server configuration
- [08-permissions.md](./08-permissions.md) - MCP tool permissions
- [07-plugin-manifest.md](./07-plugin-manifest.md) - Plugin MCP configuration

---

**Navigation:** [← Previous: Frontmatter](./10-frontmatter.md) | [Index](./README.md) | [Next: CLAUDE.md →](./12-claude-md.md)
