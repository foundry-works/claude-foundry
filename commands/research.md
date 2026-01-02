---
name: research
description: AI-powered research with five workflows (chat, consensus, thinkdeep, ideate, deep) and persistent session management
argument-hint: [chat|consensus|thinkdeep|ideate|deep|sessions|thread-*|research-*] [prompt]
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
| `deep` | `Skill(foundry:research) workflow=deep prompt="$REST"` |

### Session Resume (thread-* or research-*)

If first token matches regex `^thread-[a-zA-Z0-9]+$`:

```
Skill(foundry:research) resume=true thread_id="$1" prompt="$REST"
```

If first token matches regex `^research-[a-zA-Z0-9]+$`:

```
# Check status and fetch report if completed
mcp__...research action="deep-research-status" research_id="$1"
# If completed, also fetch report:
mcp__...research action="deep-research-report" research_id="$1"
```

### Session Management

If first token is `sessions` (or legacy `threads`):

| Sub-command | Action |
|-------------|--------|
| `sessions list` | List both threads and research sessions |
| `sessions list type=threads` | `mcp__...research action="thread-list" limit=10` |
| `sessions list type=research` | `mcp__...research action="deep-research-list" limit=10` |
| `sessions get <id>` | Get thread or research session by ID |
| `sessions delete <id>` | Delete thread or research session (confirm first) |
| `sessions status <research-id>` | `mcp__...research action="deep-research-status" research_id="$3"` |
| `sessions report <research-id>` | `mcp__...research action="deep-research-report" research_id="$3"` |

Legacy `threads` command routes to `sessions` for backwards compatibility.

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
- "Deep - comprehensive web research"
- "Manage sessions"
```

## Behavior Matrix

| Input Pattern | Expected Response | Error Handling |
|---------------|-------------------|----------------|
| `/research chat How does X work?` | Chat workflow with prompt | - |
| `/research consensus Best approach for Y?` | Consensus workflow | `NO_MODELS_AVAILABLE` |
| `/research deep ML frameworks comparison` | Deep research (background) | `RESEARCH_TIMEOUT` |
| `/research thread-abc123 follow up` | Resume thread with new prompt | `THREAD_NOT_FOUND` |
| `/research research-xyz789` | Check status, show report if done | `RESEARCH_NOT_FOUND` |
| `/research sessions list` | List all threads and research sessions | - |
| `/research sessions delete thread-xyz` | Delete session (confirm first) | `*_NOT_FOUND` |
| `/research What is the best way to...` | Auto-route (likely consensus) | - |
| `/research comprehensive survey of...` | Auto-route (likely deep) | - |
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
- "Deep - comprehensive web research (may take a few minutes)"
```

Include brief rationale for why ambiguity was detected.

## Deep Research Execution

When `deep` workflow is selected:

1. **Start**: Invoke `mcp__...research action="deep-research" query="$PROMPT"`
2. **Notify**: "Starting deep research on '{query}'. This may take a few minutes."
3. **Poll**: Check `deep-research-status` every 15 seconds
4. **Progress**: Update user at 25%, 50%, 75% completion
5. **Report**: On completion, fetch and present `deep-research-report`

If user interrupts, offer to continue in background:
```
AskUserQuestion:
"Research is still running. What would you like to do?"
Options:
- "Continue polling (stay here)"
- "Run in background (I'll check later with /research research-{id})"
- "Cancel research"
```
