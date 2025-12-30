---
name: research
description: AI-powered research with four workflows (chat, consensus, thinkdeep, ideate) and persistent thread management
argument-hint: [chat|consensus|thinkdeep|ideate|threads|thread-*] [prompt]
---

# Research Command

Entry point for AI-powered research workflows.

## Argument Routing

Parse `$ARGUMENTS` and route based on first token:

### Explicit Workflow Keywords

If first token matches a workflow keyword, invoke skill with that workflow:

| Token | Action |
|-------|--------|
| `chat` | `Skill(foundry:research) workflow=chat prompt="$REST"` |
| `consensus` | `Skill(foundry:research) workflow=consensus prompt="$REST"` |
| `thinkdeep` | `Skill(foundry:research) workflow=thinkdeep prompt="$REST"` |
| `ideate` | `Skill(foundry:research) workflow=ideate prompt="$REST"` |

### Thread Resume (thread-* ID)

If first token matches regex `^thread-[a-zA-Z0-9]+$`:

```
Skill(foundry:research) resume=true thread_id="$1" prompt="$REST"
```

### Thread Management

If first token is `threads`:

| Sub-command | Action |
|-------------|--------|
| `threads list` | `mcp__...research action="thread-list" limit=10` |
| `threads get <id>` | `mcp__...research action="thread-get" thread_id="$3"` |
| `threads delete <id>` | `mcp__...research action="thread-delete" thread_id="$3"` |

### Auto-Route (Natural Prompt)

If arguments don't match above patterns, use auto-routing:

```
Skill(foundry:research) auto_route=true prompt="$ARGUMENTS"
```

The skill analyzes intent signals and selects the appropriate workflow.

### No Arguments

Prompt user with `AskUserQuestion`:

```
"What kind of research would you like to do?"
Options:
- "Chat - iterative conversation"
- "Consensus - multiple perspectives"
- "ThinkDeep - systematic investigation"
- "Ideate - creative brainstorming"
- "Manage threads"
```

## Behavior Matrix

| Input Pattern | Expected Response | Error Handling |
|---------------|-------------------|----------------|
| `/research chat How does X work?` | Chat workflow with prompt | - |
| `/research consensus Best approach for Y?` | Consensus workflow | `NO_MODELS_AVAILABLE` |
| `/research thread-abc123 follow up` | Resume thread with new prompt | `THREAD_NOT_FOUND` |
| `/research threads list` | List recent threads | - |
| `/research threads delete thread-xyz` | Delete thread (confirm first) | `THREAD_NOT_FOUND` |
| `/research What is the best way to...` | Auto-route (likely consensus) | - |
| `/research` | Workflow selection prompt | - |

## Ambiguity Gate

When auto-routing detects ambiguous intent, present clarification:

```
AskUserQuestion:
"I detected multiple possible workflows for your query. Which approach?"
Options:
- "Chat - single model, iterative refinement"
- "Consensus - get multiple perspectives"
- "ThinkDeep - systematic investigation"
- "Ideate - creative brainstorming"
```

Include brief rationale for why ambiguity was detected.
