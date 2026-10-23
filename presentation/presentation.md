---
title: Crucible
sub_title: Code review where every finding has to survive a check
---

Why
===

* AI code review is noisy: many comments, few worth acting on
* 12 of 13 code review agents had an average signal ratio below 60%
  * Chowdhury et al., MSR 2026 <https://arxiv.org/abs/2604.03196>
  * Observational, on GitHub PRs: it shows the noise, not its cause
* Crucible's bet: post less, and only what survived a check

<!-- end_slide -->

How it works
===

```
AGENTS.md
    │             ┌─ reviewer ── verifier ─┐
PR ── build ──────┼─ reviewer ── verifier ─┼── merge ── merge risk ── post
  head and base   └─ reviewer ── verifier ─┘
```

* Agents pass work through files, not conversations
* The orchestrator only starts agents; it never reviews

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

## Research

* Repository access and execution: "catching more critical issues and raising fewer false alarms"
  * OpenAI, 2025 <https://alignment.openai.com/scaling-code-verification/>
  * A blog post about their own trained reviewer, with no numbers in the text
* All 8 models got worse as more context was pasted into the prompt
  * SWE-PRBench, 2026 <https://arxiv.org/abs/2603.26130>
  * Single-author preprint; why crucible gives agents clones to browse, not pasted context

<!-- end_slide -->

Many small, independent reviewers
===

* 16 reviewers, each with a few rules: correctness, tests, docs, yagni, …
* A reviewer starts only if the diff touches its area
* Reviewers never see each other's work; findings are merged by root cause afterwards

## Research

* Instruction following degrades as the number of instructions grows, across 10 models
  * ManyIFEval, EMNLP 2025 <https://arxiv.org/abs/2509.21051>
  * General instruction following, not code review
* Merging independent review runs: F1 15% → 22%, recall more than doubled
  * SWR-Bench, FSE 2026 <https://arxiv.org/abs/2509.01494>
  * Repeated runs of one model and prompt, not specialists
  * More recall means more noise, so the next slide matters

<!-- end_slide -->

A finding has to survive a check
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

Earlier threads
===

* Every run is a full review, even of an unchanged PR
* A reviewer sees crucible's earlier threads only after its own search
* It then settles each one: fixed, not fixed, accepted, disputed, open
* A verifier checks each verdict before anything is replied or resolved

## Research

* None: a design choice to keep earlier runs from anchoring the search

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
* Crucible itself has no evaluation yet
