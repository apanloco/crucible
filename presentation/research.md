# Code review research, mapped to panel-review

Verification marks: **[V]** checked against the abstract or full text, **[S]** only from search snippets or secondary
pages, **[M]** from memory, not re-checked. Entries marked *brief* come from `code_review_research_2026.pdf`, with its
citation errors corrected (see the end).

## 1. Running parallel agents

**SWR-Bench.** Zeng et al., FSE 2026 (PACMSE 3(FSE), FSE137). *brief* [V] <https://arxiv.org/abs/2509.01494>
- 1,000 manually verified PRs from 12 projects: 500 with real issues, 500 clean. Any comment on a clean PR is a
  false positive.
- Best tool (a single careful prompt) reaches only 18.73% F1. The multi-agent CR-Agent scores 9.22%, blamed on
  "interaction overhead and error propagation". Feeding in raw static-analysis output scores 4.87%.
- Runs are unstable: across five runs of the same model, only 27 detected issues overlapped.
- Multi-Review: n independent runs, concatenated, then one LLM call synthesizes them. F1 15.25% → 21.91% (+43.67%
  relative), recall +118.83%. Gains level off after n=5. Precision still needs work.
- Recommends presenting functional defects as mandatory and evolutionary suggestions as optional.
- **panel-review:** independent parallel lenses, not chained conversations. Aggregation buys recall, so a precision
  stage (the verifier) has to follow it.

**Debate or Vote.** Choi, Zhu, Li, NeurIPS 2025 (Spotlight). [V] <https://arxiv.org/abs/2508.17536>
- Majority voting accounts for most of the gain usually credited to multi-agent debate. In their theory, debate alone
  does not improve expected correctness.
- **panel-review:** independent lenses with no debate stage, merged by root cause.

**Towards a Science of Scaling Agent Systems.** Kim et al. (Google Research et al.), 2025. [V]
<https://arxiv.org/abs/2512.08296>
- 260 configurations: multi-agent ranged from +80.8% to −70% against a single agent, depending on how well the
  architecture fits the task.
- Coordination stops paying off once a single agent scores above about 45%.
- [S] Independent agents amplified errors 17.2x, versus 4.4x under centralized coordination with validation.
- **panel-review:** parallel lenses work on a task that splits cleanly, *because* everything funnels through verifiers
  and triage. Unverified independent lenses would amplify errors.

**Why Do Multi-Agent LLM Systems Fail?** Cemri, Pan, Yang et al., NeurIPS 2025 D&B. [V]
<https://arxiv.org/abs/2503.13657>
- 1,600+ traces across 7 frameworks, 14 failure modes (κ=0.88).
- [S] Failures split into specification/design 41.8%, inter-agent misalignment 36.9%, task verification 21.3%.
- **panel-review:** fixed output files instead of conversations, an orchestrator that does not review, and a dedicated
  verifier.

**Agentic code review at Ericsson.** Laiq, Britto, Usman, Saini, Badampudi, PROFES 2026. *brief* [V]
<https://arxiv.org/abs/2609.15877>
- Four specialist agents in parallel (readability, maintainability, reliability, performance), each with its own
  context from a code knowledge graph. An LLM orchestrator combines them. **No verification stage.**
- 206 issues on 7 commits from 4 Python projects: 197 correct (96%). Of those, 33% severe, 36% important, 31% minor.
- **panel-review:** supports specialized lenses with lens-specific context. Small sample; panel-review adds the
  verifier this design lacks.

## 2. Verifying findings

**Are LLMs reliable code reviewers?** Jin, Chen, Automated Software Engineering 33:90 (2026). *brief* [V]
<https://arxiv.org/abs/2603.00539>
- 1,400+ correct and buggy versions of HumanEval, MBPP and QuixBugs tasks; five models.
- The more the prompt asks for, the more correct code is rejected. GPT-4o on HumanEval: verdict only 26.2%, with
  explanation 58.5%, with explanation and fix 73.2%.
- 87.2% of wrong rejections: vague "logic error" 48.2%, added requirement 14.1%, boundary 13.2%, misread spec 11.7%.
- Fix-guided verification: run the original and the proposed fix on tests. Both pass → the fix was unnecessary, flip
  to accept. Only the original passes → the fix breaks working code, accept. Only the fix passes → keep the
  rejection. Wrong rejections fall 54.8% → 16.3% (HumanEval), 69.0% → 28.9% (MBPP), 51.0% → 24.0% (QuixBugs).
- Weaker where tests are shallow. Function-level benchmarks, not real PRs.
- "misjudgments can be mitigated by grounding decisions in executable evidence rather than relying solely on
  increasingly elaborate prompts."
- **panel-review:** the strongest evidence for the verifier. Lenses that must give a failure scenario and a check are
  the "full" prompt style that over-rejects; that is acceptable only because a verifier runs the check afterwards.

**Refute-or-Promote.** Agarwal, 2026. [V] <https://arxiv.org/abs/2604.19049>
- Adversarial refutation gates killed about 79% of 171 candidate defects (83% prospectively); 4 CVEs survived.
- Ten reviewer agents unanimously endorsed a vulnerability that did not exist. Only an empirical test caught it.
- **panel-review:** a verifier that tries to disprove, and execution over agreement between agents.

**Great Models Think Alike and this Undermines AI Oversight.** Goel et al., ICML 2025. [S]
<https://arxiv.org/abs/2502.04313>
- LLM judges favor models similar to themselves, and mistakes become more correlated as models get more capable.
- **panel-review:** a verifier on the same model shares the lens's blind spots. Ground verification in execution, or
  use a different model family.

**SWT-Bench.** Mündler, Müller, He, Vechev, NeurIPS 2024. [S] <https://arxiv.org/abs/2406.12952>
- Agents can write reproducing tests that fail before a fix and pass after it. Using them as a filter doubled
  SWE-Agent's precision.
- **panel-review:** verifiers running scratch tests on base and head.

**BitsAI-CR.** Sun et al. (ByteDance), FSE Companion 2025, DOI 10.1145/3696630.3728552. *brief* [V]
<https://arxiv.org/abs/2501.15134>
- RuleChecker (fine-tuned LLM, 219 rules) generates; ReviewFilter (a second LLM) verifies. Precision 57.03% → peak
  75.0%.
- The filter works better verdict-first: 77.09% precision in 1.7 s, against 65.80% in 31 s when it reasons first.
- Context: diff hunks expanded to whole functions with Tree-sitter. Duplicates removed by embedding similarity.
- **panel-review:** generate, then verify separately; merge duplicates.

**RovoDev Code Reviewer.** Tantithamthavorn et al. (Atlassian), ICSE-SEIP 2026. *brief* [V]
<https://arxiv.org/abs/2601.01129>
- Three stages: Claude 3.5 Sonnet generates, gpt-4o-mini judges factual correctness, a ModernBERT classifier
  predicts whether a comment will lead to a code change.
- The LLM judge "has a minimal impact"; the actionability classifier adds about 15 points.
- **panel-review:** an LLM-only judge adds little; panel-review's verifier runs builds and tests instead.

**Sifting the Noise.** Xiong, Zhang, ISSTA 2026. [V] <https://arxiv.org/abs/2601.22952>
- Agents cut false-positive rates on OWASP from over 92% to 6.3%, but aggressive filtering also removed real
  vulnerabilities. Results depend heavily on the model.
- **panel-review:** keep "what nobody verified" in the merge brief; count dropped findings instead of hiding them.

## 3. Context: what the lenses see

**Measuring and Exploiting Contextual Bias in LLM-Assisted Security Code Review.** Alexopoulos, Alexopoulos,
Spinellis, Mitropoulos, 2026. [V] <https://arxiv.org/abs/2603.18740>
- Describing a change as bug-free in the PR text cut vulnerability detection: GPT-4o-mini 97.2% → 3.6%, Claude 3.5
  Haiku 68.4% → 8.5%.
- Removing the PR description recovered 68.75% of missed detections; also telling the agent to ignore metadata
  recovered 93.75%. Adversarial descriptions fooled Claude Code and CodeRabbit in 32 of 33 CVEs.
- **panel-review:** challenges giving every lens the description. Perhaps only the intent and docs lenses should
  see it.

**ContextCRBench.** 2025. [V] <https://arxiv.org/abs/2511.07017>
- 67,910 entries from 153.7K issues and PRs. Textual context (issue, PR text) helps more than code context alone.
- **panel-review:** intent context helps, which pulls against the contextual-bias result.

**SWE-PRBench.** Kumar, 2026, single-author preprint. [V] <https://arxiv.org/abs/2603.26130>
- 8 frontier models find 15–31% of human-flagged issues on 350 PRs. Every model got worse as more context was
  pasted into the prompt.
- **panel-review:** worktrees to browse rather than context pasted into the prompt.

**AACR-Bench.** Alibaba, 2026. [S] <https://arxiv.org/abs/2601.19494>
- 200 PRs, 1,505 expert-verified comments, 10 languages. Whether context helps depends on model, language and
  retrieval method.

**A Roadmap for Modern Code Review.** Yang et al., ACM TOSEM 2026, DOI 10.1145/3800963. *brief* [V]
<https://arxiv.org/abs/2405.18216>
- Systematic review of 327 studies (2013–2025). Names the "Context Gap" and "Metric Misalignment". Proposes
  Context-Aware Proactivity, Value-Driven Evaluation and Human-Centric Symbiosis.
- "models often operate in isolation from project-specific history and architectural constraints"

**Gandalf.** Cândido et al., JSS 2026, DOI 10.1016/j.jss.2026.113069. *brief* [S]
<https://doi.org/10.1016/j.jss.2026.113069>
- A diff-only review agent. Context-blindness was the top barrier (reported by 10 of 17 participants); repo-wide
  context the most requested improvement. Calibrated trust dominated; blind trust is an automation-bias risk.

**WirelessCar.** Aðalsteinsson et al., ESEM 2025. *brief* [V] <https://arxiv.org/abs/2505.16339>
- 17 participants. AI-led review preferred over on-demand Q&A. Developers feared being "flooded with false
  positives". "I could miss something else, because I would focus on those improvements a lot."

**Code Review Comprehension.** Wurzel Gonçalves, Rani, Storey, Spinellis, Bacchelli, ICPC 2025. *brief* [V]
<https://arxiv.org/abs/2503.21455>
- 10 experienced reviewers, 25 real reviews. Context-building first, then inspection, including testing.
- Reviewers "contrast mental representations of expected and ideal solutions against the actual implementation."

**Code review as decision-making.** Heander, Söderberg, Rydenfält, EMSE 31:54 (2026). *brief* [V]
<https://arxiv.org/abs/2507.09637>
- 10 participants, 34 reviews. An orientation phase, then an analytical phase. Decisions include running the code and
  verifying CI. Warns that full automation risks losing knowledge transfer.

## 4. Posting less

**From Industry Claims to Empirical Reality.** Chowdhury et al., MSR 2026. [V] <https://arxiv.org/abs/2604.03196>
- 3,109 PRs. PRs reviewed only by agents merged 45.2% of the time, against 68.37% for human-reviewed. 12 of 13 agents
  had signal ratios below 60%.

**Automated Code Review In Practice.** Cihan et al. (Beko), ICSE-SEIP 2025. [V] <https://arxiv.org/abs/2412.18531>
- 4,335 PRs. 73.8% of AI comments were resolved, but median PR closure time rose from 5h52m to 8h20m.
- **panel-review:** every posted comment costs time; count MEDIUM and LOW instead of posting them.

**BitsAI-CR, retiring rules** (see section 2). Rules are dropped when "the precision is high but the Outdated Rate is
consistently low": correct, but developers do not act on them. Go Outdated Rate 26.7%; humans 35–46%.

**RovoDev, actionability** (see section 2). 38.7% of AI comments led to a code change, against 44.45% for human
comments. Median PR cycle time −31%, human comments −35.6%. Observational design.

**AI-Assisted Fixes to Code Review Comments at Scale.** Maddila et al. (Meta), 2025. [V]
<https://arxiv.org/abs/2507.13499>
- Showing AI patches to reviewers made them about 5% slower. Showing them only to authors removed the slowdown.

**Evaluating the Impact of Explainable AI on Trust in AI-Assisted Code Review.** Gao et al., ISSTA 2026. [V]
<https://arxiv.org/abs/2607.24601>
- N=34. Full explanations gave the highest trust but lower agreement; moderate explanations the highest agreement
  (89.22%).
- **panel-review:** a compact claim, scenario and fix over long rationales.

**What makes a code review useful to OpenDev developers?** Turzo, Bosu, EMSE 29:6 (2024). *brief* [V]
<https://arxiv.org/abs/2302.11686>
- Usefulness depends on "technical contributions" and on "linguistic characteristics such as comprehensibility and
  politeness."

**Hold On! Is My Feedback Useful?** Ahmed, Eisty, EMSE 30:70 (2025). *brief* [V] <https://arxiv.org/abs/2501.06738>
- Usefulness can be predicted from the comment text alone.

**From Code Review to Code Critique (ARCTIC).** Maddila, Rigby et al., 2026. [V] <https://arxiv.org/abs/2607.29516>
- Intent prediction F1 0.86. Humans rank correctness, security and performance first; AI tools over-focus on style.
- **panel-review:** the intent lens, and keeping style below the bar.

## 5. The human decides

**These Aren't the Reviews You're Looking For.** Duma et al., EASE 2026. [V] <https://arxiv.org/abs/2605.02273>
- Most AI-generated PRs get no review. When reviewed, agents dominate and humans mostly steer them.

**RADAR.** Adams et al. (Meta), 2026. [V] <https://arxiv.org/abs/2605.30208>
- 535K+ diffs. Layered gating auto-approves low-risk diffs, with 1/3 the revert rate and 1/50 the incident rate.
- **panel-review:** the opposite direction. panel-review only blocks, never approves on its own.

**Impact of modern code review practices on software quality.** McIntosh, Kamei, Adams, Hassan, EMSE 21 (2016).
*brief* [V] <https://doi.org/10.1007/s10664-015-9381-9>
- Coverage, participation and expertise "share a significant link with software quality"; coverage alone does not
  guarantee few defects. Correlational.

**Developer perceptions of modern code review processes in practice.** Witter dos Santos, Nunes, Jannach, JSS 222
(2025). *brief* [V] <https://doi.org/10.1016/j.jss.2024.112288>
- Survey at a mid-sized company. "The use of informal communication can lead to knowledge vaporization."

## 6. Lenses and checklists

**Do explicit review strategies improve code review performance?** Wurzel Gonçalves et al., EMSE 27:99 (2022).
*brief* [V] <https://doi.org/10.1007/s10664-022-10123-8>
- 67 professional developers, mostly novice reviewers. No strong relationship between guidance and performance; a
  checklist helped on the complex task. Higher cognitive load went with better performance.

**Concerns identified in code review.** Sanuri Gunawardena, Tempero, Blincoe, IST 153:107054 (2023). *brief* [V]
<https://doi.org/10.1016/j.infsof.2022.107054>
- 417 comments, 116 defect types. 38% of concerns automatically detectable. Documentation is the most common concern.
- **panel-review:** green CI first; the docs lens.

**Toward effective secure code reviews.** Charoenwet, Thongtanunam, Pham, Treude, EMSE 29:88 (2024). *brief* [V]
<https://arxiv.org/abs/2311.16396>
- 135,560 comments in OpenSSL and PHP. Concerns in 35 of 40 CWE-699 categories; memory and resource issues
  discussed less. Only 39–41% of concerns addressed; 18–20% left unfixed over disagreement.

**Google: ML-resolved comments and AutoCommenter.** Froemmgen et al., ICSE-SEIP 2024 [V]
<https://doi.org/10.1145/3639477.3639746>; Vijayvergiya et al., AIware 2024 [V] <https://arxiv.org/abs/2405.13565>
- 7.5% of reviewer comments resolved with ML-suggested edits. An LLM enforcing best practices, over 50% rated helpful.
- **panel-review:** precedent for a narrow, rules-based lens.

**Does Mutation Testing Improve Testing Practices?** Petrović, Ivanković, Fraser, Just, ICSE 2021. [S]
- Showing mutants during review improved developers' testing over time.
- **panel-review:** the tests lenses' "would this test fail if the change were reverted?"

## 7. Evaluating panel-review

**Sphinx.** Zhang et al., 2026 preprint. *brief* [V] <https://arxiv.org/abs/2601.04252>
- 2,500 PRs, checklist coverage judged by an LLM; bug-free PRs should get "No comment". Gains come from fine-tuning.

**Code Review Agent Benchmark (c-CRAB).** Zhang, Roychoudhury et al., 2026. [V] <https://arxiv.org/abs/2603.23448>
- Human reviews turned into tests. PR-agent, Devin, Claude Code and Codex together solve about 40%.

**MCR-Bench.** Zheng, Wang et al., ISSTA 2026. [V] <https://arxiv.org/abs/2608.27442>
- 2,269 multi-round tasks. Performance drops across rounds; relevant to re-running on new pushes.

**CRScore.** Naik, Alenius, Fried, Rosé, NAACL 2025. [S] <https://arxiv.org/abs/2409.19801>
- A reference-free quality metric for review comments.

## 8. Foundations

- **Bacchelli, Bird, ICSE 2013.** [M] <https://doi.org/10.1109/ICSE.2013.6606617> Finding defects is the stated goal,
  but knowledge transfer and understanding the change dominate outcomes.
- **Sadowski et al., Modern Code Review: A Case Study at Google, ICSE-SEIP 2018.** [M]
  <https://doi.org/10.1145/3183519.3183525> Small changes, fast turnaround, few reviewers.
- **Cisco/SmartBear (Cohen, 2006).** [M] Defect finding drops sharply beyond about 200–400 LOC per session. Industrial
  lore, not peer-reviewed.

## 9. One area per reviewer

**When Instructions Multiply (ManyIFEval).** EMNLP Findings 2025. [V] <https://arxiv.org/abs/2509.21051>
- 10 models (GPT-4o, Claude 3.5 Sonnet, o3-mini, DeepSeek-R1, …), text with up to 10 instructions, code with up to 6.
  "performance consistently and drastically degrades as the number of instructions increases." Instruction count
  alone predicts performance within about 10%.

**MTAC-IFBench.** Tsinghua and Zhipu AI, 2026. [V] <https://arxiv.org/abs/2609.14992>
- Multi-turn coding inside Claude Code, about 91 accumulating constraints per task. Claude Opus 4.6: constraints
  followed 82.4% → 76.2%, turns with every constraint followed 19.0% → 2.3%. Gemini 3.1 Pro: 81.6% → 58.2%,
  28.1% → 0%. Session length and constraint count grow together, so they are confounded.

**How Many Instructions Can LLMs Follow at Once? (IFScale).** Jaroslawicz et al., 2025. [V]
<https://arxiv.org/abs/2507.11538>
- 20 models, up to 500 keyword instructions; best 68% at 500. Reasoning models (o3, Gemini 2.5 Pro) stay near-perfect
  through about 150 before declining; Claude Sonnet 4 declines linearly. Bias toward earlier instructions.

**AgentIF.** 2025. [V] <https://arxiv.org/abs/2505.16944>
- 707 instructions from 50 real agentic applications, 11.9 constraints on average; models generally perform poorly.

**Perspective-based reading (human).** Mechanism is human attention, so weak transfer to agents.
- Laitenberger, El Emam, Harbich, IEEE TSE 27(5) 2001 [V] <https://doi.org/10.1109/32.922713>: 60 Bosch professionals,
  C code. Teams with one perspective each beat teams with the full checklist in 2 of 3 runs, large effect, lower cost
  per defect. Individuals did not improve. Low overlap alone is not the benefit.
- Maldonado et al., EMSE 11 2006 [V] <https://doi.org/10.1007/s10664-006-5967-6>: no team gain; inconclusive.
- Basili et al., EMSE 1 1996 [V] <https://doi.org/10.1007/BF00368702>: "I get confused trying to wear all the hats!"
- Ciolkowski, ESEM 2009 [S]: meta-analysis; beats ad-hoc, not checklists; results depend on who ran the study.

**Roles for LLM agents.**
- ChatEval, ICLR 2024 [V] <https://arxiv.org/abs/2308.07201>: same role on every agent no better than one agent (53.8%);
  different roles 60.0%. GPT-3.5, temperature 0, 80 items.
- Should we be going MAD?, ICML 2024 [V] <https://arxiv.org/abs/2311.17371>: debate does not reliably beat
  self-consistency or ensembling; persona effects fragile.
- He, Treude, Lo, TOSEM 2025 [V] <https://doi.org/10.1145/3712003>: review of 71 multi-agent SE studies; role-playing
  capability named an open gap.
- Multi-Agent LLM Committees for Beta Testing, 2025 [V] <https://arxiv.org/abs/2512.21352>: committees beat one agent
  (78% → 92–100%), but no ablation separates personas, model diversity and agent count.

## 10. Independent critics

- AgentCoder [V] <https://arxiv.org/abs/2312.13010>: tests written by the code's own agent 61% correct, by a separate
  agent 88%; same-agent tests "can be biased by the code and lose objectivity."
- CodeAgent, EMNLP 2024 [V] <https://arxiv.org/abs/2402.02172>: removing the checker agent drops confirmed
  vulnerabilities from 93% to 73%.
- LLM Critics Help Catch LLM Bugs (CriticGPT), OpenAI 2024 [V] <https://arxiv.org/abs/2407.00215>: critics out-recall
  humans but hallucinate and nitpick more; recall and hallucination rise together; human plus critic keeps recall with
  fewer hallucinations.
- A Practical Approach to Verifying Code at Scale, OpenAI, Dec 2025 [V]
  <https://alignment.openai.com/scaling-code-verification/>: repository access and execution find more critical
  issues with fewer false alarms; precision preferred over recall; stylistic comments can have negative utility;
  verifier trained separately from the generator.

## Tensions

1. Context helps intent (ContextCRBench) but biases defect detection (contextual bias), and pasted-in context dilutes
   attention (SWE-PRBench). Consider per-lens context.
2. Same-model verifiers share blind spots (Great Models Think Alike; ten agents agreeing in Refute-or-Promote).
   Execution is the tiebreaker.
3. Aggressive filtering hides real bugs (Sifting the Noise). Count dropped findings instead of discarding them.

## Corrections to the brief

- [3] Heander: the "expected/ideal solution" finding belongs to [2] Gonçalves.
- [7] SWR-Bench: published at FSE 2026.
- [8] Gandalf: 10/17 reported context-blindness; they did not "request repo-wide context".
- [11] Gunawardena: first author is Sanuri; 38% of concerns, not of the 116 types.
- [12] Sphinx: preprint.
- [13] Authors are Charoenwet, Thongtanunam, Pham, Treude, not "Yu J.". "Project-specific prioritization" unverified.
- [14] Ericsson: 69% is severe plus important; important alone is 36%.
- [16] Authors are Witter dos Santos, Nunes, Jannach, not Dogan, Tüzün. Patch-size claim unverified.
- [17] RovoDev: ICSE-SEIP 2026.
- [1] Yang: article number 271 unverified.
