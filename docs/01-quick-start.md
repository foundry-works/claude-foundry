# Quick Start

Get up and running with Claude Foundry in 5 minutes.

## What you'll learn

- How to install and configure the plugin
- How to create your first specification
- How to implement your first task

## Step 1: Install the Plugin

### 1.1 Install foundry-mcp

The plugin requires the foundry-mcp Python package:

```bash
pip install foundry-mcp
```

Verify installation:

```bash
python -m foundry_mcp.server --help
```

### 1.2 Install the Claude Code Plugin

In Claude Code, run:

```
/plugin marketplace add foundry-works/claude-foundry
/plugin install foundry@claude-foundry
```

Restart Claude Code when prompted and trust the repository.

## Step 2: Run Setup

Run the setup skill to configure Claude Foundry:

```
Use foundry-setup
```

This interactive wizard will:
- Verify foundry-mcp is installed
- Create the `specs/` directory structure
- Configure permissions
- Set up your workspace

Follow the prompts to complete configuration.

## Step 2.5: Research (Optional)

Before planning, you can explore your codebase or research best practices:

```
Use foundry-research to explore how this project handles API routes
Use foundry-research deep to research current best practices for API rate limiting
```

This is optional but recommended for larger codebases, unfamiliar projects, or when you want to research industry best practices before committing to an approach.

## Step 3: Create Your First Spec

Now let's plan a simple feature. Tell Claude what you want to build:

```
I want to add a simple health check endpoint to my API that returns
{ "status": "ok" } when the server is running.
```

Claude will invoke `foundry-spec` and guide you through:

1. **Creating a markdown plan** - outlining the approach
2. **AI review** - checking for issues
3. **Your approval** - you decide if it looks good
4. **Creating a JSON spec** - the structured plan with tasks

Example output:
```
Created spec: health-check-endpoint-2025-01-15-001
Status: pending
Tasks: 3 (in 1 phase)

Would you like to activate this spec and start implementation?
```

## Step 4: Implement Your Feature

Once the spec is activated, start implementing:

```
Use foundry-implement to work on the next task
```

Claude will:
1. Find the next task to work on
2. Show you the task details and acceptance criteria
3. Ask for your approval to start
4. Help you implement the code
5. Mark the task complete when done
6. Ask if you want to continue to the next task

Example flow:
```
Next task: task-1-1 "Create health check route handler"
File: src/routes/health.ts
Acceptance criteria:
- Returns { "status": "ok" } with HTTP 200
- No authentication required

Ready to implement? [Yes / Skip / View details]
```

## What Just Happened?

You've just experienced the core SDD workflow:

1. **Planning** (`foundry-spec`) - You described what you wanted, and Claude created a structured plan with AI review
2. **Implementation** (`foundry-implement`) - Claude guided you through each task systematically
3. **Progress tracking** - Each completed task is recorded with a journal entry

The spec file in `specs/active/` contains the full plan, task status, and journal of what was done.

## Next Steps

- **[Core Concepts](02-core-concepts.md)** - Understand specs, phases, and tasks in depth
- **[Workflow Guide](03-workflow-guide.md)** - Learn the complete workflow including review and PR creation
- **[Tutorial](04-tutorial.md)** - Follow a detailed walkthrough building a complete feature

## Quick Reference

| Skill | What it does |
|-------|--------------|
| `foundry-spec` | Create or modify specs |
| `foundry-implement` | Find next task and implement it |
| `foundry-review` | Verify implementation matches spec |
| `foundry-test` | Run tests and debug failures |
| `foundry-pr` | Create a pull request |
| `foundry-note` | Quick capture an idea for later |
| `foundry-research` | AI-powered research |
