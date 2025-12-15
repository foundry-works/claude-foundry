# Testing

> Using foundry-mcp testing tools for test discovery, execution, and debugging.

---

## Testing Philosophy

foundry-mcp treats verification as first-class:

- **Automated tests** — Defined in verification criteria
- **Fidelity reviews** — AI comparison of code vs spec
- **Manual checklists** — Human review requirements

Testing tools integrate with pytest and support external AI consultation for debugging.

---

## Test Discovery

### Discover All Tests

```
mcp__plugin_foundry_foundry-mcp__test action="discover" target="tests/"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "test_count": 150,
    "modules": [
      "tests/unit/test_auth.py",
      "tests/unit/test_models.py",
      "tests/integration/test_api.py"
    ],
    "markers": ["unit", "integration", "slow"]
  }
}
```

### Discover by Marker

```
mcp__plugin_foundry_foundry-mcp__test action="discover" target="tests/" markers="unit"
```

---

## Running Tests

### Basic Test Run

```
mcp__plugin_foundry_foundry-mcp__test action="run" target="tests/"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "passed": 145,
    "failed": 3,
    "skipped": 2,
    "duration": 12.5,
    "failures": [
      {
        "test": "tests/unit/test_auth.py::test_login_invalid",
        "error": "AssertionError: Expected 401, got 200",
        "output": "..."
      }
    ]
  }
}
```

### Quick Tests

Fast smoke tests for rapid feedback:

```
mcp__plugin_foundry_foundry-mcp__test action="run-quick"
```

**Characteristics:**
- Runs tests marked as `quick` or `smoke`
- Stops on first failure (`-x` flag)
- Limited output

### Unit Tests Only

```
mcp__plugin_foundry_foundry-mcp__test action="run-unit"
```

### With Options

```
mcp__plugin_foundry_foundry-mcp__test action="run"
  target="tests/unit/"
  markers="not slow"
  verbose=true
```

---

## Test Presets

### List Available Presets

```
mcp__plugin_foundry_foundry-mcp__test action="presets"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "presets": [
      {
        "name": "quick",
        "description": "Fast smoke tests",
        "markers": "smoke or quick",
        "options": "-x --tb=short"
      },
      {
        "name": "full",
        "description": "Complete test suite",
        "markers": "",
        "options": "-v --tb=long"
      },
      {
        "name": "unit",
        "description": "Unit tests only",
        "markers": "unit",
        "options": "-v"
      },
      {
        "name": "integration",
        "description": "Integration tests",
        "markers": "integration",
        "options": "-v --tb=long"
      }
    ]
  }
}
```

### Run with Preset

```
mcp__plugin_foundry_foundry-mcp__test action="run" preset="quick"
```

---

## Debugging Test Failures

### Understanding Failure Output

Test failures include:

```json
{
  "failures": [
    {
      "test": "tests/unit/test_auth.py::test_login_invalid",
      "error": "AssertionError",
      "message": "Expected 401, got 200",
      "traceback": "...",
      "output": "...",
      "fixture_info": ["client", "db_session"]
    }
  ]
}
```

### External AI Consultation

Route debugging to external AI tools:

```
mcp__plugin_foundry_foundry-mcp__provider action="execute"
  provider_id="gemini"
  prompt="Debug: AssertionError: Expected 401, got 200"
```

**Available tools:**
| Tool | Use Case |
|------|----------|
| `gemini` | General debugging |
| `cursor` | IDE-integrated |
| `codex` | Code-focused |

**Response:**
```json
{
  "success": true,
  "data": {
    "consultation": {
      "root_cause": "Login endpoint returns 200 for invalid credentials",
      "suggestions": [
        "Check authentication middleware",
        "Verify error handling in login view"
      ],
      "similar_patterns": [
        "This often happens when...",
      ]
    }
  }
}
```

### Investigation Workflow

1. **Run failing test** — Get detailed output
2. **Check context** — Use `code` tools for code context
3. **Consult AI** — External debugging help
4. **Fix and verify** — Run test again

```
# 1. Run the failing test
mcp__plugin_foundry_foundry-mcp__test action="run" target="tests/unit/test_auth.py::test_login_invalid" verbose=true

# 2. Get context
mcp__plugin_foundry_foundry-mcp__code action="find-function" symbol="login"

# 3. Consult
mcp__plugin_foundry_foundry-mcp__provider action="execute" provider_id="gemini" prompt="Debug: ..."

# 4. Fix and re-run
mcp__plugin_foundry_foundry-mcp__test action="run" target="tests/unit/test_auth.py::test_login_invalid"
```

---

## Verification Tasks in Specs

### Verification Types

| Type | Description | Tool |
|------|-------------|------|
| `auto` | Automated command | `test-run` |
| `fidelity` | AI comparison | `fidelity-check` |
| `manual` | Human checklist | `journal-add` |

### Spec Verification Definition

```json
{
  "task_id": "task-001",
  "verification": {
    "type": "auto",
    "command": "pytest tests/unit/test_auth.py -v",
    "expected_outcome": "All tests pass"
  }
}
```

### Phase Verification

```json
{
  "phase_id": "phase-1",
  "verification": {
    "type": "fidelity",
    "criteria": [
      "All auth endpoints implement spec requirements",
      "Error handling matches spec definitions"
    ]
  }
}
```

### Running Verification

```
mcp__plugin_foundry_foundry-mcp__verification action="execute"
  spec_id="my-feature"
  verify_id="phase-1"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "verification_results": [
      {
        "task_id": "task-001",
        "type": "auto",
        "passed": true,
        "output": "4 passed in 0.5s"
      },
      {
        "task_id": "task-002",
        "type": "fidelity",
        "passed": true,
        "findings": []
      }
    ],
    "all_passed": true
  }
}
```

---

## Test Coverage

### Running with Coverage

```
mcp__plugin_foundry_foundry-mcp__test action="run"
  target="tests/"
  verbose=true
```

**Response includes:**
```json
{
  "data": {
    "coverage": {
      "total_percent": 85,
      "by_module": {
        "src/auth/": 92,
        "src/models/": 78,
        "src/api/": 88
      },
      "report_path": "htmlcov/index.html"
    }
  }
}
```

---

## Continuous Integration

### CI-Friendly Output

```
mcp__plugin_foundry_foundry-mcp__test action="run" target="tests/" verbose=true
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All tests passed |
| 1 | Some tests failed |
| 2 | Test collection error |
| 5 | No tests found |

### CI Configuration Example

```yaml
# GitHub Actions
- name: Run Tests
  run: |
    python -m foundry_mcp.cli test run --path tests/ --format junit --output test-results.xml

- name: Upload Results
  uses: actions/upload-artifact@v3
  with:
    name: test-results
    path: test-results.xml
```

---

## Best Practices

### Test Organization

```
tests/
├── unit/           # Fast, isolated tests
├── integration/    # Cross-component tests
├── fixtures/       # Shared fixtures
└── conftest.py     # pytest configuration
```

### Marking Tests

```python
import pytest

@pytest.mark.unit
def test_user_model():
    ...

@pytest.mark.integration
def test_login_flow():
    ...

@pytest.mark.slow
def test_large_dataset():
    ...
```

### Verification in Specs

**DO:**
- Define verification for each task
- Use automated tests where possible
- Document manual verification steps

**DON'T:**
- Skip verification definitions
- Rely solely on manual testing
- Leave verification criteria vague

---

## Troubleshooting

### Tests Not Found

```json
{
  "success": false,
  "data": {
    "error_code": "NO_TESTS_FOUND",
    "remediation": "Check test path and naming conventions"
  }
}
```

**Solutions:**
1. Verify test files match `test_*.py` pattern
2. Check path is correct
3. Ensure pytest is installed

### Fixture Errors

```json
{
  "failures": [{
    "error": "fixture 'db_session' not found"
  }]
}
```

**Solutions:**
1. Check `conftest.py` for fixture definition
2. Verify fixture scope
3. Check imports

### Timeout Issues

**Solutions:**
1. Mark slow tests with `@pytest.mark.slow`
2. Use quick preset for fast feedback
3. Increase timeout in configuration

---

## Related Documentation

- **[06-Workflows](./06-workflows.md)** — Verification workflow
- **[09-LLM Providers](./09-llm-providers.md)** — Test consultation
- **[04-Tool Reference](./04-tool-reference.md)** — Test tools

---

*[Back to Index](./README.md)*
