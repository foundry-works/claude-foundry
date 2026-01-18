# Writing OpenCode Agent Skills: A Practical Guide with Examples

> Everything you need on your toolbelt to create reusable AI capabilities

If you've spent more than a few sessions with OpenCode (or any other agentic coding harness), you've probably noticed a pattern: you keep explaining the same things.

- “Our team uses conventional commits.”
- “Always filter out test accounts from analytics queries.”
- “The API response comes back nested three levels deep…”

**Agent Skills solve this.** They are files that capture your expertise once, acting as onboarding documents for your AI assistant *that it actually follows.*

OpenCode implements the **Agent Skills open standard**, meaning the same skills work across **Claude Code, VS Code, Cursor, Gemini CLI**, and many other tools.

> **Note:** Performance varies by model. Codex and Claude models (Opus/Sonnet) currently tend to leverage progressive disclosure more effectively than others.

---

## What Are Agent Skills, Really?

At their core, skills are folders containing a `SKILL.md` file. **No special tooling or deployment pipelines are required.**

The key trait is **on-demand loading**. Your AI doesn't need to know how to process PDFs until you hand it a PDF. Skills let the AI discover what's available and load the full instructions only when needed.

---

## A Brief History: From Anthropic to Open Standard

Anthropic introduced Agent Skills in **October 2025** as part of Claude Code. It was released as an open standard at **agentskills.io**.

### Platforms supporting Agent Skills

- Claude Code & Claude.ai  
- OpenCode (open-source)  
- OpenAI Codex  
- Gemini CLI  
- GitHub Copilot  
- VS Code & Cursor  
- Goose (Block)  
- Amp, Factory, Letta  

---

## The Anatomy of a `SKILL.md` File

A working skill consists of **YAML frontmatter (metadata)** and **Markdown content**.

### Example: Minimal Skill

```yaml
---
name: conventional-commits
description: Generate commit messages following conventional commit format. Use when the user asks for help with git commits.
---

# Conventional Commits

Format: `type(scope): description`

Types:
- feat: new feature
- fix: bug fix
- docs: documentation
- refactor: code restructuring
```

---

## Directory Structure

Skills live in folders named after the skill. OpenCode searches these locations in order:

1. **Project-level:** `.opencode/skill/<name>/SKILL.md`  
2. **Claude-compatible:** `.claude/skills/<name>/SKILL.md`  
3. **Global:** `~/.config/opencode/skill/<name>/SKILL.md`  

---

## Required Frontmatter Fields

- **`name`**: 1–64 characters, lowercase alphanumeric with hyphens (e.g., `pdf-processing`)  
- **`description`**: 1–1024 characters. Describes what the skill does and when to use it. This determines if the skill gets activated.

---

## Progressive Disclosure: Why Skills Scale

Skills use a **three-level architecture** to keep token usage efficient:

1. **Level 1: Metadata (Startup)**  
   AI loads just names and descriptions (~100 tokens per skill).

2. **Level 2: Instructions (Activation)**  
   AI reads the full `SKILL.md` body (recommended < 5,000 tokens).

3. **Level 3: Resources (Execution)**  
   AI loads bundled scripts or reference files only when referenced.

---

## Example Gallery

### Example: With Bundled Scripts

The AI can execute these scripts without loading their source code into the context window.

```markdown
# Database Migrations
## Quick Start
Generate a new migration:
`node scripts/create-migration.js "add_user_preferences_table"`
```

**Folder Structure:**

```text
database-migrations/
├── SKILL.md
└── scripts/
    ├── create-migration.js
    └── validate-migration.js
```

### Example: Permission-Restricted Skill

You can restrict access in `opencode.json` using wildcards:

```json
{
  "permission": {
    "skill": {
      "*": "allow",
      "internal-*": "deny",
      "experimental-*": "ask"
    }
  }
}
```

---

## Mistakes to Avoid

- **Nested Reference Chains:** Don’t point from `SKILL.md` → `advanced.md` → `details.md`. Keep references one level deep.
- **Over-Explaining:** The AI is smart. Don’t explain what a PDF is; just provide the code snippet for extraction.
- **Vague Descriptions:** “Helps with documents” is bad. Be specific: “Extract text and tables from PDF files.”
- **Windows-Style Paths:** Always use forward slashes (`/`). Backslashes break on Mac/Linux.

---

## Testing Your Skills

Anthropic recommends an iterative approach:

1. **Agent A:** Helps you write/refine the skill  
2. **Agent B:** A fresh instance used to test the skill on real tasks  
3. Bring failures from B back to A for refinement

---

## Quick Reference: File Locations

| Type | Path |
|---|---|
| Local | `.opencode/skill/<name>/SKILL.md` |
| Global | `~/.config/opencode/skill/<name>/SKILL.md` |
| Claude-Compatible | `.claude/skills/<name>/SKILL.md` |

---

## References

- OpenCode Skills Documentation  
- Agent Skills Specification  
- Anthropic Engineering Blog
