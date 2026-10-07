---
enabled: true
scope: commits
start-only-if: the PR has more than one commit
---

PRs are merged by rebase, so every commit lands in history as written, and every extra one costs each later reader
of `git log` and `git bisect`.

A PR is one commit, or several self-contained ones: each builds on its own, and no two change the same code.

Review each commit from merge base to head on its own.

A finding is:

- a commit that changes code an earlier commit in the same PR added or changed.
- a commit that does not build.
- a merge commit.
