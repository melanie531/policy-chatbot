---
description: TDD enforcement rule
globs: ["src/**/*.py", "tests/**/*.py"]
---

# TDD Rule — Policy Chatbot

## Rule
Every implementation function MUST have a corresponding test in `tests/test_<module>.py`.

## Process
1. Before writing a new function, check if a test case exists in `test-cases.md`
2. Write the test first (or simultaneously)
3. Run `pytest` to confirm the test fails (red)
4. Implement the function to make the test pass (green)
5. Refactor if needed

## Coverage
- Target: ≥80% line coverage
- Run: `pytest --cov=policy_chatbot --cov-report=term-missing`
- All Bedrock API calls must be mocked
- All FAISS operations in tests must use in-memory fixtures with `tmp_path`
