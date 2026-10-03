---
enabled: true
start-only-if: the diff adds or changes code that runs, not only comments, docs or formatting
---

Code that looks right to its author fails on the inputs the author did not try.

Review code the diff adds or changes, and the code it calls or is called by.

A finding is an input the code accepts that gives a wrong result, such as:

- empty, zero, maximum or boundary values.
- missing, empty, corrupt or unreadable files.
- non-ASCII text, Windows paths, and arguments with spaces or quotes.
- a parameter that is parsed but never reaches the code that needs it.
- a branch or fallback that never runs, or runs when it should not.
