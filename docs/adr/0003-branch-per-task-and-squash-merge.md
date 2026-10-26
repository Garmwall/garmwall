# 3. One branch per task, squash merge

- **Status:** Accepted
- **Date:** 2026-10-26

## Context

`main` is protected by a ruleset: no direct pushes, no force pushes, linear history, every change through a pull
request (see [`.github/rulesets/main-protection.json`](../../.github/rulesets/main-protection.json)).

The project notebook (*Quaderno*, cap. 8) suggests one branch per milestone. Milestones last two to four weeks, and
work happens in short sessions of one to two hours a few times a week.

## Decision

- **One branch per task**, not per milestone. A task is what fits in one or two sessions and can be reviewed on its
  own.
- Branch names: `type/short-description`, using the Conventional Commits types (`feat`, `fix`, `test`, `docs`,
  `refactor`, `perf`, `chore`, `ci`). Inside a milestone the description starts with the milestone,
  e.g. `feat/m1-stdio-transport`.
- Every branch is merged through a pull request with **squash merge**. The PR title follows
  [Conventional Commits](https://www.conventionalcommits.org) and becomes the commit message on `main`.
- Merged branches are deleted, locally and on GitHub.

## Alternatives considered

- **One branch per milestone.** A long-lived branch accumulates weeks of changes, produces one huge PR that is hard
  to review, and drifts away from `main`. CI feedback arrives late.
- **Direct commits to `main`.** Fastest, but skips CI and review, and is blocked by the ruleset anyway.
- **Merge commits instead of squash.** Keeps every intermediate commit ("wip", "fix typo") in the history of
  `main`; the ruleset requires linear history.

## Consequences

- `main` reads as a list of complete, described changes: one line per task, easy to scan and to revert.
- PRs stay small, and CI runs on every step, so problems surface early.
- Intermediate commits on a branch are free-form; only the PR title has to be clean.
- After a squash merge, git does not recognise the local branch as merged, so it has to be deleted with
  `git branch -D`.
- This deviates from the *Quaderno*; the notebook should be annotated.
