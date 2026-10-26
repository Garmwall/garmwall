# Journal

The project's logbook: one entry per session, most recent first.
Each entry answers four questions: what I did, what I learned, where I got stuck, what comes next.

Architecture decisions don't go here but in [`docs/adr/`](adr/); study notes go in [`docs/learning/`](learning/).

---

## 2026-10-26 · #5 · CI and project docs

**Done**
- Workflow `.github/workflows/ci.yml` with four jobs: `fmt`, `clippy`, `test`, `audit` (PR #5).
- Actions pinned to a full commit SHA, read-only token, `persist-credentials: false`, caching with `rust-cache`.
- The four jobs are now required checks in the `main` ruleset, with `strict` and `integration_id` (PR #6).
- Drafts of `journal.md`, ADRs 0001-0004 and `learning/M0-rust-basics.md`.

**Learned**
- An action's tag can be moved to different code, a commit SHA cannot: pinning by SHA is a supply chain defence.
- `integration_id` binds a check to the GitHub Actions app: a check with the same name created through the API does not count.
- `strict`: a PR's CI tests the branch merged with `main` as it was at that moment. If `main` changes, the result is stale, and two green PRs can break `main` together.
- `git fetch --prune` only removes `origin/…` refs for branches deleted on GitHub; local branches merged with squash must be deleted with `git branch -D`.

**Stuck on**
- How to structure the CI YAML file: solved by starting from a commented workflow.
- The question about `strict`: solved with the example of two PRs, one renaming a function and one calling it.

**Next**
- Review and merge the docs, then start M0: Rustlings and the `jsonrpc-inspect` mini tool.

**To annotate in the Quaderno**
- `resolver = "3"` instead of `"2"`: it is the default with edition 2024.
- One branch per task instead of one per milestone (ADR 0003).

---

## 2026-10-26 · #4 · Workspace lints

**Done**
- `[workspace.lints]`: `unsafe_code` forbid, `missing_docs` warn, `clippy::pedantic` warn (priority -1), `unwrap_used`/`expect_used`/`panic` deny.
- Every crate inherits them with `[lints] workspace = true`; removed the `add` placeholders and added `//!` docs to each crate (PR #4).

**Learned**
- `forbid` cannot be overridden with `#[allow]`: nobody can sneak `unsafe` in through a PR.
- An `unwrap()` on the data plane is a panic, which means a DoS of the firewall.
- Lint groups need `priority = -1`; lints are inherited as a whole block, not one by one.
- A passing check proves nothing until you have seen it fail.

**Stuck on**
- The syntax for inheriting lints (I tried `lints.rust` field by field).
- Crate doc comments written by Claude on request: **unblock used**, to be rewritten from memory.

**Next**
- Step 7: CI with GitHub Actions.

---

## 2026-10-25 · #3 · Ruleset and Cargo workspace

**Done**
- `main-protection` ruleset versioned in `.github/rulesets/` (PR #1).
- Cargo workspace with 8 crates, shared fields in `[workspace.package]`, `garmwall` binary (PR #2, #3).
- Branch naming convention `type/description`.

**Learned**
- Separate crates: boundaries enforced by the compiler, incremental builds, `pub` visibility.
- `garmwall-core` holds only pure types with no I/O (functional core, imperative shell).
- Virtual manifest: the root has no `[package]`, so `[[bin]]` goes in the crate.
- The package name (`garmwall-cli`) and the binary name (`garmwall`) can differ; in code the hyphen becomes an underscore.

**Stuck on**
- TOML syntax and workspace inheritance.
- `cargo new` had created a nested `.git` in `garmwall-core`: recovered with PR #3.

**Next**
- Step 6: workspace lints.

---

## 2026-10-25 · #2 · Repository, license and README

**Done**
- `.gitignore`, Apache-2.0 LICENSE + NOTICE ("The Garmwall Authors"), `Garmwall` GitHub org, README with the brand kit, personal profile README.

**Learned**
- In a security tool `cargo-audit` matters more: a compromised dependency is an attacker inside the firewall (ASI04).
- `Cargo.lock` is committed in an application.
- With Apache-2.0 the NOTICE file must be redistributed with the artifacts.

**Stuck on**
- README banners referenced but not committed: fixed with a dedicated commit.

**Next**
- Ruleset on `main`, then the Cargo workspace.

---

## 2026-10-25 · #1 · Toolchain, editor and cargo tools

**Done**
- Stable MSVC toolchain, RustRover with Rustfmt on save and Clippy on the fly.
- Installed `cargo-nextest`, `bacon`, `cargo-audit`, `cargo-insta`.

**Learned**
- The stable, beta and nightly channels.
- `i32` vs `usize`: a length cannot be negative.
- The `len_zero` lint: prefer `is_empty()`.

**Stuck on**
- In Git Bash, `/usr/bin/link.exe` (GNU) shadows the MSVC linker: cargo commands must run from PowerShell.

**Next**
- Step 4: repository.
