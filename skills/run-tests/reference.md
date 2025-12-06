# Run-Tests Reference

Detailed troubleshooting, failure categories, and AI tool selection guidance for the run-tests skill.

## Table of Contents

- [Tool Selection](#tool-selection)
- [Failure Categories](#failure-categories)
- [Investigation Patterns](#investigation-patterns)
- [Common Fixes](#common-fixes)
- [Advanced Troubleshooting](#advanced-troubleshooting)

---

## Tool Selection

### By Failure Type

| Failure Type | Primary Tool | Why |
|--------------|--------------|-----|
| Assertion mismatch | Codex | Code-level bug analysis |
| Exceptions | Codex | Precise code review |
| Import/packaging | Gemini | Framework expertise |
| Fixture issues | Gemini | Pytest scoping knowledge |
| Timeout/performance | Gemini + Cursor | Strategy + discovery |
| Flaky tests | Gemini + Cursor | Diagnosis + state deps |
| Multi-file issues | Cursor | Discovery + synthesis |

### Prompt Templates

**For assertion failures:**
```
Test {test_name} in {file} is failing with an assertion error.
Expected: {expected}
Actual: {actual}

The test verifies: {test_purpose}
The implementation code is: {relevant_code}

What is likely causing the mismatch?
```

**For exception failures:**
```
Test {test_name} raises {error_type}: {error_message}

Stack trace:
{stack_trace}

What is likely causing this exception and how should it be fixed?
```

---

## Failure Categories

### Assertion Failures

Expected vs actual mismatch. Investigate:

- **Test expectations correct?** - Has the expected behavior changed?
- **Implementation changed?** - Did recent changes alter output?
- **Data setup correct?** - Is test data/fixtures properly configured?

**Common causes:**
- API response format changed
- Floating point precision issues
- String encoding differences
- Order-dependent comparisons (sets, dicts)

### Exception Failures

Runtime errors during test execution:

| Error Type | Common Cause | Fix |
|------------|--------------|-----|
| `AttributeError` | Object missing attribute | Check object initialization |
| `KeyError` | Dictionary key missing | Validate data structure |
| `TypeError` | Wrong argument types | Check function signatures |
| `ValueError` | Invalid value passed | Validate input data |
| `IndexError` | List index out of range | Check collection sizes |

### Import Failures

Module loading issues:

**Check these in order:**
1. PYTHONPATH correct? (`export PYTHONPATH=src:$PYTHONPATH`)
2. `__init__.py` files exist in all package directories?
3. Circular imports? (A imports B, B imports A)
4. Missing dependencies? (`pip install -r requirements.txt`)

**Diagnosing circular imports:**
```python
# In the test file, add at the top:
import sys
print([m for m in sys.modules if 'your_module' in m])
```

### Fixture Failures

Setup/teardown issues:

| Issue | Check |
|-------|-------|
| Fixture not found | Is it in conftest.py or same file? |
| Fixture name wrong | Does the parameter name match exactly? |
| Fixture scope issue | Is scope (function/class/module/session) appropriate? |
| Fixture dependency | Does fixture depend on another fixture that fails? |

**Fixture scopes:**
- `function` - New instance per test (default)
- `class` - Shared across class
- `module` - Shared across module
- `session` - Shared across entire test session

### Timeout Failures

Tests hanging or taking too long:

**Investigation steps:**
1. Run test in isolation to confirm it's slow
2. Add debug logging to identify bottleneck
3. Check for infinite loops or blocking I/O
4. Look for missing mocks of external services

**Setting test timeouts:**
```python
@pytest.mark.timeout(30)  # 30 second limit
def test_slow_operation():
    pass
```

### Flaky Tests

Non-deterministic failures:

**Common causes:**
- Race conditions in async code
- Time-dependent logic (dates, timestamps)
- Random data generation without seeding
- Shared state between tests
- External service dependencies

**Strategies:**
- Run test multiple times: `pytest --count=10 test_file.py::test_name`
- Seed random generators: `random.seed(42)`
- Mock time-dependent functions
- Use pytest-randomly to detect order dependencies

---

## Investigation Patterns

### Systematic Approach

1. **Reproduce reliably** - Run test in isolation first
2. **Minimize scope** - Find smallest failing input
3. **Form hypothesis** - What do you think is wrong?
4. **Test hypothesis** - Add logging/assertions to verify
5. **Fix and verify** - Make minimal change, run tests

### Using Debug Output

**Add verbose pytest output:**
```bash
pytest -vvs tests/test_file.py::test_name
```

**Add print statements (captured by pytest):**
```python
def test_something():
    result = function_under_test()
    print(f"DEBUG: result = {result}")
    assert result == expected
```

**Use pytest's built-in debugging:**
```bash
pytest --pdb tests/test_file.py::test_name  # Drop into debugger on failure
```

### Reading Stack Traces

**From bottom to top:**
1. Bottom: The actual error message
2. Above: The line where it occurred
3. Higher: The call chain that led there

**Key information to extract:**
- Which file and line number
- What function was being called
- What arguments were passed (if visible)

---

## Common Fixes

### Pattern: Missing Mock

**Symptom:** Test makes real network calls, causing timeouts or failures

**Fix:**
```python
from unittest.mock import patch

@patch('module.requests.get')
def test_api_call(mock_get):
    mock_get.return_value.json.return_value = {'key': 'value'}
    result = function_under_test()
    assert result == expected
```

### Pattern: Fixture Scope Mismatch

**Symptom:** Test state leaking between tests

**Fix:** Use appropriate fixture scope or add cleanup
```python
@pytest.fixture(scope='function')  # Fresh per test
def database():
    db = create_database()
    yield db
    db.cleanup()  # Always runs after test
```

### Pattern: Assertion Message Missing

**Symptom:** Assertion fails with no helpful context

**Fix:** Add message to assertion
```python
# Bad
assert result == expected

# Good
assert result == expected, f"Expected {expected}, got {result}"
```

### Pattern: Test Order Dependency

**Symptom:** Test passes alone, fails in suite (or vice versa)

**Fix:** Find and eliminate shared state
```bash
pytest --randomly-seed=12345  # Detect order dependencies
```

---

## Advanced Troubleshooting

### Test Collection Errors

If pytest can't even collect tests:

```bash
pytest --collect-only tests/  # See what pytest finds
```

**Common causes:**
- Syntax errors in test files
- Import errors in test files
- Invalid test naming (`test_` prefix required)

### Coverage Gaps

Find untested code:

```bash
pytest --cov=src --cov-report=term-missing
```

### Parallel Test Execution

Run tests in parallel for speed:

```bash
pytest -n auto  # Uses pytest-xdist
```

**Note:** Parallel execution requires tests to be independent. If tests fail only in parallel, they likely have shared state issues.

### Test Isolation Verification

Ensure tests don't affect each other:

```bash
pytest --forked  # Run each test in subprocess
```

---

## Success Criteria

A test session is successful when:
- All tests pass
- No regressions introduced
- Root cause understood
- Fix documented (if complex)

**Post-fix checklist:**
1. Run the specific failing test
2. Run related tests in the same module
3. Run full test suite
4. Verify no new failures introduced
