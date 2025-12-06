# 6. MCP Servers

> External tool integrations via Model Context Protocol that extend Claude's capabilities with custom tools and data sources.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

MCP (Model Context Protocol) servers provide:

- **Custom tools** that Claude can invoke
- **Data sources** for context injection
- **External integrations** with APIs and services
- **Prompts** exposed as slash commands

Plugin MCP servers start automatically when the plugin is enabled.

---

## Configuration Methods

### Option A: Separate .mcp.json

Place at plugin root:

```json
{
  "database-tools": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
    "env": {
      "DB_URL": "${DB_URL}"
    }
  }
}
```

### Option B: Inline in plugin.json

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "mcpServers": {
    "plugin-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server",
      "args": ["--port", "8080"]
    }
  }
}
```

---

## Configuration Schema

### STDIO Transport (Local)

```json
{
  "mcpServers": {
    "local-tool": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/tool.js",
      "args": ["--flag", "value"],
      "env": {
        "CONFIG_PATH": "${CLAUDE_PLUGIN_ROOT}/config.json"
      },
      "cwd": "${CLAUDE_PLUGIN_ROOT}"
    }
  }
}
```

### HTTP Transport (Remote)

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}",
        "X-API-Key": "${API_KEY}"
      }
    }
  }
}
```

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `command` | string | STDIO | Executable path |
| `args` | array | MAY | Command arguments |
| `env` | object | MAY | Environment variables |
| `cwd` | string | MAY | Working directory |
| `type` | string | HTTP | Transport type (`http`) |
| `url` | string | HTTP | Server endpoint |
| `headers` | object | MAY | HTTP headers |

---

## Transport Types

### STDIO (Recommended for Local)

| Aspect | Details |
|--------|---------|
| Use case | Local tools, CLI applications |
| Latency | Microseconds |
| Communication | stdin/stdout |
| Best for | Development, local integrations |

```json
{
  "local-server": {
    "command": "node",
    "args": ["${CLAUDE_PLUGIN_ROOT}/server.js"]
  }
}
```

**Windows Note:** Use `cmd /c` wrapper:
```json
{
  "command": "cmd",
  "args": ["/c", "npx", "-y", "server-package"]
}
```

### HTTP (Recommended for Remote)

| Aspect | Details |
|--------|---------|
| Use case | Cloud services, remote APIs |
| Scalability | Multiple connections |
| Communication | HTTP requests, optional SSE streaming |
| Best for | Production, cloud deployments |

```json
{
  "remote-api": {
    "type": "http",
    "url": "https://api.service.com/mcp",
    "headers": {
      "Authorization": "Bearer ${TOKEN}"
    }
  }
}
```

### SSE (Deprecated)

| Aspect | Details |
|--------|---------|
| Status | Deprecated - legacy only |
| Issues | Unidirectional, resource intensive |
| Use | Only for existing servers |

---

## Scopes and Precedence

### Configuration Scopes

| Scope | Location | Visibility |
|-------|----------|------------|
| User | `~/.claude/settings.json` | All projects |
| Project (shared) | `.mcp.json` | Team via git |
| Project (local) | `.claude/settings.local.json` | Personal |
| Plugin | `plugin.json` or `.mcp.json` | With plugin |

### Precedence (Highest to Lowest)

1. Enterprise policies (`managed-settings.json`)
2. Command-line arguments
3. Local project settings
4. Shared project settings
5. User settings

### Requirements

| Requirement | Details |
|-------------|---------|
| SHOULD | Use project scope for team-shared servers |
| SHOULD | Use local scope for sensitive credentials |
| MAY | Use user scope for cross-project tools |

---

## Environment Variables

### Syntax

| Pattern | Example | Description |
|---------|---------|-------------|
| Basic | `${VAR}` | Required variable |
| Default | `${VAR:-default}` | With fallback value |
| Plugin path | `${CLAUDE_PLUGIN_ROOT}` | Plugin directory |
| Project path | `${CLAUDE_PROJECT_DIR}` | Project directory |

### Example

```json
{
  "mcpServers": {
    "database": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db",
      "env": {
        "DATABASE_URL": "${DATABASE_URL}",
        "LOG_LEVEL": "${LOG_LEVEL:-info}",
        "CONFIG": "${CLAUDE_PLUGIN_ROOT}/config.json"
      }
    }
  }
}
```

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Use environment variables for credentials |
| MUST NOT | Hardcode secrets in configuration |
| SHOULD | Provide defaults for non-sensitive values |
| SHOULD | Document required variables |

---

## Authentication

### Bearer Token

```json
{
  "secure-api": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${AUTH_TOKEN}"
    }
  }
}
```

### Custom Headers

```json
{
  "custom-auth": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "X-API-Key": "${API_KEY}",
      "X-Client-ID": "${CLIENT_ID}"
    }
  }
}
```

### OAuth 2.0

- Use `/mcp` command in Claude Code for OAuth authentication
- Tokens refresh automatically
- Revoke via `/mcp` menu

---

## Security Best Practices

### Credential Management

| Requirement | Details |
|-------------|---------|
| MUST | Store credentials in environment variables |
| MUST NOT | Commit credentials to version control |
| SHOULD | Use secrets managers in production |
| SHOULD | Implement short-lived tokens |

### Access Control

| Requirement | Details |
|-------------|---------|
| SHOULD | Implement RBAC per integration |
| SHOULD | Grant minimal required permissions |
| MAY | Use deny rules for sensitive files |

```json
{
  "permissions": {
    "deny": ["Read(./.env)", "Read(./secrets/**)"]
  }
}
```

### File Protection

Block access to sensitive files:

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./.git/**)"
    ]
  }
}
```

---

## Performance

### Token Limits

| Setting | Default | Environment Variable |
|---------|---------|---------------------|
| Warning threshold | 10,000 | - |
| Maximum | 25,000 | `MAX_MCP_OUTPUT_TOKENS` |

```bash
export MAX_MCP_OUTPUT_TOKENS=50000
```

### Transport Selection

| Use Case | Transport | Reason |
|----------|-----------|--------|
| Local tools | STDIO | Lowest latency |
| Remote APIs | HTTP | Scalable, reliable |
| Legacy servers | SSE | Compatibility only |

---

## Common Patterns

### Local Development Server

```json
{
  "mcpServers": {
    "dev-server": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/dev.js"],
      "env": {
        "DEBUG": "true",
        "PORT": "3000"
      }
    }
  }
}
```

### Remote API with Auth

```json
{
  "mcpServers": {
    "api-service": {
      "type": "http",
      "url": "https://api.service.com/mcp",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  }
}
```

### Multi-Server Setup

```json
{
  "mcpServers": {
    "database": {
      "command": "npx",
      "args": ["@mcp/postgres"],
      "env": {"DATABASE_URL": "${DB_URL}"}
    },
    "filesystem": {
      "command": "npx",
      "args": ["@mcp/filesystem", "${PROJECT_DIR}"]
    },
    "remote-api": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "headers": {"Authorization": "Bearer ${TOKEN}"}
    }
  }
}
```

---

## Examples

### Database Integration

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["@mcp/postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}",
        "MAX_CONNECTIONS": "${MAX_CONNECTIONS:-10}"
      }
    }
  }
}
```

### GitHub Integration

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://mcp.github.com/mcp",
      "headers": {
        "Authorization": "Bearer ${GITHUB_TOKEN}"
      }
    }
  }
}
```

### Custom Plugin Server

```json
{
  "mcpServers": {
    "plugin-tools": {
      "command": "${CLAUDE_PLUGIN_ROOT}/bin/server",
      "args": [
        "--config", "${CLAUDE_PLUGIN_ROOT}/config.json",
        "--verbose"
      ],
      "env": {
        "PLUGIN_HOME": "${CLAUDE_PLUGIN_ROOT}",
        "LOG_LEVEL": "${LOG_LEVEL:-warn}"
      }
    }
  }
}
```

---

## Anti-Patterns

### Don't: Hardcode Credentials

```json
// Bad: credentials in config
{
  "api": {
    "headers": {
      "Authorization": "Bearer sk-abc123..."
    }
  }
}
```

```json
// Good: use environment variables
{
  "api": {
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

### Don't: Use SSE for New Servers

```json
// Bad: deprecated transport
{
  "legacy": {
    "type": "sse",
    "url": "https://api.example.com/sse"
  }
}
```

```json
// Good: use HTTP
{
  "modern": {
    "type": "http",
    "url": "https://api.example.com/mcp"
  }
}
```

### Don't: Commit .env Files

```gitignore
# Good: gitignore sensitive files
.env
.env.*
secrets/
*.key
```

---

## Debugging

| Command | Purpose |
|---------|---------|
| `claude mcp list` | List all configured servers |
| `claude mcp get name` | View server details |
| `/mcp` | Check status in Claude Code |
| `claude mcp remove name` | Remove broken server |

---

## Related Documents

- [07-plugin-manifest.md](./07-plugin-manifest.md) - Plugin configuration
- [08-permissions.md](./08-permissions.md) - Permission model
- [11-mcp-consumption.md](./11-mcp-consumption.md) - How Claude uses MCPs

---

**Navigation:** [← Previous: Hooks](./05-hooks.md) | [Index](./README.md) | [Next: Plugin Manifest →](./07-plugin-manifest.md)
