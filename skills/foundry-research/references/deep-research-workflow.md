# Deep Research Workflow

Multi-phase iterative research with supervisor-driven multi-agent web source synthesis.

## Contents

- When to Use
- Architecture
- Execution Model
- MCP Operations (start, status, report, list, delete, evaluate)
- Session States
- Status Monitoring
- Clarification Handling
- User Gates
- Output Format
- Error Handling
- Integration with Session Management

## When to Use

- Complex questions requiring multiple sources
- Topics needing comprehensive coverage
- Research that benefits from iterative refinement
- Questions where surface-level answers are insufficient

## Architecture

```
Query → Clarification → Brief → Supervision → Synthesis
                                    ↑    ↓
                                    └────┘ (iterative gap-fill, up to 6 rounds)
                                       │
                              [parallel topic researchers]
```

### Phases

- **CLARIFICATION**: Binary gate — request clarification from user OR confirm understanding and proceed
- **BRIEF**: Enrich raw query into structured research brief with scope boundaries
- **SUPERVISION**: Supervisor decomposes into topics, delegates to parallel researchers, assesses coverage, iterates to fill gaps
- **SYNTHESIS**: Generate final report from compressed findings

## Execution Model

Deep research runs **in the background** by default:

1. **Start** - Initiate research session, returns `research_id`
2. **Poll** - Long-poll status until complete
3. **Report** - Retrieve final synthesized report

This allows long-running research without blocking the conversation.

## MCP Operations

### Start Research

```bash
mcp__plugin_foundry_foundry-mcp__research action="deep-research" deep_research_action="start" query="Your research question" max_iterations=3 max_sub_queries=5 max_sources_per_query=5
```

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `query` | Yes | - | Main research question |
| `deep_research_action` | No | `start` | Sub-action: start, continue, resume |
| `max_iterations` | No | 3 | Maximum refinement iterations |
| `max_sub_queries` | No | 5 | Sub-queries generated per iteration |
| `max_sources_per_query` | No | 5 | Sources fetched per sub-query |
| `follow_links` | No | true | Extract and follow links from sources |
| `max_concurrent` | No | 3 | Parallel operations |
| `timeout_per_operation` | No | 360 | Seconds per web fetch |
| `task_timeout` | No | - | Overall task timeout (seconds) |
| `provider_id` | No | - | Provider to use for research operations |

### Check Status

```bash
mcp__plugin_foundry_foundry-mcp__research action="deep-research-status" research_id="research-abc123"
```

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `research_id` | Yes | - | Research session ID |
| `wait` | No | false | Long-poll: block until state changes or timeout |
| `wait_timeout` | No | - | Max seconds to block before returning |

Returns:
```json
{
  "research_id": "research-abc123",
  "status": "in_progress",
  "changed": true,
  "progress": {
    "phase": "SUPERVISION",
    "supervision_round": 2,
    "max_supervision_rounds": 6,
    "topics_completed": 3,
    "topics_total": 5,
    "sources_processed": 28
  },
  "started_at": "2025-01-02T10:00:00Z"
}
```

### Get Report

```bash
mcp__plugin_foundry_foundry-mcp__research action="deep-research-report" research_id="research-abc123"
```

Returns comprehensive report with:
- Executive summary
- Key findings
- Source citations
- Confidence assessments
- Follow-up questions

### List Sessions

```bash
mcp__plugin_foundry_foundry-mcp__research action="deep-research-list" limit=10 completed_only=false
```

### Delete Session

```bash
mcp__plugin_foundry_foundry-mcp__research action="deep-research-delete" research_id="research-abc123"
```

### Evaluate Research

```bash
mcp__plugin_foundry_foundry-mcp__research action="deep-research-evaluate" research_id="research-abc123"
```

Returns 5-dimension quality scores (1-5 scale) with rationales:
- **Depth** — how thoroughly topics were explored
- **Source Quality** — reliability and diversity of sources
- **Analytical Rigor** — strength of reasoning and evidence use
- **Completeness** — coverage of the research question
- **Groundedness** — how well conclusions are supported by sources

## Session States

```
started → in_progress → completed
              │              │
              └→ failed      └→ (report available)
```

| State | Description | Can Resume | Has Report |
|-------|-------------|------------|------------|
| `started` | Initialized, entering clarification gate | Yes | No |
| `in_progress` | Actively researching (includes supervision rounds) | Yes | Partial |
| `completed` | All iterations done | No | Yes |
| `failed` | Error during research | Yes | Partial |

## Status Monitoring

Use long-poll to monitor progress without rapid polling.

### Long-Poll Behavior

Call `deep-research-status` with `wait=true`. The server blocks until a state change occurs or the wait timeout elapses, then returns the new state.

- `"changed": true` — progress occurred, report new state to user and poll again
- `"changed": false` — timeout elapsed with no progress, tell user "still working" and poll again

### Stall Rule

If **2 consecutive** responses return `"changed": false`, offer user options via AskUserQuestion:

```
"Research is still running but hasn't made visible progress. Options:"
- "Keep waiting"
- "Cancel and try a different query"
- "Narrow the query scope"
```

### Flow

```
1. Start research → notify user "Starting deep research, this may take a few minutes..."
2. Call deep-research-status with wait=true
3. On return:
   - changed=true → report progress to user, go to step 2
   - changed=false → tell user "still working", go to step 2
   - 2 consecutive changed=false → AskUserQuestion with options
   - status=completed → fetch and present report
   - status=failed → show error, offer retry
```

## Clarification Handling

The workflow may pause during the CLARIFICATION phase to request user input. When status shows `phase=CLARIFICATION` with a pending question:

1. Present the clarification question to the user via AskUserQuestion
2. Pass their response back to the workflow:

```bash
mcp__plugin_foundry_foundry-mcp__research action="deep-research" deep_research_action="continue" research_id="research-abc123" query="User's clarification response"
```

3. Resume status monitoring

## User Gates

| Checkpoint | Prompt |
|------------|--------|
| Start | "Starting deep research on '{query}'. This may take a few minutes." |
| Progress | "Research {phase} phase. {topics_completed}/{topics_total} topics, {sources} sources analyzed." |
| Clarification | Present clarification question from workflow to user |
| Complete | Present report with source citations |
| Failed | "Research encountered an error: {error}. Retry?" |

## Output Format

```json
{
  "research_id": "research-abc123",
  "query": "Original research question",
  "status": "completed",
  "phase": "SYNTHESIS",
  "report": {
    "summary": "Executive summary of findings",
    "findings": [
      {
        "topic": "Sub-topic",
        "content": "Detailed findings",
        "confidence": "high|medium|low",
        "sources": ["url1", "url2"]
      }
    ],
    "topic_research_results": [
      {
        "topic": "Topic name",
        "summary": "Per-topic summary",
        "sources_used": 8,
        "confidence": "high"
      }
    ],
    "contradictions": [
      {
        "claim_a": "Source A says X",
        "claim_b": "Source B says Y",
        "resolution": "Analysis of discrepancy"
      }
    ],
    "sources": [
      {
        "url": "https://...",
        "title": "Source title",
        "relevance": 0.85
      }
    ],
    "content_fidelity": {
      "degraded": false,
      "warnings": []
    },
    "follow_up_questions": [
      "Suggested follow-up question 1",
      "Suggested follow-up question 2"
    ]
  },
  "evaluation": {
    "depth": { "score": 4, "rationale": "..." },
    "source_quality": { "score": 4, "rationale": "..." },
    "analytical_rigor": { "score": 3, "rationale": "..." },
    "completeness": { "score": 4, "rationale": "..." },
    "groundedness": { "score": 5, "rationale": "..." }
  },
  "metadata": {
    "supervision_rounds": 3,
    "topics_researched": 5,
    "total_sources": 42,
    "duration_seconds": 180
  }
}
```

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `RESEARCH_NOT_FOUND` | Invalid research_id | List sessions, use valid ID |
| `RESEARCH_TIMEOUT` | Task exceeded timeout | Retry with higher `task_timeout` |
| `NO_SOURCES_FOUND` | Query too narrow/obscure | Broaden query terms |
| `RATE_LIMITED` | Too many concurrent requests | Wait and retry |
| `CONTEXT_OVERFLOW` | Context window exceeded — content may be truncated | Narrow query or reduce sources; check `content_fidelity` in report |
| `CLARIFICATION_PENDING` | Workflow paused for user clarification | Check status for pending question, respond via `continue` |

## Integration with Session Management

Deep research sessions are listed alongside threads in unified session management:

```
foundry-research sessions list
```

Shows both:
- Chat/thinkdeep/ideate threads (`thread-*`)
- Deep research sessions (`research-*`)

Sessions can be filtered:
- `type=threads` - Only conversation threads
- `type=research` - Only deep research sessions
- `type=all` - Both (default)
