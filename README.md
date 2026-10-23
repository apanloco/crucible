# Crucible

A Claude Code skill that reviews a GitHub pull request. Focused reviewers examine the PR in parallel, a verifier tries
to disprove each finding, and the survivors are posted as review comments with a merge risk. Only a high merge risk
blocks the PR.

## Install

```sh
git clone git@github.com:apanloco/crucible.git
ln -s "$PWD/crucible/skill" ~/.claude/skills/crucible
```

It needs the `gh` CLI, logged in with access to the repo under review.

## Use

```
/crucible 123
/crucible https://github.com/<owner>/<repo>/pull/123
```

## What the reviewed repo needs

Crucible reads `AGENTS.md` or `CLAUDE.md` on the base branch:

- **Build and test:** how to build and test the project, or where to find out, such as the CI pipeline. Without it,
  crucible stops.
- **Rules:** conventions the `repo-rules` reviewer checks and every fix must keep.
- **Consumers (optional):** other repos that use this one, and what their contract is. When a diff touches a contract,
  crucible clones that consumer, so agents can read how it is used and the merge risk weighs it.

## Layout

- `skill/SKILL.md`: the orchestration, from setup to posting.
- `skill/reviewers/`: one file per reviewer, each with the condition for starting it.
