---
name: crucible
description: Focused reviewers examine a pull request in parallel, verifiers try to disprove each finding, and the survivors are submitted as blocking review comments once the user approves. Use for "/crucible 123" or "/crucible <PR URL>".
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

Each run gets a run ID: 7 random hex characters, from
`head -c4 /dev/urandom | od -An -tx1 | tr -d ' \n' | cut -c1-7`. Draw again if `~/.cache/crucible/*-<run id>` exists.
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

## Resume

A run resumes the PR's newest crucible review, if there is one. Crucible marks what it posts with a footer: a review
body ends with `*crucible <run id>*` or `*crucible <run id> · follows <previous run id>*`, a finding comment with
`*crucible <run id> #<finding> · found by <reviewers>*`, and a thread reply with `*crucible <run id>*`. List the PR's
reviews and review threads with `gh api graphql`. If no review body has a crucible footer, this is a full review: skip
the rest of this section.

Otherwise the newest crucible review gives the previous run ID and, as its `commit_id`, the previous head. The previous
run's files are in `~/.cache/crucible/*-<previous run id>/`. If that directory is missing, tell the user which run ID
was not found and stop. Fetch the previous head with
`git fetch origin <sha>`; its merge base is `git merge-base <previous head> <base ref>`. Save the change since then as
`delta.diff`: `git diff --no-ext-diff <previous head> <head>` if the merge base is unchanged, otherwise
`git range-diff <previous merge base>..<previous head> <merge base>..<head>`, so a rebase onto the base branch does not
count as change.

Add to `context.md` the previous run ID, head and directory, the path to `delta.diff`, and every unresolved crucible
thread: its comment ID, finding, location, full text and every reply. Add the known findings too: every finding in the
previous run's `review.md`, posted or not, and the previous run's own known findings.

On a resume, a reviewer with `scope: commits` in its front matter is judged on, and reviews, every commit from merge
base to head as on a first run, and its message adds "Report none of the known findings in `context.md`". Every other
reviewer is judged on `delta.diff` alone, and its message says "Report only issues `<run>/delta.diff` introduces, and
none of the known findings in `context.md`" instead of "Report only issues this diff introduces". When the reviewers
start, also start one agent per unresolved crucible thread, in parallel, with this message:

> You settle one review thread on PR <n>, comment <comment id> in `<run>/context.md`. Read `<run>/context.md` first,
> and the finding's evidence in `<previous run>/<reviewer>/`. Your shell starts in the user's own
> checkout, which is not this PR: work only under `<run>`, with absolute paths or `cd` in every command. `head/` and
> `base/` are already built: run their binaries and existing tests with exactly the commands in `context.md`, and never
> edit them. To change code, create your own worktree at `<run>/threads/<comment id>/worktree/` from head, with its own
> build directory at `<run>/threads/<comment id>/target`, and delete that build directory as soon as you are done, with
> `rm -rf` on its literal absolute path.
>
> Give one verdict:
>
> - `fixed`: the finding's scenario no longer fails at head, and the fix is reasonable.
> - `not fixed`: the author says it is fixed, or the code changed, but the scenario still fails or the fix is not
>   reasonable.
> - `accepted`: the author disagrees, every factual claim in their reply holds, and their reasoning is reasonable.
> - `disputed`: the author disagrees, and a claim in their reply is false or their reasoning does not hold.
> - `open`: the author has not replied and nothing the finding is about changed.
>
> You must write the verdict, the evidence that settled it, and a reply of one or two sentences as the user would write
> it, with `cat > <run>/threads/<comment id>.md <<'EOF'`. Stop every background command you started before you write
> it.

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
description, clearly meet that condition; when in doubt, skip it. A reviewer without one always starts.

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

As each reviewer with findings finishes, start a new agent with this message:

> Try to disprove each finding in `<run>/<reviewer>/findings.md`, against the reviewer's definition of a finding in
> `<reviewer file>`. Read `<run>/context.md` first. Your shell starts in the user's own checkout, which is not this PR:
> work only under `<run>`, with absolute paths or `cd` in every command. Design and run your own check of each claim.
> Then apply its fix alone to a clean head in your worktree, and rerun the check. A finding survives only if its fix
> resolves it without breaking any rule in `<reviewer file>` or the repo rules, or any test. If a simpler fix also does,
> use that instead. Judge a fix that changes only text by reading it. Save each tested fix with
> `git diff --no-ext-diff`, which bypasses any configured diff tool, as `<run>/<reviewer>/fix-<n>.diff`. `head/` and
> `base/` are already built: run their binaries and existing tests with exactly the commands in `context.md`, and never
> edit them. Run tests that write into the source tree, such as snapshot tests, in your own worktree instead. To change
> code, such as adding a scratch test, create your own worktree at `<run>/<reviewer>/verify-worktree/`, from head unless
> the check needs base, with its own build directory at `<run>/<reviewer>/target`. Delete that build directory as soon
> as you are done, with `rm -rf` on its literal absolute path. You must write the findings that survive, each with its
> fix and the evidence that settled it, with `cat > <run>/<reviewer>/verified.md <<'EOF'`. Always write that file, with
> `None.` if nothing survives. Stop every background command you started before you write it.

## Triage

When every agent has stopped, not only written its file, check that every reviewer has a `findings.md` and every
reviewer with findings has a `verified.md`. Rerun a lane that is missing a file once; if it fails again, tell the user
which reviewer is missing. On a resume, also check that every unresolved crucible thread has a
`threads/<comment id>.md`, and rerun a missing one once. Then read every `*/verified.md` yourself and write
`review.md`. Several reviewers will often report the same problem in different words: merge findings with the same root
cause into one, keeping the evidence from every reviewer. Every posted comment is a Blocker. Write each comment as the
user would: a bold one-line claim, then mechanism, scenario, fix. End each comment with the line
`*crucible <run id> #<n> · found by <reviewers>*`, where `<n>` is the finding's number.

On a resume, `review.md` starts with the threads: for each, its comment ID, previous finding, verdict, reply and
action. The action follows the verdict:

- `fixed` or `accepted`: reply and resolve.
- `not fixed`: reply with the evidence and keep it open.
- `disputed`: ask the user whether to resolve it, or reply with the objection and keep it open.
- `open`: nothing.

End `review.md` with a merge brief: a recommendation (merge, merge after fixes, or look yourself at named spots) with
one sentence why, then the posted Blockers, CI state, what nobody verified (reviewers not applicable, checks not run),
and from `gh` other reviewers' open or dismissed threads and commits after the last approval.

Clean up, then reply to the user with these unnumbered sections, in this order:

- **PR:** its URL and title
- **Run directory**
- **Nothing is posted yet.**
- **Threads:** on a resume, each thread with its location, previous finding, verdict and action.
- **Will be posted as <event> on <short sha>:** each comment with its location, one-line claim and reviewers.
- **Merge brief**
- **Your call:** on a resume with `disputed` threads, each with the author's argument and the objection.
- **Shall I post the findings?**, or on a resume **Shall I post the findings and settle the threads?**

Write each section as a plain heading line, never as a list item, so the findings are top-level lists; the terminal
renders nested numbered lists with letters. Number only the findings, 1, 2, 3, …, and refer to findings only by those
numbers.

Post only after the user approves, as one submitted GitHub review (never pending) with `commit_id` set to the reviewed
SHA, where every comment is its own thread at its file:line. If the PR head has moved since, or a CI check that was
pending has failed, tell the user before posting. GitHub rejects the whole review if a comment is on a line outside
the diff, so anchor such a finding at the changed line that causes it (e.g. the new flag that needs docs). Put a
finding no changed line causes, such as one about a commit, in the review body, written as in `review.md`, after a
short summary. Submit as `REQUEST_CHANGES`; on the user's own PR, where GitHub forbids that, submit as `COMMENT`. With
no comments to post, ask the user whether to approve; on the user's own PR, post nothing. End every review body, an
approval's too, with `*crucible <run id>*`, or on a resume with `*crucible <run id> · follows <previous run id>*`.
On a resume, with the same approval and after posting the review, carry out each thread's action: post its reply with
`gh api repos/<owner>/<repo>/pulls/<n>/comments/<comment id>/replies`, ending with `*crucible <run id>*`, then resolve
it with the GraphQL `resolveReviewThread` mutation if its action says so. A thread that stays open still blocks, so
submit as `REQUEST_CHANGES` even with no new comments, and ask about approving only when no thread stays open.

Record each posted comment's ID, its finding and its reviewer directory in `posted.md`, so a later follow-up can go from
a thread back to its evidence. On a resume, also record each thread's verdict, reply ID and whether it was resolved.

## Cleanup

Whenever the run ends, including when it stops early, and only once every agent has stopped:
`git worktree remove --force` every worktree in the run, then `git worktree prune`, delete `refs/crucible/<run id>` and
no other ref, and delete the build directories and `<run>/clone`, with `rm -rf` on literal absolute paths only, never
on a variable. Keep the files.