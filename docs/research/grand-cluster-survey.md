# `grand-*` cluster survey

Follow-up deep-dive on the 42-repo `grand-*` cluster. Prior research touched only two
members — `grand-pattern-mono` (noted as "15 KB Rust, CI green, scalar 'vibe' graph
with JEPA predictor" in `unexplored-clusters-survey.md`) and `grand-pattern-embedded`
(one of the mission's original 11 user-named repos). The other ~40 were unexamined.
This pass enumerates all 42, clones 29 of them, reads actual source (not READMEs),
pulls real GitHub Actions logs via `gh api`, and explicitly hunts for a
`nexus-runtime`-scale hidden outlier.

**Environment constraint, same as every prior survey:** no `cargo`/`rustc`, no `pip`,
no `pytest`, no `numpy` locally. All "tests pass" claims below are verified against
downloaded GitHub Actions logs (`gh api repos/SuperInstance/<repo>/actions/...`),
not README badges and not local runs.

All clones live in `/tmp/opencode/grand-work/clones/` (shallow, `--depth 5–20`).

---

## 0. Headline finding

**No `nexus-runtime`-scale outlier. Stated explicitly and often, as the mission
requires.** The entire 42-repo cluster is ~28 K LOC of real source (measured across
29 cloned repos; the 13 uncloned are all <1 KB–1.5 KB stubs/ports and add <4 K LOC).
For scale: `nexus-runtime` alone was ~190 K LOC of Python + 7 K LOC of C. The whole
`grand-*` family is roughly **15 % the size of that one repo**.

What the cluster actually is: a single R&D sketchbook around one speculative idea —
"Grand Pattern" cellular-graph **vibe diffusion** — implemented polyglot (15+
languages) plus a set of small Rust component crates (networking, persistence,
embedded, GPU, adversarial, simulation, benchmarks). It is internally coherent and,
unusually for this mission, several of its READMEs **understate / honestly disclose**
what the code does rather than overselling it. The CI picture, however, reproduces the
mission's load-bearing "green badge lies" finding almost textbook-perfectly: **8 of
29 cloned repos have sham CI** (5 run `echo "No CI configured"`, 3 run `|| true`),
including repos whose green badges advertise a working test suite that has never
actually executed on GitHub Actions.

The one genuinely different artifact is `grand-synthesis` — a multi-LLM architecture
competition (Claude Opus / DeepSeek / GLM / Seed-pro / Kimi each submit a design for a
"Metronome Architecture"). It is intellectually the densest repo in the cluster
(publication-quality 33 KB `PAPER.md`, 30–41 KB architecture submissions per model,
cross-model critiques, a unified reference implementation), but it is a **research
notebook, not a runnable system**, and its central empirical claims are unverified:
CI runs `pytest || true`, the latest log literally prints `no tests ran`, and the one
validation script that would back the paper's drift/convergence claims is never
executed by CI.

---

## 1. The complete repo list (42, confirmed authoritative)

```
grand-pattern-abi      grand-pattern-c        grand-pattern-cli      grand-pattern-cuda
grand-pattern-design   grand-pattern-embedded grand-pattern-experiments  grand-pattern-ffi
grand-pattern-flux     grand-pattern-fortran  grand-pattern-go       grand-pattern-gpu
grand-pattern-integration  grand-pattern-java grand-pattern-kimi     grand-pattern-kit
grand-pattern-mojo     grand-pattern-mono     grand-pattern-mono-go  grand-pattern-mono-py
grand-pattern-mono-ts  grand-pattern-net      grand-pattern-opencl   grand-pattern-ptx
grand-pattern-py       grand-pattern-rs       grand-pattern-sim      grand-pattern-simd
grand-pattern-store    grand-pattern-swift    grand-pattern-topology grand-pattern-ts
grand-pattern-venue    grand-pattern-wasm     grand-pattern-zig      grand-pattern-adversarial
grand-pattern-bench    grand-pattern-bench-v2 grand-pattern-chapel   grand-pattern-claude
grand-synthesis
```

Verified via `gh repo list SuperInstance --limit 4200 --json name --jq '.[].name' | grep -i '^grand'`
— exactly 42. **41 are `grand-pattern-*`; the single outlier `grand-synthesis` is the
only non-`pattern` name.** So "grand" is not a thematic prefix grouping unrelated
projects; it groups exactly one polyglot R&D project ("Grand Pattern") plus one
adjoining research notebook (`grand-synthesis`).

### Bulk-push signal (commit-history triage)

`gh api repos/SuperInstance/<repo>/commits` shows **all 41 `grand-pattern-*` repos
were pushed on 2026-06-08** (within the same hour), each receiving a coordinated
scaffolding commit ("ci: add CI workflow", "feat: add AGENT.md ensign identity",
"feat: add memory/JOURNAL.md for ensign duty log"). The real per-repo work predates
that — e.g. `grand-pattern-mono`'s initial implementation + clippy fixes land on
2026-05-30, then the 2026-06-08 burst adds the fleet-"ensign" roleplay boilerplate
(`AGENT.md` + `memory/JOURNAL.md`) and a templated CI workflow. Commit counts are low
everywhere (4–10 for the `pattern-*` set; 14 for `grand-synthesis`). This is the
classic sketchbook pattern: real but small bursts of work, then a coordinated
templated "fleet onboarding" pass.

### Repo-size reality check

GitHub's reported `size` field is wildly misleading here and **should not be used to
triage**:

| Repo | GitHub size | Actual source LOC | Real cause of inflation |
|---|---|---|---|
| `grand-pattern-mono` | 15,372 KB | **598 LOC / 16 KB `src/lib.rs`** | git object bloat (prior survey's "15 KB" was correct about source) |
| `grand-pattern-abi` | 13,060 KB | 595 LOC / 15 KB `src/lib.rs` | git object bloat |
| `grand-pattern-mono-ts` | 11,453 KB | 486 LOC | **11 MB of `node_modules/` committed to the repo** |
| `grand-pattern-gpu` | 9,368 KB | 1,092 LOC | git bloat + a 3.7 KB precompiled SPIR-V blob (`comp.spv`) |
| `grand-pattern-adversarial` | 8,039 KB | 1,304 LOC | git object bloat |
| `grand-synthesis` | 197 KB | 3,849 LOC | mostly markdown docs + one 37 KB Python validation script |

The actual source-size ranking (cloned subset) is:

```
grand-synthesis            3849 LOC   grand-pattern-store      960
grand-pattern-go           1551 LOC   grand-pattern-cli         900
grand-pattern-adversarial  1304 LOC   grand-pattern-swift       835
grand-pattern-net          1289 LOC   grand-pattern-topology    771
grand-pattern-bench-v2     1177 LOC   grand-pattern-ffi         766
grand-pattern-embedded     1174 LOC   grand-pattern-mono-go     738
grand-pattern-gpu          1092 LOC   grand-pattern-simd        706
grand-pattern-py           1082 LOC   grand-pattern-venue       671
grand-pattern-core         1051 LOC   grand-pattern-mono-py     636
grand-pattern-zig          1042 LOC   grand-pattern-mono        598
grand-pattern-experiments  1017 LOC   grand-pattern-abi         595
grand-pattern-java          989 LOC   grand-pattern-c           574
                                     ... (all others ≤ 558 LOC)
```

Median is ~700 LOC/repo. Nothing in the uncloned tail (sizes 8–78 KB on GitHub) could
plausibly hide >2 K LOC.

---

## 2. What "Grand Pattern" actually is

Read from `grand-pattern-design/DESIGN-IDEAS.md` (42 KB ideation doc, dated
2026-05-29) and confirmed against the Rust source.

- A **CellGraph** of *rooms* connected by weighted edges.
- Each room carries a **vibe** (a scalar `f64` in the "mono" design, or a
  `D`-dimensional vector with `D ∈ {8, 13, 21}` in the earlier "Fibonacci Dual-
  Direction" design — the Fibonacci constraint is the only thing "Fibonacci" about
  the architecture; it refers to Penrose/Mandelbrot decomposition of embedding
  dimension, not to the recurrence per se).
- Each room runs a **JEPA predictor** — but in a notable honest-disclosure pattern,
  every README in the cluster explicitly says what JEPA means here: *"Weighted
  moving average predictor. Weights learn by inverse-error reinforcement"* (`mono`),
  *"Simple JEPA: exponential-weighted moving average of history"* (`adversarial`,
  `lib.rs:80`). **This is not a neural joint-embedding predictive architecture.** It
  is an EWMA. The name is borrowed; the READMEs admit it.
- Rooms **diffuse** vibe along edges (one explicit heat-equation-style update step,
  `deltas[from] += weight * diff`, see `mono/src/lib.rs:176-196`), **gossip** by
  emitting `Murmur` structs (vibe + surprise + TTL), and **decay** TTLs.

### The two-generation story (verified, not assumed)

The cluster has an internal timeline that the repos themselves document honestly:

1. **v1 — "Fibonacci Dual-Direction Architecture"** (vector vibes, dual `z_in`/`z_out`
   embeddings). This is what the polyglot ports (`-c`, `-go`, `-zig`, `-java`,
   `-swift`, `-py`, `-rs`, `-ts`, plus the GPU/embedded/net/store components)
   implement. README description string is literally `"Grand Pattern Fibonacci
   Dual-Direction Architecture - <lang> Implementation"` across all of them.
2. **The conservation law is broken in v1.** `grand-pattern-bench` (cross-language
   conservation-law benchmark) README states the finding plainly: *"This benchmark
   tests whether the double-entry bookkeeping conservation law (`|Z_in| ≈ |Z_out|`)
   holds under stress."* `grand-pattern-bench-v2` (1177 LOC) is the follow-up and its
   README gives the honest root-cause: *"100 % violation rate across 1.8 M room-tick
   pairs. Raw perceptions have higher magnitude than averaged predictions, causing
   linear drift."* It proposes three fixes (normalized bookkeeping / matched-
   magnitude / ratio conservation).
3. **v2 — "mono"** (`grand-pattern-mono`, `-mono-go`, `-mono-py`, `-mono-ts`) is the
   *correction*. Its README is explicit: *"The CORRECTED Grand Pattern — vibe is
   mono-dimensional… This makes conservation trivially hold: total vibe = sum of all
   room vibes."* `mono/src/lib.rs` test 11 (`test_conservation`) asserts the total is
   preserved to `1e-10` across 100 diffusion steps.

This is a real, small R&D arc — build thing, prove thing is broken, ship corrected
thing — and it is documented with the negative results intact. That is the **inverse
of the recurring mission finding** for this cluster specifically.

---

## 3. CI reality — the load-bearing finding for `grand-*`

I classified the `.github/workflows/ci.yml` of every cloned repo and pulled the
latest *push-event* run for each (Dependabot/`dynamic`-event runs excluded — for
`grand-synthesis` these spam the history and are not the code's CI).

### 3.1 Sham CI (8 repos — green badge, no actual testing)

| Repo | CI workflow `run:` line | What actually happens |
|---|---|---|
| `grand-pattern-c` | `echo "No CI configured — customize per project requirements"` | 43 assertions in `test/test_all.c` **never compiled, never run** |
| `grand-pattern-java` | same `echo` template | **25 `@Test` methods in `GrandPatternTest.java` never run** |
| `grand-pattern-swift` | same `echo` template | **23 `func test…` in `Tests/GrandPatternTests/` never run** |
| `grand-pattern-zig` | same `echo` template | **35 `test "…"` blocks in `src/root.zig` never run** |
| `grand-pattern-design` | same `echo` template | docs-only repo; no tests to run, correctly |
| `grand-pattern-py` | `pytest \|\| true` | runs but `|| true` masks any failure |
| `grand-pattern-mono-py` | `pytest \|\| true` | same |
| `grand-synthesis` | `pytest \|\| python -m pytest \|\| true` | downloaded log literally prints `no tests ran in 0.01s`; the actual `validation/metronome_unified.py` simulation that would substantiate the paper's claims is **never invoked by CI** |

The four language ports (C/Java/Swift/Zig) are the most misleading case: each has a
real, substantive test file (verified by grep: 43/25/23/35 test entries) and a green
badge, but CI runs `echo` instead of `make test`/`mvn test`/`swift test`/`zig build
test`. Anyone trusting the badge would believe the Java/Swift/Zig ports are
verified; they are not.

### 3.2 Real CI, tests pass clean (5 repos)

| Repo | `run:` | Verified test count from logs |
|---|---|---|
| `grand-pattern-mono` | `cargo check; cargo test; cargo clippy -- -D warnings` | **27 passed** (run 27132963561) |
| `grand-pattern-abi` | same | **16 passed** |
| `grand-pattern-adversarial` | same | **36 passed** (README claims 15 — understated!) |
| `grand-pattern-experiments` | same | **17 passed** |
| `grand-pattern-mono-ts` | `npm ci; npm test --if-present` | **29 passed** (jest). Caveat: the repo commits 11 MB of `node_modules/` — bad practice, but the tests are real and run. |

### 3.3 Real CI, tests pass but clippy fails the job (6 repos)

Identical failure mode in all six: `cargo test` passes, then `cargo clippy -- -D
warnings` exits 101 on a cosmetic lint. Same pattern as `constraint-theory-core` in
the prior survey.

| Repo | Tests passed (from log) | Clippy failure |
|---|---|---|
| `grand-pattern-core` | **43** | `clippy::manual_flatten` (`src/graph.rs:64,122` — use `.flatten()` instead of `if let Some`) |
| `grand-pattern-gpu` | **31** | clippy warning (exit 101 after tests pass) |
| `grand-pattern-embedded` | **31** | clippy warning (exit 101 after tests pass) |
| `grand-pattern-net` | **30** (2 + 30 split across crates) | clippy warning |
| `grand-pattern-store` | **26** | clippy warning |
| `grand-pattern-rs` | (not pulled individually; same template) | clippy warning |

### 3.4 Real CI in non-Rust languages (2 repos)

| Repo | `run:` | Log evidence |
|---|---|---|
| `grand-pattern-go` | `go test ./...` | Real: `ok github.com/SuperInstance/grand-pattern-go 0.002s` across **7 packages** (graph, jepa, murmur, room, signal, tick + root). Legitimate. |
| `grand-pattern-mono-go` | `go test ./...` | same shape |

### 3.5 Total verified passing tests

Summing §3.2 and §3.3: **27 + 16 + 36 + 17 + 29 + 43 + 31 + 31 + 30 + 26 = 286 real
passing tests** across 10 repos, plus the 7-package `go test` pass for `-go`/`-mono-go`.
This is real, but distributed across 12+ tiny crates averaging <30 tests each —
classroom-scale, not library-scale.

---

## 4. Sub-theme verdicts (one paragraph each, with concrete evidence)

### 4.1 The flagship: `grand-pattern-mono` (v2 "corrected" architecture)

Real, clean, small. 598 LOC of single-file Rust (`src/lib.rs`), zero dependencies
(`Cargo.toml` has no `[dependencies]`), 27 tests that pass under strict CI (`cargo
check` + `cargo test` + `cargo clippy -- -D warnings`, run 27132963561, conclusion
`success`). Code is a straightforward weighted-average "JEPA" + heat-equation graph
diffusion + TTL-gated gossip. README is honest to a fault — it explicitly says
"JEPA is a weighted reading," "Weighted moving average predictor," and "vibe is ONE
number (f64)." Conservation of total vibe is asserted to `1e-10` (test 11). The only
"JEPA"-shaped claim that overreaches is the name itself: this is an EWMA, not a
joint-embedding predictive architecture. **Verdict: the cleanest crate in the
cluster; the prior survey's positive read is upheld and if anything understated how
honest the README is.**

### 4.2 The "real systems" components: `-net`, `-store`, `-embedded`

These are the three most likely to be useful to a third party, and they are real
small systems crates, not stubs — verified by grep, not README:

- **`grand-pattern-net`** (1,289 LOC, 11 files): uses `std::net::UdpSocket` and
  `TcpListener` for real (`src/gossip.rs:1`, `src/tcp_transport.rs:1` — `UdpSocket::
  bind("0.0.0.0:{port}")`, `socket.recv_from`, `socket.send_to`). 30 tests pass;
  clippy fails the job. Implements UDP gossip + TCP framed transport + peer
  discovery + a 32-byte binary murmur wire format.
- **`grand-pattern-store`** (960 LOC, 7 files): does real file I/O. `src/persistence.rs`
  opens `fs::File::create(path)` and writes a binary format with `MAGIC` + `VERSION`
  headers + little-endian room/edge records; also JSON, CSV, and an append-only tick
  log. 26 tests pass; clippy fails.
- **`grand-pattern-embedded`** (1,174 LOC, 9 files): genuinely `#![no_std]`
  (`src/lib.rs:1` — verified), zero heap allocation, fixed-size arrays (≤32 rooms,
  ≤64 edges, ≤16 JEPA readings), 32-byte murmur messages. Feature flags `esp32` /
  `arduino`. 31 tests pass (in `tests/embedded.rs`, std-host); clippy fails.

**Verdict: all three are competent small systems crates that do what they claim, but
each is tightly coupled to the "vibe diffusion" model — they are not general-purpose
networking/persistence/embedded libraries.**

### 4.3 The polyglot-ports sub-cluster (15 repos)

`-c, -go, -zig, -java, -swift, -py, -rs, -ts, -mono-go, -mono-py, -mono-ts, -chapel,
-fortran, -mojo, -flux` (+ the GPU-on-C-like `-cuda, -opencl, -ptx`). Each implements
the same `Room`/`Vibe`/`JEPA`/`Murmur`/`CellGraph`/`tick`/`diffuse` types. Spot-
checked three:

- **`-go`** (1,551 LOC, 16 files): real `go test ./...` CI, 7 packages all `ok`.
  Multiple files per concept (`graph/`, `jepa/`, `murmur/`, `room/`, `signal/`,
  `tick/`, `vibe/`). The most substantively-organized port.
- **`-c`** (574 LOC + tests): hand-rolled `calloc`/`realloc` graph, includes a
  `<grand_pattern.h>` and a `gp_vector_db_t` (the v1 vector design with `db_in` /
  `db_out` entries — confirms the "Dual-Direction" naming). 175-line `test_all.c`
  with 43 `assert`s. CI is the `echo` sham, so none of this runs in CI.
- **`-zig`** (1,042 LOC, 9 files): same module split as `-c`, 35 `test` blocks in
  `src/root.zig`. CI is the `echo` sham.

**Verdict: a legitimate polyglot-comparison exercise — same algorithm ported to 15
languages, ranging from 430 LOC (`-rs`) to 1.5 K LOC (`-go`). Nothing stands alone as
a library; their value is the comparison itself, which is a research artifact, not a
fork target. The C/Java/Swift/Zig ports' green CI badges are misleading (§3.1).**

### 4.4 The empirical-research sub-cluster: `-experiments`, `-bench`, `-bench-v2`, `-adversarial`, `-sim`, `-venue`, `-topology`, `-simd`

This is the most intellectually honest corner of the cluster.

- **`-experiments`** (1,017 LOC + 4.8 KB `CONCLUSIONS.md`): runs 5 experiments
  (diffusion convergence, surprise cascade, conservation-under-stress, topology
  comparison, dissolution threshold) and writes up results with actual numbers:
  *"10-room ring converged to within 0.01 by tick ~460,"* *"Star: tick 35.
  Chain: tick 159. Mesh: tick 504,"* *"Surprise peaks: A=0.635, B=0.573, C=0.524,
  D=0.509, E=0.508."* Crucially, it reports a **negative result** honestly:
  *"Dissolution didn't trigger under our test conditions"* and *"⚠️ Our
  'conservation' metric measures tick-to-tick stability, not strict conservation of
  total vibe mass."* 17 tests pass.
- **`-bench` / `-bench-v2`**: the conservation-law investigation described in §2.
  `-bench` is the negative-result paper (v1 conservation broken); `-bench-v2`
  proposes three fixes (normalized / matched-magnitude / ratio conservation) and
  runs 18 configurations × 10 rooms × 10 K ticks. **No `CONCLUSIONS.md` is written
  out for `-bench-v2`** — the results live only in `cargo run` stdout, which is not
  captured by CI. So which fix "won" is not recorded; one has to run it. (Couldn't:
  no toolchain locally.)
- **`-adversarial`** (1,304 LOC, the biggest single Rust crate): 8 attack strategies
  (ConstantMax/Min, Oscillator, RandomNoise, Amplifier, Mimic, Contrarian, Collusion)
  against honest-fleet convergence, 5 metrics. **36 tests pass** (README claims 15 —
  understated, like `mono`). CI strict and green. This is the most "production-shaped"
  crate: a self-contained fuzz/adversarial harness, zero deps, real test battery.
- **`-venue`** (671 LOC): "venues are agents" — rooms develop a 7-dimensional
  personality (per-event-kind weighted JEPA memory) and a voice prompt. Claims 25
  tests; CI is real-template but I did not pull the log individually.
- **`-sim`**: "Full fleet simulation — 20 venues, 5 scenarios." README is the most
  over-claim-y in the cluster (links to `tminus-dispatcher`, `fleet-bridge`,
  `symphony-runtime`, etc. as if integrated — but those are separate repos and
  `sim` has no `Cargo.toml` `[dependencies]` pointing at them, i.e. the integration
  is aspirational, not wired).
- **`-topology`** (771 LOC) and **`-simd`** (706 LOC): topology sweep (star/chain/
  mesh/random) and cache-friendly SoA SIMD diffusion. Small, focused.

**Verdict: this sub-cluster is the intellectual core and `-experiments`/`-bench` are
genuinely honest research notebooks — a real credit to the cluster. `-adversarial` is
the single most forkable crate on pure-code-quality grounds (zero deps, 36 passing
tests, strict green CI, self-contained). But every repo is small and assumes the
"vibe" model.**

### 4.5 The keystone/glue: `-core`, `-abi`, `-ffi`, `-kit`, `-integration`, `-cli`

- **`-core`** (1,051 LOC, 9 files): the "unified core" — `graph.rs`, `topology.rs`,
  `venue.rs`, `jepa.rs`, `room.rs`, `murmur.rs`, `conservation.rs`. **43 tests pass**,
  clippy fails the job on `manual_flatten`. This is the v1 (vector) core, not mono.
- **`-abi`** (595 LOC + 2.8 KB C header `include/grand_pattern.h`): a `cdylib`/`staticlib`
  C ABI wrapper ("polyglot keystone"). 16 tests pass, CI green.
- **`-ffi`** (766 LOC): C FFI bindings to call the pattern from C/C++/Python/etc.
- **`-kit`** (668 LOC): "modular primitives glued into a toolkit."
- **`-integration`** (553 LOC): cross-component integration tests.
- **`-cli`** (900 LOC): clap-style CLI ("create, run, visualize"). Single-file.

**Verdict: real but small plumbing. `-abi` is the most credible as a standalone
artifact (a polyglot C-ABI surface with a header and tests), but its payload is the
"vibe" model, which limits standalone value.**

### 4.6 The outlier: `grand-synthesis`

The only non-`pattern` repo. 14 commits (the most in the cluster), 197 KB, Python.
**It is not a code project — it is a multi-model architecture competition.** Structure:

```
grand-synthesis/
├── COMPETITION.md            # 6 KB brief: design a "Metronome Architecture"
├── PAPER.md                  # 33 KB publication-quality synthesis paper
├── submissions/
│   ├── claude-opus/{ARCHITECTURE.md (41 KB), metronome_simulation.py (21 KB), ...}
│   ├── deepseek/{ARCHITECTURE.md (33 KB), metronome_proof.py (32 KB), ...}
│   ├── glm/{ARCHITECTURE.md (30 KB), metronome_implementation.py (24 KB), ...}
│   ├── seed-pro/{ARCHITECTURE.md (38 KB), metronome_lifecycle.py (30 KB),
│   │             lifecycle_events.json (340 KB), ...}
│   └── (kimi referenced but dir absent in clone)
├── reviews/round1-critique.md    # 23 KB cross-model critiques
├── synthesis/UNIFIED-DESIGN.md   # 37 KB merged design
└── validation/metronome_unified.py  # 37 KB "Round 6 Forgemaster" reference impl
```

The premise (from `COMPETITION.md`): N independent agents stay in sync not by
broadcasting timestamps but by each locally simulating the *same* theoretical
metronome — agreeing on a constraint tuple `θ = (T, φ₀, ε, δ)`. The unified
implementation layers Pythagorean fraction arithmetic (zero clock drift), Laman-
topology / Henneberg type-I construction with pebble-game verification, spectral-gap-
derived gossip coupling `α* = 2/(λ₂ + λ_N)`, tensor-MIDI encoding, deadband
filtering, sunset/inheritance handoff.

`PAPER.md` is genuinely well-written academic prose (abstract, Nash-equilibrium
proof sketch, spectral derivation, claim of N=20 agents converging to δ=1/16 within
500 ticks over 5 % packet loss). It is the densest single intellectual artifact in
the cluster.

**But the verification story is broken, and this is the load-bearing caveat:**

- `.github/workflows/ci.yml` runs `pytest || python -m pytest || true`. The latest
  *push* run on master (run 26304804925, 2026-05-22) prints, verbatim:
  `============================ no tests ran in 0.01s =============================`.
  The matrix of `python-version: ['3.10', '3.11', '3.12']` all "succeed" — three
  green checkmarks that mean nothing.
- The actual substantiation script, `validation/metronome_unified.py` (which exits
  non-zero if its drift/convergence/Laman/MIDI battery fails), is **never invoked by
  CI**. Its final `print("✅ UNIFIED SIMULATION PASSED" if all_ok …)` has never been
  observed by an automated runner. (Five subsequent Dependabot `dynamic`-event runs
  on master all show `failure`, but those are pip-update PRs and not the code CI.)
- I could not run the script locally: no `numpy` in this environment (the task's
  standing constraint). Reading `validation/metronome_unified.py:1-60` confirms it
  requires numpy and implements the Laman/spectral machinery it claims, but whether
  its success criteria actually pass for the claimed parameters is **unverified by
  anything automated**.

**Verdict: `grand-synthesis` is the most interesting *artifact* in the cluster and
the only one that made me pause during the outlier hunt. But it is a research
notebook, not a runnable system, and its central empirical claims are unverified —
the green CI badge is a `|| true` artifact.** If `purplepincher` wanted to adopt the
Metronome Architecture, the work-item would be "fork `grand-synthesis`, wire
`validation/metronome_unified.py` into a real `pytest`/`python -m
metronome_unified` CI step, and find out whether the paper's δ=1/16 claim actually
holds." That is a research bet, not a fork of working code.

---

## 5. Outlier hunt — explicit result

The task asked specifically to hunt for a `nexus-runtime`- / `plato-semantic-search`- /
`constraint-theory-core`-scale hidden outlier by not skipping the unglamorous repos.
I sampled the unglamorous ones deliberately: `-bench`, `-bench-v2`, `-store`,
 `-simd`, `-topology`, `-venue`, `-ffi`, `-kit`, `-integration`, `-design`, plus all
four sham-CI language ports. **No outlier found.** Concrete reasoning:

- **By scale:** the largest single-crate source in the cluster is
  `grand-pattern-adversarial` at 1,304 LOC. `nexus-runtime` was ~190 K LOC of Python
  + 7 K of C. The largest *repo* (`grand-synthesis`, 3,849 LOC) is ~50× smaller than
  `nexus-runtime` and is mostly markdown.
- **By working-system substance:** no repo in the cluster contains a wire protocol
  implementation, a bytecode VM, a real database, a hardware-firmware cross-compile,
  or any external integration that lives in *this* cluster. (`-net` has real
  sockets but is 1.3 K LOC of straightforward `std::net`; `-embedded` is `no_std` but
  its transport layer is documented stubs — "Wi-Fi/BLE/UART/I2C (compile for any
  target, implement on hardware).")
- **By CI substance:** the only repos with strict green CI are `mono`, `abi`,
  `adversarial`, `experiments`, `mono-ts`, `-go`, `-mono-go` — all averaging ~25
  tests on <1.5 K LOC. `constraint-theory-core` alone had 262 passing tests on 12.7 K
  LOC with zero runtime deps; nothing here is in that class.
- **The single "intellectually different" artifact is `grand-synthesis`**, but its
  substance is markdown + an unverified Python script. It is the closest thing to an
  outlier in *ideas*, but not in working, tested code.

A well-verified "nothing here beats what's already known" is the valid finding, as
the mission preamble promised.

---

## 6. Fork-worthy candidates for `purplepincher`

Honest list, ranked. None rise to the level of `constraint-theory-core` or
`holodeck-rust` from the prior survey.

1. **`grand-pattern-adversarial`** — *if and only if* a small, zero-dependency Rust
   harness for fuzzing graph-diffusion / weighted-predictor convergence against
   adversarial inputs is useful. Evidence: 1,304 LOC, 36 passing tests under strict
   green CI, no `[dependencies]`. Caveat: the "fleet" it models is the speculative
   "vibe" architecture, so the value is the harness pattern, not the modeled system.
2. **`grand-pattern-net`** — *conditional on* adopting the vibe-diffusion model.
   Evidence: real `std::net` UDP gossip + TCP framed transport + peer discovery,
   30 passing tests (clippy fails the job on a cosmetic lint, easily fixed).
   1,289 LOC. Caveat: tightly coupled to `Murmur`/`CellGraph`; not a general gossip
   library.
3. **`grand-synthesis`** — *as a research bet, not a fork.* The Metronome
   Architecture is a genuinely novel framing (constraint-tuple agreement vs.
   timestamp broadcast) and the `validation/metronome_unified.py` script is a
   substantive 37 KB reference implementation. But the work item is "make CI real
   and find out if the claims hold," not "drop in working code." Caveat: heavily
   themed ("Cocapn Fleet / Forgemaster ⚒️"); would need de-theming.

Everything else is either too small, too coupled to the vibe model, has sham CI, or
is a polyglot-research comparison.

---

## 7. Mission-pattern observations

For the catalog. The `grand-*` cluster is **atypical** in two ways that are worth
recording against the mission's load-bearing "READMEs oversell" finding:

1. **Several READMEs here *understate* on purpose.** `grand-pattern-mono` explicitly
   downgrades "JEPA" to "weighted reading." `-adversarial` claims 15 tests and
   actually has 36. `-experiments`/`-bench`/`-bench-v2` report negative results
   (`"100 % violation rate,"` `"dissolution didn't trigger"`) and keep them in the
   repo. This is the *opposite* of the pattern seen in `flux-tensor-midi`,
   `zeroclaw-arena`, `Equipment-*`, etc. Treat the cluster as Casey-in-a-careful-mood,
   not Casey-in-an-oversell-mood.
2. **The "green badge lies" pattern, however, reproduces almost perfectly.** 8 of 29
   cloned repos have sham CI — and the sham is concentrated exactly where you'd
   expect it to slip past: the four non-Rust/non-Go/non-TS language ports all use
   the same `echo "No CI configured"` template (so the Java/Swift/Zig/C ports
   *advertise* green badges while their very real test files have never once
   executed on GitHub Actions), and the three Python repos all use `|| true`. The
   Rust repos, by contrast, mostly have real strict CI. **Lesson reaffirmed: trust
   no badge; pull `ci.yml` + the latest push-event run log before believing any
   "tests pass" claim.**

The `node_modules/` committed to `grand-pattern-mono-ts` (11 MB of it) is a minor
data point for the "what gets committed" pattern — first time I've seen a whole
TypeScript toolchain checked into a repo in this mission.

---

## 8. Honest bottom line

The `grand-*` cluster is a coherent, small, mostly-honest polyglot R&D sketchbook
around one speculative idea (cellular vibe diffusion with a weighted-average
"JEPA"). Real source is ~28 K LOC across 42 repos; the flagship is 598 LOC. 286 real
tests pass across 10 repos; 8 repos have sham CI and their green badges lie. There is
**no `nexus-runtime`-scale outlier** — stated explicitly, as the mission requires.
The single most interesting artifact (`grand-synthesis`) is a multi-LLM architecture
competition whose paper-quality claims are unverified by its own (sham) CI. Nothing
here changes the prior survey's fork-candidate leaderboard: `constraint-theory-core`
and `holodeck-rust` remain the strongest candidates from this region of the
SuperInstance catalog.
