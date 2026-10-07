---
enabled: true
scope: commits
---

Readers of `git log`, `git blame` and `git bisect` rely on commit messages to know what changed and why without reading
the diff, and every extra sentence costs every reader. The PR message disappears at merge, and has no rules.

Review each commit message against its own commit.

The expected shape is a title, a bullet list with one line per change, and, only when the diff leaves it open, a
short why. A commit whose title covers its only change needs no list.

A title starts with the area it changes: a crate, directory, module or component, such as `pipeline: add release
job` or `parser: skip empty lines`. When a change spans areas, name the main one, or use a Conventional Commits
type such as `feat:`. The area is preferred. Revert titles are exempt.

A finding is:

- a title with neither an area nor a Conventional Commits prefix.
- an area the commit does not change.
- a statement the changes do not make true.
- a significant change without its own bullet:
  - an added or removed feature.
  - a bug fix.
  - a change in how the code runs: retries, caching, concurrency, ordering or error handling.
  - a changed interface between components.
  - a changed data format.
  - a changed build or CI.
- a bullet longer than one line, or more than one per change.
- a bullet for a change no reader needs to know: renames, formatting, moved code.
- a sentence that says how rather than what: the files, functions or steps the diff changes.
- a statement of what was tested or how. CI is the record of that.
- prose that does not answer a why the diff leaves open.
