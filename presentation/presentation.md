---
title: Crucible
sub_title: Code review where every finding has to survive a test
---

One area per reviewer
===

## Research

* Models follow a smaller share of instructions as the number of instructions grows
  * Consistent across 10 models, for text and code. ManyIFEval, EMNLP Findings 2025 <https://arxiv.org/abs/2509.21051>
* This holds for current coding agents
  * Claude Opus 4.6 in Claude Code, as constraints accumulated: **82%** to **76%** of constraints followed, **19%** to
    **2%** of turns with every constraint followed. MTAC-IFBench, 2026 <https://arxiv.org/abs/2609.14992>

## Applying the research

* Each reviewer agent gets only the instructions for its own area

<!-- end_slide -->

Demand a checkable claim
===

## Research

* Even a recent model calls correct code wrong often
  * Claude Sonnet 4.5 rejected **26%** to **62%** of correct implementations, depending on benchmark and prompt.
    Jin & Chen, ASE 2026 <https://arxiv.org/abs/2603.00539>
* Most of those wrong calls never showed where the code fails
  * The top category for all five models, Claude included: a broad claim that the logic is wrong. 48% overall.
    Jin & Chen, ASE 2026 <https://arxiv.org/abs/2603.00539>

## Applying the research

* Reviewer agents state each finding as a concrete case that can be checked
  * Vague: "the proxy precedence logic is wrong"
  * Checkable: "with `HTTPS_PROXY` set and `--proxy` unset, requests bypass the proxy"

<!-- end_slide -->

Verify independently
===

## Research

* Running the proposed fix against tests removes most false alarms
  * Claude Sonnet 4.5: **46%** to **9%** of correct code rejected. Jin & Chen, ASE 2026
    <https://arxiv.org/abs/2603.00539>
* Agreement between agents is not evidence
  * Ten agents confirmed a vulnerability that did not exist; a test exposed it. Agarwal, Refute-or-Promote, 2026
    <https://arxiv.org/abs/2604.19049>
* What the model is told about the code changes what it finds; removing it helps
  * Told the code was safe, GPT-4o-mini found **4%** of vulnerabilities instead of **97%**; Claude Opus 4.5 lost **7**
    points. Removing the PR description and commit messages restored detection. Alexopoulos et al., 2026
    <https://arxiv.org/abs/2603.18740>

## Applying the research

* A separate verifier designs its own check and tries to disprove each finding by running it
* It never sees the reviewer's idea of how to test
* Only findings that survive get a severity, and only those can block
