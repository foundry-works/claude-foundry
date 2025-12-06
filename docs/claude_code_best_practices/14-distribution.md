# 14. Plugin Distribution

> Publishing, versioning, and maintaining Claude Code plugins through marketplaces and direct distribution.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Plugin distribution covers:

- **Marketplace publishing** - Public discovery and installation
- **Direct distribution** - Git repositories and private sharing
- **Version management** - Semantic versioning and updates
- **Maintenance** - Updates, deprecation, and support

---

## Distribution Methods

### Marketplace Distribution

Publish to Claude Code marketplaces for discovery:

```bash
# Add a plugin from marketplace
claude /plugin marketplace add username/plugin-name

# Browse marketplace
claude /plugin
```

### Direct Git Distribution

Share plugins via Git repositories:

```bash
# Install from GitHub
claude /plugin add github:username/repo

# Install from any Git URL
claude /plugin add git:https://gitlab.com/user/plugin.git
```

### Local Distribution

Share plugins via filesystem:

```bash
# Install from local path
claude /plugin add /path/to/plugin

# Install from zip archive
claude /plugin add /path/to/plugin.zip
```

---

## Marketplace Submission

### Preparation Checklist

Before submitting to a marketplace:

| Requirement | Details |
|-------------|---------|
| MUST | Valid `plugin.json` with all required fields |
| MUST | `README.md` with usage documentation |
| MUST | `LICENSE` file with appropriate license |
| SHOULD | `CHANGELOG.md` with version history |
| SHOULD | Meaningful keywords for discovery |
| SHOULD | Clear description (under 200 characters) |

### Required Metadata

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Brief description for marketplace listing",
  "author": {
    "name": "Your Name",
    "email": "email@example.com"
  },
  "license": "MIT",
  "keywords": ["relevant", "keywords"],
  "repository": "https://github.com/user/plugin"
}
```

### Submission Process

1. **Prepare repository**
   - Ensure all required files present
   - Tag release with version number
   - Push to public repository

2. **Submit to marketplace**
   ```bash
   # Official marketplace (when available)
   claude /plugin marketplace submit

   # Community marketplaces follow their own process
   ```

3. **Await review**
   - Automated validation checks
   - Manual review (for official marketplace)

4. **Publication**
   - Plugin appears in marketplace
   - Users can install via `/plugin`

---

## Community Marketplaces

### Setting Up a Marketplace

Create your own plugin marketplace:

```json
// marketplace.json
{
  "name": "my-marketplace",
  "description": "Custom plugin collection",
  "plugins": [
    {
      "name": "plugin-one",
      "version": "1.0.0",
      "source": "github:user/plugin-one",
      "description": "First plugin"
    },
    {
      "name": "plugin-two",
      "version": "2.1.0",
      "source": "github:user/plugin-two",
      "description": "Second plugin"
    }
  ]
}
```

### Hosting a Marketplace

```bash
# Users add your marketplace
claude /plugin marketplace add github:your-org/marketplace

# Or via URL
claude /plugin marketplace add https://your-domain.com/marketplace.json
```

---

## Version Management

### Semantic Versioning

Follow semantic versioning (semver):

| Version Part | When to Increment |
|--------------|-------------------|
| MAJOR (X.0.0) | Breaking changes |
| MINOR (0.X.0) | New features (backward compatible) |
| PATCH (0.0.X) | Bug fixes |

### Version Lifecycle

```
1.0.0  Initial release
1.0.1  Bug fix
1.1.0  New feature
1.2.0  Another feature
2.0.0  Breaking change
```

### Breaking Changes

Document breaking changes clearly:

```markdown
## CHANGELOG

### 2.0.0 (Breaking)
- **BREAKING**: Renamed `/old-command` to `/new-command`
- **BREAKING**: Changed agent output format
- Added new skill for PDF processing

### Migration Guide
1. Update command references from `/old-command` to `/new-command`
2. Update any code parsing agent output to new format
```

---

## Update Workflow

### Releasing Updates

1. **Update version in plugin.json**
   ```json
   {
     "version": "1.1.0"
   }
   ```

2. **Update CHANGELOG.md**
   ```markdown
   ## 1.1.0 (2025-11-15)
   - Added new security scanning skill
   - Fixed hook timeout issue
   ```

3. **Tag release**
   ```bash
   git add .
   git commit -m "Release v1.1.0"
   git tag v1.1.0
   git push origin main --tags
   ```

4. **Users update**
   ```bash
   # Check for updates
   claude /plugin update

   # Update specific plugin
   claude /plugin update my-plugin
   ```

### Pre-release Versions

Use pre-release tags for testing:

```
2.0.0-alpha.1
2.0.0-beta.1
2.0.0-rc.1
2.0.0
```

---

## Documentation Requirements

### README.md Structure

```markdown
# Plugin Name

Brief description of what the plugin does.

## Installation

\`\`\`bash
claude /plugin add github:user/plugin
\`\`\`

## Features

- Feature 1
- Feature 2

## Commands

### /plugin-name:command
Description and usage.

## Agents

### agent-name
When and how to use.

## Skills

Skills included and what they do.

## Configuration

Any configuration options.

## License

MIT
```

### CHANGELOG.md Structure

```markdown
# Changelog

All notable changes to this plugin.

## [Unreleased]

## [1.1.0] - 2025-11-15
### Added
- New feature X

### Fixed
- Bug in Y

### Changed
- Behavior of Z

## [1.0.0] - 2025-10-01
- Initial release
```

---

## Deprecation

### Deprecating Components

When removing features:

1. **Announce deprecation** (1+ version before removal)
   ```markdown
   ## 1.2.0
   - **DEPRECATED**: `/old-command` will be removed in 2.0.0
     Use `/new-command` instead
   ```

2. **Add deprecation warning** (in component)
   ```yaml
   ---
   name: old-command
   description: "[DEPRECATED] Use new-command instead. Old functionality..."
   ---
   ```

3. **Remove in major version**
   ```markdown
   ## 2.0.0 (Breaking)
   - **REMOVED**: `/old-command` (use `/new-command`)
   ```

### Deprecating Entire Plugin

```markdown
# Plugin Name [DEPRECATED]

> **This plugin is deprecated.** Please use [new-plugin](link) instead.

## Migration Guide
...
```

---

## License and Attribution

### Choosing a License

| License | Use Case |
|---------|----------|
| MIT | Maximum permissiveness |
| Apache-2.0 | Permissive with patent protection |
| GPL-3.0 | Copyleft (derivative works must be open) |
| BSD-3-Clause | Similar to MIT with attribution |

### License File

Include full license text in `LICENSE` file:

```
MIT License

Copyright (c) 2025 Author Name

Permission is hereby granted...
```

### Attribution for Dependencies

If using third-party code:

```markdown
## Third-Party Licenses

This plugin includes code from:
- [library-name](link) - MIT License
- [other-lib](link) - Apache-2.0
```

---

## Discovery Optimization

### Keywords

Choose keywords that help users find your plugin:

```json
{
  "keywords": [
    "code-review",        // Function
    "security",           // Domain
    "python",             // Language
    "static-analysis"     // Technique
  ]
}
```

### Description

Write clear, searchable descriptions:

```json
// Good: Specific and searchable
{
  "description": "Automated Python code review with security scanning and style checking"
}

// Bad: Vague
{
  "description": "A helpful tool for developers"
}
```

---

## Best Practices

| Requirement | Details |
|-------------|---------|
| MUST | Include LICENSE file |
| MUST | Use semantic versioning |
| MUST | Document breaking changes |
| SHOULD | Include CHANGELOG.md |
| SHOULD | Provide migration guides for major versions |
| SHOULD | Use meaningful keywords |
| SHOULD | Keep description under 200 characters |
| MAY | Support multiple distribution methods |

---

## Related Documents

- [07-plugin-manifest.md](./07-plugin-manifest.md) - Plugin configuration
- [09-file-structure.md](./09-file-structure.md) - Directory layout
- [13-testing.md](./13-testing.md) - Testing before publication

---

**Navigation:** [← Previous: Testing](./13-testing.md) | [Index](./README.md) | [Next: Debugging →](./15-debugging.md)
