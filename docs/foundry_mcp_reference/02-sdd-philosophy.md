# Spec-Driven Development Philosophy

> Understanding the methodology that powers AI-native development workflows.

---

## What is SDD?

**Spec-Driven Development (SDD)** is a documentation-first methodology where machine-readable specifications serve as the single source of truth for software development. Unlike traditional approaches where documentation follows (or never arrives), SDD requires structured specifications *before* implementation begins.

The core insight: **specifications are contracts designed for AI coding assistants**, not just human readers. By encoding requirements, tasks, dependencies, and decisions in structured JSON, SDD enables AI tools to autonomously discover work, track progress, and maintain alignment between intention and implementation.

Think of SDD as "documentation-first for the AI age."

---

## The Problem SDD Solves

Traditional development suffers from several persistent problems:

### Code First, Document Later

Documentation is often an afterthought—written hastily before a release, or not at all. The result: codebases where intent must be reverse-engineered from implementation.

### AI Assistants Lack Context

When you ask an AI coding assistant to "implement the next feature," it has no structured way to know what "next" means, what dependencies exist, or what decisions have already been made.

### Drift Between Intentions and Code

Requirements live in wikis, designs in Figma, tasks in Jira, and code in Git. Over time, these drift apart. Which source tells the truth?

### Lost Decision Rationale

Six months later, no one remembers *why* a particular approach was chosen. The code shows *what* was built, but not the alternatives considered or trade-offs accepted.

**SDD addresses these problems by making specifications the authoritative source—structured, versioned, and designed for both human understanding and machine consumption.**

---

## Core Principles

### 1. Specifications Are the Source of Truth

Not code. Not comments. Not wikis. The spec defines what should be built, how it should behave, and what completion looks like.

**Key implications:**
- Implementation SHOULD reflect the spec
- When code and spec diverge, the spec wins (or must be updated first)
- Changes to requirements go through spec updates, not ad-hoc code changes

### 2. AI-Native by Design

Specs are JSON-first, structured for LLM consumption:

| Aspect | Design Choice | Benefit |
|--------|---------------|---------|
| **Format** | JSON, not prose | Machine-parseable, predictable structure |
| **Tasks** | Explicit status fields | Automated progress tracking |
| **Dependencies** | Structured `depends_on` arrays | Automated task ordering |
| **Metadata** | Queryable fields | AI tools can filter and search |

### 3. Progressive Task Discovery

Rather than presenting an AI assistant with a massive backlog, SDD enables *progressive discovery*:

> "What's the next actionable task given the current state?"

The spec answers this question automatically based on:
- Task status (`pending`, `in_progress`, `completed`)
- Dependencies (blocked tasks filtered out)
- Phase order (sequential progression)

### 4. Decision Traceability

Every significant decision gets journaled—not buried in commit messages or Slack threads:

```json
{
  "entry_type": "decision",
  "timestamp": "2025-12-04T10:30:00Z",
  "content": "Chose JWT over session-based auth",
  "rationale": "Better for stateless API scaling",
  "alternatives_considered": ["session cookies", "OAuth only"]
}
```

This creates an audit trail that future developers (human or AI) can consult.

### 5. Verification as First-Class

Every phase has explicit verification criteria. "Done" isn't a feeling—it's a checklist:

| Verification Type | Description | Example |
|-------------------|-------------|---------|
| `auto` | Automated tests | `pytest tests/auth/` |
| `fidelity` | Spec vs implementation review | AI compares code to requirements |
| `manual` | Human review checklist | Security audit, UX review |

---

## How SDD Differs from Traditional Approaches

| Aspect | Traditional | SDD |
|--------|-------------|-----|
| Primary consumer | Humans | AI coding assistants |
| Format | Prose, markdown, wikis | Machine-readable JSON |
| When written | After implementation | Before implementation |
| Task discovery | Manual triage | Automated next-task |
| Decision tracking | Ad-hoc (comments, PRs) | Structured journals |
| Completion criteria | Subjective | Explicit verification steps |
| Spec-code sync | Best effort | Enforced by tooling |

---

## The SDD Workflow

SDD follows a lifecycle that moves specifications through distinct phases:

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ PLAN    │───►│ EXECUTE │───►│ VERIFY  │───►│ ARCHIVE │
│ pending │    │ active  │    │completed│    │archived │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
```

### 1. Plan

Create a specification with:
- Phases (logical groupings of work)
- Tasks and subtasks
- Dependencies between tasks
- Assumptions and constraints
- Verification criteria

The spec lives in `specs/pending/` until ready.

### 2. Activate

Move the spec from pending to active using `spec-lifecycle-activate`. This signals that:
- Planning is complete
- Implementation can begin
- The spec is the source of truth

Active specs live in `specs/active/`.

### 3. Execute

AI assistants (or humans) query for the next actionable task:

```
mcp__plugin_foundry_foundry-mcp__task action="next" spec_id="my-feature"
```

Each task includes:
- Description and context
- Files to modify
- Dependencies that must be met
- Verification criteria

As work progresses, journal entries capture decisions and blockers.

### 4. Verify

Run verification steps defined for each phase:

```
mcp__plugin_foundry_foundry-mcp__verification action="execute" spec_id="my-feature" verify_id="phase-1"
```

Record results. If verification fails, the phase remains open until issues are resolved.

### 5. Complete

Mark tasks as done with completion notes:

```
mcp__plugin_foundry_foundry-mcp__task action="complete" spec_id="my-feature" task_id="task-001"
  completion_note="Implemented JWT authentication with refresh tokens"
```

Generate PR descriptions from spec context and journal entries.

### 6. Archive

Move completed specs to `specs/completed/` or `specs/archived/` for historical reference. The decision trail remains available for future consultation.

---

## Why AI-Native Matters

SDD's AI-native design unlocks capabilities that prose documentation cannot provide:

### Token Efficiency

Context windows are limited. Structured JSON specs convey maximum information in minimum tokens, leaving room for code and conversation.

### Autonomous Task Discovery

AI assistants can ask "What should I work on next?" and receive a concrete, actionable answer without human intervention.

### Tool Chaining

Specs enable multi-step autonomous workflows:

```
discover task → read dependencies → implement → verify → mark complete → discover next task
```

### Fidelity Reviews

Tooling can automatically compare implementation against spec requirements, identifying drift before it becomes technical debt.

### PR Generation

AI can synthesize journal entries, completed tasks, and phase summaries into meaningful PR descriptions—no more "various fixes and improvements."

---

## When to Use SDD

SDD adds value when:

| Scenario | Why SDD Helps |
|----------|---------------|
| **Multi-step features** | Coordination across files, systems, or time |
| **Design-first thinking** | Prevents costly rework from misunderstandings |
| **Decision traceability** | Compliance, audits, or future maintainers need context |
| **AI-assisted development** | Structured specs enable autonomous workflows |
| **Team collaboration** | Shared, unambiguous source of truth |

---

## When NOT to Use SDD

SDD adds overhead that isn't always justified:

| Scenario | Better Approach |
|----------|-----------------|
| **Trivial changes** | A typo fix doesn't need a spec |
| **Exploratory prototyping** | Spec *after* you've learned what works |
| **Emergency hotfixes** | Fix first, document retroactively if needed |
| **Simple bug fixes** | Direct fix with clear commit message |

**The goal is pragmatic adoption, not dogmatic process.** Use SDD where it adds value; skip it where it doesn't.

---

## SDD vs Other Methodologies

### SDD vs Test-Driven Development (TDD)

| Aspect | TDD | SDD |
|--------|-----|-----|
| Focus | Tests first | Specs first |
| Artifact | Test code | JSON specifications |
| Scope | Unit/function level | Feature/system level |
| Compatibility | Can combine with SDD | Can combine with TDD |

**SDD and TDD complement each other:** SDD defines *what* to build at a high level; TDD ensures *how* it's built is correct.

### SDD vs Agile/Scrum

| Aspect | Agile/Scrum | SDD |
|--------|-------------|-----|
| Planning | Sprints, stories | Structured specs |
| Tracking | Jira, boards | Spec files, journals |
| AI-native | No | Yes |
| Compatibility | Can combine | Can combine |

**SDD can operate within an Agile framework:** Specs can represent sprint work items with more structure for AI consumption.

---

## Getting Started with SDD

1. **Start Small** — Pick one feature for your first spec
2. **Use the sdd-plan Skill** — Let the skill guide spec creation
3. **Iterate** — Refine your spec structure as you learn
4. **Journal Decisions** — Build the habit of recording rationale
5. **Review and Improve** — Use fidelity checks to catch drift

---

## Related Documentation

- **[03-Spec Lifecycle](./03-spec-lifecycle.md)** — Detailed state transitions
- **[06-Workflows](./06-workflows.md)** — Practical SDD patterns
- **[04-Tool Reference](./04-tool-reference.md)** — Tools for SDD workflows

---

*[Back to Index](./README.md)*
