# 4. Hardened CI and required checks

- **Status:** Accepted
- **Date:** 2026-10-26

## Context

A firewall for AI agents sees all of an agent's traffic, including secrets. If its build pipeline or one of its
dependencies is compromised, the attacker is inside the firewall. This is the supply chain risk listed as ASI04 in
the [OWASP Top 10 for Agentic Applications](https://genai.owasp.org), and it applies to Garmwall itself, not only
to the MCP servers it protects.

CI must therefore do two jobs: check the code, and not become an attack vector itself. A check that can be skipped
or faked protects nothing.

## Decision

The workflow in [`.github/workflows/ci.yml`](../../.github/workflows/ci.yml) runs four jobs on every pull request and
on every push to `main`:

| Job | Command | What it guarantees |
|---|---|---|
| `fmt` | `cargo fmt --all --check` | Consistent formatting |
| `clippy` | `cargo clippy --workspace --all-targets -- -D warnings` | The lint policy of ADR 0002 |
| `test` | `cargo test --workspace` | Tests pass |
| `audit` | `cargo audit` | No dependency with a known vulnerability (RustSec) |

The workflow is hardened:

- **Third-party actions are pinned to a full commit SHA**, with the version in a comment. A tag can be moved to
  different code; a commit cannot.
- **New action releases are adopted after a cooldown** of about a week, so a compromised release has time to be
  noticed before it reaches this repository.
- **The token is read-only** (`permissions: contents: read`); no job needs to write.
- **`persist-credentials: false`** on checkout, so later steps cannot reuse the token stored in `.git/config`.

The four jobs are **required status checks** in the `main` ruleset:

- each check is bound to the GitHub Actions app (`integration_id: 15368`), so a status with the same name created
  through the API by anything else does not satisfy it;
- `strict` mode requires the branch to be up to date with `main`, so CI tests the code exactly as it will be after
  the merge.

## Alternatives considered

- **Actions referenced by tag (`@v4`).** Convenient and auto-updating, but the tag owner, or whoever compromises
  them, can change what runs in our CI.
- **CI as information only, without required checks.** A red build could still be merged.
- **Required checks without `strict`.** Two pull requests that pass separately can break `main` together, because
  each was tested against an older `main`.
- **A merge queue.** Solves the same problem as `strict` without manual updates, but is unnecessary overhead for a
  repository with one main contributor. To reconsider if contributions grow.

## Consequences

- No change reaches `main` without passing all four checks on up-to-date code.
- Updating an action is a deliberate change: look up the new SHA, wait the cooldown, open a PR.
- Renaming a job in the workflow requires updating the ruleset, otherwise the required check never arrives and
  every PR stays blocked.
- `cargo audit` can fail on a new advisory without any code change; that is intended, and the fix is a dependency
  update or a documented exception.
- Automated dependency updates (Dependabot or Renovate) and `cargo-deny` for licences and sources are likely
  follow-ups, each with its own ADR.
