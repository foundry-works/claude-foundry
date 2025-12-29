# Foundry Tutorial - Detailed Phase Instructions

## Phase 1: Understanding SDD

### 1.1 Explain What SDD Is

Start by teaching the user about Spec-Driven Development:

> "Let me explain what Spec-Driven Development (SDD) is and why it's powerful.
>
> **The Problem:** When building features, it's easy to jump straight into coding. But this often leads to scope creep, forgotten requirements, and difficulty tracking what's done.
>
> **The SDD Solution:** We create a detailed specification *before* writing any code. This spec becomes the single source of truth - it defines what we're building, breaks it into phases and tasks, and tracks our progress. Claude can then help you work through tasks systematically, verify your implementation matches the spec, and generate documentation automatically.
>
> Think of it like having a detailed blueprint before building a house."

Use `AskUserQuestion`:
- Question: "Make sense so far?"
- Options: "Yes, continue" / "I have a question"

If "I have a question": Pause and answer their question before continuing.

### 1.2 Walk Through the Workflow

Explain each stage of the workflow and how they connect:

```
- **Entry** → sdd-plan (Create spec)
  - → sdd-implement (Find task)
  - → [CODE] (Write code)
  - → sdd-review (Verify implementation)
  - → run-tests (Verify quality)
  - → sdd-pr → **Exit**
```

For each stage, explain its purpose conversationally:

1. **Planning** - "This is where you describe what you want to build. The sdd-plan skill helps Claude create a structured spec with phases, tasks, and acceptance criteria. It includes automatic AI review, modification, and validation."

2. **Implementation** - "Now you work through tasks one by one. sdd-implement finds your next task, gives you context, and tracks completion automatically."

3. **Review** - "sdd-review verifies your implementation matches the spec requirements."

4. **Quality** - "run-tests validates your code passes tests and helps debug failures."

5. **Delivery** - "When ready, sdd-pr creates a comprehensive PR description from your spec and journal entries."

---

## Phase 2: Hands-On Learning with a Sample Spec

**The best way to learn SDD is by doing.** Offer to create a sample spec so the user can see the structure and try the workflow.

Use `AskUserQuestion` to ask:
- Question: "Would you like to create a sample 'hello-world' spec? This lets you explore the workflow hands-on without committing to a real feature yet."
- Options: "Yes, let's try it" / "No, I'll create my own when ready"

If yes:

1. Invoke `Skill(sdd-plan) "Create a hello-world demo spec for the foundry tutorial. Requirements: A simple Python greeting utility with a greet(name='World') function that returns 'Hello, {name}!'. Include a CLI interface. Keep it simple - one implementation phase with 2 tasks."`

2. **TEACH the user about spec structure** by walking through what was created:

> "Great! I've created a sample spec. Let me walk you through its structure:
>
> **Location:** `specs/pending/hello-world-demo.json`
>
> Specs start in the `pending/` folder until you're ready to work on them. When you start, they move to `active/`, and when complete, to `completed/`.
>
> **Key sections in a spec:**
>
> - **metadata** - Title, version, status, and timestamps. This is the spec's identity.
>
> - **hierarchy** - The heart of the spec. Contains phases (major milestones) and tasks (individual work items). Each task has a status, description, and can have subtasks.
>
> - **assumptions** - Constraints and requirements that shape the implementation. Document these early to avoid surprises.
>
> - **journal** - A decision log that captures *why* you made choices, not just *what* you did. Invaluable for PRs and future reference.
>
> You interact with specs through MCP tools and skills - you don't edit the JSON directly. This keeps everything consistent and trackable."

3. Use `AskUserQuestion` to ask:
- Question: "Now that you've seen the structure, what would you like to do with this sample?"
- Options:
  - "Activate it and walk me through a task" - Best for hands-on learning
  - "Keep it in pending for later"
  - "Delete it - I understand the concept now"

4. Execute the chosen action:
   - If "activate and walk through": Call `mcp__plugin_foundry_foundry-mcp__lifecycle action="activate"`, then invoke `Skill(sdd-implement) "Sample spec activated via /foundry-tutorial. Walk the user through their first task to demonstrate the SDD workflow."`
   - If "keep in pending": Just confirm it's saved
   - If "delete": Remove it and confirm

5. After demonstrating task preparation, explain that this was a simplified introduction:

> "This tutorial showed the core workflow: `sdd-plan` to create specs and `sdd-implement` to prepare and complete tasks. The full SDD workflow also includes:
> - `sdd-review` - Verify your implementation matches the spec
> - `run-tests` - Run and debug tests
> - `sdd-pr` - Create a PR with full context from your spec and journal
>
> You can explore these as you get comfortable with the basics."

---

## Phase 3: Next Steps

**End on a helpful, encouraging note.** Give clear, actionable next steps based on their situation.

### Personalized Next Steps

Based on the user's state, recommend the most logical next action:

**If they created and activated a sample spec:**
> "You're all set! Your sample spec is active and ready. Try asking me to run `sdd-implement` - I'll show you how task preparation works and walk you through completing your first task."

**If they have existing specs:**
> "You're all set! I see you already have specs in your workspace. Run `/implement` anytime to pick up where you left off, or ask me to create a new spec with `sdd-plan`."

**If they have a clean workspace:**
> "You're all set! When you're ready to start a feature, just describe what you want to build and ask me to use `sdd-plan`. I'll create a detailed spec and we can work through it together."

### Resources for Learning More

> "A few resources if you want to go deeper:
>
> - **Re-run tutorial:** `/tutorial` is safe to run anytime if you need a refresher
> - **Re-run setup:** `/setup` to check or update your configuration
>
> Feel free to ask me questions anytime - I'm here to help you get the most out of SDD!"

### Close Warmly

End with something like:
> "That's the tour! The key thing to remember: specs first, then code. It takes a bit of discipline upfront but pays off massively in clarity, trackability, and documentation. Ready when you are!"
