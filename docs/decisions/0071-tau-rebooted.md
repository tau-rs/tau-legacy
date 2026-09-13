# ADR-0071: tau, rebooted — this repository becomes tau v1

**Status:** Accepted
**Date:** 2026-09-13
**Deciders:** tau core

> **Terminal ADR.** This is the last decision recorded in this repository. It is
> not superseded by a later ADR here; it is superseded by a different repository.

## Context

This repository was started on 2026-04-24. As of this ADR it holds 652 commits,
564 merged pull requests, ~207k lines of Rust across 36 crates (47 workspace
members), 70 numbered ADRs, and 47 lettered guidelines (17 `G`, 25 `QG`, 5 `PG`)
spread over `CONSTITUTION.md`, `GUIDELINES_CHEATSHEET.md`, and `ARCHITECTURE.md`.

Those numbers are the diagnosis, not the achievement.

**There is no frozen interface anywhere in this tree.** The closest thing is the
"two contracts" of [ADR-0055](0055-tau-identity-two-contracts.md), whose own
versioning policy ([ADR-0056](0056-contract-versioning-stability-surface.md))
still moves when a feature needs it to. Every subsystem — the workflow IR, the
bundle format, the wasm guest ABI, the MCP family, the sandbox adapters, the SDK
codegen — grew its own interface at its own pace. Nothing is a program *over*
anything else; everything is a peer that must be kept consistent with every other
peer by hand. The 47 guidelines exist because that hand-consistency does not
scale, and the guidelines did not fix it: they are the cost of the shape, paid
monthly.

The symptoms are all the same symptom:

- **Governance grew faster than the core.** A constitution, a cheat sheet, an
  escape-hatch registry, and a build-time governance gate
  ([ADR-0057](0057-root-allow-governance.md)) were required before any interface
  had stopped changing. Rules are what you write when the shape cannot
  carry the invariant itself.
- **Breadth outran depth.** 36 crates share one `target/` lock hard enough that
  the repository ships a written discipline for invoking `cargo` at all
  (`CLAUDE.md`, "CARGO RULES"). A build system that needs a manual is reporting
  that it has more surface than core.
- **Verification tiering had to be invented to stay affordable.**
  [ADR-0063](0063-ci-tiering-nightly-authority.md) moved the authoritative test
  suite to a nightly job because the pre-merge gate could no longer cover the
  workspace in reasonable time. Tiering is correct (the successor keeps it), but
  here it arrived as relief from breadth rather than as a designed funnel.
- **The load-bearing behavior was not the tested behavior.** Gate 3's
  `dev == wasm` parity assertion — the claim that the wasm profile and the native
  one agree — has never once executed, because `WasmProfile::run()` is a stub;
  it is still open as [#691](https://github.com/tau-rs/tau/issues/691). The
  sandbox and container CI lanes were dark for months without a single `#[ignore]`
  to signal it ([#648](https://github.com/tau-rs/tau/issues/648)); the fuzz
  nightly had been effectively dead for months
  ([#649](https://github.com/tau-rs/tau/issues/649)); every scheduled gate in the
  repository was silently throttled by GitHub to fire every ~11-12h regardless of
  its cron expression ([#736](https://github.com/tau-rs/tau/issues/736)). That is
  what "no frozen core to be a program over" feels like from the inside: a feature
  can be simultaneously present, reviewed, merged, and inert, and nothing in the
  shape of the system objects.

None of this is a code-quality failure. The individual subsystems are good; the
tests are real; the CI is better than most. The failure is architectural and it
is upstream of all of it: **the project never had a minimal substrate that
everything else was expressible in terms of**, so it had to govern by rule what
it could not guarantee by construction.

The mission stated in `CONSTITUTION.md` — install packages, run agents, pass
messages, observe what happens — is unchanged and still correct. The
implementation of that mission is what failed.

## Decision

**tau keeps its name and abandons its implementation.**

1. This repository is renamed `tau-rs/tau-legacy` and archived read-only. It is
   referred to as **tau v1** — the prototype — in all subsequent writing.
2. A fresh `tau-rs/tau` is created from scratch. No history, no code, and no
   governance text is carried over.
3. The successor is an **agent kernel**: a frozen seven-syscall boundary
   (`spawn`, `exit`, `wait`, `cancel`, `send`, `recv`, `attach`), an
   event-sourced core whose log *is* the kernel, capability security enforced at
   runtime, and drivers as the only border with the world. Its full framing is
   the [kernel-reboot handoff](../retrospectives/2026-09-13-kernel-reboot-handoff.md),
   committed here as tau v1's final session entry and seeded into the successor
   as its `docs/HANDOFF.md`.
4. In the successor, **`1.0` means the ABI freeze** and nothing else. Version
   numbers stop being a marketing surface.

The one sentence carried in both READMEs: *"v1 explored the territory; this tau
is the kernel."*

## Consequences

- **This repository stops.** No further PRs, no further ADRs. The 41 open issues
  are closed by archiving, not by resolution; issues describing real design
  problems are re-derivable from the handoff's §4 and §10, which is where they
  now live.
- **Old deep links move.** GitHub's rename redirect from `tau-rs/tau` to
  `tau-rs/tau-legacy` dies the moment the fresh repository claims the old name.
  Every permalink into this repository will thereafter resolve against the
  successor or 404. That is the intended outcome — traffic should find the
  successor — and it is why this ADR merges *before* the rename: the archive must
  carry its own explanation on its own front page, since the links that would
  have explained it will no longer point here.
- **The published book at the old Pages URL goes stale** and is not rebuilt.
- **crates.io is unaffected for now.** Bare `tau` is squatted by an abandoned
  2015 crate. The successor publishes as `tau-kernel` with `[[bin]] name = "tau"`
  — `cargo install tau-kernel`, then type `tau` forever, the `ripgrep`/`rg`
  pattern.
- **What survives is knowledge, not code.** The handoff's §4 (eleven resolved
  design decisions) and §7–8 (the tiered pipeline) are distilled from what this
  repository learned the expensive way. The successor starts with those answers
  instead of rediscovering them.
- **The anti-pattern is named so it can be caught.** The successor's scope-creep
  regression rule is the irreducibility test: any proposed eighth syscall must be
  shown *inexpressible* as a program over the seven. If it is expressible, it is
  libc. That test is the structural replacement for 47 guidelines.

## Alternatives considered

**Incremental refactor toward a kernel inside this repository.** Rejected on
cost, not on sentiment: the frozen boundary has to be load-bearing from the first
commit for its guarantee to mean anything, and retrofitting it here means every
one of the 36 crates becomes a migration target while continuing to ship. The
migration would be strictly larger than the rewrite, and during it the project
would have two interfaces — the old peers and the new kernel — which is the
present failure mode with an extra layer.

**Keep the code, change the governance.** Rejected: the guidelines are a symptom.
Deleting the constitution without introducing a substrate removes the only thing
currently holding the peers consistent, and leaves the same shape with less
scaffolding.

**Rename the project as well as the repository.** Rejected: the name is the
promise, not the code. Linux survived rewrites of its scheduler and entire
subsystems without becoming a different project, for exactly this reason. A new
name would discard five months of accumulated meaning to signal a change that the
ABI freeze will signal far more credibly.

**Fork rather than archive.** Rejected: a live fork invites partial migration and
implies both trees are maintained. Read-only archival makes the successor the
only place work can happen, which is the point.
