# Tutorial: Building a TODO API

Follow along as we build a complete feature using Spec-Driven Development.

## What you'll learn

- The complete SDD workflow from start to finish
- How each step feels in practice
- Tips and patterns for effective spec-driven work

## The Scenario

We're adding a TODO feature to an existing Express API. Users should be able to:
- Create todo items
- List their todos
- Mark todos as complete
- Delete todos

This is a realistic feature that touches multiple files and requires thoughtful planning.

---

## Step 1: Planning

### Starting the conversation

We begin by describing what we want:

```
I want to add a TODO feature to this API. Users should be able to
create, list, complete, and delete todo items. Each todo belongs to
a user, has a title, optional description, and completed status.
Use PostgreSQL with our existing Drizzle ORM setup.
```

Claude recognizes this needs planning and invokes `foundry-spec`.

### Creating the markdown plan

Claude first analyzes your codebase to understand existing patterns, then creates a plan:

```
Analyzing codebase...
Found: src/db/schema.ts (Drizzle schema definitions)
Found: src/routes/*.ts (existing route patterns)
Found: src/middleware/auth.ts (authentication middleware)

Creating plan for TODO feature...
```

The plan appears:

```markdown
# TODO Feature Implementation Plan

## Mission
Add CRUD operations for user-owned todo items with PostgreSQL persistence.

## Objective
Enable authenticated users to manage personal todo lists with create,
read, update (complete), and delete operations.

## Success Criteria
- [ ] Todo items persist to PostgreSQL
- [ ] Only todo owners can access their items
- [ ] All operations return appropriate HTTP status codes
- [ ] API follows existing patterns in the codebase

## Phases

### Phase 1: Database Layer
Set up the database schema and queries for todos.
**Files:** src/db/schema.ts, src/db/queries/todos.ts
**Tasks:**
- Add todos table to schema
- Create CRUD query functions
- Run migration

### Phase 2: API Routes
Implement the REST endpoints.
**Files:** src/routes/todos.ts, src/app.ts
**Tasks:**
- POST /todos - Create
- GET /todos - List user's todos
- PATCH /todos/:id/complete - Mark complete
- DELETE /todos/:id - Delete

### Phase 3: Integration & Testing
Wire everything together and test.
**Tasks:**
- Register routes in app.ts
- Add integration tests
- Manual API testing
```

### AI review

Claude runs an AI review of the plan:

```
Plan Review Results:

✓ Completeness: All CRUD operations covered
✓ Feasibility: Matches existing codebase patterns
✓ Dependencies: Phase order is logical

Suggestions:
- Consider adding validation for todo title length
- Phase 2 could include input validation middleware

Would you like to incorporate these suggestions? [Yes] [No] [View details]
```

We choose to incorporate:

```
Yes, add validation for todo title (max 200 chars) and description
(max 1000 chars).
```

### Your approval

After revisions, Claude asks for final approval:

```
Plan ready. Summary:
- 3 phases, 8 tasks
- Files: 4 new/modified
- Estimated complexity: Medium

Ready to create JSON spec? [Approve] [Revise] [Cancel]
```

We approve, and Claude creates the structured spec:

```
Spec created: todo-feature-2025-01-15-001
Status: pending
Location: specs/pending/todo-feature-2025-01-15-001.json

Would you like to activate this spec and start implementation?
[Activate] [Review spec first] [Later]
```

---

## Step 2: Activation

We activate the spec:

```
Activate the spec.
```

Claude responds:

```
Spec activated!
Moved to: specs/active/todo-feature-2025-01-15-001.json

Ready to implement. Use foundry-implement to get started.
```

---

## Step 3: Implementation

### Starting the loop

```
Use foundry-implement
```

Claude loads the spec and finds the first task:

```
Active spec: todo-feature-2025-01-15-001
Phase 1: Database Layer (0/3 tasks complete)

Next task:
task-1-1: "Add todos table to schema"
File: src/db/schema.ts

Description:
Add a 'todos' table with columns:
- id (uuid, primary key)
- userId (uuid, foreign key to users)
- title (varchar 200, not null)
- description (text, nullable)
- completed (boolean, default false)
- createdAt (timestamp)
- updatedAt (timestamp)

Acceptance criteria:
- Table references users table
- Proper indexes on userId
- Timestamps default to now; updatedAt auto-updates

Ready to implement? [Yes] [Skip] [View context]
```

### Implementing the first task

We approve, and Claude implements:

```typescript
// src/db/schema.ts (additions)

export const todos = pgTable('todos', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull().references(() => users.id),
  title: varchar('title', { length: 200 }).notNull(),
  description: text('description'),
  completed: boolean('completed').default(false).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull().$onUpdate(() => new Date()),
}, (table) => ({
  userIdIdx: index('todos_user_id_idx').on(table.userId),
}));

export type Todo = typeof todos.$inferSelect;
export type NewTodo = typeof todos.$inferInsert;
```

Claude verifies the code:

```
Verification:
✓ TypeScript compiles
✓ Imports resolve
✓ Schema follows existing patterns

Ready to mark complete? [Yes] [Review changes]
```

We confirm:

```
Task task-1-1 complete.

Journal: "Added todos table to schema with userId foreign key,
title (200 char limit), description, completed flag, and
timestamps. Added index on userId for query performance."

Progress: Phase 1 - 1/3 tasks complete

Continue to next task? [Yes] [No] [View progress]
```

### Continuing through tasks

We continue through the remaining tasks. Here's task-1-2:

```
Next task:
task-1-2: "Create CRUD query functions"
File: src/db/queries/todos.ts (new file)

Description:
Create query functions using Drizzle:
- createTodo(userId, data)
- getTodosByUser(userId)
- getTodoById(id)
- markTodoComplete(id)
- deleteTodo(id)

Acceptance criteria:
- All queries are type-safe
- getTodosByUser filters by userId
- Error handling for not found cases

Depends on: task-1-1 ✓ (completed)

Ready to implement? [Yes] [Skip] [View context]
```

After implementing all Phase 1 tasks, Claude notes the phase completion:

```
Phase 1 complete! All 3 tasks finished.

Phase summary:
- task-1-1: Added todos table ✓
- task-1-2: Created query functions ✓
- task-1-3: Migration applied ✓

Moving to Phase 2: API Routes (0/4 tasks)

Continue? [Yes] [Review Phase 1] [Stop here]
```

### Handling a blocker

In Phase 2, we encounter an issue:

```
Next task:
task-2-1: "POST /todos endpoint"
File: src/routes/todos.ts (new file)

Starting implementation...

Issue detected: No validation schema exists for todo input.
The plan mentioned validation but no schema task exists.

Options:
[Add task] - Create a validation schema task first
[Inline] - Add validation inline with this task
[Skip validation] - Implement without validation (not recommended)
```

We choose to add inline:

```
Add validation inline with this task.
```

Claude updates the task scope and implements with validation included:

```typescript
// src/routes/todos.ts

import { z } from 'zod';

const createTodoSchema = z.object({
  title: z.string().min(1).max(200),
  description: z.string().max(1000).optional(),
});

router.post('/', authMiddleware, async (req, res) => {
  const parsed = createTodoSchema.safeParse(req.body);
  if (!parsed.success) {
    return res.status(400).json({ error: parsed.error.issues });
  }

  const todo = await createTodo(req.user.id, parsed.data);
  res.status(201).json(todo);
});
```

---

## Step 4: Review

After implementing all tasks, we run a fidelity review:

```
Run foundry-review on the completed spec.
```

Claude analyzes:

```
Fidelity Review: todo-feature-2025-01-15-001

Phase 1: Database Layer
  task-1-1: Exact Match ✓
  task-1-2: Exact Match ✓
  task-1-3: Exact Match ✓

Phase 2: API Routes
  task-2-1: Minor Deviation
    - Added inline Zod validation (enhancement)
  task-2-2: Exact Match ✓
  task-2-3: Exact Match ✓
  task-2-4: Exact Match ✓

Phase 3: Integration & Testing
  task-3-1: Exact Match ✓
  task-3-2: Exact Match ✓

Summary: 10/10 tasks implemented
Deviations: 1 minor (inline validation enhancement)

Recommendation: Implementation matches spec. Minor deviation
is an enhancement, not a gap. Ready for testing.
```

---

## Step 5: Testing

We run the tests:

```
Run the tests.
```

Claude executes the test suite:

```
Running: npm test

Tests: 45 passed, 2 failed
Time: 4.2s

Failed tests:
1. POST /todos › should reject blank title
   Expected: 400
   Received: 201

2. DELETE /todos/:id › should return 404 for non-existent todo
   Expected: 404
   Received: 500

Analyzing failures...
```

Claude helps fix the issues:

```
Failure Analysis:

1. Blank title validation:
   Issue: Schema doesn't trim whitespace, so "   " passes min(1)
   Fix: Change z.string().min(1) to z.string().trim().min(1, "Title required")

2. Delete 404 handling:
   Issue: Query throws when todo not found instead of returning null
   Fix: Add null check before delete

Would you like me to fix these issues? [Yes] [View details]
```

After fixes:

```
Running: npm test

Tests: 47 passed
Time: 4.1s

All tests passing! ✓
```

---

## Step 6: Creating the PR

With everything working, we create the PR:

```
Create a PR for this spec.
```

Claude gathers context and drafts:

```
Gathering PR context...
- Spec: todo-feature-2025-01-15-001
- Commits: 8 commits on this branch
- Files changed: 6
- Journal entries: 10

Draft PR:

---

## Summary
Adds TODO feature enabling users to create, list, complete, and
delete personal todo items with PostgreSQL persistence.

## What Changed

### Key Features
- **Database**: Added `todos` table with user ownership and indexes
- **API**: New REST endpoints at `/todos`
  - POST /todos - Create todo
  - GET /todos - List user's todos
  - PATCH /todos/:id/complete - Mark complete
  - DELETE /todos/:id - Delete todo
- **Validation**: Zod schemas for input validation
- **Tests**: Integration tests for all endpoints

### Files Modified
- `src/db/schema.ts`: Todos table and indexes
- `src/db/queries/todos.ts`: CRUD queries
- `src/routes/todos.ts`: REST handlers
- `src/app.ts`: Route registration
- `tests/todos.spec.ts`: API tests

## Technical Approach
- Used existing Drizzle ORM patterns for schema and queries
- Protected all routes with auth middleware
- Added database indexes for query performance

## Implementation Details

### Phase 1: Database Layer
- [x] Added schema and queries

### Phase 2: API Routes
- [x] Implemented CRUD endpoints

### Phase 3: Integration & Testing
- [x] Registered routes and added tests

## Testing
- [x] Unit tests for query functions
- [x] Integration tests for API endpoints
- [x] Manual testing of full flow

## Spec Reference
- `specs/completed/todo-feature-2025-01-15-001.json`

## Commits
- abc1234: task-1-1: Add todos schema
- def5678: task-2-1: Implement todos routes
- ghi9012: task-3-2: Add integration tests

---

Ready to create PR? [Create] [Edit draft] [Cancel]
```

We create the PR:

```
PR created: #142 "Add TODO feature"
URL: https://github.com/your-org/your-repo/pull/142

The PR includes a link to the spec for full implementation context.
```

---

## What We Accomplished

In this tutorial, we:

1. **Planned** - Created a detailed spec with AI review
2. **Implemented** - Worked through tasks systematically
3. **Handled blockers** - Added validation inline when needed
4. **Reviewed** - Verified implementation matched the spec
5. **Tested** - Fixed failing tests with AI assistance
6. **Shipped** - Created a comprehensive PR

The spec now lives in `specs/completed/` as documentation of what was built and why.

---

## Tips from This Tutorial

### Planning
- Be specific about constraints (ORM, patterns to follow)
- Let AI review catch issues early
- Don't skip the approval gate

### Implementation
- Work through tasks in order
- Handle blockers immediately
- Write good completion notes

### Review
- Run fidelity review before testing
- Document intentional deviations
- Update spec if requirements changed

### Testing
- Use AI to help diagnose failures
- Verify fixes with full test suite
- Don't skip this step!

---

## Next Steps

- **[Command Reference](05-command-reference.md)** - Detailed options for each command
- **[Troubleshooting](06-troubleshooting.md)** - When things don't go as planned
