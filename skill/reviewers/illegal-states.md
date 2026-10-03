---
enabled: true
start-only-if: the diff adds or changes a type or function signature, or an assertion
---

An invariant the types enforce cannot be broken; one kept by comments, checks and assertions breaks on the next change.

Review code the diff adds or changes.

A finding is an invariant the code relies on that a type could enforce, such as:

- a collection that must not be empty: a non-empty type.
- a number that must not be zero, or must stay in a range: a newtype.
- fields only some combinations of which are valid: an enum.
- a value that must be validated before use: a type only validation can construct.
- values of different meaning sharing a type, such as char and byte offsets: newtypes.

The fix gives the type, either an existing one or its definition, the signatures that change, and the comments, checks
and assertions it makes deletable.
