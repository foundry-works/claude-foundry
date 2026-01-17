# Nested Bullet Shorthand (NBS) Flow Notation

Token-efficient flow diagram notation for skill documentation.

## Elements

| Element | Syntax | Purpose |
|---------|--------|---------|
| Sequence | `→` | Connect steps: `A → B → C` |
| Decision | `[condition?]` | Branch point: `[minimal?]` |
| Human gate | `(GATE: desc)` | User approval: `(GATE: approve plan)` |
| Entry/Exit | `**Bold**` | Boundaries: `**Entry**`, `**Exit**` |
| Fallback | `[else]` | Default branch: `[else] → manual` |
| Loop | `↻ back to X` | Return to step: `↻ back to Validate` |
| Reference | `§Section` | Link to section: `→ see §Verification` |
| Error | `[error?]` | Error path: `[blocked?] → HandleBlocker` |
| MCP call | backticks | Tool invocation: `` `task action="prepare"` `` |

## Structure

```
- **Entry** → FirstStep
  - [condition?] → BranchA → NextStep
  - [else] → BranchB
    - (GATE: user approval) → Continue
    - [error?] → HandleError
  - FinalStep → **Exit**
```

- Indentation shows nesting/branching depth
- Sibling bullets are alternatives (OR)
- Sequential arrows are steps (AND)

## Example: Test Runner

```
- **Entry** → RunTests
  - [pass?] → **Exit**
  - [fail?] → Investigate
    - `provider action="list"` → [tools available?]
      - [yes] → Consult → Synthesize
      - [no] → skip
    - Fix → Verify
      - [pass?] → **Exit**
      - [fail?] → ↻ back to Investigate
```

## Lint Checklist

- Use one root bullet per flow block; nested bullets represent branch paths and rejoin steps.
- Replace inline `|` with sibling bullets at the same indent.
- Decisions use `[condition?]` and fallbacks use `[else]` when needed.
- Every `→` must connect two steps; never start a line with `→`.
- Loops use `↻ back to StepName`.
- Gates are `(GATE: description)` as standalone steps.
- Avoid bracketed labels like `Step[foo]`; use plain text or branches.

## Token Efficiency

NBS achieves ~65% reduction vs ASCII box diagrams:

| Format | Tokens (typical) |
|--------|------------------|
| ASCII box art | 150-200 |
| Vertical ASCII | 80-120 |
| NBS | 50-70 |
