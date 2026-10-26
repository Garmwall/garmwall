# M0 · Rust basics

Study notes for milestone M0 (Quaderno cap. 6 ①-④), updated at the end of each session.
The first part covers the **setup**, which is already Rust: cargo, workspaces and lints are the tools you will use for the whole project.

---

## Part 0 · Setup: cargo, workspace, lints

### Cargo and the toolchain

- **`rustup`** manages Rust versions, like `nvm` for Node. Channels: `stable`, `beta`, `nightly`. We use `stable`.
- **`cargo`** is `composer`, `artisan` and `make` in one: dependencies, build, tests, lints, formatting.
- **`Cargo.lock`** is committed in applications (Garmwall ships a binary), like `composer.lock`: reproducible builds.

### Workspace

A **workspace** is a monorepo of crates sharing one `Cargo.lock` and one `target/` folder.

```toml
# Root Cargo.toml: a "virtual manifest", with no [package]
[workspace]
members = ["crates/*"]
resolver = "3"          # default with edition 2024

[workspace.package]
edition = "2024"        # crates inherit it with edition.workspace = true
```

- **Why many crates:** the compiler enforces the boundaries. If `garmwall-core` doesn't depend on `garmwall-proxy`, nobody can import the proxy from core by mistake. Builds are also incremental: change one crate and only it and its dependents get recompiled.
- **Functional core, imperative shell:** `garmwall-core` holds only pure types, with no I/O. It is fast to compile and trivial to test. I/O lives in the outer crates.
- **Package name vs binary name:** the package is `garmwall-cli`, the binary is `garmwall` (`[[bin]] name = "garmwall"`). In code the hyphen becomes an underscore: `use garmwall_core::...`.

> **Analogy:** like an Angular monorepo with several libraries under `projects/`. The difference is that in Rust a circular import between crates simply does not compile.

### Lints

A **lint** is a static check that flags suspicious code. Some come from `rustc`, others from **clippy** (Rust's "PHPStan").

| Level | Effect | Can `#[allow]` override it? |
|---|---|---|
| `allow` | silent | — |
| `warn` | warning (an error in CI with `-D warnings`) | yes |
| `deny` | error | yes |
| `forbid` | error | **no** |

```toml
[workspace.lints.clippy]
pedantic = { level = "warn", priority = -1 }  # group: lower priority
unwrap_used = "deny"                           # single lint: wins over the group
```

```toml
# in every crate: the whole block is inherited, not lint by lint
[lints]
workspace = true
```

- **Why no `unwrap()`:** it panics on `None` or `Err`. On the data plane a panic stops the firewall, which is a DoS. Use `?` or a `match` instead.
- **Why `forbid` on `unsafe_code`:** with `deny`, a PR could add `#[allow(unsafe_code)]` and get through. With `forbid` it can't.

### Doc comments

```rust
//! Documents the item that CONTAINS the comment (here: the whole crate).

/// Documents the item that FOLLOWS (here: the function).
pub fn decode() {}
```

With `missing_docs` every `pub` item needs its `///`. Pedantic also wants backticks around technical names (`doc_markdown`): `` `SQLite` ``, not SQLite.

---

## Mistakes made and how I fixed them

| Mistake | Cause | Fix |
|---|---|---|
| `members = [crates*]` is invalid | In TOML strings need quotes, and the right glob is `crates/*` | `members = ["crates/*"]` |
| `virtual manifest specifies a bin section` | `[[bin]]` put in the root `Cargo.toml`, which has no `[package]` | Moved to the `garmwall-cli` `Cargo.toml` |
| Crate missing from the PR | `cargo new` inside a repo had created a nested `.git` | Deleted the nested `.git` and added the crate in a new PR |
| `lint_groups_priority` | The `pedantic` group had the same priority as single lints | `priority = -1` on the group |
| Clippy passed "too easily" | Crates did not inherit the workspace lints | `[lints] workspace = true` in every crate |
| `[lints.rust] unsafe_code.workspace = true` | Lints cannot be inherited field by field | Inherit the whole block: `[lints] workspace = true` |
| Wrong linker in Git Bash | GNU's `/usr/bin/link.exe` shadows the MSVC one | Run cargo from PowerShell |

---

## Interview questions

1. What is the difference between a crate and a package? And between a `lib` crate and a `bin` crate?
2. Why split a project into several crates instead of using only modules?
3. Why is `unwrap()` dangerous in a network service? What do you use instead?
4. What is the difference between `deny` and `forbid`? When do you choose `forbid`?
5. Why is `Cargo.lock` committed in an application but not always in a library?

---

## Flashcards

**Q:** What does `[lints] workspace = true` do in a crate's `Cargo.toml`?
**A:** The crate inherits **all** the lints in `[workspace.lints]`. You cannot inherit a single lint.

**Q:** Why does `pedantic` have `priority = -1`?
**A:** It is a group: with a lower priority, single lints (e.g. `unwrap_used = "deny"`) can override it. Order in the file doesn't matter.

**Q:** `deny` or `forbid` for `unsafe_code`, and why?
**A:** `forbid`: it cannot be overridden with `#[allow]`, so no PR can quietly bring `unsafe` back.

**Q:** What happens if the data plane calls `unwrap()` on a `None`?
**A:** A panic: the thread stops and the firewall stops working, which is a DoS. Use `?` or `match`.

**Q:** What is the difference between `//!` and `///`?
**A:** `//!` documents the item that contains it (the crate, the module); `///` documents the item that follows.

**Q:** What is a virtual manifest?
**A:** A `Cargo.toml` with `[workspace]` but no `[package]`: it defines the workspace and is not a crate itself.

**Q:** Why does `garmwall-core` do no I/O?
**A:** Functional core, imperative shell: pure types, fast to compile and easy to test. I/O lives in the outer crates.

---

## Part 1 · Syntax, ownership and borrowing

*To be filled in during M0.*

## Part 2 · Structs, enums and `match`

*To be filled in during M0.*

## Part 3 · `Option`, `Result` and `?`

*To be filled in during M0.*

## Part 4 · Traits, modules and tests

*To be filled in during M0.*

## Project · `jsonrpc-inspect`

*To be filled in during M0.*
