# Plugin Development Guide

> A comprehensive methodology for building Claude Code plugins from concept to distribution.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

This guide walks through the complete plugin development process:

1. **Discovery** - Understanding the problem
2. **Design** - Choosing features and architecture
3. **Implementation** - Building incrementally
4. **Testing** - Validation approaches
5. **Distribution** - Publishing and maintenance

Plus a **worked example** building a real plugin from scratch.

---

## Part 1: Development Lifecycle

### Phase 1: Discovery

Before writing code, understand what you're building.

**Questions to answer:**

| Question | Why It Matters |
|----------|----------------|
| What problem does this solve? | Defines scope and success criteria |
| Who will use it? | Just you, your team, or public? |
| What workflows does it support? | Identifies which features to use |
| What already exists? | Avoid duplicating effort |

**Discovery checklist:**

- [ ] Problem statement written in one sentence
- [ ] Target users identified
- [ ] Key workflows listed (3-5 max)
- [ ] Existing solutions researched

---

### Phase 2: Requirements Analysis

Translate discovery into concrete requirements.

**Categorize requirements:**

| Category | Examples |
|----------|----------|
| **Functional** | "User can deploy with one command" |
| **Behavioral** | "Claude remembers our coding standards" |
| **Integration** | "Connects to our internal API" |
| **Security** | "Never accesses .env files" |

**Map requirements to features:**

| Requirement Type | Likely Feature |
|------------------|----------------|
| User-triggered action | Command |
| Complex autonomous workflow | Agent |
| Reusable expertise | Skill |
| Always-on guidelines | CLAUDE.md |
| Event-driven automation | Hook |
| External tool integration | MCP Server |

---

### Phase 3: Architecture

Design your plugin structure before implementing.

**Key decisions:**

1. **Which features to use?** (See [01-choosing-features.md](./01-choosing-features.md))
2. **How do features interact?**
3. **What permissions are needed?**
4. **How will it be distributed?**

**Architecture template:**

```
Plugin: [name]
Purpose: [one sentence]

Features:
├── Commands: [list with purpose]
├── Agents: [list with purpose]
├── Skills: [list with purpose]
├── Hooks: [list with triggers]
└── MCP Servers: [list with integrations]

Permissions needed:
├── Tools: [Bash, Read, Write, etc.]
└── Files: [patterns]

Distribution:
├── Scope: [personal/team/public]
└── Location: [git repo, marketplace]
```

---

### Phase 4: Implementation

Build incrementally, validating as you go.

**Implementation order:**

1. **Scaffold** - Create directory structure and plugin.json
2. **Core feature** - Build the most important feature first
3. **Supporting features** - Add features that enhance the core
4. **Integration** - Connect features together
5. **Polish** - Add error handling, documentation

**Incremental validation:**

| After Building | Validate |
|----------------|----------|
| Each command | Invoke with `/command`, check behavior |
| Each agent | Trigger delegation, verify results |
| Each skill | Check auto-activation in context |
| Each hook | Trigger event, verify execution |
| MCP server | Check `/mcp` status, invoke tools |

---

### Phase 5: Testing

Validate your plugin works correctly.

**Testing approaches:**

| Type | How |
|------|-----|
| **Manual** | Use the plugin in real workflows |
| **Edge cases** | Test with unusual inputs |
| **Permissions** | Verify security constraints work |
| **Integration** | Test feature combinations |

**Testing checklist:**

- [ ] All commands invoke correctly
- [ ] Agents activate when expected
- [ ] Skills trigger on matching context
- [ ] Hooks fire on events
- [ ] MCP servers connect and respond
- [ ] Permissions block what they should
- [ ] Documentation matches behavior

---

### Phase 6: Distribution

Share your plugin with others.

**Distribution options:**

| Method | Audience | Setup |
|--------|----------|-------|
| Git repository | Team | Clone and link |
| Marketplace | Public | Publish with metadata |
| Local only | Personal | No distribution needed |

**Pre-distribution checklist:**

- [ ] README.md with usage examples
- [ ] LICENSE file included
- [ ] CHANGELOG.md started
- [ ] No hardcoded secrets
- [ ] Tested on fresh install

---

## Part 2: Decision Framework

### Analyzing Requirements

For each requirement, ask:

```
1. WHO triggers this?
   ├── User explicitly → Command
   ├── Claude automatically → Skill or Agent
   └── System event → Hook

2. WHAT is the scope?
   ├── Simple, quick action → Command
   ├── Complex, multi-step → Agent
   └── Knowledge/expertise → Skill

3. WHEN should it happen?
   ├── On demand → Command
   ├── Always (standards) → CLAUDE.md
   ├── Context-dependent → Skill
   └── Event-triggered → Hook

4. WHERE does it run?
   ├── Main conversation → Command, Skill, CLAUDE.md
   ├── Isolated context → Agent
   └── External system → MCP Server
```

### Feature Combinations

Sometimes you need multiple features together:

| Combination | Use Case |
|-------------|----------|
| Skill + Agent | Expertise that needs isolated execution |
| Command + Hook | User action with pre/post automation |
| CLAUDE.md + Skill | Standards + detailed expertise |
| MCP + Agent | External data with complex processing |

**Example: Code Review Plugin**

```
CLAUDE.md     → Team coding standards (always active)
Skill         → Security expertise (auto-activated)
Agent         → Deep review workflow (isolated)
Command       → /review shortcut (user trigger)
Hook          → Format on save (automation)
```

### Trade-off Analysis

| Decision | Trade-off |
|----------|-----------|
| Command vs Skill | Explicit control vs auto-activation |
| Agent vs Skill | Isolation vs main context |
| CLAUDE.md vs Skill | Simplicity vs progressive disclosure |
| Hook vs Command | Automatic vs manual |
| MCP vs Bash | Standard protocol vs simple scripts |

---

## Part 3: Worked Example

Let's build a **"Code Quality Toolkit"** plugin from scratch.

### Step 1: Discovery

**Problem**: Our team needs consistent code quality checks and patterns.

**Users**: Development team (5 people)

**Workflows**:
1. Quick code review before commits
2. Security vulnerability scanning
3. Performance analysis
4. Following team coding standards

**Existing solutions**: None integrated with Claude Code

---

### Step 2: Requirements

| Requirement | Type | Priority |
|-------------|------|----------|
| Apply team coding standards | Behavioral | Must |
| Quick review command | Functional | Must |
| Security expertise | Functional | Should |
| Performance analysis | Functional | Should |
| Auto-format on save | Automation | Nice |

---

### Step 3: Architecture

```
Plugin: code-quality-toolkit
Purpose: Ensure consistent code quality across the team

Features:
├── CLAUDE.md: Team coding standards
├── Commands:
│   └── /review - Quick code review
├── Skills:
│   ├── security-analysis/ - Security expertise
│   └── performance-analysis/ - Performance expertise
├── Agents:
│   └── deep-reviewer - Thorough review workflow
└── Hooks:
    └── PostToolUse on Write - Auto-format

Permissions:
├── Tools: Read, Grep, Glob, Bash(eslint:*), Bash(prettier:*)
└── Deny: Read(.env*)

Distribution: Team git repository
```

---

### Step 4: Implementation

#### 4.1 Scaffold

Create the directory structure:

```
code-quality-toolkit/
├── .claude-plugin/
│   └── plugin.json
├── commands/
├── agents/
├── skills/
├── hooks/
├── CLAUDE.md
├── README.md
└── LICENSE
```

#### 4.2 Plugin Manifest

`.claude-plugin/plugin.json`:

```json
{
  "name": "code-quality-toolkit",
  "version": "1.0.0",
  "description": "Team code quality standards and review tools",
  "author": {
    "name": "Development Team"
  },
  "license": "MIT",
  "keywords": ["code-quality", "review", "security"]
}
```

#### 4.3 Team Standards (CLAUDE.md)

`CLAUDE.md`:

```markdown
# Code Quality Standards

## Style
- TypeScript with strict mode
- 2-space indentation
- Single quotes, no semicolons
- Max line length: 100 characters

## Patterns
- React functional components only
- Custom hooks for shared logic
- Zustand for state management
- Error boundaries around async operations

## Naming
- Components: PascalCase
- Hooks: useCamelCase
- Files: kebab-case.tsx
- Constants: SCREAMING_SNAKE_CASE

## Testing
- Jest for unit tests
- React Testing Library for components
- Minimum 80% coverage for new code

## Security
- Never commit secrets
- Validate all user input
- Use parameterized queries
- Sanitize output for XSS
```

#### 4.4 Quick Review Command

`commands/review.md`:

```yaml
---
description: Quick code review of recent changes
argument-hint: [file-or-pattern]
allowed-tools: Read, Grep, Glob, Bash(git diff:*)
---

Review the code for quality issues.

If a file or pattern is provided, review: $ARGUMENTS
Otherwise, review recent changes: !`git diff --name-only HEAD~1`

Check for:
1. Coding standard violations (see CLAUDE.md)
2. Potential bugs or logic errors
3. Security concerns
4. Performance issues
5. Missing error handling

Provide a summary with:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (nice to have)
```

#### 4.5 Security Skill

`skills/security-analysis/SKILL.md`:

```yaml
---
name: security-analysis
description: Analyze code for security vulnerabilities. Use when reviewing code for security issues or auditing for vulnerabilities.
allowed-tools: Read, Grep
---

# Security Analysis Expertise

## Vulnerability Categories

### Injection
- SQL injection via string concatenation
- Command injection in shell calls
- LDAP/XPath injection

### Authentication & Authorization
- Hardcoded credentials
- Weak password policies
- Missing auth checks
- Privilege escalation paths

### Data Exposure
- Sensitive data in logs
- Unencrypted storage
- Exposed API keys
- Information leakage in errors

### XSS & Output
- Unsanitized user input in HTML
- innerHTML usage
- Template injection

## Analysis Workflow

1. Identify data entry points
2. Trace data flow through the code
3. Check for validation and sanitization
4. Verify authentication/authorization
5. Review error handling
6. Check dependency vulnerabilities

## Severity Ratings

- **Critical**: Exploitable with immediate impact
- **High**: Exploitable with significant impact
- **Medium**: Requires specific conditions
- **Low**: Minimal impact or unlikely
```

#### 4.6 Performance Skill

`skills/performance-analysis/SKILL.md`:

```yaml
---
name: performance-analysis
description: Analyze code for performance issues. Use when reviewing code for performance or investigating slow operations.
allowed-tools: Read, Grep
---

# Performance Analysis Expertise

## Common Issues

### React Performance
- Missing memo/useMemo/useCallback
- Unnecessary re-renders
- Large component trees
- Missing keys in lists

### Data & Algorithms
- N+1 query patterns
- Inefficient loops
- Missing indexes
- Large data in memory

### Network
- Excessive API calls
- Missing caching
- Large payloads
- No pagination

### Bundle Size
- Unused dependencies
- Missing code splitting
- Large imports

## Analysis Workflow

1. Identify hot paths and frequently called code
2. Check for algorithmic complexity issues
3. Review data fetching patterns
4. Analyze component render patterns
5. Check for memory leaks

## Recommendations Format

- Impact: High/Medium/Low
- Effort: High/Medium/Low
- Suggestion: Specific fix
```

#### 4.7 Deep Review Agent

`agents/deep-reviewer.md`:

```yaml
---
name: deep-reviewer
description: Thorough code review with security, performance, and best practices analysis
tools: Read, Grep, Glob
model: sonnet
---

You are an expert code reviewer performing a thorough analysis.

## Review Process

1. **Understand Context**
   - What does this code do?
   - What is it part of?
   - Who will use it?

2. **Check Standards Compliance**
   - Review against team coding standards
   - Check naming conventions
   - Verify patterns are followed

3. **Security Review**
   - Apply security analysis expertise
   - Check for OWASP Top 10 issues
   - Verify input validation

4. **Performance Review**
   - Apply performance analysis expertise
   - Check for bottlenecks
   - Review data patterns

5. **Best Practices**
   - Error handling
   - Edge cases
   - Testability
   - Maintainability

## Output Format

### Summary
[2-3 sentence overview]

### Critical Issues
[Must fix before merge]

### Warnings
[Should fix]

### Suggestions
[Nice to have]

### Positive Observations
[What's done well]
```

#### 4.8 Auto-Format Hook

`hooks/hooks.json`:

```json
{
  "PostToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "command",
          "command": "prettier --write \"$FILE\" 2>/dev/null || true",
          "timeout": 10
        }
      ]
    }
  ]
}
```

---

### Step 5: Testing

**Test the command:**
```
/review src/components/Button.tsx
```

**Test skill activation:**
```
User: "Check this code for security issues"
[security-analysis skill should activate]
```

**Test agent delegation:**
```
User: "Do a thorough review of the authentication module"
[deep-reviewer agent should be invoked]
```

**Test hook:**
```
[Edit a file, verify prettier runs]
```

---

### Step 6: Distribution

**README.md:**

```markdown
# Code Quality Toolkit

A Claude Code plugin for team code quality standards.

## Installation

1. Clone this repository
2. Link as a Claude Code plugin

## Features

- `/review` - Quick code review
- Security analysis (auto-activated)
- Performance analysis (auto-activated)
- Deep review agent for thorough analysis
- Auto-formatting on save

## Team Standards

See CLAUDE.md for our coding standards.
```

**Final structure:**

```
code-quality-toolkit/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   └── review.md
├── agents/
│   └── deep-reviewer.md
├── skills/
│   ├── security-analysis/
│   │   └── SKILL.md
│   └── performance-analysis/
│       └── SKILL.md
├── hooks/
│   └── hooks.json
├── CLAUDE.md
├── README.md
├── LICENSE
└── CHANGELOG.md
```

---

## Part 4: Best Practices

### Development Patterns

| Pattern | Description |
|---------|-------------|
| **Start small** | Build one feature, validate, then expand |
| **Validate early** | Test each component before building more |
| **Document as you go** | Don't leave documentation for the end |
| **Version from start** | Use semantic versioning from v1.0.0 |

### Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Building everything at once | Implement incrementally |
| Skipping testing | Test each feature immediately |
| Hardcoding values | Use environment variables |
| Vague descriptions | Write specific, actionable text |
| Overly broad tools | Restrict to minimum needed |

### Maintenance

| Activity | Frequency |
|----------|-----------|
| Review CLAUDE.md accuracy | Monthly |
| Update dependencies | As needed |
| Audit permissions | Quarterly |
| Gather user feedback | Ongoing |

### Evolution Strategy

```
v1.0 → Core functionality working
v1.1 → User feedback incorporated
v1.2 → Additional features
v2.0 → Breaking changes (if needed)
```

---

## Summary Checklist

### Before Starting
- [ ] Problem clearly defined
- [ ] Users identified
- [ ] Requirements documented

### During Development
- [ ] Features chosen based on requirements
- [ ] Directory structure created
- [ ] Each feature tested individually
- [ ] Features integrated and tested together

### Before Distribution
- [ ] README.md complete
- [ ] LICENSE included
- [ ] No secrets in code
- [ ] Permissions are minimal
- [ ] Documentation matches behavior

---

## Related Documents

- [01-choosing-features.md](./01-choosing-features.md) - Feature selection guide
- [07-plugin-manifest.md](./07-plugin-manifest.md) - plugin.json reference
- [09-file-structure.md](./09-file-structure.md) - Directory conventions

---

**Navigation:** [Index](./README.md) | [Next: Choosing Features →](./01-choosing-features.md)
