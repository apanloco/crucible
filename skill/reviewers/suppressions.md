---
enabled: true
start-only-if: the diff adds a suppression, changes lint or CI config, or removes, ignores or weakens a test
---

A check that is switched off stops protecting every later change, not just this one.

Review suppressions, lint config, tests and CI the diff adds, changes or removes.

A finding is a check the diff disables, loosens or bypasses:

- an inline suppression: `#[allow(...)]`, `#[expect(...)]`, `// eslint-disable`, `# noqa`, `@SuppressWarnings`, or
  code hidden from tests with `#[cfg(not(test))]`.
- a lint lowered or removed, or an advisory ignored, in project config such as `Cargo.toml`, `clippy.toml` or
  `deny.toml`.
- a test ignored or deleted while the behaviour it covered remains, or an assertion loosened until it passes.
- a CI step removed, made optional or allowed to fail, or a flag such as `-D warnings` dropped.

A suppression is not a finding when the check is wrong for this code and a comment next to it says why.

The fix makes the code pass the check.
