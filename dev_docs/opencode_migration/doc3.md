# OpenCode Migration Notes: Leaving Claude Code Without Losing Your Workflow

---

I wrote most of this post with OpenCode, as a summary of my migration — and then edited it myself.

I’d been quietly annoyed at Claude Code’s arbitrary, opaque quota limits for a while. I kept tolerating it not because I felt I couldn’t live without Opus or Sonnet (most models are pretty good these days if you have the slightest idea of what you’re doing), but because I’d invested in custom commands, agents, and skills for months — and all of that *just works* for me in Claude Code.

And the thought of rebuilding everything sounded painful. Then the “OpenCode max subscription ban” landed, and that was my last straw.

Obviously, I had to disconnect OpenCode from my Claude subscription, but it meant I had no reason left to keep paying them **$200/month** (well, already reduced to **$100** recently — and I bought some other subscriptions with the difference).

No warning. No clarity. And there were reports of accounts being banned for “misuse.”

People might justify it with *“they can do whatever they want with THEIR subscription,”* but then *“customers are free to leave as well.”* It felt like a toxic dependency: good when it worked, exhausting when it didn’t.

I decided to start cutting the cord for good.

I spent a few hours learning how OpenCode organizes commands, agents, and skills and realized I could migrate without nuking my Claude setup. That became the mission:

> **Keep my Claude assets usable while making them first‑class in OpenCode.**

---

## The three pillars I had to understand

OpenCode is structured around three things, and once I understood them the migration was mostly plumbing:

- **Commands:** slash commands with frontmatter  
- **Agents:** explicit roles with prompts + tool permissions  
- **Skills:** reusable instruction bundles loaded on demand  

---

## Migration strategy (keep Claude intact, add OpenCode wrappers)

I wanted **zero rewrites** in my Claude files. The simplest path was **wrappers + symlinks** so Claude stays the source of truth.

---

### 1) Skills: symlink Claude → OpenCode

This lets both tools use the same skills.

```bash
# Project
ln -s .claude/skills .opencode/skill

# User
ln -s ~/.claude/skills ~/.config/opencode/skill
```

---

### 2) Commands: wrap Claude commands with frontmatter

OpenCode needs frontmatter. Claude doesn’t. Wrappers let OpenCode read Claude commands without edits.

Example wrapper:

```yaml
---
description: Enforce code discipline checklist
agent: build
---
@.claude/commands/enforce-code-disciplines.md
```

I did this for all project commands (including the `priming/` folder) and all user‑level commands in `~/.claude/commands/`.

---

### 3) Agents: wrap Claude prompts

OpenCode agents can point to a prompt file, which makes them perfect wrappers for Claude agents.

```yaml
---
description: Senior code reviewer
mode: subagent
prompt: "{file:~/.claude/agents/senior-code-reviewer.md}"
---
```

I wrapped all my Claude agents as subagents to preserve behavior.

---

## Verification checks (how I proved it worked)

These were the concrete checks that confirmed OpenCode was seeing everything:

### Run

```bash
opencode debug skill
```

### Confirm

- Commands show up in the `/` palette  
- Agents show up in the `@` picker  
- Skills list correctly in `opencode debug skill`  

If skills don’t show up, I enabled them in `opencode.json`:

```json
{
  "tools": {
    "read": true,
    "write": true,
    "edit": true,
    "bash": true,
    "skill": true
  }
}
```

---

## Bonus: I turned the playbook into a skill

I didn’t want to repeat this on every repo, so I started by asking OpenCode to write a migration skill that:

- Scans Claude commands, agents, and skills  
- Creates OpenCode wrappers and symlinks  
- Verifies discovery  

Now migration is repeatable and documented instead of a one‑off bash ritual.

---

## Tips

- You can invoke skills by name in plain language. Unlike Claude Code, you don't have to keep asking it to “invoke the skill properly” (instead of just reading a single `SKILL.md` file and ignoring everything else). OpenCode invoked the skill and read every file mentioned in it.
- Skills can load fine even if the UI doesn’t surface them — use `opencode debug skill`.
- Wrappers won’t load without frontmatter.

---

## Final take

If Claude Code’s limits are starting to feel arbitrary, there is a clean exit ramp. You don’t have to throw away your existing commands or skills. OpenCode can run them while Claude remains intact.

I’m now fully migrated without losing anything, and it feels like getting my workflow back.

I do plan to use Claude Code for planning at times, but now I have options.

And honestly, I was only flirting with OpenCode for the last few months, but watching accounts being banned by Anthropic because customers didn’t like their tooling was a *dic\** move. And it made me realize that I just can’t let myself be fully dependent on such a company.

Now, if Anthropic suddenly decides to ban my account for some random reason, I can just walk away without being devastated.

Even with Opus 4.5, I have to spend time ensuring that the code is as per my preferences and standards — so for me, the loss would have been leaving behind the workflow that just worked for me. But now it seems that OpenCode is the best place for it to fit one-to-one.

If anyone wants the playbook or migration skill, here you go:

- **Repo:** https://github.com/SmrutAI/opencode-migration

Just install it, and it migrates everything.

Soon I’ll include a way to reuse Claude's `settings.json` and hooks, which are the last bit of attachment I have with Claude. (Safety net)

---
