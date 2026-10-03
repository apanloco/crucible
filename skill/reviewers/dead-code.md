---
enabled: true
start-only-if: the diff adds or changes an item, parameter, field, option or branch, or removes a use of one
---

Every line a PR adds costs every future reader, so it must be needed.

Review code the diff adds, and code the diff leaves unused.

A finding is code that can be deleted with no change in behaviour:

- commented-out code.
- code nothing reaches: unused items, parameters, fields, options or branches.
