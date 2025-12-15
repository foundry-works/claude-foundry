# SDD-Plan-Review Reference

Detailed workflows, examples, and edge cases for the sdd-plan-review skill.

## Table of Contents

- [Review Dimensions](#review-dimensions)
- [Feedback Categories](#feedback-categories)
- [Interpreting Consensus](#interpreting-consensus)
- [Output Format](#output-format)
- [Error Handling](#error-handling)
- [Model Coordination](#model-coordination)
- [Configuration](#configuration)
- [Example Invocations](#example-invocations)
- [Best Practices](#best-practices)

---

## Review Dimensions

Every review examines specs across 6 dimensions:

### 1. Completeness
- All sections present
- Sufficient detail for implementation
- No gaps in requirements
- Dependencies fully specified

### 2. Clarity
- Unambiguous descriptions
- Specific acceptance criteria
- Clear success metrics
- Well-defined boundaries

### 3. Feasibility
- Realistic time estimates
- Achievable dependencies
- Available resources
- Technical viability

### 4. Architecture
- Sound design decisions
- Proper abstractions
- Scalability considerations
- Integration patterns

### 5. Risk Management
- Risks identified
- Edge cases covered
- Mitigation strategies
- Fallback plans

### 6. Verification
- Comprehensive test plan
- Quality gates defined
- Acceptance criteria testable
- Coverage expectations

---

## Feedback Categories

Findings are organized by category:

### Missing Information
Gaps where spec needs more detail:
- Undefined behavior for edge cases
- Missing error handling requirements
- Incomplete API contracts
- Unspecified data formats

### Design Concerns
Architectural choices needing reconsideration:
- Tight coupling between components
- Missing abstraction layers
- Scalability limitations
- Maintenance burden

### Risk Flags
Security, performance, or reliability risks:
- Authentication/authorization gaps
- Data exposure vulnerabilities
- Performance bottlenecks
- Single points of failure

### Feasibility Questions
Estimate or dependency concerns:
- Unrealistic timelines
- Missing expertise
- External dependencies
- Resource constraints

### Enhancement Suggestions
Non-critical improvements:
- Code organization
- Documentation additions
- Testing improvements
- Developer experience

### Clarification Requests
Ambiguous language or requirements:
- Multiple interpretations possible
- Conflicting requirements
- Undefined terms
- Unclear priorities

---

## Interpreting Consensus

### Reviewer Consensus Levels

**Strong Consensus:**
- Multiple reviewers identified same issue
- High confidence in finding
- Prioritize for immediate attention

**Diverse Perspectives:**
- Different reviewers raised different concerns
- Indicates breadth of review coverage
- All perspectives valuable

**Conflicting Views:**
- Reviewers disagree on approach
- Both perspectives documented
- Requires human judgment

### Priority Assignment

| Level | Criteria | Action |
|-------|----------|--------|
| **CRITICAL** | Security vulnerabilities, blockers | Address immediately |
| **HIGH** | Design flaws, missing info | Address before implementation |
| **MEDIUM** | Improvements, unclear requirements | Consider addressing |
| **LOW** | Nice-to-have enhancements | Note for future |

### Consensus Detection Logic

Consensus is calculated based on **responding models only**, not requested models.

| Models Requested | Models Responded | Finding Agreement | Consensus Level |
|------------------|------------------|-------------------|-----------------|
| 3 | 3 | 3/3 agree | Strong |
| 3 | 3 | 2/3 agree | High |
| 3 | 2 | 2/2 agree | High (with reduced confidence note) |
| 3 | 2 | 1/2 flags | Moderate |
| 3 | 1 | 1/1 flags | Low (single-model) |
| 3 | 0 | N/A | Review failed |

**Edge case - partial failures:** If only 1 model responds due to timeouts or errors, all its findings are tagged as "single-model, lower confidence" and deprioritized by 1 level.

### Single-Model Weight Reduction

Single-model findings are reduced by exactly 1 priority level:

| Original Priority | Adjusted Priority |
|-------------------|-------------------|
| CRITICAL | HIGH |
| HIGH | MEDIUM |
| MEDIUM | LOW |
| LOW | LOW (floor) |

**Why:** Single-model findings lack cross-validation. The priority reduction reflects reduced confidence, not dismissal.

### Synthesis Algorithm

When merging findings from multiple models:

**1. Deduplication**
Findings addressing the same issue (same file/component + similar recommendation) are merged into one entry, listing all models that flagged it.

**2. Priority Assignment**
- Multi-model findings: Use the highest priority any model assigned
- Single-model findings: Apply weight reduction (see table above)

**3. Conflict Resolution**
- If models give conflicting recommendations for the same issue, document both under "Conflicting Views"
- Do NOT attempt to resolve conflicts - surface them for human judgment
- Conflicting findings retain their original priority levels

**4. Category Assignment**
Use the category from the first model that flagged the issue. If models disagree on category, use the more severe category.

---

## Output Format

### Markdown Report Structure

```markdown
## Feedback Summary

### Risk Flags (CRITICAL)
1. Missing authentication on admin endpoints
   Priority: CRITICAL | Flagged by: gemini, codex
   Impact: Unauthorized access to sensitive operations
   Recommendation: Add JWT validation middleware

### Feasibility Questions (HIGH)
2. Time estimates may be unrealistic for Phase 2
   Priority: HIGH | Flagged by: codex
   Impact: Timeline risk - OAuth integration underestimated
   Recommendation: Revisit estimates (+50% suggested)

### Design Concerns (MEDIUM)
3. Tight coupling between auth and user services
   Priority: MEDIUM | Flagged by: gemini
   Impact: Maintenance complexity
   Recommendation: Introduce interface abstraction

### Enhancement Suggestions (LOW)
4. Consider adding request logging
   Priority: LOW | Flagged by: cursor-agent
   Impact: Debugging and monitoring
   Recommendation: Add structured logging middleware
```

### JSON Summary Structure

```json
{
  "spec_id": "user-auth-001",
  "review_type": "full",
  "models_consulted": 3,
  "summary": {
    "critical": 1,
    "high": 3,
    "medium": 5,
    "low": 2
  },
  "findings": [
    {
      "id": 1,
      "category": "risk_flag",
      "priority": "critical",
      "title": "Missing authentication on admin endpoints",
      "flagged_by": ["gemini", "codex"],
      "impact": "Unauthorized access to sensitive operations",
      "recommendation": "Add JWT validation middleware"
    }
  ],
  "consensus": {
    "strong_consensus": 1,
    "diverse_perspectives": 4,
    "conflicting_views": 0
  }
}
```

### Output File Locations

| Type | Path | Purpose |
|------|------|---------|
| Markdown | `specs/.reviews/{spec-id}-review-{type}.md` | Human-readable report |
| JSON | `specs/.reviews/{spec-id}-review-{type}.json` | Machine-readable data |

---

## Error Handling

| Error | Behavior | Impact |
|-------|----------|--------|
| Timeout (>120s) | Retry with backoff | Longer wait |
| Rate limit (429) | Sequential fallback | Slower execution |
| Auth failure | Skip tool, continue | Reduced confidence |
| Parse failure | Use other responses | No impact if >=2 succeed |
| All tools fail | Return error with troubleshooting | Review fails |

### Graceful Degradation

| Models Succeeded | Confidence Level | Behavior |
|------------------|------------------|----------|
| 3/3 | High | Full review |
| 2/3 | Medium | Review continues with note |
| 1/3 | Low | Warning issued |
| 0/3 | Failed | Review fails with troubleshooting |

### Common Troubleshooting

**No tools available:**
1. Check `mcp__plugin_foundry_foundry-mcp__review action="list-tools"` output
2. Verify provider configuration
3. Ensure API keys are set

**Timeout issues:**
1. Check network connectivity
2. Reduce review scope (use `quick` type)
3. Retry with increased timeout

**Parse failures:**
1. Check provider response format
2. Verify spec JSON is valid
3. Review error logs for details

---

## Model Coordination

### Parallel Execution

- All available models consulted simultaneously
- ThreadPoolExecutor manages concurrent requests
- Each model gets same prompt with spec content
- Independent failures don't block others

### Execution Timeline

```
T+0s:   Dispatch reviews to all available models
T+60s:  Fast models typically complete
T+120s: Timeout for slow models
T+130s: Synthesize available responses
T+150s: Generate report and summary
```

### Resource Management

- Timeout per model: 120 seconds
- Max review time: 300 seconds (5 minutes)
- Min models required: 1
- Recommended models: 2

---

## Configuration

### Default Values

| Parameter | Value | Description |
|-----------|-------|-------------|
| `min_models` | 1 | Minimum models for valid review |
| `recommended_models` | 2 | Recommended for quality feedback |
| `timeout_per_model` | 120s | Per-model timeout |
| `max_review_time` | 300s | Total review timeout |
| `summary_max_lines` | 50 | Max lines in conversation summary |

### Output Isolation Settings

| Setting | Value | Description |
|---------|-------|-------------|
| `write_to_files` | true | Always write to files |
| `suppress_verbose_output` | true | Don't echo full review |
| `return_format` | summary | Return summary to conversation |
| `ensure_directory` | true | Create `.reviews/` if missing |
| `overwrite_existing` | true | Replace previous reviews |

---

## Example Invocations

### Quick Review (Time-Constrained)

```bash
Skill(foundry:sdd-plan-review) "Run a quick review on spec simple-feature-001. Focus on completeness and clarity."
```

### Full Review (Complex Spec)

```bash
Skill(foundry:sdd-plan-review) "Run a full review on spec user-auth-001. Analyze all 6 dimensions and provide comprehensive feedback."
```

### Security Review (Auth System)

```bash
Skill(foundry:sdd-plan-review) "Run a security review on spec payment-processing-001. Focus on risk management, auth, and data handling."
```

### Feasibility Review (Tight Timeline)

```bash
Skill(foundry:sdd-plan-review) "Run a feasibility review on spec q4-migration-001. Check estimates and dependencies for Phase 1."
```

---

## Best Practices

### DO

- Verify available tools before starting review
- Select review type based on spec risk/complexity
- Present findings organized by priority
- Note reviewer consensus in report
- Provide actionable recommendations
- Return report paths for full context
- Handle partial failures gracefully

### DON'T

- Skip tool verification step
- Use full review for trivial specs
- Ignore single-model findings entirely
- Modify specifications (use `sdd-modify`)
- Echo full review content to conversation
- Make up findings not from reviewer output
- Override priority based on personal judgment

### When to Escalate

Consider escalating to human review when:
- All models return conflicting views
- Critical security findings detected
- Architectural decisions affect multiple systems
- Estimates significantly exceed resources
- Compliance or regulatory concerns raised

---

## Integration with SDD Workflow

### Typical Workflow

```
1. sdd-plan creates specification
2. sdd-plan-review provides feedback (this skill)
3. sdd-modify applies approved changes
4. sdd-plan-review re-reviews if needed
5. sdd-next begins implementation
```

### Handoff to sdd-modify

After review, findings can flow to `sdd-modify`:
1. Human reviews findings and approves changes
2. `sdd-modify` applies approved modifications
3. Re-review confirms changes addressed findings

### Skip Conditions

Skip plan review when:
- Spec has <5 tasks and uses standard patterns
- Implementation already underway
- Exploratory/prototype work
- Simple CRUD operations
- Time constraints don't allow review
