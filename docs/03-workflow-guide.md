# Workflow Guide

The complete Spec-Driven Development workflow explained step by step.

## What you'll learn

- How to plan features with `sdd-plan`
- How to implement tasks with `/implement`
- How to verify your work with `sdd-review`
- How to debug tests with `run-tests`
- How to create PRs with `sdd-pr`
- Supporting skills for research and refactoring

## The SDD Workflow

```
/research → describe intent → /implement → (auto-verify) → /sdd-pr
    │             │               │              │              │
 Explore      Claude          Work on       Verify tasks     Ship it
 codebase     creates         tasks via     auto-dispatch
 or web       spec            dependency    to sdd-review
                              order         or run-tests
```

**Key insight:** You don't manually call `sdd-review` or `run-tests`. Specs include verification tasks that `/implement` auto-dispatches when reached.

Each step has a specific skill. Let's walk through each one.

---

## 0. Research (Optional)

**When to use:** Before planning, especially in unfamiliar codebases, when design decisions matter, or when you want to research industry best practices.

### Starting research

```
/research How does this project handle authentication?
/research deep Current best practices for JWT refresh tokens 2025
```

### Research workflows

| Workflow | Use for |
|----------|---------|
| `chat` | Quick questions, single-model conversation |
| `consensus` | Design decisions, get multiple AI perspectives |
| `thinkdeep` | Systematic investigation of complex topics |
| `ideate` | Creative brainstorming, exploring options |
| `deep` | Comprehensive web research (runs in background) |

Research helps you make informed decisions before committing to a plan. Use `deep` for web research on best practices, libraries, or industry standards.

---

## 1. Planning with sdd-plan

**When to use:** Starting a new feature, complex bug fix, refactoring, or any work that benefits from upfront planning.

### What happens

The planning workflow has several stages:

```
Understand → Analyze → Plan → Review → Approve → Create Spec → Activate
```

### Starting a plan

Simply describe what you want to build:

```
I want to add user authentication with JWT tokens to my Express API.
```

Claude will invoke `sdd-plan` automatically, or you can ask directly:

```
Use sdd-plan to create a spec for adding JWT authentication.
```

### Stage 1: Creating the markdown plan

First, Claude creates a markdown plan outlining the approach:

```markdown
# JWT Authentication Implementation Plan

## Mission
Add secure JWT-based authentication to the Express API.

## Objective
Enable users to register, login, and access protected routes using JWT tokens.

## Success Criteria
- [ ] Users can register with email/password
- [ ] Users can login and receive a JWT
- [ ] Protected routes reject invalid/missing tokens
- [ ] Tokens expire and can be refreshed

## Phases

### Phase 1: Foundation
- Set up JWT utilities
- Create user model
- Add password hashing

### Phase 2: Authentication Routes
- Register endpoint
- Login endpoint
- Token refresh endpoint

### Phase 3: Protected Routes
- Auth middleware
- Apply to protected routes
- Test integration
```

### Stage 2: AI review

The plan is reviewed by AI for:

- **Completeness** - Are all requirements covered?
- **Feasibility** - Is this achievable given the codebase?
- **Clarity** - Are tasks specific enough?
- **Dependencies** - Are task dependencies correct?

You'll see a summary of any issues found.

### Stage 3: Your approval

This is a **mandatory gate**. You must approve before proceeding:

```
Plan review complete. Ready to create JSON spec?

[Approve] [Revise] [Cancel]
```

- **Approve** - Convert to structured spec
- **Revise** - Make changes to the plan
- **Cancel** - Abandon this spec

### Stage 4: JSON spec creation

Once approved, the plan becomes a structured JSON spec with:

- Unique spec ID (e.g., `jwt-auth-2025-01-15-001`)
- Phases with ordered tasks
- Dependencies between tasks
- Acceptance criteria for each task
- Verification nodes at phase boundaries

### Stage 5: Activation

The spec starts in `pending`. When ready:

```
Spec created. Would you like to activate it now?
```

Activating moves the spec to `active/` and makes it available for `/implement`.

### Tips for good specs

- **Be specific** about what you want
- **Mention constraints** (libraries to use, patterns to follow)
- **Include edge cases** you want handled
- **Break large features** into multiple specs if needed (keep specs under 6 phases)

---

## 2. Implementing with /implement

**When to use:** After you have an active spec and want to make progress.

### Starting implementation

```
/implement
```

Claude finds your active spec and recommends the next task based on:
- Dependencies (can't start a task if its dependencies aren't done)
- Phase order (earlier phases first)
- Task order within phases

### The implementation loop

```
Find Task → Show Plan → Approve → Implement → Verify → Complete → Next?
```

#### Step 1: Task selection

```
Active spec: jwt-auth-2025-01-15-001

Next recommended task:
task-1-1: "Create JWT utility module"
File: src/utils/jwt.ts
Acceptance criteria:
- Generate access tokens with RS256
- Verify token signatures
- Extract user ID from token
Depends on: (none)

Ready to start? [Yes] [Skip] [View spec]
```

#### Step 2: Implementation

After approval, Claude helps you implement the task:

- Creates or modifies the specified files
- Follows your codebase patterns
- Addresses acceptance criteria

#### Step 3: Verification

Before marking complete, Claude verifies:

- Code compiles/has no syntax errors
- Imports resolve correctly
- Basic sanity checks pass

#### Step 4: Completion

```
Task task-1-1 complete.

Journal entry: "Created JWT utility with sign/verify functions using
jsonwebtoken library. Added type definitions for token payload.
Verified imports work."

Continue to next task? [Yes] [No] [View progress]
```

### Handling blockers

Sometimes a task can't proceed:

```
Task task-2-1 is blocked.

Issue: "The user model doesn't exist yet"

Options:
[Resolve] - Fix the issue and continue
[Skip] - Move to a different task
[Add dependency] - Mark as depending on another task
[Block] - Mark task as blocked for later
```

### Execution modes

Control how much prompting you want:

| Flag | Behavior |
|------|----------|
| (none) | Approve each task, implement inline |
| `--auto` | Less prompting between tasks |
| `--delegate` | Use subagent for each task |
| `--parallel` | Run multiple subagents (with `--delegate`) |
| `--model opus` | Use Opus model for delegation |

Examples:

```bash
# Interactive (default) - most control
/implement

# Autonomous - fewer prompts, you still implement
/implement --auto

# Delegated - subagent implements, you approve
/implement --delegate

# Parallel delegation - fast, less control
/implement --delegate --parallel
```

### Progress tracking

Track progress anytime:

```
Show me the progress on the current spec.
```

Claude shows:
- Completed tasks (with journal entries)
- Current task (if any)
- Remaining tasks
- Overall phase completion

---

## 3. Reviewing with sdd-review

**When to use:** After implementing a phase or the entire spec, before final testing.

### What it does

`sdd-review` compares your implementation against the spec:

- Did you implement what the spec said?
- Are there any deviations?
- Are deviations intentional or accidental?

### Running a review

```
Run sdd-review on phase 1.
```

Or for the whole spec:

```
Run sdd-review on the entire spec.
```

### The review process

1. **Spec evolution check** - What changed during implementation?
2. **LSP structural check** - Do specified functions/classes exist?
3. **AI fidelity analysis** - Deep comparison of spec vs code
4. **Deviation assessment** - Categorize any differences

### Deviation categories

| Category | Meaning | Action |
|----------|---------|--------|
| **Exact Match** | Code matches spec perfectly | None needed |
| **Minor Deviation** | Small difference, no functional impact | Document it |
| **Major Deviation** | Significant difference | Investigate |
| **Missing** | Specified feature not implemented | Fix or update spec |

### Review output

```
Fidelity Review: Phase 1 - Foundation

task-1-1 "Create JWT utility": Exact Match
task-1-2 "User model": Minor Deviation
  - Added 'lastLogin' field not in spec
  - Impact: Low (enhancement)
task-1-3 "Password hashing": Exact Match

Overall: 3/3 tasks implemented
Deviations: 1 minor (enhancement)
Recommendation: Proceed to Phase 2
```

### When to update the spec

If you made intentional changes during implementation:

```
Update the spec to reflect the 'lastLogin' field addition.
```

Claude will modify the spec to match reality, keeping documentation accurate.

---

## 4. Testing with run-tests

**When to use:** When tests fail or you need to debug test issues systematically.

### Running tests

```
Run the tests for this project.
```

Or invoke directly:

```
Use run-tests to debug the failing tests.
```

### What it does

1. **Runs your test suite** (pytest, jest, go test, etc.)
2. **Categorizes failures** (assertion, exception, import, etc.)
3. **Gathers context** (related code, test setup)
4. **Helps you fix** (AI-assisted diagnosis)
5. **Verifies fixes** (re-runs failing tests)

### Failure categories

| Category | Typical cause | Fix approach |
|----------|--------------|--------------|
| **Assertion** | Logic error | Check expected vs actual |
| **Exception** | Runtime error | Fix the throwing code |
| **Import** | Missing dependency | Install or fix import path |
| **Setup** | Fixture/config issue | Fix test setup |
| **Timeout** | Performance issue | Optimize or increase timeout |
| **Flaky** | Non-deterministic | Add waits, fix race conditions |

### AI consultation

For complex failures, Claude can consult AI for deeper analysis:

```
This test is failing intermittently. Can you investigate?
```

The AI will:
- Analyze the test and related code
- Form hypotheses about the cause
- Suggest fixes with reasoning

---

## 5. Creating PRs with sdd-pr

**When to use:** When your spec is complete and you're ready to submit for review.

### What it does

`sdd-pr` creates a comprehensive pull request by gathering:

- **Spec metadata** - Title, description, mission
- **Completed tasks** - What was implemented
- **Git history** - Commits during implementation
- **Journal entries** - Decisions made along the way
- **Spec evolution** - Requirements added during work

### Creating a PR

```
Create a PR for this spec.
```

### The PR creation flow

1. **Context gathering** - Collects all relevant information
2. **Draft generation** - AI writes comprehensive PR description
3. **Your review** - You approve or revise the draft
4. **Branch push** - Pushes to remote if needed
5. **PR creation** - Creates the pull request

### PR template

```markdown
## Summary
Brief description of what this PR accomplishes.

## What Changed

### Key Features
- Bullet points of key changes
- Organized by feature/component

### Files Modified
- `path/to/file.ext`: Short summary

## Technical Approach
Explanation of how it was implemented.

## Implementation Details
### Phase 1: Foundation
- ✅ Key task

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing performed

## Spec Reference
- `specs/completed/<spec-id>.json`

## Commits
- abc1234: task-1-1: Short description
```

### Why spec-driven PRs are better

| Traditional PR | Spec-Driven PR |
|---------------|----------------|
| "Added auth" | Full context of why and how |
| Reviewer guesses intent | Intent is documented |
| Lost context over time | Journal preserves decisions |

---

## Supporting Skills

### sdd-refactor

**When to use:** Renaming, extracting functions, moving code, or cleaning up.

```
Rename the function 'validateUser' to 'authenticateUser' across the codebase.
```

Uses LSP (Language Server Protocol) for:
- **Impact analysis** - Shows all references before changing
- **Safe renaming** - Updates all usages automatically
- **Verification** - Confirms all references updated

### /research

**When to use:** Complex investigations, design decisions, or learning.

```
/research What are the security implications of storing JWTs in localStorage?
```

Research workflows:
- **chat** - Single-model conversation
- **consensus** - Multiple AI perspectives
- **thinkdeep** - Systematic investigation
- **ideate** - Creative brainstorming
- **deep** - Comprehensive web research

### /bikelane

**When to use:** Quick capture of ideas or issues without interrupting flow.

```
/bikelane add Remember to add rate limiting to auth endpoints
```

Later, review captured items:

```
/bikelane list
```

---

## Workflow Examples

### Starting fresh

```
1. /research to understand codebase (optional)
2. Describe your feature naturally
3. Claude creates spec, you review and approve
4. /implement to work through tasks
5. Verification tasks auto-dispatch when reached
6. /sdd-pr when spec is complete
```

### Resuming work

```
1. /implement
2. Claude finds active spec
3. Shows next task
4. Continue from where you left off
```

### Fixing issues

```
1. Verify task fails (or you ask to run tests)
2. AI helps diagnose
3. Fix the issues
4. Continue with /implement
```

---

## Next Steps

- **[Tutorial](04-tutorial.md)** - Follow along building a complete feature
- **[Command Reference](05-command-reference.md)** - Detailed options for each command
