# Response Contract

> Understanding the standardized response-v2 envelope used by all foundry-mcp tools.

---

## Standard Envelope Structure

All foundry-mcp tools return responses in the standardized v2 envelope format:

```json
{
  "success": true,
  "data": { ... },
  "error": null,
  "meta": {
    "version": "response-v2",
    "request_id": "req_abc123",
    "warnings": [],
    "pagination": { ... },
    "rate_limit": { ... },
    "telemetry": { ... }
  }
}
```

This consistent structure enables:
- Predictable response parsing
- Machine-readable error handling
- LLM-friendly output
- Consistent metadata propagation

---

## Field Semantics

### Top-Level Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `success` | boolean | YES | Whether the operation completed successfully |
| `data` | object | YES | Operation payload (empty object `{}` if no data) |
| `error` | string \| null | YES | Error message when `success` is `false` |
| `meta` | object | YES | Metadata including version and operational info |

### The `success` Field

- `true` — Operation completed successfully
- `false` — Operation failed; check `error` and `data.error_*` fields

**Important:** `success: true` with `meta.warnings` is valid—warnings are non-fatal issues.

### The `data` Field

Contains the operation's payload. Structure varies by tool:

```json
// Task query response
{
  "data": {
    "tasks": [...],
    "total_count": 15,
    "filtered_count": 5
  }
}

// Spec get response
{
  "data": {
    "spec": { ... },
    "hierarchy": { ... }
  }
}

// Empty but successful
{
  "data": {}
}
```

### The `error` Field

- `null` when `success: true`
- Human-readable error string when `success: false`

```json
{
  "success": false,
  "error": "Spec not found: invalid-spec-id",
  "data": { ... }
}
```

---

## Metadata Fields

The `meta` object contains operational metadata:

```json
{
  "meta": {
    "version": "response-v2",
    "request_id": "req_abc123",
    "warnings": ["Deprecated field used"],
    "pagination": {
      "cursor": "eyJvZmZzZXQiOjIwfQ==",
      "has_more": true,
      "total_count": 150,
      "page_size": 20
    },
    "rate_limit": {
      "limit": 100,
      "remaining": 95,
      "reset_at": "2025-12-04T12:00:00Z"
    },
    "telemetry": {
      "duration_ms": 45,
      "cache_hit": true
    }
  }
}
```

### Metadata Field Semantics

| Key | Required | Description |
|-----|----------|-------------|
| `version` | YES | Response schema version (`response-v2`) |
| `request_id` | SHOULD | Correlation ID for logs/traces |
| `warnings` | SHOULD | Non-fatal issues (array of strings) |
| `pagination` | MAY | Cursor-based pagination info |
| `rate_limit` | MAY | Rate limiting status |
| `telemetry` | MAY | Performance metrics |

---

## Error Response Structure

When `success` is `false`, the `data` object contains structured error context:

```json
{
  "success": false,
  "data": {
    "error_code": "VALIDATION_ERROR",
    "error_type": "validation",
    "remediation": "Provide a non-empty spec_id parameter",
    "details": {
      "field": "spec_id",
      "constraint": "required",
      "received": null
    }
  },
  "error": "Validation failed: spec_id is required",
  "meta": {
    "version": "response-v2",
    "request_id": "req_xyz789"
  }
}
```

### Standard Error Fields

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `error_code` | SHOULD | string | Machine-readable classification (SCREAMING_SNAKE_CASE) |
| `error_type` | SHOULD | string | Error category for routing |
| `remediation` | SHOULD | string | Actionable guidance to fix |
| `details` | MAY | object | Context-specific error info |

### Error Type Categories

| error_type | HTTP Analog | Description | Retry? |
|------------|-------------|-------------|--------|
| `validation` | 400 | Invalid input data | No, fix input |
| `authentication` | 401 | Invalid credentials | No, re-authenticate |
| `authorization` | 403 | Insufficient permissions | No |
| `not_found` | 404 | Resource doesn't exist | No |
| `conflict` | 409 | State conflict (duplicate) | Maybe |
| `rate_limit` | 429 | Too many requests | Yes, after delay |
| `feature_flag` | 403 | Feature not enabled | No |
| `internal` | 500 | Server-side error | Yes, with backoff |
| `unavailable` | 503 | Service unavailable | Yes, with backoff |

---

## Common Error Codes

| Error Code | Description | Remediation |
|------------|-------------|-------------|
| `VALIDATION_ERROR` | Invalid input parameters | Check parameter values |
| `NOT_FOUND` | Resource doesn't exist | Verify spec_id/task_id |
| `ALREADY_EXISTS` | Duplicate resource | Use different ID |
| `INVALID_STATE` | Operation not allowed in current state | Check lifecycle state |
| `DEPENDENCY_ERROR` | Unmet dependencies | Complete dependent tasks first |
| `RATE_LIMIT_EXCEEDED` | Too many requests | Wait and retry |
| `FEATURE_DISABLED` | Feature flag not enabled | Enable feature flag |
| `INTERNAL_ERROR` | Server error | Report issue, retry |

---

## Edge Case Handling

### Empty But Successful Results

```json
{
  "success": true,
  "data": {
    "tasks": [],
    "total_count": 0
  },
  "error": null,
  "meta": { "version": "response-v2" }
}
```

When no results match, return:
- `success: true` (query succeeded)
- Empty arrays/zero counts in `data`
- `error: null`

### Partial Work Completed

```json
{
  "success": true,
  "data": {
    "processed": 8,
    "failed": 2,
    "failures": [
      { "task_id": "task-003", "error": "Missing file" }
    ]
  },
  "error": null,
  "meta": {
    "version": "response-v2",
    "warnings": ["2 tasks failed to process"]
  }
}
```

When work partially succeeds:
- `success: true` (overall operation continued)
- Details in `data` about what failed
- `meta.warnings` for visibility

### Blocked or Incomplete State

```json
{
  "success": true,
  "data": {
    "task_id": "task-001",
    "status": "blocked",
    "blocker": "Waiting for API approval"
  },
  "error": null,
  "meta": {
    "version": "response-v2",
    "warnings": ["Task is blocked and cannot proceed"]
  }
}
```

---

## Pagination Pattern

For operations that return many results:

```json
{
  "success": true,
  "data": {
    "tasks": [ ... ],  // 20 items
  },
  "meta": {
    "version": "response-v2",
    "pagination": {
      "cursor": "eyJvZmZzZXQiOjIwfQ==",
      "has_more": true,
      "total_count": 150,
      "page_size": 20
    }
  }
}
```

### Pagination Fields

| Field | Description |
|-------|-------------|
| `cursor` | Opaque string to get next page |
| `has_more` | Whether more results exist |
| `total_count` | Total matching items (if available) |
| `page_size` | Items per page |

### Getting Next Page

```
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id="my-feature" cursor="eyJvZmZzZXQiOjIwfQ=="
```

---

## Response Handling in Skills

### Checking Success

```python
response = mcp__plugin_foundry_foundry-mcp__spec(action="get", spec_id="my-feature")

if response["success"]:
    spec = response["data"]["spec"]
    # Process spec
else:
    error = response["error"]
    remediation = response["data"].get("remediation", "")
    # Handle error
```

### Checking Warnings

```python
if response["meta"].get("warnings"):
    for warning in response["meta"]["warnings"]:
        # Log or display warning
```

### Handling Pagination

```python
all_tasks = []
cursor = None

while True:
    response = mcp__plugin_foundry_foundry-mcp__task(
        action="query",
        spec_id="my-feature",
        cursor=cursor
    )
    all_tasks.extend(response["data"]["tasks"])

    if not response["meta"]["pagination"]["has_more"]:
        break
    cursor = response["meta"]["pagination"]["cursor"]
```

---

## Versioning

### Current Version

All tools currently use `response-v2`:

```json
{
  "meta": {
    "version": "response-v2"
  }
}
```

### Version Negotiation

Clients can declare supported versions via capability negotiation:

```
mcp__plugin_foundry_foundry-mcp__server action="capability-negotiate"
  client_capabilities={"response_contract": "v2"}
```

### Feature Flags

The `response_contract_v2` feature flag controls the response format:

| Flag Status | Behavior |
|-------------|----------|
| Enabled (default) | Use v2 envelope |
| Disabled | Legacy format (deprecated) |

---

## Best Practices

### For Tool Consumers

1. **Always check `success`** before accessing `data`
2. **Handle `meta.warnings`** — they may indicate issues
3. **Use `error_code`** for programmatic error handling
4. **Follow `remediation`** hints in errors
5. **Implement pagination** for list operations

### For Integration Development

1. **Never assume data shape** — always validate
2. **Log `request_id`** for debugging
3. **Respect `rate_limit`** metadata
4. **Handle partial success** gracefully

---

## Examples

### Successful Task Query

```json
{
  "success": true,
  "data": {
    "tasks": [
      {
        "task_id": "task-001",
        "description": "Implement login",
        "status": "completed"
      },
      {
        "task_id": "task-002",
        "description": "Implement logout",
        "status": "pending"
      }
    ],
    "total_count": 2
  },
  "error": null,
  "meta": {
    "version": "response-v2",
    "request_id": "req_12345"
  }
}
```

### Validation Error

```json
{
  "success": false,
  "data": {
    "error_code": "VALIDATION_ERROR",
    "error_type": "validation",
    "remediation": "Spec ID must be non-empty and contain only alphanumeric characters and hyphens",
    "details": {
      "field": "spec_id",
      "constraint": "pattern",
      "pattern": "^[a-z0-9-]+$",
      "received": "Invalid Spec ID!"
    }
  },
  "error": "Validation failed: spec_id contains invalid characters",
  "meta": {
    "version": "response-v2",
    "request_id": "req_67890"
  }
}
```

### Not Found Error

```json
{
  "success": false,
  "data": {
    "error_code": "NOT_FOUND",
    "error_type": "not_found",
    "remediation": "Check that the spec_id exists in specs/active/ or specs/pending/",
    "details": {
      "resource_type": "spec",
      "resource_id": "nonexistent-spec"
    }
  },
  "error": "Spec not found: nonexistent-spec",
  "meta": {
    "version": "response-v2"
  }
}
```

---

## Related Documentation

- **[04-Tool Reference](./04-tool-reference.md)** — Tool catalog
- **[11-Troubleshooting](./11-troubleshooting.md)** — Error resolution

---

*[Back to Index](./README.md)*
