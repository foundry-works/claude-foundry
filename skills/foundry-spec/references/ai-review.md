# AI Plan Review

Get AI-powered feedback on markdown plans before converting to JSON specs.

## Running a Plan Review

```bash
mcp__plugin_foundry_foundry-mcp__plan action="review" plan_path="specs/.plans/feature-name.md"
```

All 6 dimensions are always assessed: Completeness, Architecture, Sequencing, Feasibility, Risk, Clarity.

## Plan Review Output Location

Reviews are saved to: `specs/.plan-reviews/<plan-name>-review.md`

## Plan Review Output Structure

```markdown
# Plan Review: {Plan Name}

## Summary
{Overall assessment}

## Dimensions

### Completeness
**Score:** {1-5}
**Findings:**
- {Finding 1}
- {Finding 2}
**Recommendations:**
- {Recommendation 1}

### Architecture
**Score:** {1-5}
**Findings:**
- {Finding 1}
**Recommendations:**
- {Recommendation 1}

[... additional dimensions ...]

## Critical Blockers
1. {Blocker requiring resolution before proceeding}

## Questions for Stakeholder
1. {Question needing clarification}

## Verdict
{APPROVED | NEEDS_REVISION | BLOCKED}
```

## Spec Review (Spec-vs-Plan Comparison)

The spec review compares the JSON spec against its source plan to identify translation gaps — places where the plan's intent was lost, diluted, or silently modified during spec creation. It requires a linked `plan_path` in spec metadata.

```bash
mcp__plugin_foundry_foundry-mcp__review action="spec-review" spec_id="{spec-id}"
```

### Comparison Dimensions

The spec review evaluates 7 comparison dimensions:

1. **Coverage** — Are all plan phases and tasks represented in the spec?
2. **Fidelity** — Does the spec accurately reflect the plan's intent?
3. **Success Criteria Mapping** — Are plan success criteria traceable to spec tasks?
4. **Constraints Preserved** — Are plan constraints reflected in the spec?
5. **Risks Preserved** — Are plan risks carried forward into the spec?
6. **Open Questions Preserved** — Are unresolved questions tracked in the spec?
7. **Undocumented Additions** — Does the spec contain work not in the plan?

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `review_path` | string | Path to the saved review output |
| `verdict` | string | `aligned`, `deviation`, or `incomplete` |

### Verdict Values

| Verdict | Meaning |
|---------|---------|
| `aligned` | Spec faithfully implements the plan |
| `deviation` | Spec diverges from the plan in significant ways |
| `incomplete` | Spec is missing plan elements |

### Spec Review Output Location

Spec reviews are saved to: `specs/.spec-reviews/{spec_id}-spec-review.md`

## Iteration Workflow

```
1. Create plan → plan action="create" (MANDATORY)
2. Fill in content → Read + Edit
3. Run AI review → plan action="review"
4. Read feedback → specs/.plan-reviews/
5. Revise plan → address critical/high findings
6. Re-review → plan action="review"
7. ↻ Repeat steps 4-6 until no critical/high issues remain
8. HUMAN APPROVAL GATE → present clean plan + review summary
   - [approved] → continue
   - [revise] → back to step 2
   - [abort] → exit
9. Convert to spec → authoring action="spec-create"
10. Run spec review → review action="spec-review"
11. Read findings → parse-feedback
12. Fix spec → address critical/high findings (authoring/task MCP tools)
13. Re-review → review action="spec-review"
14. ↻ Repeat steps 11-13 until no critical/high issues remain
15. Present clean spec + review summary to user
```

## Common Review Feedback

| Issue | Typical Recommendation |
|-------|------------------------|
| Vague tasks | Add specific acceptance criteria |
| Missing dependencies | Explicit task ordering needed |
| No verification | Add test requirements |
| Scope creep | Split into multiple specs |
| Missing risks | Add risk assessment section |
