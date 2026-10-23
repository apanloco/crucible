---
title: Crucible
sub_title: Finds more. Proves all of it.
---

One pass sees a slice
===

* AI review finds a fraction of the issues, and a different fraction every run

## Research

* 8 frontier models found 15–31% of the issues human reviewers flagged
  * SWE-PRBench, 2026 <https://arxiv.org/abs/2603.26130>
  * Single-author preprint
* Five runs of the same model overlapped on only 27 identified issues
  * SWR-Bench, FSE 2026 <https://arxiv.org/abs/2509.01494>

## Experience

* Fix what a reviewer found, rerun it, and new findings appear

## Crucible

* Search wide, then verify everything it found

<!-- end_slide -->

How it works
===

```
AGENTS.md
    │             ┌─ reviewer ── verifier ─┐
PR ── build ──────┼─ reviewer ── verifier ─┼── merge ── merge risk ── post
  head and base   └─ reviewer ── verifier ─┘
```

* The orchestrator stays neutral: it only starts agents, never reviews, never relays a finding
  * So when it triages, it has formed no opinion of its own
  * And a verifier reads the reviewer's own words from a file, not a retelling

<!-- end_slide -->

The repo tells crucible what matters
===

* `AGENTS.md` on the base branch says:
  * how to build and test: without it, crucible stops
  * the rules: a reviewer checks them, and every fix must keep them
  * the consumers: other repos that use this one, and what their contract is
* For a CLI tool, for example:
  * a web app runs the binary: CLI flags and output formats are a contract
  * an editor extension starts its MCP server: MCP tools are a contract
* When the diff touches a contract, crucible clones that consumer, so findings and the merge risk are checked against
  real use

<!-- end_slide -->

Specialized reviewers
===

* 16 reviewers, each with a few rules: correctness, tests, docs, yagni, …
* The orchestrator starts a reviewer only if the diff clearly meets its condition, and shows why for each in a table
* Reviewers never see each other's work; findings are merged by root cause afterwards
* Every run is a full review: earlier threads are shown only after a reviewer's own search, then settled

## Research

* Instruction following degrades as the number of instructions grows, across 10 models
  * ManyIFEval, EMNLP 2025 <https://arxiv.org/abs/2509.21051>
  * General instruction following, not code review

<!-- end_slide -->

Then prove it
===

* A reviewer must give a concrete scenario and a fix, not "the logic is wrong"
* A separate verifier tries to disprove it, by running it when reading does not settle it

## Research

* 48% of wrong rejections of correct code were a broad "logic error" claim
* Running the proposed fix against tests: Claude Sonnet 4.5 rejected 46% → 9% of correct HumanEval code
  * Jin & Chen, 2026 <https://arxiv.org/abs/2603.00539>
  * Function-level benchmarks, not real PRs; weaker where tests are shallow
* Ten reviewer agents unanimously endorsed an OpenSSL vulnerability that did not exist; one test rejected it
  * Agarwal, Refute-or-Promote, 2026 <https://arxiv.org/abs/2604.19049>
  * Single-author preprint, and one example, not a rate

<!-- end_slide -->

One merge risk, and you decide
===

* 🔴 high can hurt users · 🟡 low costs later · ⚪ zero is polish
* Only high blocks the merge; crucible never approves for you
* It posts without asking: a post is easy to undo

## Research

* "presenting functional defects as mandatory requirements and evolutionary suggestions as optional recommendations"
  * SWR-Bench, FSE 2026 <https://arxiv.org/abs/2509.01494>
  * The authors' recommendation, not something they measured

<!-- end_slide -->

What the research does not back (yet)
===

* Reviewers read the PR description, which can bias them
  * Framed as bug-free, GPT-4o-mini found 4% of vulnerabilities instead of 97%; Claude Opus 4.5 lost 7 points
  * Redacting the description and commit metadata restored detection
  * Alexopoulos et al., 2026 <https://arxiv.org/abs/2603.18740>
* A verifier on the same model may share the reviewer's blind spots
  * Model mistakes grow more similar as models grow more capable
  * Goel et al., 2025 <https://arxiv.org/abs/2502.04313>
  * Running checks instead of agreeing is crucible's answer, but untested
* No head-to-head comparison of crucible with other AI reviewers yet
