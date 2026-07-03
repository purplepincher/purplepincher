# Ternary / Conservation-Law Repo Cluster — Assessment

Scope: repos owned by `SuperInstance` whose names contain `ternary` or `conservation`, plus the manually identified `delta-clt` and `noether-bridge`.  The cluster is one of the largest coherent themes in the SuperInstance account (~300 `ternary-*` repos and ~80 `conservation-*` repos).  This note focuses on the applied-looking and explanatory repos, not the whole long tail.

## 1. What the conservation law actually claims

The central narrative is stated most clearly in `conservation-languages/README.md` and `conservation-languages/PAPER.md`:

* **The identity:** `γ + η = C` is presented as a conservation law.  In the docs it is explicitly identified with Shannon's chain rule:
  ```
  H(X) = I(X;G) + H(X|G)
  ```
  i.e. `C = H(X)` (total Shannon entropy), `γ = I(X;G)` (mutual information, the "predictable/coupled" part), and `η = H(X|G)` (conditional entropy, the "surprise/noise" part).
* **The fleet cancellation claim:** For `n` independent agents each voting ternary `{-1, 0, +1}` with equal probability, the aggregate sum `S = Σ sᵢ` concentrates near zero.  They define a cancellation factor `1 − |S|/n` and a "cancellation delta" `δ(n) = |S|/n`.
* **The concrete prediction:** `δ(n) = (1/√n)(1 − 3/(2n))`, with `γ_efficiency(n) = 1 − δ(n)`.  At `n = 1,000,000` this predicts ~99.9% cancellation (`PAPER.md:48-52`).
* **The broader pitch:** The docs claim this is universal — markets, democracies, neural networks, immune systems, etc. — and that the ternary alphabet is uniquely optimal because it is zero-mean and carries maximum entropy `log₂(3)`.

So the claim has two very different parts: (1) a genuine information-theory identity repackaged as a conservation law, and (2) a specific `δ(n)` formula about cancellation in ternary voting.

## 2. Is it rigorous?

### 2.1 The `γ + η = C` identity

It **is** rigorous, but it is not new and it is not a physical conservation law.  It is the standard Shannon chain rule.  `conservation-cli/src/conservation.rs:253-285` computes it correctly from samples:

```rust
let hx = shannon_entropy(&x_dist);
let hg = shannon_entropy(&g_dist);
let hxg = joint_entropy(&joint);
let gamma = hx + hg - hxg;        // I(X;G)
let eta = hxg - hg;               // H(X|G)
let c = hx;
```

`native-conservation-core/src/ternary_alu.c` also has `ternary_conservation_analyze()` that computes Shannon entropy and mutual information from two ternary arrays.  These implementations are sound as far as they go.

The leap from "Shannon chain rule" to "conservation law of information" is semantic packaging.  Nothing in the code enforces or proves an additional physical invariant.

### 2.2 The `δ(n)` cancellation formula

This part is **not rigorous**.  The formula `δ(n) = (1/√n)(1 − 3/(2n))` is an ad-hoc approximation with the wrong leading coefficient.

For i.i.d. uniform ternary `sᵢ ∈ {-1,0,1}`:

* `Var(sᵢ) = E[sᵢ²] = 2/3`.
* By the CLT, `S ≈ N(0, 2n/3)`, so `E[|S|] ≈ √(4n/(3π)) ≈ 0.6515 √n`.
* Therefore `δ(n) = E[|S|]/n ≈ 0.6515 / √n`.

The repo's formula predicts roughly `1/√n`, which is about **50% too large**.  `PAPER.md:28-31` actually derives the correct `0.6515/√n` coefficient in the proof sketch, then immediately states a theorem whose coefficient is `1.0`.  The "Error vs MC" table in `PAPER.md:47-52` cannot be reproduced from the stated model.

I ran an independent Monte Carlo simulation and the repo's own `delta-clt/delta_clt.py`:

| n | actual `E|S|/n` | repo's `δ(n)` | drift |
|---|------------------|---------------|-------|
| 50 | 0.092 | 0.137 | 0.045 |
| 100 | 0.065 | 0.099 | 0.034 |
| 500 | 0.029 | 0.045 | 0.016 |

`delta_clt.py` calls this "VERIFIED within bounds" because its acceptance threshold is `abs(eta_mean - eta_theoretical) < 0.10` (`delta_clt.py:299,352`).  That is a loose tolerance, not a proof.  The script also layers on unrelated "dependency graph" and "colony" checks with hand-tuned weight mappings (`delta_clt.py:161-264`) that have no derivation from the ternary model.

### 2.3 The verification repos

* `delta-clt` is a Python script that prints tables.  It has no unit tests, no assertions, and no formal proof.  Its "verification" is visual matching against an empirically wrong formula.
* `conservation-verify` is a Rust harness around a `Simulation` trait; the bundled `DeterministicSimulation` is just algebra, not a real multi-agent simulation.  Baseline expected values are magic constants.
* `conservation-verify-c` builds and its 34 assertions pass, but the "law" it verifies is that a fixed 59.5/29.5/10.5% random distribution produces a few derived statistics within loose thresholds.
* `conservation-explorer` is an HTML visualization whose entropy value is hard-wired to approach `log₂(3)` regardless of input.
* `conservation-languages` benchmarks measure cancellation factor, not `γ + η = C`, and its throughput figures are unchecked in this environment.

**Bottom line:** the cluster has a sound-but-trivial information-theory identity at its core, wrapped in an incorrect/ad-hoc cancellation formula and a great deal of evocative prose.  The "verification" is Monte Carlo curve-fitting with generous tolerances, not a mathematical proof.

## 3. Per notable repo assessment

| Repo | Language | What it does | Maturity / quality | Useful as standalone? |
|------|----------|--------------|--------------------|-----------------------|
| `ternary-pid` | Rust | Position-form PID quantized to `{-1,0,+1}`; anti-windup, derivative filter, cascade, feedforward (`src/lib.rs:40-117`) | Has CI, ~14 unit tests.  No `dt`, no output clamping, non-standard cascade wiring (`src/lib.rs:114`), `PLUG_AND_PLAY.md` shows a 3-arg API that does not exist. | **Most usable applied tool** — a decent bang-bang / three-state actuator controller skeleton, but needs engineering polish. |
| `ternary-svm` | Rust | Linear SVM for ternary feature vectors using PEGASOS; binary and one-vs-one 3-class (`src/lib.rs:240-584`) | CI, ~25 tests, CSV CLI.  `kernel_hint` is a no-op placeholder; untrained model returns `0` silently; only linear kernel. | Usable minimal ternary-feature classifier for small datasets. |
| `ternary-entropy` | Rust | Shannon entropy, conditional entropy, MI, KL/JS divergence, cross-entropy for ternary distributions (`src/lib.rs:95-292`) | CI, clean tests, `#![forbid(unsafe_code)]`. | Clean, usable tiny information-theory crate. |
| `ternary-types` | Rust | Core `Ternary` enum, `TritVector`, `TernaryMatrix`, packed 2-bit storage, Z₃ arithmetic | CI, tests.  `#[no_std]` is broken for `Vec`-using modules; README/doc drift; `packed` feature forces nightly. | Useful low-level ternary primitives if you fix the no-std path. |
| `ternary-arithmetic` | Rust | Arithmetic coder for `{-1,0,+1}` alphabet (`src/lib.rs:113-322`) | CI, 20 tests.  Panics on bad input; no streaming API; `compression_ratio` baseline is one byte/symbol. | Minimal usable entropy codec for ternary streams. |
| `ternary-compress` | Rust | RLE, sparse ternary storage, dictionary coding, 2-bit packing (`src/lib.rs:7-133`) | CI, 7 tests.  License mismatch (README MIT / Cargo.toml Apache-2.0); dictionary decode fragile; no GPU integration despite README claim. | Toy compression primitives, not production. |
| `ternary-fleet-packing` | Rust + Python | Ternary vector packing schemes: 2-bit, sign+nonzero, base-3 5-trit/byte, RLE (`src/lib.rs:79-309`); Python backup has vector ops and wire format | Rust crate depends on optional git crates not used in code; Python tests pass. | Low-level ternary packing library; Python side is more coherent. |
| `ternary-route` | Rust | Toy load balancer with `+1/0/-1` health tiers, min-load routing, weighted round-robin (`src/lib.rs`) | CI, 8 tests.  `weighted_route(&self)` never advances state; `rebalance()` resets loads to average; no async/network. | Not production-usable; a teaching demo. |
| `ternary-search` | Rust | Generic graph search (BFS/DFS/A*/beam) on graphs whose nodes carry a `Ternary` label (`src/lib.rs:109-413`) | CI, 17 tests.  `Ternary` label is mostly decorative; algorithms are generic. | Toy graph-search crate. |
| `ternary-search-rs` | Rust | Axum HTTP server for brute-force cosine similarity over 384-dim crate embeddings (`src/vectors.rs:69-251`) | No tests, no CI, no `Cargo.lock`, hard-coded paths to `/home/phoenix/...`.  Never quantizes vectors to ternary. | Not usable as-is; plausible small-dataset demo if rewritten. |
| `ternary-control` | Rust | Position-form PID, bang-bang hysteresis, FSM, first-order plant sim (`src/lib.rs:59-370`) | CI, ~24 tests.  `StabilityAnalysis::is_stable` checks `|v| < tol` instead of error-vs-setpoint, contradicting README example. | More educational than production. |
| `ternary-thermostat` | Rust | Ternary thermostat (`-1/0/+1` cool/idle/heat) with PID and multi-zone diffusion (`src/lib.rs`) | CI, 9 tests.  `src/main.rs` is "Hello, world!"; README uses invalid named-arg syntax; `regulate()` lacks real hysteresis. | Skeleton only, not an HVAC controller. |
| `ternary-rhythm` | Rust | Euclidean rhythms, meter, syncopation, swing, genetic evolver on ternary patterns | CI (triggers on `main`, repo branch is `master`), ~60 tests.  Optional SIMD dependency path is broken; genre rules are arbitrary. | Toy music-pattern utility. |
| `ternary-hamiltonian` | Rust | `Z₃` modular arithmetic phase-space toy with symplectic Euler / Störmer-Verlet steps (`src/lib.rs:32-279`) | CI, 30+ tests.  README API does not match code; integrator ignores the `Hamiltonian` struct. | Educational/demo only. |
| `ternary-logic` | Rust | Kleene K3, Łukasiewicz L3, Bochvar B3, Gödel-Dummett G3 truth tables, entailment, simple formula AST | CI, ~20 tests.  Non-ASCII `GödelDummett` variant name; modal operators are one-place truth-functions, not Kripke semantics. | Reference/educational three-valued logic crate. |
| `ternary-cuda-kernels` | Rust + CUDA | PTX-embedding host + CUDA kernels for ternary jam session, matmul, harmony reduction | CI, Rust tests for packing.  No GPU launch binding; matmul kernel does not implement advertised XNOR+popcount; `load_ptx()` loads the same file for all three kernels. | Incomplete tech demo. |
| `ternary-benchmark` | Rust | Claims to benchmark FP32 vs ternary kernels, evolutionary games, Lotka-Volterra ecology | CI configured but crate is almost certainly unbuildable: `lib.rs` never declares modules, `TernaryChoice`/`play_iterated` are used but not defined, two incompatible `BenchmarkResult` types. | Broken scaffold. |
| `ternary-fleet` | None | Umbrella repo with 21 empty sub-directories (`ternary-conv`, `ternary-svm`, etc.) and only `CROSS-POLLINATION.md` | Nothing compiles or runs. | Not a product. |
| `ternary-fleet-integration` | Rust | Vote aggregator, health report, JSON dashboard emitter, toy Axum relay (`src/ternary_aggregator.rs`, `src/bin/dash_relay.rs`) | Tests but no CI/README.  No actual integration tests; conservation thresholds in `CROSS-POLLINATION.md` are not enforced. | Demo dashboard backend only. |
| `conservation-cli` | Rust | CLI for `theory`, `prove`, `bench`, `shannon`, `haar`; computes chain rule and MC cancellation (`src/conservation.rs:253-285`) | Unit tests, no CI.  `bench` hard-codes `/home/phoenix/repos/conservation-languages/`; `prove` runs MC once and discards the result. | Toy entropy/demo tool. |
| `conservation-law` | Rust | Generic "conservation law framework": component sums, invariant detection, flux network, thermodynamics analogies (`src/law.rs`, `src/invariants.rs`) | CI, ~50 tests.  Users must supply values that already sum to `C`; thermodynamics module hard-codes `entropy += 0.1 dt`. | Lightweight constraint-checking vocabulary, not a physics engine. |
| `conservation-law-rs` | Rust | Lagrangian/Hamiltonian mechanics toy, symplectic integrator, Noether symmetry labels, fleet budget circuit breaker | CI, proptest.  README example files are missing; `TimeTranslationSymmetry` zeroes velocity; `CAPABILITY.toml` lists exports not in code. | Small educational mechanics library; fleet mapping is analogy. |
| `conservation-api` | Python/FastAPI | Spectral graph analysis: Laplacian eigenvalues, Fiedler value, eigenvector centrality (`main.py:137-190`) | No tests/CI.  `py_compile` passes.  Reuses Laplacian eigenvectors for adjacency centrality by accident; sparse inputs densified. | Usable toy spectral-graph calculator, not a router/SVM/PID. |
| `conservation-action` | GitHub Action | Composite action that calls `npx superinstance-mcp` tool `conservation_check` (`action.yml:24-35`) | No tests.  `fleet-api-url` input is declared but ignored; relies on unpublished MCP package. | Not verifiable; outsourcing a CI gate to an external tool. |
| `conservation-explorer` | HTML/JS | One-page visualization of `γ+η=C` and `δ(n)` with sliders and hard-coded benchmark bars | No tests/CI.  Entropy is faked to hit `log₂(3)`. | Decorative demo page. |
| `conservation-protocol` | Rust | Graph-Laplacian gossip: agents broadcast Laplacian rows, compute algebraic connectivity (`src/laplacian.rs`, `src/gossip.rs`) | CI, tests.  No real networking; `detect_negative_eigenvalues()` only checks diagonal entries; conservation check is just `gamma + eta == C`. | Toy consensus/demo library. |
| `conservation-checker` | Rust | Runtime tracker that checks a scalar does not drop below baseline minus tolerance (`src/lib.rs`) | CI, 53+ tests.  Panics on unknown names; no timestamps/async/export. | Usable single-process invariant monitor. |
| `native-conservation-core` | C + Python | Ring buffer, ternary dot/conv/matmul, Haar transform, CUDA kernel stub (`src/conservation_core.c`, `src/ternary_alu.c`) | `make bench`/`make alu` build and run; `make cuda` needs `nvcc`.  Python bindings need a `.so` the Makefile does not build.  Ring-buffer benchmark ignores failed pushes. | Usable SPSC ring buffer and ternary ALU primitives, but rough packaging. |
| `noether-bridge` | Rust | Registers symmetry names with arbitrary `gamma_coeff`, checks `gamma + h == C` | CI, 43 tests.  `C` can be set to the mean of inputs, making conservation circular. | Tautological label checker. |
| `conservation-docs` | Markdown/LaTeX | ~100+ research essays, two paper drafts, claims of 9-language SDK/204 tests | No code, no build, no tests.  Internal documents contradict the conservation hypothesis. | Readable prose only, not software. |

### Key observations across the applied repos

* The **ternary aspect** is real in many repos (actual `{-1,0,+1}` enums, arithmetic, packing, SVM features).
* The **conservation-law aspect** is almost entirely a **naming/metaphor layer**.  With rare exceptions (`conservation-cli`'s chain-rule calculator, `native-conservation-core`'s entropy functions), the code does not compute, enforce, or verify `γ + η = C`.
* **Doc/code drift is common**: `PLUG_AND_PLAY.md`, `README.md`, and `Cargo.toml` often describe APIs, features, or example files that do not exist.
* **Many repos are single-commit scaffolds** with empty directories, missing modules, or hard-coded host paths.

## 4. Recommendation for purplepincher

**Is this cluster a fork candidate as a practical tool?**  Mostly no.  The cluster reads as SuperInstance's own research sketchbook: a large number of small, thematically linked experiments, not a mature product line.

* **Do not fork for the conservation-law theory.**  The theory is a repackaged Shannon chain rule plus an incorrect `δ(n)` cancellation formula, supported by loose Monte Carlo tolerances and a lot of prose.  It is not a rigorous new physical or mathematical law.
* **Do not fork the umbrella/conservation infrastructure repos.**  `ternary-fleet`, `conservation-action`, `conservation-api`, `conservation-explorer`, `conservation-law`, `noether-bridge`, etc. are either empty, thin wrappers, or visualization toys.
* **Possible narrow exceptions if you need a starting point:**
  * `ternary-pid` — closest to a genuinely usable tool.  It is a real three-state actuator PID with tests and CI.  Fork it only if you are willing to add sample-time scaling, output clamping, and fix the cascade wiring and documentation.
  * `ternary-svm` — a minimal but working linear SVM for ternary features.  Usable for small demos or embedded classification after adding model serialization and fixing the silent-untrained behavior.
  * `ternary-entropy` and `ternary-types` — small, clean primitive libraries that do what they say on the tin.
  * `ternary-arithmetic` / `ternary-fleet-packing` (Python side) / `native-conservation-core` — usable low-level ternary encoding/ALU building blocks, but expect to do integration work.

**Verdict:** Treat the entire ternary/conservation cluster as an **interesting, highly prolific research sketchbook** rather than a source of production libraries.  If anything is worth forking, it is `ternary-pid` as a three-state controller seed — but fork it for the PID, not for the conservation-law framing.
