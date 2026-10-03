---
enabled: true
start-only-if: the diff adds or changes branches, loops, conversions or modules
---

The simplest code that does the job has the fewest places to be wrong.

Review code the diff adds or changes.

A finding is code a different formulation does with the same behaviour, or the same minus a defect, while removing at
least one of: a branch, a guard, a conversion, a special case, a constant, a loop, a module or a layer.
Shorter syntax alone is not a finding.
