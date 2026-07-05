# Survey of Unexplored SuperInstance Clusters

## Scope and method

This survey covers the previously un-researched prefix clusters in GitHub user `SuperInstance`:
`flux-*` (214 repos), `agent-*` (79), `constraint-*` (46), `si-*` (~42, excluding the 3 already-studied dev-tooling repos), `crdt-*` (9), plus proportional samples of `grand-*`, `spectral-*`, `forge-*`, `graph-*`, `holodeck-*`, `zeroclaw-*`, `snapkit-*`, `tropical-*`, `sheaf-*`, `eisenstein-*`, `tile-*`, `Equipment-*`, and `gpu-*`.

I pulled the full repo list (`gh repo list SuperInstance --limit 4200`), fetched metadata for 596 target-prefix repos, sampled by commit count and size, cloned promising candidates, and inspected code and CI logs. **Local execution was not possible in this environment: `cargo`/`rustc` and `pytest` are not installed.** Therefore, claims about passing tests come from GitHub Actions logs downloaded via `gh api repos/SuperInstance/<repo>/actions/runs` and workflow files read from `.github/workflows/ci.yml`.

The recurring mission finding — that READMEs oversell the code — continues here. Several repos have green CI badges that mean little because the workflow ends every command with `|| true`.

---

## `flux-*` (214 repos) — Mostly bulk artifacts and aspirational specs

Verdict: **templated/bulk-generated noise with a few real-but-unintegrated specs.** No `nexus-runtime`-scale outlier.

The cluster is dominated by tiny repos (median size 27 KB) and a handful of huge ones that appear to be workspace dumps:
- `flux-isa-edge` (265 KB, Makefile, description "Preserved workspace artifact") and `flux-isa-std` (49 KB, same description) are artifact dumps, not maintained code.
- `flux-tensor-midi` is the most active by commits (≥100) and the largest Python repo (185 KB, ~430 K LOC). Its CI shows `ruff` reports **40,409 errors**, `mypy src/` fails with "Cannot read file 'src': No such file or directory", and `pytest --tb-short -q` errors because the flag is unrecognized — yet the run is marked **success** because every step uses `|| true`. The README promises a 4-D tensor MIDI system with PLATO rooms, gene-regulatory networks, and neural music cortex; the actual code is a sprawling collection of ~1,000 Python files with no coherent architecture or working test suite.
- `flux-vm-v3` (131 KB Rust, ~4.4 K LOC) describes a terminating constraint VM with 60 opcodes and SHA-256 proof certificates. Its CI is strict (`cargo fmt --check`, `cargo clippy -- -D warnings`, `cargo test --release`, `cargo bench --no-run`) and has **failed on every recent run**, so the implementation is not currently building clean.
- `flux-importer` (29 KB Rust) is a Flux-bytecode-to-synthetic-MIR bridge for `cuda-oxide`. It is well-documented but depends on the rest of the cuda-oxide/plato stack; standalone value is unclear.
- `flux-bridge`, `flux-baton`, `flux-crypto`, `flux-deploy`, `flux-genome-rs`, and `flux-hardware` were spot-checked; they are either small stubs or partial integrations tied to the broader flux fleet narrative.

No repo in this cluster demonstrates a working, self-contained system with passing tests under a strict CI.

---

## `agent-*` (80 repos) — Thin wrappers and one external fork

Verdict: **speculation and stubs; nothing fork-worthy.**

The entire cluster totals only ~12 MB. The top repos by commits are:
- `agent-harness-generator` (100 commits, 9.9 MB TypeScript/Rust). This is a **GitHub fork of `ruvnet/metaharness`** (`gh api` returns `"fork": true, "parent": "ruvnet/metaharness"`). It is not original SuperInstance work and adds no fleet-specific value; ignore for fork candidacy.
- `agent-skills-affyossmani` (100 commits, 330 B, no detected language) is essentially empty.
- `agent-knowledge` (52 commits, 277 B Rust) and `agent-operations` (13 commits, 255 B Python) are tiny concept repos.
- `agent-forge` (12 commits, 68 B TypeScript) is a stub.

None have meaningful CI or tests. The cluster looks like a brainstorming area for agent tooling concepts.

---

## `constraint-*` (46 repos) — One real, well-tested crate

Verdict: **`constraint-theory-core` is the strongest fork candidate found in this entire survey.** The rest of the cluster is supporting documentation or thin bindings.

- `constraint-theory-core` (139 KB Rust, ~12.7 K LOC, 74 commits). README claims "184 tests, zero deps." The `Cargo.toml` has **no runtime dependencies** (only `rand` and `criterion` as dev-deps). Downloaded CI logs for the latest actual project run (run #69, 2026-06-10) show:
  - `cargo test --verbose`: **262 tests passed** (136 + 30 + 54 + 42 across lib/integration/doc tests), 0 failed, 2 ignored.
  - The job matrix was canceled after clippy produced warnings, so the workflow run is marked **failure**, but the code itself compiles and the test suite passes on stable Rust.
  - Implements Pythagorean triple snapping, KD-tree spatial indexing, CSP/AC-3/backtracking/CDCL solvers, holonomy checking, sheaf cohomology, Ricci flow, Laman rigidity, and several quantizers.
  This is a real, substantial, zero-dependency Rust crate with broad geometric/constraint machinery. **Worth a deeper fork evaluation.**
- `constraint-theory-llvm` (21 KB Rust, 26 commits), `constraint-synth` (16 KB Python, 32 commits), `constraint-theory-ecosystem` (3.5 KB Python, 45 commits), `constraint-theory-python` (375 B Python, 32 commits), and `constraint-theory-research` (2.2 KB Python, 24 commits) are bindings, notes, or experiment scripts around the core idea. They do not stand alone.

---

## `si-*` (~42 repos) and `crdt-*` (9 repos) — Real but small utilities

Verdict: **Competent small crates, not outliers.**

- `si-variational-bayes` (61 KB Rust) is a variational-inference crate with an extensive README and ~32 claimed tests, but it has **no CI runs**.
- `si-cli` (67 B Rust, 8 commits) is a real clap-based CLI for scanning `CAPABILITY.toml` files; depends on `anyhow`, `clap`, `colored`, etc. Its CI run failed.
- `si-runtime-python`, `si-runtime-js`, `si-runtime-wasm`, `si-fleet-api` are tiny runtime stubs (~60 B each, 6–8 commits).
- The `crdt-*` repos (`crdt-core`, `crdt-map`, `crdt-gcounter`, `crdt-gset`, `crdt-lwwreg`, `crdt-orset`, `crdt-pnvector`, `crdt-sync`, `crdt-bench`) are small, focused Rust implementations of G-Counter, PN-Counter, G-Set, OR-Set, LWW-Register, etc. `crdt-core` and `crdt-map` have **successful CI runs**. Code inspection shows correct CRDT semantics (e.g., `merge` takes component-wise max for G-Counter). They are real but classroom-sized; no distributed-systems integration.

The CRDT cluster is conceptually sound but does not approach the scale of `git-native-agents` or `nexus-runtime`.

---

## `Equipment-*` (14 repos) — TypeScript specs with failing CI

Verdict: **READMEs describe a product; the repos lack test wiring.**

All 14 are TypeScript packages. The largest and most active:
- `Equipment-NLP-Explainer` (35 KB, 32 commits). README describes a full NLP explanation engine with confidence explanations, reasoning chains, audit trails, and multi-language support. The repo contains `src/__tests__/nlp.test.ts` and `vitest.config.ts`, but **`package.json` has no `test` script** — only `build`, `clean`, and `prepublishOnly`. CI fails with `npm error Missing script: "test"`.
- `Equipment-Escalation-Router` (30 KB, 24 commits). Promises "Bot→Brain→Human with 40x cost reduction." Same pattern: no `test` script, CI fails on `npm test`.
- `Equipment-Context-Handoff`, `Equipment-Memory-Hierarchy`, `Equipment-Monitoring-Dashboard`, `Equipment-CellLogic-Distiller`, `Equipment-Hardware-Scaler`, `Equipment-Self-Improvement` follow the same template: well-written README, TypeScript source, no working test command, CI red.

These are specification documents packaged as npm modules, not production-ready libraries.

---

## `zeroclaw-*` (9 repos) — Game-learning concept with broken CI

Verdict: **Real Python code, but CI does not actually run tests.**

- `zeroclaw-agent-early-version` (46 KB Python, 100 commits) is archived per its own README.
- `zeroclaw-arena` (765 B Python, 79 commits, ~26.7 K LOC) describes a tile-based Monte Carlo self-play framework for Tic-Tac-Toe, Connect 4, Go 9×9, and Texas Hold'em. It has real tests in `tests/test_zeroclaw.py`. However, CI runs `pytest || true` and the test collection fails with `ModuleNotFoundError: No module named 'numpy'` because the workflow only installs `pytest`, not the package dependencies. The green badge is an artifact of `|| true`.
- `zeroclaw-crew`, `zeroclaw-plato`, `zeroclaw-scout` are tiny stubs.

The idea is coherent, but the repo is not currently testable in CI and depends on unspecified NumPy usage.

---

## `holodeck-*` (10 repos) — Real, working MUD/agent environment

Verdict: **The second strongest fork candidate.** `holodeck-rust` is a real, tested system.

- `holodeck-rust` (221 B Rust, 44 commits, ~4.3 K LOC) is a Tokio async TCP MUD server with rooms, NPCs, gauges, combat, poker, comms, and permissions. CI runs `cargo build --release`, `cargo test`, `cargo fmt --check`, and `cargo clippy`. Downloaded CI logs for the latest run show:
  - **68 tests passed** (33 + 29 + 5 + 1 across test suites), 0 failed.
  - The README understates this (claims 9 tests).
- `holodeck-c`, `holodeck-zig`, `holodeck-go`, `holodeck-studio` are partial ports or smaller experiments with mixed CI status.

`holodeck-rust` is a concrete, runnable multi-agent environment. If the Cocapn fleet's MUD/agent-server concept is useful to `purplepincher`, this is the repo to fork, not a speculation.

---

## `forge-*` (20 repos) — Plato-tile tooling, not standalone

Verdict: **Supporting crates for the Plato/ForgeFlux pipeline; not independent products.**

- `forge-memory` (52 KB Rust, 6 commits), `forge-pipeline` (32 KB Rust, 6 commits), `forge-a2a` (26 KB Rust, 6 commits), `forge-soniqo` (25 KB Rust, 6 commits) are real Rust crates for tile memory, pipeline orchestration, A2A messaging, and audio decomposition. They are well-scoped but tightly coupled to the Plato agent ecosystem. CI status is mixed.
- Standalone `forge-*` differs from the previously studied `plato-forge-*` variants; the standalone set is newer and more modular, but still assumes the Plato runtime.

Useful only if `purplepincher` is already adopting the Plato tile model.

---

## Smaller math-themed and specialty clusters (sampled)

Verdict: **Mostly real, self-contained Rust crates — more coherent than `lau-*`/`ternary-*` but still isolated from one another.**

- `grand-pattern-mono` (15 KB Rust, 10 commits) — scalar "vibe" graph with JEPA predictor; CI green.
- `spectral-fingerprint` (35 KB Rust, 5 commits) — AST spectral fingerprinting with power iteration; CI red.
- `graph-coloring-rs` (25 B Rust, 8 commits) — graph coloring algorithms, claimed 55 tests; CI red.
- `sheaf-laplacian` (5 KB Rust, 7 commits) — cellular sheaf Laplacian; CI green.
- `tropical-neural` (11 KB Rust, 7 commits) — tropical semiring / ReLU network representation; CI red.
- `eisenstein-quantize` (23 KB Rust, 7 commits) — A₂ lattice quantization; mostly empty implementation, CI red.
- `snapkit-rs` (20 KB Rust, 4 commits) — snapshot/diff/merge toolkit; no CI runs.
- `tile-*` and `gpu-*` are small CUDA/OpenCL/NEON experiments with a few commits each.

These are closer to a collection of independent math demos than a coordinated platform. None show cross-repo integration or fleet-scale architecture. `sheaf-laplacian` and `grand-pattern-mono` are the most complete in this group.

---

## Fork-worthy candidates

Only two repos in this entire unexplored set rise to "actually inspect for forking":

1. **`SuperInstance/constraint-theory-core`**
   - Evidence: ~12.7 K LOC Rust, **zero runtime dependencies**, **262 passing tests** in downloaded CI logs, implements a unique Pythagorean-manifold snapping engine plus KD-tree, CSP solvers, holonomy, cohomology, and quantizers.
   - Caveat: Latest CI run is red because the matrix cancels after clippy warnings/fmt issues on beta; stable tests pass.
   - Why fork: deterministic geometric constraint engine is a concrete, self-contained primitive unlike anything else surveyed.

2. **`SuperInstance/holodeck-rust`**
   - Evidence: ~4.3 K LOC Rust, Tokio async TCP MUD server, **68 passing tests** in strict CI (`cargo test`, `cargo fmt`, `cargo clippy`).
   - Caveat: Tightly themed around the Cocapn fleet; would need domain adaptation.
   - Why fork: it is a working multi-agent environment, not a spec.

## Honest bottom line

No `nexus-runtime`-scale hidden outlier appeared. The pattern from prior research holds: the vast majority of these repos are README-driven sketches, bulk-pushed artifacts, or forks of external work (`agent-harness-generator`). The exceptions are `constraint-theory-core` and `holodeck-rust`, both of which have real code and passing tests under reasonable CI. Everything else is either not buildable, not tested, or not original enough to merit a fork for `purplepincher`.
