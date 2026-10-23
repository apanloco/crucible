---
enabled: true
start-only-if: context.md lists a cloned consumer
---

A repo that uses this one breaks on what changed, whether or not the change was meant, and its owners find out last.

Review every use, in the consumers listed in `context.md`, of what the diff changes.

A finding is a use that head breaks: a flag it passes, output it parses, or a function, type or field it calls.

A break a commit message names, together with the consumer it breaks, is intended, and not a finding.

Give the consumer's file:line and the head code it no longer matches. The fix keeps the old contract working in this
repo, such as an alias for a renamed flag, or names the break and the consumer in the commit message.
