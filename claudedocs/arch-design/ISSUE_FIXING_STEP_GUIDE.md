# ╔══════════════════════════════════════════════════════════════════╗
# ║                   ISSUE FIXING STEP GUIDE                        ║
# ║            Implementation, Testing & Fix Iteration               ║
# ╚══════════════════════════════════════════════════════════════════╝

---

## Purpose

Workflow for implementing fixes after troubleshooting, writing tests, and handling failures during the test phase. Defines boundaries for autonomous fixes vs. user approval.

---

## Development Principles

### Hard Rules

| Rule | Enforcement |
|------|-------------|
| No backwards compatibility | Unless user specifically requests it |
| No fallbacks | EVER |
| No defaults | EVER |

> These rules exist to surface problems early rather than hide them behind silent behaviors.

---

## Test Organization

### Directory Structure

```
project_root/
├── src/
│   ├── module_a/
│   │   └── handler.py
│   └── module_b/
│       └── processor.py
└── tests/                    ← One level down from root
    ├── module_a/             ← Mirrors src/ structure
    │   └── test_handler.py
    └── module_b/
        └── test_processor.py
```

### Rules

- Tests live in `/tests/` directory (one down from root)
- Subfolder structure mirrors main project
- Test files only - no production code in `/tests/`
- Naming: `test_<original_filename>.py`

---

## Test Framework Options

### Primary: pytest

**Use when**: Most cases. Modern, flexible, minimal boilerplate.

```python
# tests/module_a/test_handler.py
import pytest
from module_a.handler import process_data

def test_process_data_happy_path():
    result = process_data({"key": "value"})
    assert result["status"] == "ok"

def test_process_data_raises_on_empty():
    with pytest.raises(ValueError):
        process_data({})

@pytest.fixture
def sample_artifact():
    return load_artifact("fixtures/sample.json")

def test_with_fixture(sample_artifact):
    result = process_data(sample_artifact)
    assert result is not None
```

**Strengths**: Simple assertions, powerful fixtures, great plugin ecosystem
**Run**: `pytest tests/` or `pytest tests/module_a/test_handler.py::test_specific`

---

### Secondary: unittest

**Use when**: Need compatibility with existing unittest-based projects, or prefer class-based organization.

```python
# tests/module_a/test_handler.py
import unittest
from module_a.handler import process_data

class TestProcessData(unittest.TestCase):

    def setUp(self):
        self.sample = {"key": "value"}

    def test_happy_path(self):
        result = process_data(self.sample)
        self.assertEqual(result["status"], "ok")

    def test_raises_on_empty(self):
        with self.assertRaises(ValueError):
            process_data({})

if __name__ == "__main__":
    unittest.main()
```

**Strengths**: Standard library (no install), class-based setUp/tearDown
**Run**: `python -m unittest discover tests/` or `python -m unittest tests.module_a.test_handler`

---

### Lightweight Alternatives

| Framework | Size | Best For | Install |
|-----------|------|----------|---------|
| **doctest** | Built-in | Inline examples in docstrings | None |
| **nose2** | Small | unittest extension, autodiscovery | `pip install nose2` |
| **ward** | Small | Modern, descriptive test names | `pip install ward` |
| **hypothesis** | Medium | Property-based testing, edge cases | `pip install hypothesis` |

#### doctest Example

```python
def add(a, b):
    """Add two numbers.

    >>> add(2, 3)
    5
    >>> add(-1, 1)
    0
    """
    return a + b
```

Run: `python -m doctest module.py`

#### hypothesis Example (property-based)

```python
from hypothesis import given, strategies as st

@given(st.integers(), st.integers())
def test_add_commutative(a, b):
    assert add(a, b) == add(b, a)
```

---

### Framework Selection

| Situation | Choice |
|-----------|--------|
| New project | pytest |
| Existing unittest codebase | unittest (stay consistent) |
| Quick docstring examples | doctest |
| Finding edge cases | hypothesis + pytest |
| Minimal dependencies | unittest (stdlib) |

---

## Fix Attempt Limits

### Per-Category Limits

| Code Category | Max Fix Attempts | Notes |
|---------------|------------------|-------|
| Code changed/added/removed in current plan | 2 | Direct plan work |
| Test files | Unlimited | Fix liberally |
| Import path/name/call signature errors | 1 each | Basic wiring only |
| All other code | 0 | Do not touch |

### Global Maximum

> **After 7 total fix attempts (across all categories), STOP and escalate to user.**

This prevents infinite loops on stubborn issues.

### What Counts as "Other Code"

- Code not modified by current plan
- Pre-existing logic
- Unrelated modules
- Anything outside plan scope

> If "other code" needs changes, STOP and escalate to user.

---

## Fix Iteration Documentation

### Required: Fix Log Document

When entering fix iteration, create:

```
YYYYMMDD_<plan_name>_fixlog.md
```

**Location**: Same directory as plan file (cross-link them)

### Fix Log Format

```markdown
# Fix Log: [Plan Name]
**Parent Plan**: [link to plan doc]
**Created**: YYYY-MM-DD HH:MM
**Total Attempts**: 0/7

## Fix Attempts

### Attempt 1 - HH:MM
- **File**: path/to/file.py
- **Category**: [plan-code | test-file | import-fix | other]
- **Issue**: [brief description]
- **Change**: [what was changed]
- **Result**: PASS | FAIL

### Attempt 2 - HH:MM
...
```

**Keep entries brief** - just enough to track what was tried.

### Cross-Linking

In plan doc, add:
```markdown
**Fix Log**: [link to fixlog.md]
```

In fix log, header contains:
```markdown
**Parent Plan**: [link to plan doc]
```

---

## Approval Gates

### Always Requires User Approval

| Change Type | Action |
|-------------|--------|
| Logic changes | STOP → Ask user |
| Architecture changes | STOP → Ask user |
| Changes outside plan scope | STOP → Ask user |
| New dependencies | STOP → Ask user |
| 7+ total fix attempts | STOP → Ask user |

### Autonomous (Within Limits)

| Change Type | Action |
|-------------|--------|
| Fixing code from current plan | Up to 2 attempts |
| Fixing test files | Fix liberally |
| Import/signature corrections | 1 attempt each |

---

## Test Writing Guidelines

### Keep Objectives in Mind

Before writing tests, answer:
1. **What is this code producing?** - Expected outputs/behaviors
2. **What makes a GOOD test?** - Tests the right thing, not just any thing
3. **What are failure modes?** - Edge cases, error conditions

### Test Quality Checklist

- [ ] Tests actual behavior, not implementation details
- [ ] Covers happy path AND error cases
- [ ] Self-contained (no external dependencies)
- [ ] Deterministic (same input = same output)
- [ ] Fast (no unnecessary waits/sleeps)

---

## Mock Data & Fixtures

### Mocking is Acceptable and Encouraged

**Goal**: Self-contained tests that don't require external systems or live data.

### Sources for Mock Data

| Source | When to Use |
|--------|-------------|
| Fixture files in `/tests/fixtures/` | Reusable across tests |
| Inline test data | Simple, single-use cases |
| Factory functions | Complex object construction |
| External test data files | Large datasets, JSON/YAML/CSV |

### Fixture Organization

```
tests/
├── fixtures/
│   ├── sample_input.json
│   ├── expected_output.json
│   └── edge_cases/
│       └── malformed_input.json
├── conftest.py          ← Shared pytest fixtures
└── module_a/
    └── test_handler.py
```

### Fixture Pattern

```python
import pytest
import json
from pathlib import Path

FIXTURES_DIR = Path(__file__).parent / "fixtures"

@pytest.fixture
def sample_input():
    with open(FIXTURES_DIR / "sample_input.json") as f:
        return json.load(f)

@pytest.fixture
def expected_output():
    with open(FIXTURES_DIR / "expected_output.json") as f:
        return json.load(f)

def test_processing(sample_input, expected_output):
    result = process(sample_input)
    assert result == expected_output
```

---

## Workflow Summary

```
IMPLEMENT:  Execute plan changes
                ↓
TEST:       Write/run unit tests
                ↓
        ┌── PASS ──→ Done, update plan STATUS: RESOLVED
        │
    FAIL?
        │
        ↓
    Check total attempts (max 7)
        │
        ├── >= 7 attempts?
        │       → STOP, escalate to user
        │
        └── < 7 attempts, check category:
                │
                ├── Plan code (attempts < 2)?
                │       → Create/update fixlog
                │       → Attempt fix
                │       → Re-test
                │
                ├── Test file issue?
                │       → Fix liberally
                │       → Update fixlog
                │       → Re-test
                │
                ├── Import/signature (attempts < 1)?
                │       → Log attempt
                │       → Fix wiring
                │       → Re-test
                │
                └── Other code OR category limits exceeded?
                        → STOP
                        → Update fixlog
                        → Escalate to user
```

---

## Quick Reference

```
PRINCIPLES: No backwards compat | No fallbacks | No defaults

FRAMEWORKS: pytest (primary) | unittest (secondary) | doctest/hypothesis (special cases)

TESTS:      /tests/ mirrors project structure
            test_<filename>.py naming

FIX LIMITS: Plan code = 2 | Tests = unlimited | Imports = 1 | Other = 0
            GLOBAL MAX = 7 total attempts

DOCS:       Create fixlog.md next to plan
            Cross-link both ways
            Track total attempts
            Keep entries brief

APPROVAL:   Logic/arch changes = ALWAYS ask user
            Out-of-scope = ALWAYS ask user
            7+ attempts = ALWAYS ask user

FIXTURES:   /tests/fixtures/ for reusable test data
            conftest.py for shared pytest fixtures
```

---

*See also: `TROUBLESHOOTING_STEP_GUIDE.md`, `BaseResearchInstructions.md`*
