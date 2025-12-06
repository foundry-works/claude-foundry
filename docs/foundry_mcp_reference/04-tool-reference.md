# Tool Reference

> Complete reference for all foundry-mcp MCP tools.

---

## Tool Naming Conventions

foundry-mcp tools follow strict naming conventions:

### Format
```
prefix-verb[-modifier]
```

- **prefix** — Domain of the operation (`spec-`, `task-`, `doc-`, etc.)
- **verb** — Action being performed (`create`, `list`, `validate`, etc.)
- **modifier** — Optional qualifier (`-quick`, `-fix`, etc.)

### Common Prefixes

| Prefix | Domain | Example |
|--------|--------|---------|
| `spec-` | Specification operations | `spec-create`, `spec-validate` |
| `task-` | Task operations | `task-next`, `task-complete` |
| `plan-` / `phase-` | Planning utilities | `plan-format`, `phase-check-complete` |
| `journal-` | Decision tracking | `journal-add`, `journal-list` |
| `doc-` / `code-` | Documentation/code analysis | `doc-stats`, `code-find-class` |
| `test-` | Testing operations | `test-run`, `test-discover` |
| `review-` | Review workflows | `review-spec`, `review-parse-feedback` |
| `pr-` | Pull request workflows | `pr-context`, `pr-create` |
| `provider-` | LLM provider management | `provider-list`, `provider-status` |
| `sdd-` | Environment helpers | `sdd-verify-toolchain` |

### Invocation Pattern

From claude-foundry skills, invoke tools as:
```
mcp__foundry-mcp__<tool-name>
```

**Example:**
```
mcp__foundry-mcp__spec-validate with spec_id="my-spec"
```

---

## Tools by Domain

### Spec Operations

Tools for creating, managing, and validating specifications.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `spec-create` | Create a new specification | `spec_id`, `title`, `description`, `phases` |
| `spec-get` | Retrieve a spec by ID | `spec_id` |
| `spec-get-hierarchy` | Get full spec hierarchy | `spec_id` |
| `spec-list` | List specs by status | `status`, `limit`, `cursor` |
| `spec-list-basic` | Lightweight spec listing | `status` |
| `spec-list-by-folder` | List specs in specific folder | `folder` |
| `spec-find` | Find specs matching criteria | `query`, `status`, `date_range` |
| `spec-validate` | Validate spec structure | `spec_id`, `strict` |
| `spec-fix` | Auto-fix spec issues | `spec_id`, `dry_run` |
| `spec-validate-fix` | Validate and fix in one call | `spec_id` |
| `spec-stats` | Get spec statistics | `spec_id` |
| `spec-render` | Render spec as markdown | `spec_id`, `format` |
| `spec-render-progress` | Render progress summary | `spec_id` |
| `spec-update-frontmatter` | Update spec metadata | `spec_id`, `updates` |
| `spec-reconcile-state` | Reconcile spec state | `spec_id` |

#### Lifecycle Operations

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `spec-lifecycle-move` | Move spec between folders | `spec_id`, `target_folder` |
| `spec-lifecycle-activate` | Activate a pending spec | `spec_id` |
| `spec-lifecycle-complete` | Mark spec as completed | `spec_id`, `completion_notes` |
| `spec-lifecycle-archive` | Archive a completed spec | `spec_id` |
| `spec-lifecycle-state` | Get current lifecycle state | `spec_id` |

---

### Task Operations

Tools for working with tasks within specifications.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `task-next` | Get next actionable task | `spec_id` |
| `task-prepare` | Prepare task with context | `spec_id`, `task_id` |
| `task-get` | Get task details | `spec_id`, `task_id` |
| `task-info` | Get detailed task info | `spec_id`, `task_id` |
| `task-query` | Query tasks with filters | `spec_id`, `status`, `phase_id` |
| `task-list` | List all tasks | `spec_id`, `status` |
| `task-check-deps` | Check task dependencies | `spec_id`, `task_id` |
| `task-update-status` | Update task status | `spec_id`, `task_id`, `status` |
| `task-start` | Mark task as in_progress | `spec_id`, `task_id` |
| `task-complete` | Mark task as completed | `spec_id`, `task_id`, `notes`, `journal_entry` |
| `task-progress` | Get task progress metrics | `spec_id` |

#### Blocker Operations

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `task-block` | Block a task | `spec_id`, `task_id`, `reason`, `metadata` |
| `task-unblock` | Unblock a task | `spec_id`, `task_id`, `resolution` |
| `task-list-blocked` | List all blocked tasks | `spec_id` |

---

### Journal Operations

Tools for tracking decisions and audit trails.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `journal-add` | Add a journal entry | `spec_id`, `task_id`, `entry`, `entry_type` |
| `journal-list` | List journal entries | `spec_id`, `task_id`, `limit` |
| `journal-list-unjournaled` | Find tasks without journals | `spec_id` |
| `journal-bulk-add` | Add multiple entries | `spec_id`, `entries` |

---

### Planning & Phase Operations

Tools for planning and phase management.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `plan-format` | Format plan as markdown | `spec_id` |
| `plan-report-time` | Report time metrics | `spec_id` |
| `phase-list` | List all phases | `spec_id` |
| `phase-check-complete` | Check if phase is complete | `spec_id`, `phase_id` |
| `phase-report-time` | Report phase time metrics | `spec_id`, `phase_id` |

---

### Documentation & Code Analysis

Tools for querying codebase documentation and code intelligence.

#### Documentation Statistics

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `doc-stats` | Get documentation status | `path` |
| `doc-scope` | Get scoped documentation | `path`, `view` |
| `doc-search` | Search documentation | `query`, `scope` |
| `doc-dependencies` | Get dependency graph | `path` |
| `doc-context` | Get comprehensive context | `path` |

#### Code Analysis

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `code-find-class` | Find class definition | `class_name`, `path` |
| `code-find-function` | Find function definition | `function_name`, `path` |
| `code-trace-calls` | Trace function calls | `function_name` |
| `code-impact-analysis` | Analyze change impact | `path` |
| `code-get-callers` | Get function callers | `function_name` |
| `code-get-callees` | Get functions called | `function_name` |

---

### Testing Operations

Tools for test discovery and execution with pytest.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `test-run` | Run tests | `path`, `markers`, `verbose` |
| `test-run-quick` | Run quick smoke tests | `path` |
| `test-run-unit` | Run unit tests only | `path` |
| `test-discover` | Discover available tests | `path`, `format` |
| `test-presets` | List available presets | — |
| `test-consult` | Consult external AI for debugging | `error`, `context`, `tool` |

---

### Review Operations

Tools for AI-powered spec review and fidelity checking.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `review-spec` | AI review of specification | `spec_id`, `focus_areas` |
| `review-parse-feedback` | Parse review feedback | `feedback_text` |
| `fidelity-check` | Compare implementation vs spec | `spec_id`, `task_id` |
| `verify-check` | Run verification steps | `spec_id`, `phase_id` |
| `verify-fix` | Fix verification issues | `spec_id`, `phase_id` |

---

### PR Workflow Operations

Tools for creating pull requests with spec context.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `pr-context` | Gather PR creation context | `spec_id` |
| `pr-create` | Create a pull request | `spec_id`, `title`, `body`, `base_branch` |
| `task-create-commit` | Create commit for task | `spec_id`, `task_id`, `message` |

---

### Session & Context Operations

Tools for managing development sessions and context.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `session-work-mode` | Get/set work mode | `mode` |
| `session-generate-marker` | Generate session marker | — |
| `session-context` | Get context usage info | — |

---

### Provider Operations

Tools for LLM provider management.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `provider-list` | List available providers | — |
| `provider-status` | Get provider status | `provider_name` |
| `provider-execute` | Execute with specific provider | `provider_name`, `prompt` |

---

### Discovery & Capability Operations

Tools for discovering available functionality.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `tool-list` | List all available tools | `category` |
| `tool-list-categories` | List tool categories | — |
| `tool-get-schema` | Get tool JSON schema | `tool_name` |
| `capability-get` | Get capability info | `capability_name` |
| `capability-negotiate` | Negotiate capabilities | `client_capabilities` |
| `sdd-server-capabilities` | Get server capabilities | — |

---

### Environment Operations

Tools for environment setup and verification.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `sdd-verify-toolchain` | Verify development tools | — |
| `sdd-init-workspace` | Initialize specs workspace | `path` |
| `sdd-cache-manage` | Manage cache | `action` |
| `env-verify-environment` | Verify environment setup | — |

---

## Common Parameters

Parameters that appear across multiple tools:

| Parameter | Type | Description |
|-----------|------|-------------|
| `spec_id` | string | Unique specification identifier |
| `task_id` | string | Task identifier within a spec |
| `phase_id` | string | Phase identifier within a spec |
| `status` | string | Status filter: `pending`, `in_progress`, `completed`, `blocked` |
| `path` | string | File or directory path |
| `limit` | integer | Maximum results to return |
| `cursor` | string | Pagination cursor for next page |
| `dry_run` | boolean | Preview changes without applying |

---

## Feature Flags

Some tools are gated by feature flags:

| Flag | Tools Enabled | Default |
|------|---------------|---------|
| `response_contract_v2` | All (standardized responses) | Enabled |
| `environment_tools` | `sdd-verify-toolchain`, `env-*` | Enabled |
| `spec_helpers` | `spec-reconcile-state`, `spec-update-frontmatter` | Enabled |
| `planning_tools` | `plan-*`, `phase-*` | Enabled |

Check available flags:
```
mcp__foundry-mcp__capability-get with capability_name="feature_flags"
```

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
mcp__foundry-mcp__task-next with spec_id="my-feature-spec"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "task": {
      "task_id": "task-001",
      "description": "Implement user authentication",
      "status": "pending",
      "dependencies": []
    },
    "context": {
      "phase": "Phase 1: Core Implementation",
      "blocking_tasks": []
    }
  },
  "meta": { "version": "response-v2" }
}
```

### Validate a Spec

```
mcp__foundry-mcp__spec-validate with spec_id="my-feature-spec"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "valid": true,
    "issues": [],
    "stats": {
      "total_tasks": 15,
      "completed_tasks": 3,
      "pending_tasks": 12
    }
  },
  "meta": { "version": "response-v2" }
}
```

### Run Tests

```
mcp__foundry-mcp__test-run with path="tests/unit" markers="quick"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "passed": 42,
    "failed": 0,
    "skipped": 2,
    "duration": 3.5,
    "failures": []
  },
  "meta": { "version": "response-v2" }
}
```

---

## Related Documentation

- **[05-Response Contract](./05-response-contract.md)** — Response envelope details
- **[06-Workflows](./06-workflows.md)** — Tool usage in workflows
- **[08-Integration Patterns](./08-integration-patterns.md)** — How skills use these tools

---

*[Back to Index](./README.md)*
