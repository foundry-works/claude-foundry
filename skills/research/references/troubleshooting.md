# Troubleshooting

Common issues and resolutions for research workflows.

## Thread Issues

### THREAD_NOT_FOUND
**Symptom:** Error when resuming with thread ID.

**Causes:**
- Thread was deleted or archived
- Typo in thread ID
- Thread expired (soft deleted after 30 days)

**Resolution:**
1. Verify thread ID format: `thread-[alphanumeric]`
2. List available threads: `/research threads list`
3. Start new thread if original unavailable

### Thread Corruption
**Symptom:** Thread loads but history is incomplete or garbled.

**Resolution:**
1. Export available messages
2. Delete corrupted thread
3. Start new thread with exported context

## Consensus Issues

### NO_MODELS_AVAILABLE
**Symptom:** All requested models failed to respond.

**Causes:**
- API rate limits exceeded
- Network connectivity issues
- Model service outage

**Resolution:**
1. Wait 30 seconds and retry
2. Try with fewer models
3. Fall back to single-model chat

### Synthesis Failure
**Symptom:** Models responded but synthesis failed.

**Causes:**
- Responses too divergent to synthesize
- Model outputs malformed

**Resolution:**
1. View raw responses with `strategy="all_responses"`
2. Manually summarize if needed
3. Retry with different models

## ThinkDeep Issues

### Investigation Loops
**Symptom:** Same hypotheses re-examined repeatedly.

**Causes:**
- Insufficient evidence to progress
- Circular dependencies in hypotheses

**Resolution:**
1. Review hypothesis list for duplicates
2. Add new evidence sources manually
3. Force conclusion with current confidence levels:
   ```
   "Please conclude with current findings, even if confidence is low"
   ```

### Max Depth Exceeded
**Symptom:** `MAX_DEPTH_EXCEEDED` error.

**Resolution:**
1. Summarize findings so far
2. Start new investigation thread with narrower scope
3. Use `depth="quick"` for focused analysis

## Ideate Issues

### Stuck in Phase
**Symptom:** Unable to progress to next ideation phase.

**Causes:**
- Insufficient ideas generated
- User approval gate not answered

**Resolution:**
1. Check for pending user prompt
2. Force phase transition:
   ```
   `/research ideate phase=convergent`
   ```
3. Add ideas manually before converging

### Idea Overflow
**Symptom:** Too many ideas to manage (>50).

**Resolution:**
1. Force early convergence
2. Apply quick filter: "Keep only top 20 by feasibility"
3. Split into multiple ideation sessions by theme

## General Issues

### Context Bloat
**Symptom:** Slow responses, context limit warnings.

**Causes:**
- Long thread history
- Multiple large responses

**Resolution:**
1. Start new thread (history persisted in old thread)
2. Use `/clear` then resume
3. Summarize thread before continuing:
   ```
   "Summarize our discussion so far, then continue"
   ```

### Timeouts
**Symptom:** No response within expected time.

**Causes:**
- Complex query processing
- Model service delays
- Network issues

**Resolution:**
1. Wait up to 60 seconds for complex queries
2. Retry with simpler prompt
3. Check service status if persistent

### Invalid Parameters
**Symptom:** Parameter validation errors.

**Resolution:**
1. Check parameter types (string vs array)
2. Verify required parameters present
3. Review MCP Contract in SKILL.md

## Error Code Reference

| Code | Meaning | Action |
|------|---------|--------|
| `THREAD_NOT_FOUND` | Thread ID invalid or deleted | List threads, start new |
| `NO_MODELS_AVAILABLE` | All models failed | Retry or reduce model count |
| `MAX_DEPTH_EXCEEDED` | ThinkDeep hit depth limit | Conclude or narrow scope |
| `INVALID_PHASE` | Unknown ideate phase | Use: divergent, convergent, selection, elaboration |
| `INVALID_THREAD_ID` | Malformed thread ID | Check format: thread-[alphanumeric] |
