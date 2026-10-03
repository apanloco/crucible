---
enabled: true
start-only-if: the diff adds or changes code that reads or writes files, or talks to the network
---

Review every file and network operation the diff adds or changes.

A finding is:

- a write that replaces a file's content in place, instead of writing a temporary file in the same directory and
  renaming it over the original.
- a write whose error is lost, such as a buffered writer dropped without a checked flush.
- a network call without a timeout.
