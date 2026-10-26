# 1. Record architecture decisions

- **Status:** Accepted
- **Date:** 2026-10-26

## Context

Garmwall is a security tool built in public over many months, mostly by one person learning Rust along the way.
Many choices (crate layout, lints, CI hardening, protocol handling, policy engine) will look arbitrary to a reader
six months from now, including the author. The code shows *what* was done, but not *why*, nor which alternatives
were rejected.

For a security project this matters more than usual: reviewers and contributors need to know which properties are
deliberate guarantees (fail-closed, no panics on the data plane, pinned dependencies) and which are accidents.

## Decision

We record every significant architectural decision as an **Architecture Decision Record (ADR)**, following the
lightweight format described by Michael Nygard in
[Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions).

- ADRs live in `docs/adr/`, one Markdown file per decision, named `NNNN-short-title.md`, numbered sequentially.
- Each ADR has: **Status**, **Date**, **Context**, **Decision**, **Alternatives considered**, **Consequences**.
- Status is one of `Proposed`, `Accepted`, `Deprecated`, or `Superseded by ADR-NNNN`.
- Accepted ADRs are not rewritten. If a decision changes, a new ADR supersedes the old one, and the old one only
  gets its status updated. The history of reasoning stays intact.
- An ADR is added in the same pull request as the change it describes.
- [`docs/adr/README.md`](README.md) keeps an index.

A decision deserves an ADR when it is hard to reverse, affects more than one crate, deviates from the project
notebook (*Quaderno*), or a future reader would reasonably ask "why is it like this?".

## Alternatives considered

- **No written record, rely on commit messages and PR descriptions.** Commits explain a change, not a standing
  decision; the reasoning gets scattered across history and is hard to find.
- **A single `DECISIONS.md` file.** Easy to start, but grows into a wall of text, and changes to old decisions
  overwrite the original reasoning.
- **A wiki outside the repository.** Drifts from the code and is not reviewed in pull requests.

## Consequences

- Decisions are reviewable in pull requests together with the code that implements them.
- New contributors can read `docs/adr/` to understand the design without reverse-engineering it.
- Writing an ADR takes a few minutes per decision; that cost is accepted.
- Deviations from the *Quaderno* are tracked explicitly instead of silently.
