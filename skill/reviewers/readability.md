---
enabled: true
start-only-if: the diff adds or changes a ternary conditional operator
---

Code is read far more often than written, and these constructs make the reader stop and decode.

Review code the diff adds or changes.

A finding is:

- a ternary conditional operator (`a ? b : c`).
