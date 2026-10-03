---
enabled: true
start-only-if: the diff adds or changes code that runs, or tests
---

Tests exist to fail when the project's code is wrong.

Review tests the diff adds or changes, and the code the diff changes.

A finding is:

- changed behaviour no test would catch if the change were reverted. This means a test per behaviour, not per line:
  trivial wiring needs none.
- a test that does not test what it says it tests: it still passes with the project code it claims to cover reverted.
  A test that is explicitly about a dependency, by its name or module, is fine.
- an assertion a plausible regression passes.
- a test that catches no regression another test does not also catch. Name the other test.
