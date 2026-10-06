---
name: python-package-fix-conventions
description: Use when fixing a Python package so tests pass and review-bot conventions must also be satisfied.
---
1. Run the test suite from the workspace root with the package import path set correctly; if imports fail, fix the environment or package layout before editing logic.
2. Read docstrings and failing tests as the specification, then inspect public functions for missing type annotations and add annotations to every parameter and return value.
3. After each code change, rerun the relevant tests until the whole suite passes.
4. When you fix bugs, add a regression test file under the tests directory with one test function per bug fixed, and make sure it passes without modifying existing tests.
5. Record each fix in the changelog under `## Unreleased` as bullets in the form `- fix(<function name>): <short description>`.
6. Self-check:
   - all public functions have parameter and return annotations
   - regression tests exist for each fixed bug
   - changelog entries were added under `## Unreleased`
   - full test suite passes
