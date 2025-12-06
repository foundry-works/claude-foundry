# Architecture Overview

> Understanding foundry-mcp's layered architecture and integration with Claude Code.

---

## High-Level Architecture

foundry-mcp follows a layered architecture where business logic is transport-agnostic, enabling both MCP server and CLI interfaces to share the same core functionality.

```
┌─────────────────────────────────────────────────────────────┐
│                    MCP Server (server.py)                   │
│              FastMCP + Tool Registration                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Tools Layer (tools/*.py)                  │
│         MCP Tool Implementations + Response Formatting       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Core Layer (core/*.py)                    │
│         Business Logic (transport-agnostic)                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    CLI Layer (cli/*.py)                      │
│         Click Commands + JSON Output                         │
└─────────────────────────────────────────────────────────────┘
```

---

## Layer Responsibilities

| Layer | Location | Responsibility |
|-------|----------|----------------|
| **Server** | `server.py` | MCP server creation, tool registration, transport handling |
| **Tools** | `tools/*.py` | MCP tool implementations, parameter validation, response formatting |
| **Core** | `core/*.py` | Pure business logic, no transport dependencies |
| **CLI** | `cli/*.py` | Command-line interface using Click, JSON output |

### Key Design Principle

**All business logic MUST live in `core/`**. Tools and CLI commands are thin wrappers that:
1. Parse and validate input parameters
2. Call core functions
3. Format responses for their transport (MCP or CLI)

This ensures:
- Consistent behavior across MCP and CLI
- Single source of truth for business rules
- Easier testing (core functions can be unit tested without transport)
- Feature parity between interfaces

---

## Component Details

### Server Layer (`server.py`)

The entry point for MCP connections:

```python
# Simplified structure
from fastmcp import FastMCP

mcp = FastMCP("foundry-mcp")

# Tools are registered from tools/*.py modules
from foundry_mcp.tools import tasks, validation, lifecycle, ...

# Server handles transport and tool routing
```

**Responsibilities:**
- FastMCP server initialization
- Tool registration and discovery
- Resource and prompt registration
- Transport handling (stdio, SSE, etc.)

### Tools Layer (`tools/*.py`)

MCP tool implementations organized by domain:

| Module | Tools | Purpose |
|--------|-------|---------|
| `tasks.py` | `task-next`, `task-prepare`, `task-complete` | Task operations |
| `validation.py` | `spec-validate`, `spec-audit` | Spec validation |
| `lifecycle.py` | `spec-activate`, `spec-complete`, `spec-archive` | Lifecycle transitions |
| `journal.py` | `journal-add`, `journal-get` | Decision tracking |
| `docs.py` | `doc-stats`, `doc-scope`, `doc-search` | Code documentation |
| `testing.py` | `test-run`, `test-discover` | pytest integration |
| `review.py` | `review-spec`, `fidelity-check` | AI-powered review |
| `pr_workflow.py` | `pr-context`, `pr-create` | PR generation |
| `authoring.py` | `spec-create`, `spec-update` | Spec creation |
| `rendering.py` | `spec-render`, `plan-format` | Spec visualization |

**Tool Structure:**

```python
@mcp.tool()
async def task_next(spec_id: str = None) -> dict:
    """Find the next actionable task."""
    # 1. Validate parameters
    # 2. Call core function
    result = core.task.find_next_task(spec_id)
    # 3. Format response
    return responses.success_response(result)
```

### Core Layer (`core/*.py`)

Transport-agnostic business logic:

| Module | Purpose |
|--------|---------|
| `spec.py` | Spec file I/O, discovery, parsing |
| `task.py` | Task operations and dependency resolution |
| `journal.py` | Journal entry management |
| `lifecycle.py` | Spec state transitions |
| `validation.py` | Schema and input validation |
| `responses.py` | Response envelope helpers |
| `pagination.py` | Cursor-based pagination |
| `feature_flags.py` | Feature flag management |
| `observability.py` | Logging and metrics |
| `security.py` | API key validation, rate limiting |
| `ai_consultation.py` | LLM provider abstraction |
| `discovery.py` | Tool registry and capabilities |

### CLI Layer (`cli/*.py`)

Command-line interface mirroring MCP tools:

```bash
python -m foundry_mcp.cli task next --specs-dir ./specs
python -m foundry_mcp.cli spec validate --spec-id my-spec
```

The CLI:
- Outputs JSON for machine consumption
- Shares core logic with MCP tools
- Provides the same capabilities as MCP interface

---

## Integration with Claude Code

### How claude-foundry Connects

```
┌─────────────────────┐      MCP Protocol       ┌─────────────────────┐
│   Claude Code       │◄──────────────────────►│   foundry-mcp       │
│   + claude-foundry  │                         │   MCP Server        │
└─────────────────────┘                         └─────────────────────┘
         │                                                │
         │  Skill invokes                                 │
         ▼                                                ▼
┌─────────────────────┐                         ┌─────────────────────┐
│   sdd-plan skill    │──────────────────────►│   spec-create tool   │
│   sdd-next skill    │◄──────────────────────│   task-next tool     │
│   run-tests skill   │                         │   test-run tool      │
└─────────────────────┘                         └─────────────────────┘
```

### Tool Invocation Pattern

claude-foundry skills invoke foundry-mcp tools using the MCP naming convention:

```
mcp__foundry-mcp__<tool-name>
```

**Examples:**
- `mcp__foundry-mcp__spec-create` — Create a new specification
- `mcp__foundry-mcp__task-next` — Get next actionable task
- `mcp__foundry-mcp__doc-stats` — Check documentation status

### Data Flow

1. **User Request** → Claude Code receives user intent
2. **Skill Activation** → Claude selects appropriate skill (e.g., `sdd-plan`)
3. **Tool Invocation** → Skill invokes MCP tool with parameters
4. **Core Processing** → Tool calls core functions
5. **Response** → Standardized JSON response returned
6. **Presentation** → Skill formats response for user

---

## Key Modules Deep Dive

### Response Helpers (`core/responses.py`)

All responses use standardized helpers:

```python
from foundry_mcp.core.responses import success_response, error_response

# Success response
success_response(
    data={"spec_id": "...", "status": "active"},
    warnings=["Schema version outdated"]
)

# Error response
error_response(
    error="Spec not found",
    error_code="NOT_FOUND",
    error_type="not_found",
    remediation="Check spec_id exists in specs/ directory"
)
```

### Feature Flags (`core/feature_flags.py`)

Capabilities are gated by feature flags:

| Flag | Description |
|------|-------------|
| `response_contract_v2` | Use standardized v2 response envelope |
| `environment_tools` | Enable environment verification tools |
| `spec_helpers` | Enable spec authoring helpers |
| `planning_tools` | Enable planning/phase tools |

Flags are declared in `mcp/capabilities_manifest.json` and can be:
- Enabled by default
- Enabled via environment variable
- Negotiated with clients

### Pagination (`core/pagination.py`)

Large result sets use cursor-based pagination:

```python
{
  "data": {
    "tasks": [...],
  },
  "meta": {
    "pagination": {
      "cursor": "eyJvZmZzZXQiOjIwfQ==",
      "has_more": true,
      "total_count": 150,
      "page_size": 20
    }
  }
}
```

---

## Directory Structure

```
src/foundry_mcp/
├── __init__.py
├── server.py              # MCP server entry point
├── config.py              # Configuration management
│
├── core/                  # Business logic (transport-agnostic)
│   ├── spec.py            # Spec file operations
│   ├── task.py            # Task operations
│   ├── journal.py         # Journal management
│   ├── lifecycle.py       # Spec lifecycle transitions
│   ├── validation.py      # Spec validation
│   ├── responses.py       # Response envelope helpers
│   ├── feature_flags.py   # Feature flag management
│   ├── pagination.py      # Cursor-based pagination
│   ├── observability.py   # Logging and metrics
│   ├── security.py        # Authentication, rate limiting
│   ├── ai_consultation.py # LLM provider abstraction
│   └── discovery.py       # Tool registry
│
├── tools/                 # MCP tool implementations
│   ├── tasks.py           # Task-related tools
│   ├── validation.py      # Validation tools
│   ├── lifecycle.py       # Lifecycle tools
│   ├── journal.py         # Journal tools
│   ├── docs.py            # Code documentation tools
│   ├── testing.py         # Test runner tools
│   ├── review.py          # LLM review tools
│   ├── authoring.py       # Spec creation tools
│   ├── rendering.py       # Spec rendering tools
│   ├── pr_workflow.py     # PR generation tools
│   └── providers.py       # LLM provider tools
│
├── cli/                   # Command-line interface
│   ├── main.py            # CLI entry point
│   ├── output.py          # JSON output formatting
│   └── commands/          # Click command modules
│
├── resources/             # MCP resources (foundry:// URIs)
│   └── specs.py
│
└── prompts/               # MCP prompts (workflow templates)
    └── workflows.py
```

---

## Transport-Agnostic Design Benefits

### Why This Matters

1. **Consistency** — MCP and CLI produce identical results
2. **Testability** — Core logic tested without transport overhead
3. **Flexibility** — Add new transports without changing business logic
4. **Debugging** — Use CLI for quick testing when MCP is unavailable

### Example: Same Logic, Different Transport

**MCP Tool:**
```python
@mcp.tool()
async def task_next(spec_id: str = None) -> dict:
    result = core.task.find_next_task(spec_id)
    return responses.success_response(result)
```

**CLI Command:**
```python
@click.command()
@click.option('--spec-id')
def task_next(spec_id):
    result = core.task.find_next_task(spec_id)
    click.echo(json.dumps(responses.success_response(result)))
```

Both call the same `core.task.find_next_task()` function.

---

## Related Documentation

- **[04-Tool Reference](./04-tool-reference.md)** — Complete tool listing
- **[05-Response Contract](./05-response-contract.md)** — Response envelope details
- **[08-Integration Patterns](./08-integration-patterns.md)** — Skill-to-tool mapping

---

*[Back to Index](./README.md)*
