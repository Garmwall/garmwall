# 2. Strict lints across the whole workspace

- **Status:** Accepted
- **Date:** 2026-10-26

## Context

Garmwall sits on the path of every message between an AI agent and its tools. A panic on the data plane does not
just fail one request: it takes the firewall down, which is a denial of service, and depending on how the host
reacts it can mean tool calls flowing unchecked or the agent losing all its tools.

The project notebook (*Quaderno*, cap. 8) planned `clippy::unwrap_used = "deny"` and `clippy::pedantic` on the
**core crates** only. All eight crates are new and empty today, so enabling strict lints now costs nothing, while
enabling them later on a large codebase means hundreds of warnings to fix at once.

## Decision

Lints are configured once in the root `Cargo.toml` under `[workspace.lints]`, and **every** crate inherits them with
`[lints] workspace = true`:

| Lint | Level | Why |
|---|---|---|
| `unsafe_code` | `forbid` | No `unsafe` anywhere; `forbid` cannot be overridden by a local `#[allow]`. |
| `missing_docs` | `warn` | Every public item is documented. Promoted to an error by `-D warnings` in CI. |
| `clippy::pedantic` | `warn`, `priority = -1` | Stricter idioms everywhere. The group has lower priority so single lints can override it. |
| `clippy::unwrap_used` | `deny` | No panics from `Option`/`Result` handling. |
| `clippy::expect_used` | `deny` | Same as above: `expect` is `unwrap` with a message. |
| `clippy::panic` | `deny` | No explicit `panic!` calls. |

CI runs `cargo clippy --workspace --all-targets -- -D warnings`, so every warning blocks a merge.

When a lint must be silenced, it is done locally, on the smallest item possible, with a written reason:
`#[expect(clippy::lint_name, reason = "...")]`. `#[expect]` is preferred over `#[allow]` because it warns when the
suppression is no longer needed.

## Alternatives considered

- **Pedantic on core crates only, as planned in the *Quaderno*.** Lower friction in the CLI and API crates, but the
  boundary is arbitrary: the CLI also parses untrusted input, and a split policy is one more thing to remember.
- **Lints declared per crate.** Eight copies of the same table drift apart over time.
- **`#![deny(...)]` attributes in each `lib.rs`.** Same drift problem, and invisible from `Cargo.toml`.
- **Default lints only.** Too permissive for a security tool.

## Consequences

- One place to read and change the lint policy.
- New crates are strict from their first line, as long as they add `[lints] workspace = true`.
- Pedantic sometimes flags code that is fine (e.g. `must_use_candidate`, `module_name_repetitions`); those cases get
  a local `#[expect]` with a reason, or the lint is relaxed in the workspace through a new ADR.
- Tests also fall under `unwrap_used`, `expect_used` and `panic`. If this becomes too noisy, a `clippy.toml` with
  the `allow-*-in-tests` options is the expected follow-up; it is not added until it is actually needed.
- This deviates from the *Quaderno*; the notebook should be annotated.
