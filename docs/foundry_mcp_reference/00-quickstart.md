# Quickstart Guide

> Get from zero to productive with foundry-mcp in under 10 minutes.

---

## Prerequisites

Before installing foundry-mcp, ensure you have:

| Requirement | Version | Check Command |
|-------------|---------|---------------|
| Python | 3.10+ | `python --version` |
| pip or uv | Latest | `pip --version` or `uv --version` |
| Claude Code | Latest | Check VS Code extensions |

---

## Installation Methods

### Option 1: Run Instantly with uvx (Recommended)

The fastest way to get started—no installation required:

```bash
uvx foundry-mcp
```

This downloads and runs foundry-mcp in an isolated environment.

### Option 2: Install from PyPI

For persistent installation:

```bash
pip install foundry-mcp
```

Verify installation:

```bash
foundry-mcp --version
```

### Option 3: Install from Source (Development)

For contributors or custom modifications:

```bash
# Clone the repository
git clone https://github.com/tylerburleigh/foundry-mcp.git
cd foundry-mcp

# Install with test dependencies
pip install -e ".[test]"

# Verify
python -c "import foundry_mcp; print('foundry-mcp installed')"
```

---

## Claude Code Configuration

### Step 1: Open MCP Settings

In VS Code with Claude Code:
1. Open Command Palette (`Cmd+Shift+P` / `Ctrl+Shift+P`)
2. Search for **"Claude Code: Configure MCP Servers"**
3. Select to open your MCP configuration

### Step 2: Add foundry-mcp Server

Add the following to your `mcpServers` configuration:

**Using uvx (recommended):**

```json
{
  "mcpServers": {
    "foundry-mcp": {
      "command": "uvx",
      "args": ["foundry-mcp"],
      "env": {
        "FOUNDRY_MCP_SPECS_DIR": "/path/to/your/project/specs"
      }
    }
  }
}
```

**Using pip installation:**

```json
{
  "mcpServers": {
    "foundry-mcp": {
      "command": "foundry-mcp",
      "env": {
        "FOUNDRY_MCP_SPECS_DIR": "/path/to/your/project/specs"
      }
    }
  }
}
```

### Step 3: Set Your Specs Directory

Replace `/path/to/your/project/specs` with the actual path to where you want specs stored. If the directory doesn't exist, create it:

```bash
mkdir -p /path/to/your/project/specs/{pending,active,completed,archived}
```

---

## Verifying Installation

### Method 1: Check MCP Server Status

In Claude Code, use the `/mcp` command to see connected servers:

```
/mcp
```

You should see `foundry-mcp` listed with its tools.

### Method 2: List Available Tools

Ask Claude to list foundry-mcp tools:

```
What tools are available from foundry-mcp?
```

You should see 40+ tools including `spec-create`, `task-next`, `doc-stats`, etc.

### Method 3: Run a Test Command

Try a simple tool invocation:

```
Use mcp__plugin_foundry_foundry-mcp__environment action="verify" to check the environment
```

---

## Your First SDD Workflow

Let's walk through a complete spec-driven development cycle.

### Step 1: Check Documentation Availability

Before planning, verify codebase documentation is available:

```
Use mcp__plugin_foundry_foundry-mcp__code action="doc-stats" to check documentation status
```

If documentation isn't available, the tool will guide you on how to generate it.

### Step 2: Create a Specification

Use the `sdd-plan` skill to create a new specification:

```
I want to add a user authentication feature. Use skill sdd-plan to create a specification.
```

The skill will:
1. Gather requirements through questions
2. Analyze existing code patterns
3. Generate a structured JSON specification
4. Validate and save to `specs/pending/`

### Step 3: Activate the Spec

Move from planning to implementation:

```
Use mcp__plugin_foundry_foundry-mcp__lifecycle action="activate" spec_id="your-spec-id"
```

This moves the spec from `pending/` to `active/`.

### Step 4: Find the Next Task

Use the `sdd-next` skill to get actionable work:

```
Use skill sdd-next to find what to work on
```

The skill will:
1. Query for active specs
2. Find the next actionable task based on dependencies
3. Provide implementation context

### Step 5: Complete Tasks and Journal

As you work, update task status and journal decisions:

```
Use skill sdd-update to mark task "task-id" as completed with notes about what was implemented
```

### Step 6: Create a Pull Request

When all tasks are complete, generate a PR:

```
Use skill sdd-pr to create a pull request for the completed spec
```

---

## Specs Directory Structure

After setup, your specs directory should look like:

```
specs/
├── pending/      # New specs awaiting activation
├── active/       # Currently being worked on
├── completed/    # Finished specs with journals
└── archived/     # Historical reference
```

Each spec is a JSON file containing:
- Metadata (ID, timestamps, status)
- Phases with tasks and subtasks
- Dependencies between tasks
- Verification criteria
- Decision journals

---

## Common Configuration Options

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `FOUNDRY_MCP_SPECS_DIR` | Path to specs directory | Auto-detected |
| `FOUNDRY_MCP_LOG_LEVEL` | Logging level | `INFO` |
| `FOUNDRY_MCP_WORKFLOW_MODE` | Execution mode | `single` |

### Optional TOML Configuration

Create `foundry-mcp.toml` in your project root for persistent settings:

```toml
[workspace]
specs_dir = "./specs"

[logging]
level = "INFO"

[workflow]
mode = "single"
auto_validate = true
```

---

## Next Steps

Now that foundry-mcp is running:

1. **[01-Architecture](./01-architecture.md)** — Understand system design
2. **[02-SDD Philosophy](./02-sdd-philosophy.md)** — Learn the methodology
3. **[04-Tool Reference](./04-tool-reference.md)** — Explore all available tools
4. **[06-Workflows](./06-workflows.md)** — Master common patterns

---

## Troubleshooting Quick Fixes

### Server Not Starting

```bash
# Check Python version
python --version  # Must be 3.10+

# Try running directly
python -m foundry_mcp.server
```

### Tools Not Appearing

1. Restart Claude Code
2. Check MCP configuration syntax
3. Verify `FOUNDRY_MCP_SPECS_DIR` path exists

### Permission Errors

Ensure your specs directory is writable:

```bash
chmod -R 755 /path/to/specs
```

For more help, see [11-Troubleshooting](./11-troubleshooting.md).

---

*[Back to Index](./README.md)*
