# Review Types

All fidelity reviews use a single unified review flow. The scope (phase or task) determines the breadth of analysis.

## 1. Phase Review
**Scope:** Single phase within specification (typically 3-10 tasks)
**When to use:** Phase completion checkpoints, before moving to next phase
**Output:** Phase-specific fidelity report with per-task breakdown
**Best practice:** Use at phase boundaries to catch drift before starting next phase

## 2. Task Review
**Scope:** Individual task implementation (typically 1 file)
**When to use:** Critical task validation, complex implementation verification
**Output:** Task-specific compliance check with implementation comparison
**Best practice:** Use for high-risk tasks (auth, data handling, API contracts)

**Note:** For full spec reviews, run phase-by-phase reviews for better manageability and quality.

## Plan-Enhanced Review

When a spec has a linked `plan_path`, fidelity reviews are automatically enhanced with spec-vs-plan comparison. This is not a separate review type — it augments standard phase or task reviews with additional plan alignment checks.

- **Auto-triggered:** Detected from spec metadata; no user action needed
- **Response field:** `plan_enhanced` boolean indicates whether plan comparison was performed
- **Report path:** Available via `review_path` in the response
