# Core Concepts

Understand the philosophy and key concepts behind Spec-Driven Development.

## What you'll learn

- What Spec-Driven Development is and why it matters
- The key building blocks: specs, phases, tasks, and verification
- How specs move through their lifecycle
- The human-in-the-loop philosophy

## What is Spec-Driven Development?

**Spec-Driven Development (SDD)** is a methodology where you create a detailed specification *before* writing code, then implement systematically against that spec.

Think of it like architecture before construction:

| Traditional Coding | Spec-Driven Development |
|-------------------|------------------------|
| Start coding immediately | Plan first, code second |
| Requirements evolve in your head | Requirements are documented |
| Progress is subjective | Progress is measurable |
| "Is it done?" is unclear | Completion is verifiable |

### Why Plan Before Code?

1. **Clarity** - Writing a plan forces you to think through the problem before solving it
2. **Alignment** - You and Claude share the same understanding of what needs to be built
3. **Quality** - AI review catches issues before you write a single line of code
4. **Tracking** - You can see exactly what's done and what's left
5. **Context** - When creating PRs, the spec provides full context for reviewers

## Key Concepts

### Specs (The Blueprint)

A **spec** is a structured document that defines what you're building. It contains:

- **Title and description** - What this feature/fix is about
- **Mission** - A single sentence capturing the core objective
- **Phases** - Major milestones to complete
- **Tasks** - Individual work items within each phase
- **Journal** - A log of decisions and progress

Specs are stored as JSON files in the `specs/` folder:

```
specs/
├── pending/     # New specs not yet started
├── active/      # Currently being worked on
├── completed/   # Finished specs
└── archived/    # Old/abandoned specs
```

### Phases (Milestones)

**Phases** are major milestones that group related tasks. A typical spec might have:

- **Phase 1: Foundation** - Setup and infrastructure
- **Phase 2: Core Implementation** - Main functionality
- **Phase 3: Integration** - Wiring things together
- **Phase 4: Testing & Polish** - Quality assurance

Phases help you:
- Break large features into manageable chunks
- Track progress at a high level
- Order work logically (Phase 2 often depends on Phase 1)

### Tasks (Work Items)

**Tasks** are individual units of work. Each task has:

- **Title** - What needs to be done
- **Description** - Details about the work
- **File path** - Which file(s) to modify (for implementation tasks)
- **Acceptance criteria** - How to know it's done
- **Dependencies** - Which tasks must complete first
- **Status** - pending, in_progress, completed, or blocked

Example task:
```
task-1-2: "Add validation middleware"
File: src/middleware/validate.ts
Acceptance criteria:
- Validates request body against schema
- Returns 400 with error details on failure
- Passes valid requests through
Depends on: task-1-1 (schema definition)
```

### Task Types

Not all tasks are code changes:

| Type | Purpose | Example |
|------|---------|---------|
| **task** | Write code | "Implement user authentication" |
| **verify** | Quality gate | "Run fidelity review on Phase 1" |
| **research** | Investigation | "Research JWT vs session auth" |

### Verification (Quality Gates)

**Verification nodes** are special tasks that check quality:

- **foundry-test** - Execute the test suite
- **fidelity** - Check if code matches the spec
- **manual** - Human verification checklist

These are typically placed at the end of phases to ensure quality before moving on.

## The Spec Lifecycle

Specs move through four stages:

```
pending  →  active  →  completed  →  archived
   │           │           │            │
 Draft     Working     Delivered     History
```

### 1. Pending

A spec starts in `pending` after creation. It's been planned and reviewed, but work hasn't started yet.

**What happens here:**
- AI reviews the plan for issues
- You approve or revise the plan
- The spec waits for activation

### 2. Active

When you're ready to work, the spec moves to `active`. Only one spec is typically active at a time to maintain focus.

**What happens here:**
- Tasks are implemented one by one
- Progress is tracked in the spec's journal
- Phases complete as their tasks finish

### 3. Completed

When all tasks are done and verified, the spec moves to `completed`.

**What happens here:**
- The spec serves as documentation of what was built
- PR creation references the completed spec
- Journal entries document decisions made

### 4. Archived

Old specs that are no longer relevant can be archived.

**What happens here:**
- Spec is preserved for historical reference
- No longer appears in active workflows

## Human-in-the-Loop Philosophy

Claude Foundry is designed for **collaboration**, not automation. Key decisions always involve you:

### Approval Gates

| Gate | When it happens |
|------|-----------------|
| Plan approval | Before converting plan to JSON spec |
| Spec activation | Before starting implementation |
| Task start | Before beginning each task |
| Task completion | Before marking work done |
| Blocker handling | When a task can't proceed |

### Why Not Fully Automated?

1. **You understand your codebase** - Claude suggests, you decide
2. **Context matters** - Some decisions need human judgment
3. **Quality control** - Catching issues early prevents rework
4. **Learning** - You stay informed about what's being built

### Execution Modes

For repetitive tasks, you can reduce friction:

| Mode | Approval level | Use when |
|------|---------------|----------|
| Interactive | Every task | Learning, complex work |
| Autonomous (`--auto`) | Less prompting | Confident, routine tasks |
| Delegated (`--delegate`) | Subagent handles | Many similar tasks |

## The Journal

Every spec maintains a **journal** - a log of what happened:

```json
{
  "journal": [
    {
      "timestamp": "2025-01-15T10:30:00Z",
      "task_id": "task-1-1",
      "entry": "Implemented schema validation. Used Zod for type-safe validation. Tests pass."
    },
    {
      "timestamp": "2025-01-15T11:00:00Z",
      "task_id": "task-1-2",
      "entry": "Added middleware. Integrated with existing error handler."
    }
  ]
}
```

The journal:
- Documents decisions for future reference
- Provides context for PR descriptions
- Helps when resuming work after a break
- Creates institutional knowledge

## Summary

| Concept | Definition |
|---------|------------|
| **Spec** | The complete plan for a feature/fix |
| **Phase** | A milestone grouping related tasks |
| **Task** | A single unit of work |
| **Verification** | Quality checks (tests, reviews) |
| **Journal** | Log of progress and decisions |
| **Lifecycle** | pending → active → completed → archived |

## Next Steps

- **[Workflow Guide](03-workflow-guide.md)** - Learn how to use each skill in the workflow
- **[Tutorial](04-tutorial.md)** - Build a complete feature step by step
