---
enabled: true
start-only-if: the diff adds or changes CLI flags, output, config, environment variables, requests to servers or persisted files, or the code that produces them
---

Customers notice what changed, not what the PR meant to change, and most breakage is a change nobody intended.

Review every path from the code the diff changes to what users, scripts and servers can observe.

A finding is a difference between base and head in:

- CLI flags, defaults, exit codes or help text.
- output scripts or programs read, such as stdout and files.
- config and environment variables, and their precedence.
- requests sent to servers: endpoints, headers, auth, payloads.
- files written by one version and read by another.

A difference a commit message names is intended, and not a finding.

Give the input that shows the difference between the base and head binaries. The fix restores the old behaviour, or
names the change in the commit message.
