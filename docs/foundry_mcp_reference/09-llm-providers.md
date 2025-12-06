# AI Consultation (CLI Providers)

> How foundry-mcp invokes external AI tools via CLI for consultation workflows.

---

## Overview

foundry-mcp uses a **CLI-based provider architecture** for AI consultation. Rather than making direct API calls, it invokes external CLI tools (Gemini CLI, Cursor Agent, Codex) via subprocess. This design:

- Leverages existing CLI tool authentication
- Provides consistent read-only enforcement
- Enables caching and fallback patterns
- Works with any CLI tool that outputs JSON

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                 ConsultationOrchestrator                    │
│        (Central coordinator for AI workflows)               │
└─────────────────────────────────────────────────────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            ▼                 ▼                 ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  Gemini CLI     │  │  Cursor Agent   │  │  Codex CLI      │
│  Provider       │  │  Provider       │  │  Provider       │
└─────────────────┘  └─────────────────┘  └─────────────────┘
         │                   │                   │
         ▼                   ▼                   ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ subprocess.run  │  │ subprocess.run  │  │ subprocess.run  │
│ gemini -m ...   │  │ cursor-agent    │  │ codex exec ...  │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### Key Components

| Component | Purpose |
|-----------|---------|
| **ConsultationOrchestrator** | Selects provider, manages caching, normalizes responses |
| **Provider Registry** | Discovers available CLI tools on PATH |
| **ProviderDetector** | Checks binary availability via `which` + health probe |
| **ProviderContext** | Base class for CLI invocation with lifecycle hooks |
| **ResultCache** | Filesystem-based caching at `.cache/foundry-mcp/consultations/` |

---

## Available CLI Providers

| Provider | Binary | Detection | Use Case |
|----------|--------|-----------|----------|
| **Gemini** | `gemini` | `which gemini` | General AI consultation |
| **Cursor Agent** | `cursor-agent` | `which cursor-agent` | IDE-integrated analysis |
| **Codex** | `codex` | `which codex` | Code-focused tasks |

### Provider Selection Priority

1. Explicit `provider_id` in request
2. `FOUNDRY_AI_PROVIDER_PRIORITY` environment variable
3. First available provider from: `["gemini", "codex", "cursor-agent"]`

---

## Provider-Specific Details

### Gemini CLI

**Binary:** `gemini`

**Invocation:**
```bash
gemini -m gemini-2.5-flash --output-format json -p "prompt here"
```

**Key Options:**
| Flag | Purpose |
|------|---------|
| `-m MODEL` | Model selection (e.g., `gemini-2.5-flash`, `pro`) |
| `--output-format json` | JSON output for parsing |
| `--allowed-tools` | Read-only tool enforcement |
| `-p "PROMPT"` | The prompt to execute |

**Read-Only Enforcement:**
- Uses `--allowed-tools` flag to restrict operations
- Blocks piped commands (security limitation documented)

**Environment Variables:**
```bash
FOUNDRY_GEMINI_AVAILABLE_OVERRIDE=true  # Force availability
FOUNDRY_GEMINI_BINARY=/custom/path/gemini  # Custom binary path
```

---

### Cursor Agent CLI

**Binary:** `cursor-agent`

**Invocation:**
```bash
cursor-agent --print --output-format json
```

**Key Features:**
- Config-based permission system
- Automatic backup/restore of user config
- Allowlist/denylist for tool access

**Read-Only Enforcement:**

Cursor Agent uses a JSON config file at `~/.cursor/cli-config.json`:

```json
{
  "permissions": [
    {"allow": "Read(**)"},
    {"allow": "Grep(**)"},
    {"allow": "Shell(git)"}
  ],
  "approvalMode": "allowlist"
}
```

The provider:
1. Backs up existing user config
2. Writes read-only config
3. Executes command
4. Restores original config (even on failure)

**Environment Variables:**
```bash
FOUNDRY_CURSOR_AGENT_AVAILABLE_OVERRIDE=true
FOUNDRY_CURSOR_AGENT_BINARY=/custom/path/cursor-agent
```

---

### Codex CLI

**Binary:** `codex`

**Invocation:**
```bash
codex exec --sandbox read-only "prompt here"
```

**Key Features:**
- Native OS-level sandboxing
- JSONL output format
- No custom permission config needed

**Read-Only Enforcement:**

Codex uses OS-native sandboxing via `--sandbox read-only`:

| Platform | Mechanism |
|----------|-----------|
| macOS | Seatbelt |
| Linux | Landlock + seccomp |
| Windows | Restricted Token |

This is enforced at the OS level—more secure than config-based approaches.

**Environment Variables:**
```bash
FOUNDRY_CODEX_AVAILABLE_OVERRIDE=true
FOUNDRY_CODEX_BINARY=/custom/path/codex
```

---

## Provider Detection

### How Availability is Determined

```
1. Check environment override
   FOUNDRY_{PROVIDER}_AVAILABLE_OVERRIDE=true → available

2. Check test mode
   FOUNDRY_PROVIDER_TEST_MODE=1 → not available (for testing)

3. Find binary on PATH
   which gemini → /usr/local/bin/gemini

4. Run health probe
   gemini --help → exits 0 → available
```

### Detection Flow

```python
# Simplified detection logic
def is_available(provider_id):
    # 1. Override check
    if os.environ.get(f"FOUNDRY_{provider_id.upper()}_AVAILABLE_OVERRIDE") == "true":
        return True

    # 2. Test mode check
    if os.environ.get("FOUNDRY_PROVIDER_TEST_MODE") == "1":
        return False

    # 3. PATH check
    binary = shutil.which(provider_binary[provider_id])
    if not binary:
        return False

    # 4. Health probe
    result = subprocess.run([binary, "--help"], capture_output=True)
    return result.returncode == 0
```

---

## Environment Variables

### Provider Selection

```bash
# Priority order for provider selection
FOUNDRY_AI_PROVIDER_PRIORITY="gemini,codex,cursor-agent"
```

### Per-Provider Overrides

```bash
# Force provider availability (bypass PATH detection)
FOUNDRY_GEMINI_AVAILABLE_OVERRIDE=true
FOUNDRY_CODEX_AVAILABLE_OVERRIDE=false
FOUNDRY_CURSOR_AGENT_AVAILABLE_OVERRIDE=true

# Custom binary paths
FOUNDRY_GEMINI_BINARY=/opt/tools/gemini
FOUNDRY_CODEX_BINARY=/usr/local/bin/codex
FOUNDRY_CURSOR_AGENT_BINARY=/path/to/cursor-agent
```

### Testing & Development

```bash
# Disable real CLI probes (for unit tests)
FOUNDRY_PROVIDER_TEST_MODE=1

# Additional tool PATH
FOUNDRY_TOOL_PATH=/usr/local/bin:/opt/tools
```

### Cache Configuration

```bash
# Cache directory (default: .cache/foundry-mcp/consultations)
FOUNDRY_AI_CACHE_DIR=/custom/cache/path

# Cache TTL in hours (default: 24)
FOUNDRY_AI_CACHE_TTL_HOURS=12
```

---

## Caching

### How Caching Works

Consultation results are cached to avoid redundant CLI calls:

**Cache Location:**
```
.cache/foundry-mcp/consultations/{workflow}/{cache_key}.json
```

**Cache Key Generation:**
```python
key_parts = [prompt_id, json.dumps(context, sort_keys=True), model]
cache_key = sha256("|".join(key_parts)).hexdigest()[:32]
```

**Cache Entry Format:**
```json
{
  "content": "AI response content",
  "provider_id": "gemini",
  "model_used": "gemini:2.5-flash",
  "tokens": {"input": 100, "output": 50, "total": 150},
  "timestamp": 1701234567.89,
  "ttl": 3600
}
```

### Cache Behavior

| Scenario | Behavior |
|----------|----------|
| Cache hit (valid) | Return cached result immediately |
| Cache miss | Execute CLI, cache result |
| Cache expired | Execute CLI, update cache |
| Cache disabled | Always execute CLI |

---

## Consultation Workflows

### Available Workflows

| Workflow | Purpose | Used By |
|----------|---------|---------|
| `PLAN_REVIEW` | Review spec quality | `review-spec` tool |
| `FIDELITY_REVIEW` | Compare code vs spec | `fidelity-check` tool |
| `DOC_GENERATION` | Generate documentation | Doc generation tools |
| `TEST_CONSULT` | Debug test failures | `test-consult` tool |

### Using Consultation

From MCP tools:

```
mcp__foundry-mcp__test-consult with
  error="AssertionError in test_login"
  context="[test output and code]"
  provider="gemini"
```

The orchestrator:
1. Checks cache for matching request
2. Selects available provider (or uses specified)
3. Builds prompt from template
4. Executes CLI subprocess
5. Parses JSON response
6. Caches result
7. Returns normalized response

---

## Response Normalization

All providers normalize to a standard result:

```json
{
  "success": true,
  "data": {
    "content": "AI analysis content...",
    "provider_id": "gemini",
    "model_used": "gemini:2.5-flash",
    "tokens": {
      "input": 150,
      "output": 200,
      "total": 350
    },
    "duration_ms": 2500,
    "cache_hit": false
  }
}
```

### Provider Status Values

| Status | Description | Retryable |
|--------|-------------|-----------|
| `SUCCESS` | Normal completion | - |
| `TIMEOUT` | Request timed out | Yes |
| `NOT_FOUND` | Binary not on PATH | No |
| `INVALID_OUTPUT` | Malformed response | No |
| `ERROR` | Generic error | Yes |
| `CANCELED` | User canceled | No |

---

## Graceful Degradation

When no CLI providers are available:

### With Provider Available

```json
{
  "success": true,
  "data": {
    "review": {
      "content": "Detailed AI analysis...",
      "ai_generated": true,
      "provider_id": "gemini"
    }
  }
}
```

### Without Provider

```json
{
  "success": true,
  "data": {
    "review": {
      "structural_analysis": {...},
      "validation_results": {...},
      "ai_generated": false
    }
  },
  "meta": {
    "warnings": ["No AI providers available, using structural analysis"]
  }
}
```

### Fallback Behavior by Tool

| Tool | With AI | Without AI |
|------|---------|------------|
| `review-spec` | AI-powered analysis | Structural validation |
| `fidelity-check` | AI comparison | Rule-based checking |
| `pr-context` | AI-enhanced summary | Template-based summary |
| `test-consult` | External debugging | Error - no fallback |

---

## Security

### Read-Only Enforcement

Each provider enforces read-only access differently:

| Provider | Mechanism | Enforcement Level |
|----------|-----------|-------------------|
| Gemini | `--allowed-tools` flag | Application |
| Cursor | Config file allowlist | Application |
| Codex | `--sandbox read-only` | OS (kernel) |

**Recommendation:** Codex provides the strongest isolation via OS-level sandboxing.

### Prompt Injection Protection

- System prompts include security warnings
- Input context is sanitized
- Output is validated against expected format

**Piped Command Warning (Gemini):**
```
IMPORTANT: Avoid piped commands (e.g., cat file | wc -l).
Piped commands bypass tool allowlist checks in Gemini CLI.
```

---

## Troubleshooting

### Provider Not Found

```json
{
  "error_code": "PROVIDER_NOT_FOUND",
  "remediation": "Install CLI tool or set override: FOUNDRY_GEMINI_AVAILABLE_OVERRIDE=true"
}
```

**Solutions:**
1. Install the CLI tool (e.g., `pip install gemini-cli`)
2. Add tool to PATH
3. Use environment override for testing

### Timeout Errors

**Solutions:**
1. Increase timeout in request
2. Use a faster model
3. Check network/CLI tool performance

### Invalid Output

```json
{
  "error_code": "INVALID_OUTPUT",
  "remediation": "CLI output was not valid JSON"
}
```

**Solutions:**
1. Verify CLI tool version
2. Check for stderr output
3. Enable debug logging

### Checking Provider Status

```
mcp__foundry-mcp__provider-list
```

Shows all providers with availability status and detection method.

---

## Related Documentation

- **[07-Configuration](./07-configuration.md)** — General configuration
- **[10-Testing](./10-testing.md)** — Test consultation features
- **[11-Troubleshooting](./11-troubleshooting.md)** — Error resolution

---

*[Back to Index](./README.md)*
