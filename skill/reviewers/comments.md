---
enabled: true
start-only-if: the diff adds or changes comments other than doc comments
---

Comments exist to tell a reader of the code what the code itself cannot.

Review comments, not doc comments, commented-out code or other files.

Go through every comment the diff adds or changes, one sentence at a time. A sentence is a finding if it:
- is false.
- restates what a reader can see in the code: names, types, attributes or logic.
- says what clearer code could say: a name, a constant or a type.
- repeats another sentence in the same comment, or a doc the comment points to.
- describes how the code got there instead of what it is.
- points to a file. Nothing checks such links, so they break silently; references belong in doc comments, as links to code items.
