---
name: research
description: AI-powered research skill with four workflows - chat (single-model conversation), consensus (multi-model synthesis), thinkdeep (systematic investigation), and ideate (creative brainstorming). Supports persistent threads for conversation continuity.
---

# Research Skill

## Overview

- **Purpose:** AI-powered research with multiple reasoning strategies
- **Scope:** Four workflows, persistent thread management
- **Entry:** `/research` command or `Skill(foundry:research)`

### Flow

> `[x?]`=decision `(GATE)`=user approval `→`=sequence

```
- **Entry** → [explicit?] → Dispatch | [thread-id?] → Resume
  | [threads?] → ThreadMgmt | [no args?] → (GATE) | AutoRoute
- **Dispatch** → Execute → PersistThread → Response + thread_id
```

## MCP Tooling

| Router | Actions |
|--------|---------|
| `research` | `chat`, `consensus`, `thinkdeep`, `ideate`, `thread-list`, `thread-get`, `thread-delete` |

## MCP Contract

| Action | Required | Optional | Errors |
|--------|----------|----------|--------|
| `chat` | `prompt` | `thread_id`, `model` | `THREAD_NOT_FOUND` |
| `consensus` | `prompt` | `models[]`, `strategy` | `NO_MODELS_AVAILABLE` |
| `thinkdeep` | `prompt` | `thread_id`, `depth` | `MAX_DEPTH_EXCEEDED` |
| `ideate` | `prompt` | `thread_id`, `phase` | `INVALID_PHASE` |
| `thread-*` | `thread_id` | `limit` | `THREAD_NOT_FOUND` |

## Workflow Selection

| Signal | Workflow |
|--------|----------|
| Follow-up, iteration | `chat` |
| Multiple perspectives | `consensus` |
| Complex problem | `thinkdeep` |
| Brainstorming | `ideate` |

## User Gates

- No args: workflow selection
- Ambiguous: clarify before auto-route
- Consensus: strategy selection
- Ideate: phase transition

## Output Formats

| Workflow | Response |
|----------|----------|
| `chat` | `{response, thread_id, model}` |
| `consensus` | `{responses[], synthesis, strategy}` |
| `thinkdeep` | `{findings[], confidence, thread_id}` |
| `ideate` | `{ideas[], phase, selected[]}` |

## References

- [Chat](./references/chat-workflow.md) | [Consensus](./references/consensus-workflow.md)
- [ThinkDeep](./references/thinkdeep-workflow.md) | [Ideate](./references/ideate-workflow.md)
- [Threads](./references/thread-management.md) | [Auto-Route](./references/auto-routing.md)
