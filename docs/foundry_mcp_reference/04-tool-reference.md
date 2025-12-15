# Tool Reference

> Complete reference for all foundry-mcp MCP tools using the consolidated router architecture.

---

## Router Architecture

foundry-mcp uses 17 consolidated routers, each handling related operations via an `action` parameter.

### Invocation Pattern

From claude-foundry skills, invoke tools as:
```
mcp__plugin_foundry_foundry-mcp__<router> action="<action>" [parameters...]
```

**Example:**
```
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="my-spec"
mcp__plugin_foundry_foundry-mcp__task action="complete" spec_id="my-spec" task_id="task-1-1"
```

---

## Routers by Domain

### spec — Specification Operations

Operations for querying, validating, and managing specifications.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `get` | Retrieve a spec by ID | `spec_id` |
| `get-hierarchy` | Get full spec hierarchy | `spec_id` |
| `list` | List specs by status | `status`, `limit`, `cursor` |
| `list-basic` | Lightweight spec listing | `status` |
| `list-by-folder` | List specs in specific folder | `directory` |
| `find` | Find specs matching criteria | `status` |
| `validate` | Validate spec structure | `spec_id` |
| `fix` | Auto-fix spec issues | `spec_id`, `dry_run` |
| `validate-fix` | Validate and fix in one call | `spec_id` |
| `stats` | Get spec statistics | `spec_id` |
| `render` | Render spec as markdown | `spec_id` |
| `render-progress` | Render progress summary | `spec_id` |

---

### task — Task Operations

Operations for managing tasks within specifications.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `next` | Get next actionable task | `spec_id` |
| `prepare` | Prepare task with context | `spec_id`, `task_id` |
| `get` | Get task details | `spec_id`, `task_id` |
| `info` | Get detailed task info | `spec_id`, `task_id` |
| `query` | Query tasks with filters | `spec_id`, `status`, `parent` |
| `list` | List all tasks | `spec_id`, `status` |
| `check-deps` | Check task dependencies | `spec_id`, `task_id` |
| `update-status` | Update task status | `spec_id`, `task_id`, `status` |
| `start` | Mark task as in_progress | `spec_id`, `task_id` |
| `complete` | Mark task as completed | `spec_id`, `task_id`, `journal_entry` |
| `progress` | Get task progress metrics | `spec_id` |
| `block` | Block a task | `spec_id`, `task_id`, `reason`, `blocker_type` |
| `unblock` | Unblock a task | `spec_id`, `task_id`, `resolution` |
| `list-blocked` | List all blocked tasks | `spec_id` |

---

### journal — Decision Tracking

Operations for journal entries and audit trails.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `add` | Add a journal entry | `spec_id`, `title`, `content`, `task_id`, `entry_type` |
| `list` | List journal entries | `spec_id`, `task_id`, `limit` |

**Entry types:** `decision`, `deviation`, `blocker`, `note`, `status_change`

---

### lifecycle — Spec Lifecycle

Operations for moving specs through lifecycle stages.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `move` | Move spec between folders | `spec_id`, `to_folder` |
| `activate` | Activate a pending spec | `spec_id` |
| `complete` | Mark spec as completed | `spec_id` |
| `archive` | Archive a completed spec | `spec_id` |
| `state` | Get current lifecycle state | `spec_id` |

---

### authoring — Spec Creation & Modification

Operations for creating and structurally modifying specifications.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `spec-create` | Create a new specification | `name`, `title`, `description` |
| `phase-add` | Add a phase to a spec | `spec_id`, `phase_id`, `title` |
| `task-add` | Add a task to a spec | `spec_id`, `title`, `parent` |
| `assumption-add` | Add an assumption | `spec_id`, `text`, `assumption_type` |
| `revision-add` | Add a revision entry | `spec_id`, `changes` |
| `update-frontmatter` | Update spec metadata | `spec_id`, `key`, `value` |

---

### review — Review Operations

Operations for AI-powered reviews and fidelity checking.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `fidelity` | Compare implementation vs spec | `spec_id`, `task_id` |
| `parse-feedback` | Parse review feedback | `feedback` |

---

### plan — Planning Operations

Operations for planning and plan review.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `review` | AI review of specification | `name`, `plan_path` |

---

### pr — Pull Request Operations

Operations for creating PRs with spec context.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `create` | Create a pull request | `spec_id`, `title`, `base_branch` |
| `context` | Gather PR creation context | `spec_id` |

---

### test — Testing Operations

Operations for test discovery and execution.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `run` | Run tests | `target`, `markers`, `verbose` |
| `run-quick` | Run quick smoke tests | `target` |
| `run-unit` | Run unit tests only | `target` |
| `discover` | Discover available tests | `target` |
| `presets` | List available presets | — |

---

### code — Code Analysis

Operations for querying codebase documentation and code intelligence.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `find-class` | Find class definition | `symbol` |
| `find-function` | Find function definition | `symbol` |
| `trace-calls` | Trace function calls | `symbol` |
| `impact-analysis` | Analyze change impact | `symbol` |
| `get-callers` | Get function callers | `symbol` |
| `get-callees` | Get functions called | `symbol` |

---

### provider — LLM Provider Management

Operations for managing LLM providers.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `list` | List available providers | — |
| `status` | Get provider status | `provider_id` |
| `execute` | Execute with specific provider | `provider_id`, `prompt` |

---

### verification — Verification Operations

Operations for verification steps and results.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `add` | Add verification result | `spec_id`, `verify_id`, `result` |
| `execute` | Run verification task | `spec_id`, `verify_id` |
| `format-summary` | Format verification summary | `spec_id` |

---

### environment — Environment Setup

Operations for environment setup and verification.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `verify` | Verify environment setup | — |
| `setup` | Run setup process | `path` |
| `init-workspace` | Initialize specs workspace | `path` |

---

### health — Health Checks

Operations for server health monitoring.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `liveness` | Basic liveness check | — |
| `readiness` | Full readiness check | — |
| `check` | Comprehensive health check | `include_details` |

---

### error — Error Tracking

Operations for error monitoring and patterns.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `list` | List errors | `limit`, `since` |
| `get` | Get error details | `error_id` |
| `stats` | Get error statistics | — |
| `patterns` | Analyze error patterns | `min_count` |

---

### metrics — Metrics Tracking

Operations for metrics collection and analysis.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `query` | Query metrics | `metric_name`, `since`, `until` |
| `list` | List available metrics | — |
| `summary` | Get metrics summary | — |
| `stats` | Get detailed statistics | — |

---

### server — Server Information

Operations for discovering server capabilities.

| Action | Description | Key Parameters |
|--------|-------------|----------------|
| `capabilities` | Get server capabilities | — |
| `tool-list` | List all available tools | `category` |
| `tool-schema` | Get tool JSON schema | `tool_name` |

---

## Common Parameters

Parameters that appear across multiple routers:

| Parameter | Type | Description |
|-----------|------|-------------|
| `action` | string | **Required.** The operation to perform |
| `spec_id` | string | Specification identifier |
| `task_id` | string | Task identifier within a spec |
| `phase_id` | string | Phase identifier within a spec |
| `status` | string | Status filter: `pending`, `in_progress`, `completed`, `blocked` |
| `dry_run` | boolean | Preview changes without applying |
| `limit` | integer | Maximum results to return |
| `cursor` | string | Pagination cursor for next page |

---

## Response Format

All tools return the standardized v2 response envelope:

```json
{
  "success": true,
  "data": { ... },
  "error": null,
  "meta": {
    "version": "response-v2",
    "request_id": "req_abc123",
    "warnings": [],
    "pagination": { ... }
  }
}
```

See [05-Response Contract](./05-response-contract.md) for details.

---

## Usage Examples

### Find Next Task

```
mcp__plugin_foundry_foundry-mcp__task action="next" spec_id="my-feature-spec"
```

### Validate a Spec

```
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="my-feature-spec"
```

### Complete a Task

```
mcp__plugin_foundry_foundry-mcp__task action="complete" spec_id="my-spec" task_id="task-1-1" journal_entry="Implemented authentication with JWT tokens. All tests passing."
```

### Run Tests

```
mcp__plugin_foundry_foundry-mcp__test action="run" target="tests/unit" markers="quick"
```

### Move Spec to Active

```
mcp__plugin_foundry_foundry-mcp__lifecycle action="activate" spec_id="my-spec"
```

---

## Related Documentation

- **[05-Response Contract](./05-response-contract.md)** — Response envelope details
- **[06-Workflows](./06-workflows.md)** — Tool usage in workflows
- **[08-Integration Patterns](./08-integration-patterns.md)** — How skills use these tools

---

*[Back to Index](./README.md)*
