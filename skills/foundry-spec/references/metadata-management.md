# Metadata Management Reference

Structured metadata actions for enriching specs with constraints, risks, open questions, and success criteria extracted from approved plans.

## When to Use

Use these actions during the **plan-to-spec transition** (after plan approval, during spec creation). The workflow is:

1. **Extract** — Identify constraints, risks, questions, and success criteria from the approved plan
2. **Populate** — Add each item to the spec via the authoring MCP actions below
3. **Validate** — Spec review automatically checks that plan metadata is preserved

## Actions

### constraint-add

Add a constraint to the spec (technical, business, or resource limitation).

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="constraint-add" spec_id="{spec-id}" text="Must maintain backward compatibility with v2 API"
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `spec_id` | Yes | Target specification ID |
| `text` | Yes | Constraint description |

### constraint-list

List all constraints on a spec.

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="constraint-list" spec_id="{spec-id}"
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `spec_id` | Yes | Target specification ID |

### risk-add

Add a risk entry with likelihood, impact, and mitigation strategy.

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="risk-add" spec_id="{spec-id}" description="OAuth provider rate limits during peak usage" likelihood="medium" impact="high" mitigation="Implement token caching with 5-minute TTL"
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `spec_id` | Yes | Target specification ID |
| `description` | Yes | Risk description |
| `likelihood` | Yes | `low`, `medium`, or `high` |
| `impact` | Yes | `low`, `medium`, or `high` |
| `mitigation` | Yes | Mitigation strategy |

### risk-list

List all risks on a spec.

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="risk-list" spec_id="{spec-id}"
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `spec_id` | Yes | Target specification ID |

### question-add

Add an open question that needs resolution.

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="question-add" spec_id="{spec-id}" text="Which OAuth scopes are required for the admin flow?"
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `spec_id` | Yes | Target specification ID |
| `text` | Yes | Question text |

### question-list

List all open questions on a spec.

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="question-list" spec_id="{spec-id}"
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `spec_id` | Yes | Target specification ID |

### success-criterion-add

Add a measurable success criterion.

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="success-criterion-add" spec_id="{spec-id}" text="All protected endpoints return 401 without valid token"
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `spec_id` | Yes | Target specification ID |
| `text` | Yes | Success criterion text |

### success-criteria-list

List all success criteria on a spec.

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="success-criteria-list" spec_id="{spec-id}"
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `spec_id` | Yes | Target specification ID |

## Workflow Example

After plan approval, enrich the spec with metadata from the plan:

```bash
# 1. Create spec linked to the plan
mcp__plugin_foundry_foundry-mcp__authoring action="spec-create" name="feature-name" template="empty" plan_path="specs/.plans/feature-name.md" plan_review_path="specs/.plan-reviews/feature-name-review.md"

# 2. Add constraints from the plan
mcp__plugin_foundry_foundry-mcp__authoring action="constraint-add" spec_id="{spec-id}" text="Must maintain backward compatibility with v2 API"
mcp__plugin_foundry_foundry-mcp__authoring action="constraint-add" spec_id="{spec-id}" text="Maximum 200ms response time on auth endpoints"

# 3. Add risks from the plan
mcp__plugin_foundry_foundry-mcp__authoring action="risk-add" spec_id="{spec-id}" description="OAuth provider rate limits" likelihood="medium" impact="high" mitigation="Token caching with TTL"
mcp__plugin_foundry_foundry-mcp__authoring action="risk-add" spec_id="{spec-id}" description="Migration downtime for existing sessions" likelihood="low" impact="high" mitigation="Rolling deployment with session compatibility"

# 4. Add open questions
mcp__plugin_foundry_foundry-mcp__authoring action="question-add" spec_id="{spec-id}" text="Which OAuth scopes are required for admin flow?"

# 5. Add success criteria
mcp__plugin_foundry_foundry-mcp__authoring action="success-criterion-add" spec_id="{spec-id}" text="All protected endpoints return 401 without valid token"
mcp__plugin_foundry_foundry-mcp__authoring action="success-criterion-add" spec_id="{spec-id}" text="Token refresh completes within 100ms"

# 6. Verify metadata was captured
mcp__plugin_foundry_foundry-mcp__authoring action="constraint-list" spec_id="{spec-id}"
mcp__plugin_foundry_foundry-mcp__authoring action="risk-list" spec_id="{spec-id}"

# 7. Continue with phases, then spec review validates preservation
```

## Validation

When a spec has a linked `plan_path`, the spec review automatically checks that plan metadata (constraints, risks, open questions, success criteria) is preserved in the spec. Missing or divergent metadata will appear in the review findings.
