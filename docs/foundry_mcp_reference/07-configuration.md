# Configuration

> Environment variables, TOML configuration, and Claude Code setup.

---

## Configuration Hierarchy

foundry-mcp loads configuration from multiple sources in order of priority:

```
1. Environment Variables    (highest priority - overrides all)
       ↓
2. TOML Configuration      (foundry-mcp.toml)
       ↓
3. Default Values          (built-in defaults)
```

Higher priority sources override lower ones.

---

## Environment Variables

### Core Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `FOUNDRY_MCP_SPECS_DIR` | Path to specs directory | Auto-detected from workspace |
| `FOUNDRY_MCP_LOG_LEVEL` | Logging level (DEBUG, INFO, WARNING, ERROR) | `INFO` |
| `FOUNDRY_MCP_WORKFLOW_MODE` | Execution mode: `single`, `autonomous`, `batch` | `single` |
| `FOUNDRY_MCP_CONFIG_FILE` | Path to TOML configuration file | Auto-detected |

### Security Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `FOUNDRY_MCP_API_KEYS` | Comma-separated API keys for authentication | Disabled |
| `FOUNDRY_MCP_WORKSPACE_ROOTS` | Allowed workspace paths | Current directory |
| `FOUNDRY_MCP_RATE_LIMIT` | Rate limit per minute | 100 |

### Feature Flags

| Variable | Description | Default |
|----------|-------------|---------|
| `FOUNDRY_MCP_FEATURE_FLAGS` | Comma-separated enabled flags | Based on rollout |
| `FOUNDRY_MCP_RESPONSE_CONTRACT` | Force response contract version | Auto-negotiated |

### LLM Provider Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `OPENAI_API_KEY` | OpenAI API key | Not set |
| `ANTHROPIC_API_KEY` | Anthropic (Claude) API key | Not set |
| `GEMINI_API_KEY` | Google Gemini API key | Not set |
| `FOUNDRY_MCP_LLM_PROVIDER` | Default provider | Auto-detected |
| `FOUNDRY_MCP_LLM_TIMEOUT` | LLM request timeout (seconds) | 30 |

### Example

```bash
export FOUNDRY_MCP_SPECS_DIR="/path/to/project/specs"
export FOUNDRY_MCP_LOG_LEVEL="DEBUG"
export FOUNDRY_MCP_WORKFLOW_MODE="autonomous"
export ANTHROPIC_API_KEY="sk-ant-..."
```

---

## TOML Configuration

Create `foundry-mcp.toml` in your project root for persistent settings.

### Complete Example

```toml
# Workspace configuration
[workspace]
specs_dir = "./specs"
workspace_roots = ["/path/to/allowed/dirs"]

# Logging configuration
[logging]
level = "INFO"
structured = true
include_timestamps = true

# Workflow configuration
[workflow]
mode = "single"
auto_validate = true
journal_enabled = true
context_limit_percent = 85

# LLM provider configuration
[llm]
provider = "anthropic"
model = "claude-3-sonnet"
timeout = 30
fallback_provider = "openai"

# Security configuration
[security]
require_api_key = false
allowed_keys = []
rate_limit = 100
workspace_scoping = true

# Feature flags
[feature_flags]
enabled = [
    "response_contract_v2",
    "environment_tools",
    "spec_helpers",
    "planning_tools"
]
```

### Configuration Sections

#### [workspace]

| Key | Type | Description |
|-----|------|-------------|
| `specs_dir` | string | Path to specs directory |
| `workspace_roots` | array | Allowed workspace paths |

#### [logging]

| Key | Type | Description |
|-----|------|-------------|
| `level` | string | Log level (DEBUG, INFO, WARNING, ERROR) |
| `structured` | bool | Use structured JSON logging |
| `include_timestamps` | bool | Include timestamps in logs |

#### [workflow]

| Key | Type | Description |
|-----|------|-------------|
| `mode` | string | Workflow mode: single, autonomous, batch |
| `auto_validate` | bool | Validate specs on load |
| `journal_enabled` | bool | Enable automatic journaling |
| `context_limit_percent` | int | Context usage threshold (for autonomous) |

#### [llm]

| Key | Type | Description |
|-----|------|-------------|
| `provider` | string | Default provider (anthropic, openai, gemini) |
| `model` | string | Model identifier |
| `timeout` | int | Request timeout in seconds |
| `fallback_provider` | string | Fallback if primary fails |

#### [security]

| Key | Type | Description |
|-----|------|-------------|
| `require_api_key` | bool | Require API key for access |
| `allowed_keys` | array | Valid API keys |
| `rate_limit` | int | Requests per minute |
| `workspace_scoping` | bool | Restrict to workspace roots |

#### [feature_flags]

| Key | Type | Description |
|-----|------|-------------|
| `enabled` | array | List of enabled feature flags |

---

## Claude Code MCP Configuration

### Location

MCP server configuration lives in your Claude Code settings. Access via:

1. Command Palette → **"Claude Code: Configure MCP Servers"**
2. Or directly edit `.claude/settings.json`

### Configuration Format

```json
{
  "mcpServers": {
    "foundry-mcp": {
      "command": "uvx",
      "args": ["foundry-mcp"],
      "env": {
        "FOUNDRY_MCP_SPECS_DIR": "/path/to/project/specs",
        "FOUNDRY_MCP_LOG_LEVEL": "INFO",
        "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

### Using uvx (Recommended)

```json
{
  "mcpServers": {
    "foundry-mcp": {
      "command": "uvx",
      "args": ["foundry-mcp"],
      "env": {
        "FOUNDRY_MCP_SPECS_DIR": "/path/to/specs"
      }
    }
  }
}
```

### Using pip Installation

```json
{
  "mcpServers": {
    "foundry-mcp": {
      "command": "foundry-mcp",
      "env": {
        "FOUNDRY_MCP_SPECS_DIR": "/path/to/specs"
      }
    }
  }
}
```

### Using Python Module

```json
{
  "mcpServers": {
    "foundry-mcp": {
      "command": "python",
      "args": ["-m", "foundry_mcp.server"],
      "env": {
        "FOUNDRY_MCP_SPECS_DIR": "/path/to/specs"
      }
    }
  }
}
```

### Environment Variable References

Use `${VAR_NAME}` to reference system environment variables:

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}",
    "FOUNDRY_MCP_SPECS_DIR": "${PROJECT_ROOT}/specs"
  }
}
```

---

## Work Mode Configuration

### SDD Config File

Create `.claude/sdd_config.json` for SDD-specific settings:

```json
{
  "work_mode": "single",
  "context_limit_percent": 85,
  "auto_journal": true,
  "default_spec_folder": "active",
  "verification_level": "strict"
}
```

### Work Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `single` | One task at a time, human in loop | Default, most control |
| `autonomous` | Continuous execution until limit | Batch processing |
| `batch` | Process multiple specs | CI/CD pipelines |

### Setting Work Mode

Via environment:
```bash
export FOUNDRY_MCP_WORKFLOW_MODE="autonomous"
```

Via TOML:
```toml
[workflow]
mode = "autonomous"
```

---

## Security Configuration

### API Key Authentication

Enable API key requirement:

```toml
[security]
require_api_key = true
allowed_keys = ["key-prod-abc123", "key-dev-xyz789"]
```

Pass key via environment:
```bash
export FOUNDRY_MCP_API_KEY="key-prod-abc123"
```

### Workspace Scoping

Restrict operations to specific directories:

```toml
[security]
workspace_scoping = true

[workspace]
workspace_roots = [
    "/home/user/projects/myapp",
    "/home/user/projects/myapp-tests"
]
```

Operations outside these paths will be rejected.

### Rate Limiting

Configure request limits:

```toml
[security]
rate_limit = 100  # requests per minute
```

When exceeded, responses include:
```json
{
  "meta": {
    "rate_limit": {
      "limit": 100,
      "remaining": 0,
      "reset_at": "2025-12-04T12:05:00Z"
    }
  }
}
```

---

## Feature Flags

### Available Flags

| Flag | Description | Default |
|------|-------------|---------|
| `response_contract_v2` | Use v2 response envelope | Enabled |
| `environment_tools` | Enable env verification tools | Enabled |
| `spec_helpers` | Enable spec authoring helpers | Enabled |
| `planning_tools` | Enable planning tools | Enabled |

### Enabling Flags

Via environment:
```bash
export FOUNDRY_MCP_FEATURE_FLAGS="response_contract_v2,planning_tools"
```

Via TOML:
```toml
[feature_flags]
enabled = ["response_contract_v2", "planning_tools"]
```

### Checking Flag Status

```
mcp__plugin_foundry_foundry-mcp__server action="capability-get" capability_name="feature_flags"
```

---

## Directory Setup

### Creating Specs Directory

```bash
mkdir -p specs/{pending,active,completed,archived}
```

### Recommended Structure

```
project/
├── specs/
│   ├── pending/
│   ├── active/
│   ├── completed/
│   └── archived/
├── foundry-mcp.toml
├── .claude/
│   └── sdd_config.json
└── src/
```

### Permissions

Ensure the specs directory is writable:

```bash
chmod -R 755 specs/
```

---

## Configuration Validation

### Verify Environment

```
mcp__plugin_foundry_foundry-mcp__environment action="verify"
```

**Response shows:**
- Configuration sources loaded
- Active settings
- Missing requirements
- Warnings

### Common Issues

| Issue | Solution |
|-------|----------|
| Specs dir not found | Check `FOUNDRY_MCP_SPECS_DIR` path |
| Permission denied | Ensure write access to specs/ |
| Config not loading | Check TOML syntax |
| API key rejected | Verify key in `allowed_keys` |

---

## Example Configurations

### Development Setup

```toml
[logging]
level = "DEBUG"
structured = false

[workflow]
mode = "single"
auto_validate = true

[llm]
provider = "anthropic"
timeout = 60

[security]
require_api_key = false
```

### Production Setup

```toml
[logging]
level = "INFO"
structured = true

[workflow]
mode = "single"
auto_validate = true

[llm]
provider = "anthropic"
timeout = 30

[security]
require_api_key = true
allowed_keys = ["${FOUNDRY_API_KEY}"]
rate_limit = 50
workspace_scoping = true
```

### CI/CD Setup

```toml
[logging]
level = "WARNING"
structured = true

[workflow]
mode = "batch"
auto_validate = true

[security]
require_api_key = true
rate_limit = 200
```

---

## Related Documentation

- **[00-Quickstart](./00-quickstart.md)** — Initial setup
- **[09-LLM Providers](./09-llm-providers.md)** — Provider configuration
- **[11-Troubleshooting](./11-troubleshooting.md)** — Config issues

---

*[Back to Index](./README.md)*
