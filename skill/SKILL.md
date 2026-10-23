---
name: crucible
description: Focused reviewers examine a pull request in parallel, verifiers try to disprove each finding, and the survivors are posted automatically as review comments with a merge risk, blocking the PR only when that risk is high. Use for "/crucible 123" or "/crucible <PR URL>".
argument-hint: <PR number or URL>
disable-model-invocation: true
---

The argument is a PR number or URL. Without one, do not guess: list the open PRs where the user's review is requested
and ask which one to review. If the PR is not open, warn the user that it is closed or merged, and continue.

Do not review the code yourself, and do not pass findings between agents yourself. Agents read and write files, and you
only start them.

The repo rules are `AGENTS.md`/`CLAUDE.md` on the base branch.

## Gate

The repo rules or README on the base branch must say clearly how to build, test and run the project, directly or by
pointing to where it is, such as the CI pipeline. Read them, and anything they point to, from GitHub with `gh`. Follow
such a pointer, but never search for build instructions on your own. If they are missing or leave a choice, tell the
user what is missing and stop: that is a repo problem to surface, not to work around.

## Run directory

Each run gets a run ID: 4 random hex characters, from `head -c2 /dev/urandom | od -An -tx1 | tr -d ' \n'`. Draw
again if it has no letter, so it cannot pass for a number, or if `~/.cache/crucible/*-<run id>` exists. GitHub links
only hex words of 7 or more characters to commits, so a run ID never becomes a commit link.
The run's directory is `~/.cache/crucible/<owner>-<repo>-<n>-<run id>/`. Set it up:

1. Work in the current repo if it is the PR's repo; otherwise clone it into `<run>/clone`.
2. Fetch the base branch.
3. Fetch the PR into a private ref named after the run ID, never into a branch:
   `git fetch origin pull/<n>/head:refs/crucible/<run id>`.
4. Check out the PR head and its merge base with the base branch as detached git worktrees, `<run>/head/` and
   `<run>/base/`.

Never touch the user's checked-out files. Registering worktrees and fetching the PR ref in the user's clone is fine, as
long as cleanup removes them.

Keep `sessions.md` in the run directory: the path to your own transcript, and, as you start each agent, its role (e.g.
`correctness reviewer`, `correctness verifier`) and transcript path, which is
`~/.claude/projects/<project>/<session>/subagents/agent-<id>.jsonl`.

## Context

Write `context.md` in the run directory: the run ID and start time, the PR title, description and linked issue, the
reviewed head and merge-base commit SHAs, every commit from merge base to head with its SHA and full message, the paths
to `head/` and `base/`, the diff (`git diff --no-ext-diff <base ref>...<head ref>`) saved as `pr.diff`, where the repo
rules are, and the CI checks on the head commit that fail or are pending, with their names and links, and nothing about
passing ones.

When the repo rules list consumers, other repositories that use this one through a contract, clone each consumer whose
contract the diff touches, at its default branch, with `git clone --depth 1` into `<run>/consumers/<repo>`; when in
doubt whether a contract is touched, clone it. Never read a local checkout of a consumer instead: it may be stale. List
in `context.md` each cloned consumer, its contract as the repo rules state it, and its path, so agents can read how a
changed contract is used.

## Previous threads

Every run is a full review, even of an unchanged head: a reviewer seldom finds everything at once, so earlier runs must
not narrow or steer the search. They count only through their threads, which each reviewer settles after its own
search.

Crucible marks what it posts with a footer: a review body ends with `*crucible <run id>*`, a finding comment with
`*crucible <run id> finding <n> · found by <reviewers>*`, and a thread reply with `*crucible <run id> · <verdict>*`,
each followed by ` · posted automatically`. Older footers have 7-character run IDs, `#<n>` for `finding <n>`, a
`follows <run id>`, or no verdict; read them too. Before starting the reviewers, list the PR's review threads with
`gh api graphql`. A previous thread is one that starts with a crucible finding comment, in a review the user posted,
and is unresolved, or was resolved, from `resolvedBy`, by the PR's author or by someone other than the user: on their
own PR, the user resolving a thread is the author resolving it. A resolved thread stops being one once a later crucible
review body lists it as fixed, or once it ends with a crucible reply whose verdict is `fixed` or `accepted`. A crucible
thread in someone else's review is another reviewer's thread, for the merge brief.

Keep the previous threads out of the run directory until a reviewer has written its `findings.md`, so no reviewer reads
them before its own search. Then write `<run>/<reviewer>/previous.md` with every previous thread whose footer names that
reviewer: its comment ID, run ID and finding number, location, full text, every reply, and whether and by whom it was
resolved. Continue that reviewer with SendMessage:

> Your own review is done: never edit `findings.md` again. `<run>/<reviewer>/previous.md` holds review threads that
> crucible posted on earlier runs of your reviewer. Settle each one at head, by reading, against the verdicts in the
> Previous threads section of `<this file>`; a thread the author resolved is only ever `fixed` or
> `resolved, still present`. A verifier checks each verdict next. You must write, for each thread, its comment ID, the
> verdict, the evidence that settled it, a reply of one or two sentences as the user would write it, and, when the
> scenario still fails, whether your `findings.md` found it again, as `finding <n>`, or missed it, with
> `cat > <run>/<reviewer>/threads.md <<'EOF'`. Stop every background command you started before you write it.

The verdicts:

- `fixed`: the finding's scenario no longer fails at head, and the fix is reasonable.
- `not fixed`: the author says it is fixed, or the code changed, but the scenario still fails or the fix is not
  reasonable.
- `accepted`: the author disagrees, every factual claim in their reply holds, and their reasoning is reasonable.
- `disputed`: the author disagrees, and a claim in their reply is false or their reasoning does not hold.
- `open`: the author has not replied and the scenario still fails at head.
- `resolved, still present`: the author resolved the thread, and the scenario still fails at head.

A thread the author resolved is only ever `fixed` or `resolved, still present`, and gets no reply. A thread none of
whose reviewers started is `not checked`, and so is one whose reviewers' verdicts disagree; it gets no reply.

## Builds

Every tree has its own build directory: `<run>/target/head` and `<run>/target/base`, and `<run>/<reviewer>/target` for
a verifier's worktree. Never let two trees share one, or their binaries overwrite each other.

Build and test with the commands the gate found, with the same flags as CI, so the results match CI's.

Build `head/` and `base/`, including the tests. If a build fails, tell the user and stop. Record in `context.md` the
exact build and test commands with their environment for each tree, and the absolute paths to the head and base
binaries.

## Reviewers

Every file in `reviewers/` next to this one with `enabled: true` in its front matter is a reviewer. A reviewer with
`start-only-if` in its front matter starts only if the diff and its commit list, read alone and not with the PR
description, clearly meet that condition; when in doubt, skip it. A condition on `context.md` is met by what it lists.
A reviewer without one always starts.

Before starting any reviewer, you must print this table in your reply to the user, and add it to `sessions.md`:

| Reviewer | Started | Reason |
| --- | --- | --- |
| `<name>` | yes / no | the change in the diff that meets or misses its condition, or "always starts" |

It has one row for every reviewer, started or not, so the user can see why each one runs or is skipped. Then start one
agent per started reviewer, in parallel, with this message:

> You are one of several reviewers working in parallel on PR <n>, each on a different area. Yours is described in
> `<reviewer file>`. Read `<run>/context.md` first. Your shell starts in the user's own checkout, which is not this PR:
> work only under `<run>`, with absolute paths or `cd` in every command. `head/` and `base/` are shared and read-only:
> read code there, but never edit, build, run or check out anything. Report only issues this diff introduces, and only
> those you can fix. For each, give file:line, or the commit SHA for a finding about a commit, the claim, why it
> breaks your reviewer's rules, and the simplest concrete fix as the code or text to write. Do not say how to check the
> claim. You must write them with `cat > <run>/<reviewer>/findings.md <<'EOF'`. Always write that file, with `None.` if
> you found nothing. Stop every background command you started before you write it.

## Verify

As each reviewer finishes, with findings or a `threads.md`, start a new agent with this message:

> Try to disprove each finding in `<run>/<reviewer>/findings.md`, against the reviewer's definition of a finding in
> `<reviewer file>`. Read `<run>/context.md` first. Your shell starts in the user's own checkout, which is not this PR:
> work only under `<run>`, with absolute paths or `cd` in every command. Check each claim in the cheapest way that
> settles it. Judge a fix whose whole effect you can see in the diff by reading it, without running code. Apply any
> other fix alone to a clean head in your worktree, rerun the check, and run the narrowest tests that cover the code it
> changes. Run nothing else, such as linters, formatters, release builds or other test suites: CI runs them on whatever
> the author pushes. A finding survives only if its fix resolves it without breaking any rule in `<reviewer file>` or
> the repo rules, or a test. If a simpler fix also does, use that instead. Save each fix with
> `git diff --no-ext-diff`, which bypasses any configured diff tool, as `<run>/<reviewer>/fix-<n>.diff`. `head/` and
> `base/` are already built: run their binaries and existing tests with exactly the commands in `context.md`, and never
> edit them. Run tests that write into the source tree, such as snapshot tests, in your own worktree instead. To change
> code, such as adding a scratch test, create your own worktree at `<run>/<reviewer>/verify-worktree/`, from head unless
> the check needs base, with its own build directory at `<run>/<reviewer>/target`. Delete that build directory as soon
> as you are done, with `rm -rf` on its literal absolute path. Wait for a background command by its completion notice,
> never by polling `pgrep -f`, which matches the polling command's own command line and never stops. You must write
> the findings that survive, each with its fix, the evidence that settled it, and, for a fix you judged by reading,
> why its whole effect is in the diff, with `cat > <run>/<reviewer>/verified.md <<'EOF'`. Always write that file, with
> `None.` if nothing survives. If `<run>/<reviewer>/threads.md` exists, also try to disprove each thread's verdict,
> against the verdicts in the Previous threads section of `<this file>`, where a thread the author resolved is only
> ever `fixed` or `resolved, still present`, by running the thread's scenario at head where
> reading does not settle it. You must write each verdict that holds, or the one that replaces it, with the evidence
> and reply, with `cat > <run>/<reviewer>/threads-verified.md <<'EOF'`. Stop every background command you started
> before you write it.

## Triage

When every agent has stopped, not only written its file, check that every reviewer has a `findings.md` and every
reviewer with findings has a `verified.md`. Rerun a lane that is missing a file once; if it fails again, tell the user
which reviewer is missing. Likewise, every reviewer with a `previous.md` must have a `threads.md` and a
`threads-verified.md`. Then read every `*/verified.md` and `*/threads-verified.md` yourself and write `review.md`. Drop
a finding with the same root cause as a previous thread, whichever reviewer settled it: that thread carries it. If that
thread's verdict is `fixed`, keep the finding instead, and make the thread `not checked`. Several reviewers
will often report the same problem in different words: merge findings with the same root cause into one, keeping the
evidence from every reviewer. Then compare each finding with every crucible finding comment on the PR from earlier
runs, resolved or not. A finding whose fix undoes what an earlier finding asked for, such as restoring code it had
removed or calling redundant code it had added, survives only if its evidence shows the earlier finding was wrong; its
comment then names that finding as `crucible <run id> finding <n>` and says why. Write each comment as the user would:
a one-line claim, then mechanism, scenario, fix. End each comment with the line
`*crucible <run id> finding <n> · found by <reviewers>*`, where `<n>` is the finding's number. Write a commit as the
word "commit" and its plain SHA, never in backticks, so GitHub links it and no one takes it for a run ID. Never put
`#` before a finding's number: GitHub links `#<n>` to issue or PR `<n>`. Write "finding <n>" instead.

With previous threads, `review.md` starts with them: for each, its comment ID, run ID and finding, verdict, reply and
action. The action follows the verdict:

- `fixed` on an unresolved thread, or `accepted`: reply and resolve.
- `fixed` on a thread the author resolved: nothing.
- `not fixed`: reply with the evidence and keep it open.
- `disputed`: reply with the objection and keep it open.
- `open`, `resolved, still present` or `not checked`: nothing. The author decides what to fix; the merge risk counts
  what stays.

End `review.md` with a merge brief: CI state, what nobody verified, including every `not checked` thread, and from
`gh` other reviewers' open or dismissed threads and commits after the last approval. A check counts as not verified
only if neither the run nor CI ran it; a check that CI runs and passes on the reviewed head is verified, even if no
agent ran it.

## Review summary

Whoever decides to merge reads the risk first and the findings second, so every run ends with a merge risk that they
can judge in a minute. When `review.md` is written, start one agent with this message:

> Assess the risk of merging PR <n> at the reviewed head, following the Review summary section of `<this file>`. Read
> `<run>/context.md`, `<run>/sessions.md`, `<run>/review.md`, every `<run>/*/verified.md` and
> `<run>/*/threads-verified.md`, and read the consumers in `context.md` to see whether real users reach a changed path.
> Your shell starts in the user's own checkout, which is not this PR: work only under `<run>`, with absolute paths or
> `cd` in every command. `head/` and `base/` are shared and read-only: read code there only to place a finding, and
> report no new findings. You must write the assessment with `cat > <run>/risk.md <<'EOF'`. Stop every background
> command you started before you write it.

The assessor judges only what the run established: the verified findings, the thread verdicts, CI, and what no
reviewer covered. It takes each finding's reviewers from its footer in `review.md`, and gives each finding a risk:
what merging it as it is can cost.

- `high`: merging can hurt users: a verified wrong result, crash, or lost or leaked data on a path real users reach,
  including an opt-in path that a product they use turns on; a security or credential issue; or a change to a
  persisted format, released API or CLI contract that a revert does not undo.
- `low`: merging hurts no one now, but makes harm more likely or costs whoever works on the code next: a latent bug
  such as an argument order nothing checks, a broken repo rule, a false or missing doc or message, or a wrong result
  only on a path no user reaches.
- `zero`: merging costs nothing but polish: the code is right, and the fix only makes it simpler or clearer.

The merge risk is the highest risk of any finding, and of any thread that is `not fixed`, `disputed`, `open`,
`resolved, still present` or `not checked`, so it says whether merging the PR as it is can hurt someone, and decides
the review's event. A failing CI check makes it `high`. A runtime change with no finding is `low`, and a diff that
changes no behaviour is `zero`. A changed path no test covers stays `low`, and the rationale names it.

The merge risk only makes sense to someone who knows what the PR changes, and the user has not read it. So "What it
does" says in at most three sentences what merging changes for those who maintain the code: what users, scripts and
servers observe differently, and what the code newly carries that someone must keep working, such as a flag, environment
variable, dependency, contract or persisted format. It comes from the diff, not the PR description, which tells the
author's intent, and says nothing of how the change is built. It judges nothing; the rationale does, without repeating
it.

The rationale and each bullet under it are at most two sentences; the detail belongs in the findings' comments. The
rationale names only the findings and facts that set the level. When the level rests on something the run could
not see, such as how a product outside the repo uses the changed path or a `not checked` thread, it says so and what
would settle it, such as "high if the editor calls `reconfigure`; ask the author". A pending CI check sets no level,
and the merge brief reports it.

`risk.md` holds exactly this, the review summary, with nothing before or after it:

```
**Merge risk: <Level>**

**What it does:** <What merging changes, as its maintainer sees it.>

**Rationale:** <The findings, as "finding <n>", or facts that set the level.>

**If this PR is merged as is:**
- Who is affected: <the default path or an opt-in path, and who uses it>
- Reversibility: <full, partial or none> — <what a revert after merging undoes, and what stays>
- To lower the risk: <the smallest step that drops the level, or "nothing to lower">

**Findings of crucible <run id> on commit <short sha>:**

| # | Risk | Finding | Reviewers |
| --- | --- | --- | --- |
| <n> | <risk mark> | <what goes wrong, in at most 8 words> | <reviewers> |

Risk: 🔴 **high** can hurt users if merged · 🟡 **low** harms no one now but costs later · ⚪ **zero** polish only
```

The risk mark is 🔴 for `high`, 🟡 for `low` and ⚪ for `zero`, alone in its cell. GitHub squeezes narrow columns
beside a long one until it breaks words, so the Finding text stays short enough for the table to fit, and a single
emoji cannot break. The number connects a row to its inline comment, whose footer carries the run ID and the same
number. The inline comment carries the location, so the table has none; a finding in the review body names what it
is about, such as "Commit message: …".

Findings are listed by risk, `high`, `low`, `zero`, and by number within a risk. With previous threads, a second table
follows, with a row for every thread that is not `fixed`, and after it one line for the `fixed` ones:

```
**Threads from earlier crucible runs:**

| Thread | Outcome |
| --- | --- |
| Crucible <run id> finding <n>: <what it was about, in at most 8 words> | **<verdict>** · <its action> · author: <reason> |

**Fixed since earlier runs:** crucible <run id> findings <n>, <n>–<n> · crucible <run id> findings <n>
```

The action is the one `review.md` gives the thread, such as "reply and resolve" or "none". The author's reason is their
last reply on the thread, in at most 10 words, quoted when short, or "no reply" when they resolved or left it without
writing anything, so a silent dismissal stands out. When a finding that sets the level was resolved without a reason,
the rationale says so. The fixed line is what tells later runs that those threads are settled, so it names every
`fixed` thread, by run ID and finding number.

When the assessor has stopped, renumber the findings in `risk.md` and `review.md` 1, 2, 3, … in the table's order,
including each comment's footer, start each finding with the line `**Finding <n> · <risk mark> <risk>**` above its
claim, and put the review summary at the top of `review.md`. The same review summary goes at the top of the posted
review body and in the reply to the user, so both read the same thing.

## Post

Crucible posts without asking, so nothing waits for the user; a post is easy to revert instead.

Write `<run>/preview.html`: one page that shows what is posted, in posting order, with the markdown rendered as GitHub
renders it. It has a card for the review body, one card per inline comment headed by its number and `file:line`, and
one card per thread reply headed by the thread's previous run ID and number. Render the exact text that is posted, with
`marked` from `cdn.jsdelivr.net` and its `breaks` option on, since GitHub turns newlines into line breaks, and in
GitHub's comment width and table layout, so a squeezed column shows. Follow the system's light or dark theme, and keep
the page local: it holds the repo's code.

Post one submitted GitHub review (never pending) with `commit_id` set to the reviewed SHA, where every comment is its
own thread at its file:line. Before posting, check that the review holds every finding in `risk.md`: one comment for
each finding on a code line, and the rest in the body; if one is missing, fix the review before posting it. If the PR
head moved during the run, post nothing and tell the user to run crucible again. GitHub rejects the whole review if a
comment is on a line outside the diff, so anchor such a finding at the changed line that causes it (e.g. the new flag
that needs docs).

The review body opens with `*Written by crucible, run by <user>.*`, then the review summary. After it, under the
heading `**Findings not on a code line:**`, come the findings no changed line causes, such as one about a commit,
written as in `review.md` but without a footer of their own: the review body's footer covers them. End the body with
`*crucible <run id> · posted automatically*`; every comment and thread reply footer
ends with ` · posted automatically` too.

GitHub renders every newline in a review as a line break, so post each paragraph as one line: join the wrapped lines of
`review.md`, and keep the line breaks of code blocks, list items, tables and headings.

The merge risk picks the event: `REQUEST_CHANGES` for `high`, `COMMENT` for `low` and `zero`. Every event posts the same
comments; only `REQUEST_CHANGES` blocks the merge. When it blocks, the body's second line is
`*Blocked automatically: merge risk high. <user> lifts it by approving or dismissing this review.*` On the user's own
PR, where GitHub forbids `REQUEST_CHANGES`, submit as `COMMENT`. Never approve on the user's behalf: a `COMMENT` does
not lift an earlier `REQUEST_CHANGES`, so when the user's newest earlier review requested changes and the risk is not
`high`, tell them it still blocks the PR and ask whether to approve. With no findings and no open thread, ask too.

After posting the review, carry out each previous thread's action: post its reply with
`gh api repos/<owner>/<repo>/pulls/<n>/comments/<comment id>/replies`, then resolve it with the GraphQL
`resolveReviewThread` mutation if its action says so.

Record each posted comment's ID, its finding and its reviewer directory in `posted.md`, so a later follow-up can go from
a thread back to its evidence. Also record each previous thread's verdict, reply ID and whether it was resolved.

Clean up, then reply to the user with these unnumbered sections, in this order:

- **PR:** its URL and title
- **Posted as <event> on <short sha>:** the review's URL, and when it blocked, how to lift it.
- **Run directory** and **Preview:** the paths to the run directory and `preview.html`.
- **Review summary:** `risk.md`, verbatim.
- **Merge brief**
- **Your call:** each `disputed` thread with the author's argument and the objection posted, and whether to approve
  when an earlier block of the user's could be lifted.

Write each section as a plain heading line, never as a list item; the terminal renders nested numbered lists with
letters. Number only the findings, 1, 2, 3, …, and refer to findings only by those numbers.

When the user asks to lift a block, dismiss that review with
`gh api -X PUT repos/<owner>/<repo>/pulls/<n>/reviews/<review id>/dismissals`, with a message saying they lifted it.

## Cleanup

Whenever the run ends, including when it stops early, and only once every agent has stopped:
`git worktree remove --force` every worktree in the run, then `git worktree prune`, delete `refs/crucible/<run id>` and
no other ref, and delete the build directories, `<run>/clone` and `<run>/consumers`, with `rm -rf` on literal absolute
paths only, never on a variable. Keep the files.