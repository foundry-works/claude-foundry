# Skill Reference

Detailed documentation for all Claude Foundry skills.

## How to Invoke Skills

Skills are invoked by asking Claude to use them naturally:

```
Use foundry-spec to create a plan for user authentication
Use foundry-implement to work on the next task
```

Or invoke directly with `Skill(foundry:skill-name)`.

---

## Skills

### foundry-setup

First-time configuration and verification.

**When to use:**
- Setting up Claude Foundry for the first time
- Verifying your installation is working
- Re-configuring after changes

**Invoking:**
```
Use foundry-setup to configure Claude Foundry
Use foundry-setup --check to verify installation
```

**Options:**
| Option | Description |
|--------|-------------|
| `--check` | Only run verification, don't configure |

**What it does:**
1. Verifies foundry-mcp is installed
2. Checks Python and Git availability
3. Creates workspace directories (`specs/`)
4. Configures foundry-mcp.toml if needed
5. Verifies permissions

---

### foundry-spec

Create and manage specifications.

**When to use:**
- Starting a new feature
- Complex bug fixes
- Refactoring projects
- Any work benefiting from upfront planning

**Workflow stages:**
1. **Understand** - Clarify requirements
2. **Analyze** - Explore codebase
3. **Plan** - Create markdown plan
4. **Review** - AI review for issues
5. **Approve** - Human approval gate
6. **Create** - Convert to JSON spec
7. **Activate** - Move to active

**Invoking:**
```
Use foundry-spec to create a spec for user authentication.
```

Or describe the feature and let Claude recognize the need:
```
I want to add user authentication with JWT tokens.
```

**Key actions:**
| Action | What it does |
|--------|--------------|
| Create plan | Generate markdown plan |
| Review plan | Run AI review |
| Create spec | Convert plan to JSON |
| Modify spec | Update existing spec |
| Validate spec | Check for issues |
| Activate spec | Move to active |

---

### foundry-implement

Task execution and progress tracking.

**When to use:**
- After a spec is activated
- When resuming work
- To check what's next

**Invoking:**
```
Use foundry-implement to work on the next task
Use foundry-implement --auto for less prompting
Use foundry-implement --delegate to use subagents
Use foundry-implement --delegate --parallel for parallel execution
```

**Options:**
| Option | Description |
|--------|-------------|
| `--auto` | Reduce prompts between tasks |
| `--delegate` | Use subagent for each task |
| `--parallel` | Run multiple subagents concurrently (requires `--delegate`) |
| `--model MODEL` | Model size for delegation: `small` (haiku), `medium` (sonnet), `large` (opus) |

Defaults come from `[implement]` in `foundry-mcp.toml`.

**Execution modes:**
| Mode | Flags | Best for |
|------|-------|----------|
| Interactive | (none) | Learning, complex work |
| Autonomous | `--auto` | Routine tasks, faster flow |
| Delegated | `--delegate` | Many tasks, fresh context per task |
| Parallel | `--delegate --parallel` | Independent tasks, max speed |

**Task lifecycle:**
```
pending → in_progress → completed
                     ↘ blocked
```

**Key actions:**
| Action | What it does |
|--------|--------------|
| Prepare | Find next recommended task |
| Start | Mark task in_progress |
| Complete | Mark task completed |
| Block | Mark task blocked |
| Add dependency | Link tasks |

---

### foundry-review

Verify implementation matches specification.

**When to use:**
- After completing a phase
- Before final testing
- To check implementation quality

**Invoking:**
```
Run foundry-review on phase 1.
Run foundry-review on the entire spec.
Use foundry-review to check task-2-3.
```

**Review types:**
| Type | Scope |
|------|-------|
| Task review | Single task |
| Phase review | All tasks in phase |
| Spec review | Entire specification |

**Deviation categories:**
| Category | Meaning |
|----------|---------|
| Exact Match | Code matches spec |
| Minor Deviation | Small difference |
| Major Deviation | Significant difference |
| Missing | Not implemented |

---

### foundry-research

AI-powered research workflows.

**When to use:**
- Complex investigations
- Design decisions needing multiple perspectives
- Learning about unfamiliar topics
- Web research for best practices

**Invoking:**
```
Use foundry-research to explore how authentication works in this codebase
Use foundry-research consensus to get perspectives on Redis vs PostgreSQL for sessions
Use foundry-research thinkdeep to investigate why this test is flaky
Use foundry-research deep to research API rate limiting best practices 2025
```

**Workflows:**
| Workflow | Description | Use for |
|----------|-------------|---------|
| `chat` | Single-model conversation | Quick questions, iteration |
| `consensus` | Multiple AI perspectives | Design decisions, trade-offs |
| `thinkdeep` | Systematic investigation | Complex debugging, analysis |
| `ideate` | Creative brainstorming | Exploring options |
| `deep` | Web research (background) | Comprehensive research |

**Thread management:**
```
Use foundry-research to continue thread-abc123: What about security implications?
Use foundry-research to list sessions
Use foundry-research to get session research-xyz789
```

---

## Configuration

### foundry-mcp.toml

Project-level configuration file.

**Location:** Project root

**Key settings:**
```toml
[workspace]
specs_dir = "./specs"         # Where specs are stored

[implement]
auto = false                  # Default for --auto flag
delegate = false              # Default for --delegate flag
parallel = false              # Default for --parallel flag
model = "small"               # Model size for delegation (small=haiku, medium=sonnet, large=opus)

[consultation]
priority = [
  "[cli]codex:gpt-5.2-codex",
  "[cli]opencode:gpt-5.2-codex",
  "[cli]gemini:pro",
  "[cli]cursor-agent:composer-1",
  "[cli]claude:opus",
]
default_timeout = 300         # Seconds for AI calls

[research]
default_provider = "[cli]codex:gpt-5.2-codex"
```

---

## Quick Reference Card

### Common skill invocations
```
Use foundry-setup --check              # Verify installation
Use foundry-spec to plan [feature]     # Create a specification
Use foundry-implement                  # Start/resume work
Use foundry-implement --auto           # Less prompting
Use foundry-review on phase 1          # Verify implementation
Use foundry-research chat [question]   # Quick research
```

---

## Next Steps

- **[Troubleshooting](06-troubleshooting.md)** - When things don't work as expected
