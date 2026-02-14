# tmux Integration for Claude Foundry

## Overview

This document explores how tmux session control could enhance Claude Foundry's spec-driven development workflows. The goal is to extend Foundry's existing session management with persistent, observable execution environments.

## Current State

Claude Foundry already has sophisticated session management:
- Session state persistence across `/clear` boundaries
- Pause/resume semantics with automatic triggers
- Background subagent spawning via `Task(run_in_background=true)`
- Batch operations for parallel task execution

**What's missing:** True process persistence, interactive debugging, dev server lifecycle management, and live user observability.

## What tmux Would Add

| Capability | Current State | With tmux |
|------------|---------------|-----------|
| Process persistence | Tied to Claude session | Survives disconnect |
| Dev servers | Manual start/stop | Managed lifecycle |
| Interactive debugging | Limited | Full REPL/debugger access |
| User visibility | Poll status | `tmux attach` live view |
| True parallelism | Subagents | Multiple independent sessions |
| Long-running processes | Timeout limits | Unlimited duration |

## Integration Points

### 1. Persistent Autonomous Sessions

Wrap `foundry-implement --auto` in tmux for sessions that survive terminal disconnection:

```
tmux: foundry-auto
┌─────────────────────────────────────┐
│ foundry-implement --auto            │
│ [spec: auth-system]                 │
│ ✓ Task 1.1 completed                │
│ ✓ Task 1.2 completed                │
│ → Task 1.3 in_progress...           │
└─────────────────────────────────────┘
```

User can disconnect, return hours later, `tmux attach` to see progress.

### 2. Dev Environment Orchestration

When implementing features, tasks often need running services:

```
tmux: dev-environment
┌──────────────┬──────────────┐
│ npm run dev  │ flask run    │
│ (frontend)   │ (api)        │
├──────────────┼──────────────┤
│ docker-compose up           │
│ (postgres, redis)           │
└─────────────────────────────┘
```

`foundry-implement` could spin these up before coding, tear down after. Task metadata could declare required services.

### 3. Test Watch Mode

Instead of running tests once, maintain a persistent watcher:

```
tmux: test-watcher
┌─────────────────────────────────────┐
│ pytest-watch src/                   │
│ [waiting for changes...]            │
│                                     │
│ PASSED: 47 | FAILED: 0 | 2.3s       │
└─────────────────────────────────────┘
```

Claude implements code, watcher gives instant feedback, no re-run overhead.

### 4. Multi-Spec Parallel Execution

Different specs in different tmux windows for true parallelism:

```
tmux windows:
[0] spec:auth-system   [1] spec:api-refactor   [2] spec:ui-redesign
```

Each runs `--auto` independently. User monitors via window switching.

### 5. Deep Research Background Sessions

`foundry-research --deep` can take a long time. Run it in tmux:

```
tmux: research
┌─────────────────────────────────────┐
│ deep-research: "auth patterns 2025" │
│ Phase 2/4: Synthesizing sources...  │
│ Sources analyzed: 23/40             │
└─────────────────────────────────────┘
```

Continue other work while research runs in background.

### 6. Interactive Debugging

When tests encounter failures, drop into a real debugger:

```
tmux: debug
┌─────────────────────────────────────┐
│ (Pdb) l                             │
│  12     def authenticate(user):     │
│  13  →      token = gen_token(user) │
│  14         return token            │
│ (Pdb) p user                        │
│ {'id': None, 'email': '...'}        │
└─────────────────────────────────────┘
```

Actually step through code, inspect state, test hypotheses interactively.

### 7. Live Monitoring Dashboard

A dedicated tmux layout for observability:

```
tmux: foundry-dashboard
┌─────────────────┬───────────────────┐
│ SESSION STATUS  │ TASK PROGRESS     │
│ spec: auth      │ ████████░░ 80%    │
│ state: active   │ 8/10 tasks done   │
│ errors: 0       │                   │
├─────────────────┼───────────────────┤
│ LOGS                                │
│ [14:23] Task 1.8 completed          │
│ [14:24] Starting Task 1.9...        │
└─────────────────────────────────────┘
```

## Existing tmux MCP Servers

Several tmux MCP implementations exist:

| Project | Stars | Status | Notes |
|---------|-------|--------|-------|
| [nickgnd/tmux-mcp](https://github.com/nickgnd/tmux-mcp) | 196 | Most mature | 4 contributors, active maintenance |
| [michael-abdo/tmux-claude-mcp-server](https://github.com/michael-abdo/tmux-claude-mcp-server) | 11 | Specialized | Hierarchical Claude orchestration |
| [jonrad/tmux-mcp](https://github.com/jonrad/tmux-mcp) | 6 | POC | Not production-ready |

**Recommendation:** Start with `nickgnd/tmux-mcp` for straightforward integration, or evaluate `michael-abdo/tmux-claude-mcp-server` if hierarchical agent orchestration is desired.

## Implementation Approach

### Option A: New Skill (`foundry-session`)

A dedicated skill for managing tmux-backed execution environments:

```
/foundry-session start <spec-id>     # Create tmux session for spec
/foundry-session attach              # Show attach command
/foundry-session status              # Check all running sessions
/foundry-session stop <spec-id>      # Tear down session
```

Integrates with existing `session-config` state machine.

### Option B: Enhance Existing Skills

Add tmux support as flags/options to existing skills:

```
/foundry-implement --auto --tmux     # Run in persistent tmux session
# Test watcher can be set up as a tmux pane
/foundry-research deep --tmux        # Background research session
```

### Option C: Environment Declarations in Specs

Add environment requirements to task/phase metadata:

```json
{
  "phase_id": "phase-1",
  "environment": {
    "services": ["frontend:npm run dev", "api:flask run"],
    "watch": "pytest-watch src/",
    "layout": "dev-standard"
  }
}
```

Foundry automatically manages tmux lifecycle based on spec requirements.

## MCP Tool Design

Core tools needed (wrapping tmux-mcp or custom):

```
foundry-tmux:
  - session-create(spec_id, layout?)     # Create session with optional layout
  - session-destroy(spec_id)             # Clean up session
  - session-list()                       # List active sessions
  - pane-send(spec_id, pane, command)    # Send command to pane
  - pane-capture(spec_id, pane)          # Get pane output
  - service-start(spec_id, name, cmd)    # Start named service in pane
  - service-stop(spec_id, name)          # Stop service
  - layout-apply(spec_id, layout)        # Apply predefined layout
```

Predefined layouts:
- `minimal` - Single pane for main work
- `dev-standard` - Main + services + logs
- `debug` - Main + debugger + test watcher
- `dashboard` - Status + progress + logs

## Integration with Existing Session Management

Foundry's `session-config` already tracks:
- `session_id`, `spec_id`, `state`
- `started_at`, `paused_at`
- `pause_reason`, `completed_tasks`, `current_task`
- `error_count`, `context_percentage`

tmux integration would add:
- `tmux_session` - tmux session name
- `tmux_layout` - active layout
- `services` - running service panes
- `attach_command` - command for user to attach

Pause/resume semantics map naturally:
- `pause` → tmux session persists, work stops
- `resume` → reconnect to existing tmux session, continue work
- `end` → tear down tmux session, clean up

## User Experience

### Starting Work
```
User: /foundry-implement --auto --tmux
Claude: Created tmux session 'foundry-auth-system'
        To observe: tmux attach -t foundry-auth-system
        Starting autonomous implementation...
```

### Checking Status
```
User: /foundry-session status
Claude: Active sessions:
        - foundry-auth-system (spec: auth-system)
          State: active, Tasks: 8/10, Errors: 0
          Attach: tmux attach -t foundry-auth-system
```

### User Takes Over
```
$ tmux attach -t foundry-auth-system
[Now seeing Claude's work in real-time, can type to interact]
```

## Open Questions

1. **Session naming** - Use spec ID? Allow custom names?
2. **Cleanup policy** - Auto-destroy on spec completion? Keep for review?
3. **Multiple Claude instances** - One spec per Claude, or orchestrate multiple?
4. **Error recovery** - If tmux session dies, how to recover state?
5. **Resource limits** - Max concurrent sessions? Pane limits?

## Next Steps

1. Evaluate `nickgnd/tmux-mcp` integration complexity
2. Prototype `foundry-session` skill with basic start/stop/status
3. Test with `foundry-implement --auto` workflow
4. Design service declaration schema for specs
5. Build predefined layouts for common workflows
