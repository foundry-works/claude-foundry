# Integration Patterns

> How claude-foundry skills interface with foundry-mcp tools.

---

## Integration Overview

claude-foundry is a Claude Code plugin that provides skills orchestrating foundry-mcp backend tools.

```
┌─────────────────────────────────────────────────────────────┐
│                       claude-foundry                        │
│                    (Claude Code Plugin)                     │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  sdd-plan   │  │  sdd-next   │  │ run-tests   │  ...   │
│  │   skill     │  │   skill     │  │   skill     │        │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘        │
│         │                │                │                 │
└─────────┼────────────────┼────────────────┼─────────────────┘
          │                │                │
          │     MCP Protocol (tool calls)   │
          │                │                │
┌─────────▼────────────────▼────────────────▼─────────────────┐
│                       foundry-mcp                           │
│                    (MCP Server Backend)                     │
│                                                             │
│  spec-create  task-next  test-run  doc-stats  pr-create    │
│  spec-validate task-complete ...                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Skill-to-Tool Mapping

Each skill orchestrates specific foundry-mcp tools:

### sdd-plan

Creates specifications using planning tools.

| Router | Action | Purpose in Skill |
|--------|--------|------------------|
| `code` | `doc-stats` | Check documentation availability |
| `code` | `find-class` | Find existing class patterns |
| `code` | `find-function` | Find existing implementations |
| `authoring` | `spec-create` | Create the specification |
| `spec` | `validate` | Validate before saving |
| `spec` | `fix` | Auto-fix issues |

### sdd-next

Finds and prepares the next actionable task.

| Router | Action | Purpose in Skill |
|--------|--------|------------------|
| `spec` | `find` | Discover active specs |
| `spec` | `list` | List specs by status |
| `task` | `next` | Find next actionable task |
| `task` | `prepare` | Get full task context |

### sdd-update

Tracks progress and updates task status.

| Router | Action | Purpose in Skill |
|--------|--------|------------------|
| `task` | `update-status` | Change task status |
| `task` | `complete` | Mark task done |
| `task` | `start` | Mark task in progress |
| `journal` | `add` | Add journal entries |
| `task` | `block` | Record blockers |
| `task` | `unblock` | Clear blockers |

### sdd-validate

Validates specifications and identifies issues.

| Router | Action | Purpose in Skill |
|--------|--------|------------------|
| `spec` | `validate` | Run validation |
| `spec` | `fix` | Auto-fix issues |
| `spec` | `stats` | Get statistics |

### sdd-fidelity-review

Reviews implementation against specifications.

| Router | Action | Purpose in Skill |
|--------|--------|------------------|
| `review` | `fidelity` | Compare code vs spec |
| `verification` | `execute` | Run verification |
| `code` | `impact-analysis` | Check impact |

### sdd-plan-review

AI-powered specification review.

| Router | Action | Purpose in Skill |
|--------|--------|------------------|
| `review` | `spec-review` | AI review of spec |
| `review` | `parse-feedback` | Parse review results |
| `journal` | `add` | Record review findings |

### sdd-pr

Creates pull requests with spec context.

| Router | Action | Purpose in Skill |
|--------|--------|------------------|
| `pr` | `context` | Gather PR information |
| `pr` | `create` | Create the PR |
| `journal` | `list` | Get decision history |
| `task` | `list` | Get completed tasks |

### run-tests

Executes tests with debugging support.

| Router | Action | Purpose in Skill |
|--------|--------|------------------|
| `test` | `run` | Run tests |
| `test` | `run-quick` | Quick smoke tests |
| `test` | `discover` | Find available tests |
| `test` | `presets` | List test presets |
| `provider` | `execute` | External AI debugging |

### doc-query

Queries codebase documentation.

| Router | Action | Purpose in Skill |
|--------|--------|------------------|
| `code` | `doc-stats` | Check doc status |
| `code` | `find-class` | Find class definitions |
| `code` | `find-function` | Find functions |
| `code` | `get-callers` | Find function callers |
| `code` | `get-callees` | Find function callees |
| `code` | `trace-calls` | Build call graphs |
| `code` | `impact-analysis` | Analyze change impact |

---

## Tool Invocation Patterns

### Basic Pattern

Skills invoke tools using the router+action pattern:

```
mcp__plugin_foundry_foundry-mcp__<router> action="<action>"
```

With parameters:

```
mcp__plugin_foundry_foundry-mcp__task action="next" spec_id="my-feature"
```

### Named Parameters

All parameters are passed by name:

```
mcp__plugin_foundry_foundry-mcp__authoring action="spec-create"
  name="feature-2025-12-04"
  title="My Feature"
  description="Implement the feature"
```

### Response Handling

Skills check the response structure:

```python
response = mcp__plugin_foundry_foundry-mcp__task(action="next", spec_id="my-feature")

if response["success"]:
    task = response["data"]["task"]
    # Process task
else:
    error = response["error"]
    remediation = response["data"].get("remediation", "")
    # Handle error
```

---

## Context Flow

### How Skills Gather Context

```
┌───────────────┐
│ User Request  │
└───────┬───────┘
        │
        ▼
┌───────────────┐    ┌───────────────┐
│ Skill Loads   │───►│ doc-stats     │  Check docs available
└───────┬───────┘    └───────────────┘
        │
        ▼
┌───────────────┐    ┌───────────────┐
│ Get Context   │───►│ doc-scope     │  Get module info
│               │    │ doc-search    │  Find related code
│               │    │ spec-get      │  Get spec details
└───────┬───────┘    └───────────────┘
        │
        ▼
┌───────────────┐
│ Process with  │
│ Full Context  │
└───────────────┘
```

### Progressive Disclosure

Skills load context progressively to manage token usage:

1. **Initial check** — Minimal queries (e.g., `doc-stats`)
2. **Focused context** — Specific queries (e.g., `doc-scope` for one module)
3. **Deep context** — Full context only when needed (e.g., `doc-context`)

---

## Error Handling in Skills

### Success Path

```
response = tool_call()
if response["success"]:
    # Extract data
    data = response["data"]
    # Continue processing
```

### Error Path

```
response = tool_call()
if not response["success"]:
    error_code = response["data"].get("error_code")
    remediation = response["data"].get("remediation")

    if error_code == "NOT_FOUND":
        # Handle missing resource
    elif error_code == "VALIDATION_ERROR":
        # Handle validation issue
    else:
        # Report error to user
```

### Warning Handling

```
if response["meta"].get("warnings"):
    for warning in response["meta"]["warnings"]:
        # Log or display warning to user
```

### Pagination Handling

```
all_results = []
cursor = None

while True:
    response = tool_call(cursor=cursor)
    all_results.extend(response["data"]["items"])

    pagination = response["meta"].get("pagination", {})
    if not pagination.get("has_more"):
        break
    cursor = pagination["cursor"]
```

---

## Best Practices

### DO: Use MCP Tools Directly

```
# Correct: Use MCP tool
response = mcp__plugin_foundry_foundry-mcp__spec(action="get", spec_id="my-feature")
spec = response["data"]["spec"]
```

### DON'T: Read Spec Files Directly

```
# Wrong: Reading raw JSON
with open("specs/active/my-feature.json") as f:
    spec = json.load(f)  # Don't do this!
```

**Why:** Tools provide validation, context, and consistent response format.

### DO: Check Response Success

```
response = mcp__plugin_foundry_foundry-mcp__task(action="next", spec_id="my-feature")
if response["success"]:
    task = response["data"]["task"]
```

### DON'T: Assume Success

```
# Wrong: No success check
response = mcp__plugin_foundry_foundry-mcp__task(action="next", spec_id="my-feature")
task = response["data"]["task"]  # May fail if success: false
```

### DO: Use Helper Tools for Context

```
# Get relevant context
doc_response = mcp__plugin_foundry_foundry-mcp__code(action="find-function", symbol="authenticate")
# Use context for planning
```

### DON'T: Read All Files Manually

```
# Wrong: Manual file reading
files = glob.glob("src/auth/**/*.py")
for f in files:
    content = read_file(f)  # Inefficient, no structure
```

---

## Workflow Integration Examples

### Example: Planning Workflow

```python
# 1. Check documentation
doc_status = mcp__plugin_foundry_foundry-mcp__code(action="doc-stats")

if not doc_status["success"]:
    # Handle missing docs
    return

# 2. Get planning context
context = mcp__plugin_foundry_foundry-mcp__code(action="find-function", symbol="authenticate")

# 3. Create specification
spec_response = mcp__plugin_foundry_foundry-mcp__authoring(
    action="spec-create",
    name="auth-feature-2025-12-04",
    title="Authentication Feature"
)

# 4. Validate
validation = mcp__plugin_foundry_foundry-mcp__spec(
    action="validate",
    spec_id="auth-feature-2025-12-04"
)

if not validation["data"]["valid"]:
    # Fix issues
    mcp__plugin_foundry_foundry-mcp__spec(action="fix", spec_id="auth-feature-2025-12-04")
```

### Example: Task Execution

```python
# 1. Find next task
next_task = mcp__plugin_foundry_foundry-mcp__task(action="next", spec_id="auth-feature-2025-12-04")

if not next_task["success"] or not next_task["data"].get("task"):
    # No tasks available
    return "All tasks completed or blocked"

task = next_task["data"]["task"]

# 2. Get full context
context = mcp__plugin_foundry_foundry-mcp__task(
    action="prepare",
    spec_id="auth-feature-2025-12-04",
    task_id=task["task_id"]
)

# 3. Implement task (skill guides implementation)
# ...

# 4. Complete task
mcp__plugin_foundry_foundry-mcp__task(
    action="complete",
    spec_id="auth-feature-2025-12-04",
    task_id=task["task_id"],
    completion_note="Implemented login endpoint with JWT"
)
```

---

## Skill Configuration

Skills can read configuration from foundry-mcp via the server router:

```
# Get server capabilities
caps = mcp__plugin_foundry_foundry-mcp__server(action="capabilities")

# Check health status
health = mcp__plugin_foundry_foundry-mcp__health(action="check")
```

---

## Related Documentation

- **[04-Tool Reference](./04-tool-reference.md)** — Complete tool catalog
- **[06-Workflows](./06-workflows.md)** — Common patterns
- **[05-Response Contract](./05-response-contract.md)** — Response handling

### claude-foundry Skill Documentation
- [../claude_code_best_practices/04-skills.md](../claude_code_best_practices/04-skills.md) — How to build skills
- [../claude_code_best_practices/11-mcp-consumption.md](../claude_code_best_practices/11-mcp-consumption.md) — MCP in Claude Code

---

*[Back to Index](./README.md)*
