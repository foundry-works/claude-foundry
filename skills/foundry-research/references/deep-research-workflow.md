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

## Pre-Launch Confirmation Gate

Before starting a deep research session, **always** present the composed query and parameters to the user for approval. Deep research is the most resource-intensive workflow (up to 30 minutes), so the user must have a chance to review and refine before launch.

### Flow

1. Compose the research query from the user's request
2. Present via AskUserQuestion:
   - The **full query** that will be sent to the research engine
   - **Key parameters** if non-default (iterations, sub-queries, sources, profile)
   - Options: **"Start research" / "Edit query" / "Adjust parameters"**
3. If user selects "Edit query": accept their revised text and re-present for confirmation
4. If user selects "Adjust parameters": ask which parameters to change, re-present
5. Only call `deep-research` with `deep_research_action="start"` after explicit approval

### Example Gate Prompt

```
I'll run deep research with this query:

> "{composed_query}"

Parameters: {iterations} iterations, {sub_queries} sub-queries, {sources}/query sources, profile: {profile}

This typically takes up to 30 minutes.
```

Options: "Start research (Recommended)", "Edit query", "Adjust parameters"

## MCP Operations

**Parameter discipline:** Only set optional parameters when the user explicitly requests them. Do NOT fabricate values for `task_timeout`, `timeout_per_operation`, `max_concurrent`, or `provider_id`. These have server-managed defaults that are tuned for typical research sessions. Setting `task_timeout` arbitrarily (e.g., 600s) can prematurely kill research that legitimately takes up to 30 minutes.

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
| `max_concurrent` | No | - | Parallel operations (server-managed default) |
| `timeout_per_operation` | No | - | Seconds per web fetch (server-managed default) |
| `task_timeout` | No | - | Overall task timeout in seconds (server-managed default — do NOT set unless user requests it) |
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

## Expected Timing

Deep research is a multi-phase background process. Typical durations:

| Phase | Typical Duration | Notes |
|-------|-----------------|-------|
| CLARIFICATION | 5-15s | Quick; may pause for user input |
| BRIEF | 10-30s | Query enrichment |
| SUPERVISION | 2-10 min | **Longest phase.** Runs up to 6 rounds of parallel topic research with iterative gap-filling. Each round decomposes topics, fetches sources, assesses coverage, then identifies remaining gaps. Multiple rounds with no visible `changed` events is normal during source fetching. |
| SYNTHESIS | 15-60s | Compresses findings into final report |

**Total end-to-end: typically up to 30 minutes** depending on query breadth, number of sub-queries, and source availability.

The SUPERVISION phase is expected to be the longest. During this phase, parallel topic researchers are fetching and analyzing web sources. Status polls may return `"changed": false` multiple times — this does not mean the research is stuck. It means the long-poll timeout elapsed before a reportable state change occurred, while background work continues.

## Do NOT Supplement with External Searches

**While deep research is running, do NOT call WebSearch, WebFetch, tavily_search, tavily_extract, or any other web/research tools.** The deep research workflow handles all source gathering internally through its parallel topic researchers.

Only use external search tools if the user **explicitly** asks you to search independently (e.g., "also search for X while we wait").

When deep research seems slow, the correct response is to report the current status and keep polling — not to start your own parallel research. The deep research supervisor is designed to iteratively fill coverage gaps; doing your own searches duplicates effort and clutters the conversation.

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

**Important:** 2 consecutive `changed=false` is the threshold — do not take alternative action before reaching it. A single `changed=false` during the SUPERVISION phase is routine and expected.

### Flow

```
1. Present composed query + params to user for approval (Pre-Launch Gate)
2. On approval → start research → notify user "Starting deep research — this can take up to 30 minutes depending on query breadth."
3. Call deep-research-status with wait=true
4. On return:
   - changed=true → report progress to user, go to step 3
   - changed=false → tell user "still working", go to step 3
   - 2 consecutive changed=false → AskUserQuestion with options
   - status=completed → fetch and present report
   - status=failed → show error, offer retry
5. NEVER start your own web searches as a "supplement" or "alternative approach"
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
| **Pre-launch** | **Present composed query + parameters for user approval before calling start** |
| Start | "Starting deep research on '{query}'. This can take up to 30 minutes depending on query breadth." |
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
