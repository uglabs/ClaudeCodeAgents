# Plan: Testing Smith Agent Prompts

## Problem

Agent prompts (smith.md, karen.md, etc.) are instructions for Claude, not executable code. Traditional unit testing doesn't apply. LLM outputs are non-deterministic.

## Testing Layers

### Layer 1: Static Validation (Easy)

**What:** Verify markdown structure and required elements exist.

**Tests:**
- [ ] Required sections present (Overview, Workflow, etc.)
- [ ] Agent references (@karen, @python-pro) are valid
- [ ] Version history format correct
- [ ] No broken internal links

**Implementation:**
```python
# tests/test_agent_structure.py
import re
from pathlib import Path

def test_smith_has_required_sections():
    content = Path("smith.md").read_text()
    required = ["## Overview", "## Quick Commands", "## Phase 1:", "## Phase 2:"]
    for section in required:
        assert section in content, f"Missing: {section}"

def test_agent_references_valid():
    content = Path("smith.md").read_text()
    refs = re.findall(r"@(\w+[-\w]*)", content)
    valid_agents = {"Jenny", "karen", "python-pro", "code-quality-pragmatist", ...}
    for ref in refs:
        assert ref in valid_agents, f"Unknown agent: @{ref}"
```

**Effort:** 1-2 hours

---

### Layer 2: Trigger Detection (Medium)

**What:** Verify natural language triggers map to correct workflows.

**Tests:**
- [ ] "junior worked on X" → review workflow
- [ ] "@smith ?" → status check
- [ ] "@smith commit" → commit workflow
- [ ] Ambiguous inputs → ask for clarification

**Implementation:**
```python
# tests/test_triggers.py
import pytest

REVIEW_TRIGGERS = [
    "junior worked on src/auth/",
    "review the API code",
    "check what alex did on payments",
    "a new dev touched the config",
]

STATUS_TRIGGERS = [
    "@smith ?",
    "@smith status?",
    "what's the current state?",
]

@pytest.mark.parametrize("input", REVIEW_TRIGGERS)
def test_review_trigger(input, smith_classifier):
    assert smith_classifier.classify(input) == "review_workflow"

@pytest.mark.parametrize("input", STATUS_TRIGGERS)
def test_status_trigger(input, smith_classifier):
    assert smith_classifier.classify(input) == "status_check"
```

**Challenge:** Need a classifier. Options:
1. Rule-based regex (brittle but fast)
2. Small embedding model (better but more infra)
3. Claude itself with constrained output (accurate but slow/costly)

**Effort:** 1-2 days

---

### Layer 3: Agent Call Verification (Medium-Hard)

**What:** Verify correct agents are called in correct order.

**Tests:**
- [ ] Review workflow calls @karen then @code-quality-pragmatist
- [ ] Big changes call @architect-reviewer
- [ ] UI changes call @ui-comprehensive-tester

**Implementation:**
```python
# tests/test_agent_calls.py
from unittest.mock import MagicMock, call

def test_review_calls_agents_in_order(mock_task_tool):
    smith.handle("review src/auth/")

    calls = mock_task_tool.call_args_list
    agent_order = [c.kwargs["subagent_type"] for c in calls]

    assert "karen" in agent_order
    assert "code-quality-pragmatist" in agent_order
    assert agent_order.index("karen") < agent_order.index("code-quality-pragmatist")
```

**Challenge:** Need to intercept Task tool calls. Options:
1. Mock the Claude API, inspect tool calls in response
2. Use Claude Code hooks to log agent invocations
3. Run in test mode that logs but doesn't execute

**Effort:** 2-3 days

---

### Layer 4: Output Shape Validation (Medium)

**What:** Verify outputs match expected structure.

**Tests:**
- [ ] Review output contains 🔴/🟡/⚪ categories
- [ ] Status output contains branch info
- [ ] Commit messages follow conventional format

**Implementation:**
```python
# tests/test_output_shape.py
import re

def test_review_output_has_categories(smith_output):
    output = smith.review("src/auth/")

    assert "🔴" in output or "Broken" in output
    assert "🟡" in output or "Overbuilt" in output
    assert "⚪" in output or "Nitpick" in output

def test_review_asks_permission():
    output = smith.review("src/auth/")

    # Should ask before fixing
    assert re.search(r"(Fix|Proceed|Continue)\?", output)
```

**Challenge:** LLM outputs vary. Need fuzzy matching or semantic similarity.

**Effort:** 1-2 days

---

### Layer 5: Golden Tests / Snapshots (Easy-Medium)

**What:** Capture known-good outputs, diff against them.

**Tests:**
- [ ] Review of sample repo matches baseline
- [ ] Status check output matches baseline
- [ ] Error handling matches baseline

**Implementation:**
```python
# tests/test_golden.py
import difflib

def test_review_golden(snapshot):
    output = smith.review("fixtures/junior-auth-code/")

    # First run: creates snapshot
    # Later runs: compares to snapshot
    snapshot.assert_match(output, "review_auth_golden.txt")
```

**Challenge:**
- LLM non-determinism causes flaky tests
- Need semantic diff, not literal diff
- Snapshots go stale

**Mitigation:**
- Use embedding similarity instead of string diff
- Set similarity threshold (e.g., 0.85)
- Review/update snapshots regularly

**Effort:** 1 day setup, ongoing maintenance

---

### Layer 6: End-to-End Eval Suite (Hard)

**What:** Run full workflows, score outputs with rubric.

**Tests:**
- [ ] Given junior code with known issues, does Smith find them?
- [ ] Given working code, does Smith avoid false positives?
- [ ] Does simplification actually simplify?

**Implementation:**
```python
# evals/test_review_e2e.py
EVAL_CASES = [
    {
        "name": "finds_hardcoded_secret",
        "input_dir": "fixtures/hardcoded-jwt/",
        "expected_findings": ["hardcoded", "secret", "security"],
        "severity": "red",
    },
    {
        "name": "catches_factory_overengineering",
        "input_dir": "fixtures/premature-factory/",
        "expected_findings": ["factory", "one", "provider", "abstraction"],
        "severity": "yellow",
    },
]

@pytest.mark.parametrize("case", EVAL_CASES)
def test_eval_case(case, smith):
    output = smith.review(case["input_dir"])

    # Check severity
    if case["severity"] == "red":
        assert "🔴" in output

    # Check findings mentioned (fuzzy)
    output_lower = output.lower()
    matches = sum(1 for kw in case["expected_findings"] if kw in output_lower)
    assert matches >= len(case["expected_findings"]) * 0.7  # 70% threshold
```

**Effort:** 3-5 days + fixtures

---

## Recommended Approach

### Phase 1: Quick Wins (1-2 days)
1. Layer 1: Static validation
2. Layer 5: Golden tests with semantic similarity

### Phase 2: Core Testing (1 week)
3. Layer 2: Trigger detection with rule-based classifier
4. Layer 4: Output shape validation

### Phase 3: Full Coverage (2+ weeks)
5. Layer 3: Agent call verification
6. Layer 6: E2E eval suite

---

## Test Infrastructure Needed

```
tests/
├── conftest.py           # Fixtures, mocks
├── test_agent_structure.py
├── test_triggers.py
├── test_output_shape.py
├── test_golden.py
├── fixtures/
│   ├── junior-auth-code/  # Sample code with issues
│   ├── premature-factory/ # Over-engineered code
│   └── clean-code/        # Code without issues
└── snapshots/
    └── review_auth_golden.txt

evals/
├── eval_runner.py        # Run evals, generate report
├── cases/
│   └── review_cases.yaml # Eval case definitions
└── results/
    └── 2025-01-15.json   # Eval results
```

---

## Tools to Consider

| Tool | Use Case | Pros | Cons |
|------|----------|------|------|
| **pytest** | Test runner | Standard, familiar | - |
| **promptfoo** | Prompt testing | Built for this, good eval framework | JS-based |
| **deepeval** | LLM evals | Python, semantic scoring | Learning curve |
| **instructor** | Structured outputs | Type-safe LLM responses | Adds dependency |
| **sentence-transformers** | Semantic similarity | Lightweight embeddings | Needs model download |

---

## Open Questions

1. **Where to run?** CI/CD (costly) vs local (manual)?
2. **Flaky test tolerance?** Allow X% failure rate?
3. **Eval scoring?** Human review vs automated?
4. **Fixtures maintenance?** Who updates sample code?

---

## Next Steps

1. [ ] Decide on Phase 1 scope
2. [ ] Set up test directory structure
3. [ ] Implement static validation tests
4. [ ] Create 2-3 fixture repos with known issues
5. [ ] Run first eval, establish baseline
