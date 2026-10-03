---
enabled: true
start-only-if: the diff adds or changes a trait, generic, function, parameter, field, option or setting
---

Code should solve today's problem. Generality nothing uses yet costs every reader now, and rarely fits the need when it
arrives.

Review code the diff adds or changes, and existing code the diff leaves with a single use.

A finding is complexity no current use needs:

- a trait, interface or generic with a single implementation or instantiation.
- a wrapper, helper or layer of indirection with a single caller, that inlining would make simpler.
- a parameter, option, field or setting that every caller sets to the same value.
- handling of inputs no caller passes or states no code produces. Input from users, files, the network or the
  environment can be anything, so handling it is needed.
- extension points for variants that do not exist yet: registries, hooks, builders, strategies, version fields.

A newtype is not a finding, even with a single use: it makes the compiler enforce a meaning.

Name the simpler version, and show that every current use works with it.
