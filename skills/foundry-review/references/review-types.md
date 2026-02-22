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

## Spec Review (Spec-vs-Plan Comparison)

The spec review compares the JSON spec against its source plan to catch translation gaps. It evaluates 7 dimensions: coverage, fidelity, success criteria mapping, and preservation of constraints, risks, and open questions, plus undocumented additions.

- **Requires:** `plan_path` in spec metadata pointing to the source plan
- **Verdict:** `aligned`, `deviation`, or `incomplete`
- **Report path:** Available via `review_path` in the response
