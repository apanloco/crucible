---
enabled: true
start-only-if: the diff adds or renames a type, function, field, parameter, variable or module, or changes what one holds or does
---

A reader trusts a name before reading what it names, so a name that says something false misleads every reader.

Review names the diff adds, and existing names whose type, contents or behaviour the diff changes.

A finding is a name that claims something false about what it names, such as:

- a data structure word that does not match the structure: a `Map` that maps nothing, a `List` that is a set, a
  `Queue` that is not first in, first out.
- a name taken over from a sibling or a previous version, whose structure or behaviour the diff changed.
- a verb or prefix that promises what the code does not do: a `get_` that computes or mutates, an `is_` or `has_` that
  does not return a boolean, a `try_` that cannot fail.
- a unit, plurality or property the value does not have: `_ms` holding seconds, a plural holding one item, `sorted`
  that is not sorted.

A name that is true but could be shorter, more idiomatic or more to taste is not a finding.

Show the false claim from the code, give a name that states what it is, and list every use the rename changes.
