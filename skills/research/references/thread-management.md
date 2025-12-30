# Thread Management

Persistent conversation threads across research workflows.

## Thread Ownership

**foundry-mcp is the single source of truth** for all thread data. The skill never caches or stores thread state locally.

## Thread ID Format

Pattern: `^thread-[a-zA-Z0-9]{8,16}$`

Examples:
- Valid: `thread-abc123def456`
- Invalid: `thread-`, `abc123`, `thread_abc`

**Invalid ID Handling:**
```
Error: INVALID_THREAD_ID
Message: "Thread ID must match pattern thread-[alphanumeric]"
```

## Thread Lifecycle

```
created → active → archived → deleted
   │         │         │
   │         │         └→ Soft delete (recoverable for 30 days)
   │         └→ No activity for 7 days
   └→ First message sent
```

| State | Description | Can Resume | Can Delete |
|-------|-------------|------------|------------|
| `created` | Thread initialized, no messages | Yes | Yes |
| `active` | Has messages, recently used | Yes | Yes |
| `archived` | Inactive >7 days, auto-archived | Yes | Yes |
| `deleted` | Soft deleted, pending purge | No | N/A |

## MCP Operations

### thread-list
```bash
mcp__plugin_foundry_foundry-mcp__research action="thread-list" limit=10 status="active"
```

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `limit` | No | 10 | Max threads to return |
| `status` | No | all | Filter: active, archived, all |

### thread-get
```bash
mcp__plugin_foundry_foundry-mcp__research action="thread-get" thread_id="thread-abc123"
```

Returns full thread with message history.

### thread-delete
```bash
mcp__plugin_foundry_foundry-mcp__research action="thread-delete" thread_id="thread-abc123"
```

## Deletion Confirmation Gate

Before deleting, always confirm:

```
AskUserQuestion:
"Delete thread {thread_id}? This removes {n} messages."
Options:
- "Yes, delete permanently"
- "No, keep thread"
- "Archive instead"
```

## Surfacing Threads as Interactive Choices

When listing threads, present as selectable options:

```
AskUserQuestion:
"Select a thread to resume:"
Options:
- "thread-abc123: 'API design discussion' (3 messages, 2h ago)"
- "thread-def456: 'Performance analysis' (7 messages, 1d ago)"
- "Start new thread"
- "Cancel"
```

## Storage Interface

### Read Operations
- **Always** read from MCP on resume (no local cache)
- Thread context loaded fresh each session

### Write Operations
- Messages persisted to MCP after each exchange
- Metadata updated (timestamps, turn count)

### Cache Rules
- No persistent client-side cache
- In-memory context valid only for current session
- Refresh on each `/research thread-* resume`

## Data Handling Policy

| Aspect | Policy |
|--------|--------|
| Retention | Active: indefinite, Archived: 90 days, Deleted: 30 days |
| Hard delete | After 30-day soft delete period |
| Soft delete | Recoverable via support request |
| Redaction | User can request message redaction without thread deletion |

## Auto-Cleanup Behavior

| Trigger | Action |
|---------|--------|
| No activity 7 days | Auto-archive |
| Archived 90 days | Auto-delete (soft) |
| Soft deleted 30 days | Hard delete (permanent) |
| User request | Immediate soft delete |

Auto-cleanup runs daily. Users notified before archive/delete via system message.
