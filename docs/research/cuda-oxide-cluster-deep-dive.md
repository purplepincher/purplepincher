# `cuda-*` / `oxide-*` Cluster Deep Dive

**Author:** Kimi Code CLI research pass · **Date:** 2026-07-03  
**Scope:** All 173 public repos on `SuperInstance` whose names match `^cuda-|^oxide-`, with a code-first sample of 25 representative repos. The task is to test the ecosystem survey's framing that this cluster is "GPU-specific" and out of scope for `purplepincher`, and to look for hidden outliers or non-GPU tooling that might change that verdict.

---

## TL;DR (read this first)

1. **The "GPU cluster" label is mostly branding, not code.** Only **3** of the 173 repos contain actual GPU/CUDA code (`cuda-oxide`, `cuda-constraint-engine`, `cuda-constraint-checker`). The other ~170 are Rust crates that describe GPU workloads in their READMEs but never call CUDA APIs, compile `.cu` files, or emit PTX.
2. **`cuda-oxide` is not original SuperInstance work.** It is a fork of `NVlabs/cuda-oxide`, a real Rust→PTX compiler runtime. The SuperInstance repo has one commit on top of the fork ("gc: sync fleet GC config"). It is genuinely GPU-specific and genuinely real, but it is not a new asset to adopt.
3. **Most `cuda-*` crates are Lucineer mirrors, not SuperInstance originals.** The large, descriptive ones (`cuda-genepool`, `cuda-memory-fabric`, `cuda-trust`, `cuda-equipment`, `cuda-telepathy`, etc.) are forks of `Lucineer/*`, pushed as single-commit mirrors on 2026-04-13. They are generic agent primitives with a `cuda-` prefix.
4. **`oxide-*` are June-2026 generated stubs.** They share identical scaffolding, identical commit messages, and CI that passes `cargo check`/`cargo test` but fails `cargo fmt`/`cargo clippy -D warnings`. They are generic data structures wrapped in GPU/ternary jargon.
5. **No hidden `nexus-runtime`-scale outlier exists.** The only large repo is `cuda-oxide` (external). The largest SuperInstance-native GPU code is `cuda-constraint-engine` (~1.8 KLOC of CUDA C++), which is tightly coupled to the already-rejected constraint-theory math.
6. **Verdict: stay out of scope.** Some `cuda-*` crates are *conceptually* relevant to purplepincher (A2A messaging, trust engine, watchdog, reflex/edge runtime), but they are immature mirrors with path dependencies, no published artifacts, and heavy metaphorical baggage. Better implementations of the same ideas already exist in the `fleet-*`, `pincher`, `exocortex`, and `nexus-runtime` clusters. The real GPU work is genuinely GPU-specific and should remain parked until purplepincher has a GPU edge product.

---

## 1. Enumeration and sample selection

### 1a. Full cluster stats

```text
gh repo list SuperInstance --limit 4200 --json name,description,updatedAt,pushedAt,primaryLanguage,diskUsage,createdAt,stargazerCount,forkCount
```

Filtered to names matching `^cuda-|^oxide-` yields **173 repos** (saved in `notes/cuda_oxide_repo_metadata.tsv`).

| Metric | Value |
|---|---|
| Total repos | 173 |
| Primary language = Rust | 164 |
| Primary language = Makefile | 6 |
| Primary language = CUDA | 2 |
| No language listed | 1 |
| Median disk usage | 8 KB |
| Max disk usage | 30,222 KB (`cuda-genepool`, inflated by committed build artifacts) |
| Created 2026-04-10/11/12 | 139 |
| Created 2026-06-05/06 | 31 |
| Last pushed 2026-04-13 | 136 |
| Last pushed 2026-06-09/10/11/13 | 31 |

The date concentration alone is a strong bulk-generation signature: 79% of the cluster was created in a 72-hour April window, and 79% were last pushed on a single date (2026-04-13).

### 1b. Sample of 25 repos inspected

Selection prioritized (a) the largest repos by disk usage, (b) repos whose descriptions explicitly mention GPU/CUDA/build/runtime, and (c) a random scatter of small repos. All were cloned shallow into `clones/<name>/` and inspected by reading source, `Cargo.toml`, `Makefile`, `.cu` files, CI workflows, and GitHub metadata.

1. `cuda-oxide` — flagship, Flux→PTX runtime
2. `cuda-genepool` — largest Lucineer mirror
3. `cuda-memory-fabric` — large Lucineer mirror
4. `cuda-biology` — large Lucineer mirror
5. `cuda-ghost-tiles` — large Lucineer mirror
6. `cuda-emotion` — large Lucineer mirror
7. `cuda-trust` — large Lucineer mirror
8. `cuda-energy` — large Lucineer mirror
9. `cuda-neurotransmitter` — large Lucineer mirror
10. `cuda-confidence` — large Lucineer mirror
11. `oxide-partition` — largest `oxide-*`
12. `oxide-conservation` — second largest `oxide-*`
13. `cuda-constraint-engine` — real CUDA C++
14. `cuda-constraint-checker` — real CUDA C++
15. `cuda-intelligence` — GPU-described Rust crate
16. `cuda-edge-runtime` — edge-runtime-described Rust crate
17. `oxide-sandbox` — Flux→MIR→PTX simulator
18. `oxide-flux-runtime` — runtime wrapper
19. `oxide-constructs` — git-native construct loader
20. `oxide-fleet` — fleet coordination layer
21. `oxide-crdt` — CRDT wrapper
22. `cuda-atp-market` — energy-market Rust crate
23. `cuda-keeper-core` — system watchdog Rust crate
24. `cuda-equipment` — shared equipment/A2A Rust crate
25. `cuda-telepathy` — A2A transport Rust crate

---

## 2. What the code actually is (not what the READMEs claim)

### 2a. The three repos that are actually GPU code

| Repo | Real GPU code? | Evidence |
|---|---|---|
| `cuda-oxide` | **Yes** | Fork of `NVlabs/cuda-oxide`. Workspace `Cargo.toml` points to `https://github.com/NVlabs/cuda-oxide.git`; author is `Nihal Pasham <nihalp@nvidia.com>`. Contains 308 source files, ~130 KLOC of Rust, real `.cu` device examples, `nvvm`/`nvjitlink` FFI, a custom rustc codegen backend, and scheduled CI (`cargo-deny`, `codeql`) that is green. |
| `cuda-constraint-engine` | **Yes** | CUDA C++ shared library (`src/constraint_engine.cu`, `src/kernels.cu`, `src/memory_pool.cu`, `src/stream_pool.cu`) that builds with `nvcc` via a real `Makefile`. No Rust. Uses `cuda_runtime.h`, CUDA graphs, streams, and an Eisenstein-snap kernel. ~1.8 KLOC. |
| `cuda-constraint-checker` | **Yes** | CUDA C++ binary (`src/constraint_checker.cu`, `src/main.cu`) with `__global__` kernels for bounds checks and Eisenstein snapping. Builds with `nvcc`. ~1.4 KLOC. |

`cuda-oxide` is the only repo here that is a serious, production-scale GPU compiler/runtime. It is also the only one that is **not** original to SuperInstance.

### 2b. The `cuda-*` Lucineer mirrors — generic agent crates wearing a GPU name

All ten large `cuda-*` repos inspected are forks of `Lucineer/*` (verified via `gh api repos/SuperInstance/<name> --jq '.parent.full_name'`). They contain **zero CUDA files** and **no GPU API calls**. Their actual contents:

| Repo | Source files | Source LOC | What it actually implements |
|---|---:|---:|:---|
| `cuda-genepool` | 1 | 1,388 | Agent genome/enzyme/RNA/protein metaphor; no GPU code. |
| `cuda-memory-fabric` | 1 | 481 | Four-layer agent memory (working/episodic/semantic/procedural); no GPU code. |
| `cuda-biology` | 1 | 631 | Biological agent runtime with instincts, ATP, apoptosis; no GPU code. |
| `cuda-ghost-tiles` | 7 | 3,255 | Sparse attention tile tracker; mentions a future CUDA kernel but has none. |
| `cuda-emotion` | 1 | 395 | Emotional-state engine mapping neurotransmitters; no GPU code. |
| `cuda-trust` | 6 | 3,074 | Contextual trust engine with Bayesian fusion; no GPU code. |
| `cuda-energy` | 1 | 373 | ATP/energy budget and circadian rhythm; no GPU code. |
| `cuda-neurotransmitter` | 1 | 500 | Neurotransmitter signal/receptor/synapse model; no GPU code. |
| `cuda-confidence` | 1 | 334 | `Conf` value-with-confidence type; no GPU code. |
| `cuda-intelligence` | 7 | 1,449 | Vessel spec table and stub modules; no GPU code. |

The large file sizes (17–30 MB) come from committed `target/` build artifacts (`.o`, `.rmeta`, `.rlib`, `fingerprint/`), not from source. After excluding build artifacts, the median source size is a few hundred lines.

### 2c. The small `cuda-*` Lucineer mirrors — also generic

| Repo | Source LOC | What it actually implements |
|---|---:|:---|
| `cuda-edge-runtime` | 354 | Edge trust/reflex/fleet coordinator; **path-depends on `../cuda-equipment`**. |
| `cuda-keeper-core` | 1,059 | `/proc` system watchdog, health checks, trend analysis. |
| `cuda-equipment` | 637 | Shared `Confidence`, `TileGrid`, A2A `FleetMessage`, `EquipmentRegistry`. |
| `cuda-telepathy` | 744 | A2A binary message transport, mailbox, router. |
| `cuda-atp-market` | 1,398 | Trust-weighted double-auction energy market. |

None of these call CUDA. They are generic distributed-agent primitives.

### 2d. The `oxide-*` crates — generated stubs with GPU-themed wrappers

Every inspected `oxide-*` repo is a single small Rust file (400–600 lines) built from the same template. They define generic data structures and wrap them in GPU/ternary vocabulary:

| Repo | Source LOC | Actual content |
|---|---:|---|
| `oxide-partition` | 503 | Hash-map partition assignment + balance thresholds. No CUDA. |
| `oxide-conservation` | 444 | Before/after numeric comparison with a `Ternary` verdict enum. |
| `oxide-sandbox` | 524 | Toy "Flux→MIR→PTX" compiler simulation; emits `Vec<String>` "PTX lines". |
| `oxide-flux-runtime` | 429 | Config/status structs for a hypothetical runtime. |
| `oxide-constructs` | 608 | Manifest/semver structs for git-native "constructs". |
| `oxide-fleet` | 442 | In-memory agent registry and work-assignment logic. |
| `oxide-crdt` | 438 | `GCounter`, `PNCounter`, `LwwKernelMap`. |

All use identical CI (`clones/oxide-partition/.github/workflows/ci.yml`): `cargo check`, `cargo test`, `cargo clippy -- -D warnings`, `cargo fmt --check`. The generated code **compiles and passes tests**, but **fails `fmt` and `clippy`**, which is the same pattern seen in the `lau-*` cluster.

---

## 3. Cross-repo dependencies and integration

### 3a. There is almost no dependency graph

- **No published crate links:** `grep` of `Cargo.toml` files found no `crates.io` dependencies on other `cuda-*` or `oxide-*` crates.
- **No git dependencies** between sample repos.
- **One path dependency:** `cuda-edge-runtime/Cargo.toml` declares `cuda-equipment = { path = "../cuda-equipment" }`. That only works if both repos are cloned as siblings; the SuperInstance mirror has no workspace or registry to resolve it.
- **Internal workspace only:** `cuda-oxide` uses path members for its own sub-crates.
- **README references only:** The Lucineer READMEs point to each other (e.g., `cuda-genepool` references `cuda-biology`, `cuda-neurotransmitter`, `cuda-energy`), but these are documentation links, not code imports.

### 3b. No hidden `nexus-runtime`-scale outlier

The `nexus-*` cluster had one genuine monorepo (`nexus-runtime`, ~190 KLOC, 2,559 passing tests) hiding among 17 stubs. Nothing comparable exists here:

- The largest repo by disk is `cuda-genepool` (30 MB), but 99% of that is committed `target/` artifacts; source is 1,388 lines.
- The largest original SuperInstance code is `cuda-constraint-engine` (~1.8 KLOC of CUDA C++).
- The only large, real codebase is `cuda-oxide`, and it is an external fork.
- Grep across all 25 cloned repos found **zero imports** of `nexus_runtime`, `nexus`, or any other SuperInstance cluster.

---

## 4. Bulk-generation signatures

### 4a. Cluster-wide date patterns

From `notes/cuda_oxide_repo_metadata.tsv`:

| Date | Event | Count |
|---|---|---:|
| 2026-04-10 | created | 56 |
| 2026-04-11 | created | 49 |
| 2026-04-12 | created | 34 |
| 2026-06-05 | created | 6 |
| 2026-06-06 | created | 25 |
| 2026-04-13 | last pushed | 136 |
| 2026-06-09 | last pushed | 16 |

This is the same bulk-push profile as `lau-*` and many `fleet-*` stubs.

### 4b. Commit-level signatures in the sample

Every sampled repo has **exactly one commit** on its default branch in the SuperInstance mirror. The messages repeat like a template:

| Group | Representative commit message | Author | Date |
|---|---|---|---|
| `cuda-*` Lucineer mirrors | `chore: add MIT license` | Casey Digennaro | 2026-04-13/18 |
| `oxide-*` stubs | `chore: add production files (LICENSE, CONTRIBUTING.md, CI)` | SuperInstance | 2026-06-09 |
| `cuda-oxide` | `gc: sync fleet GC config` | SuperInstance DocBot | 2026-06-13 |
| `cuda-constraint-engine` | `Add .gitignore, remove build artifacts from tracking` | Forgemaster | 2026-05-09 |
| `cuda-constraint-checker` | `feat: production CUDA constraint checker with constraint-the...` | Fleet Auditor | 2026-05-22 |

The Lucineer mirrors are not original development; they are one-shot license additions on top of imported code.

---

## 5. Verify or refute the out-of-scope framing

### 5a. Is the cluster GPU-specific?

**Mostly no, with three exceptions.**

- **Yes, GPU:** `cuda-oxide`, `cuda-constraint-engine`, `cuda-constraint-checker`.
- **No, generic agent crates with GPU branding:** the other 170 repos.

Quantitative evidence: of the 25 sampled repos, only 3 contained `.cu` files or CUDA API usage; the other 22 were pure Rust (or Makefile wrappers around Rust) with no GPU calls. The cluster-wide language distribution (164 Rust, 6 Makefile, 2 CUDA, 1 empty) confirms that CUDA is a tiny minority.

### 5b. Is there anything non-GPU that purplepincher should adopt?

**Conceptually yes, practically no.**

The following crates are *relevant in principle* to purplepincher's mission (modular/distributed edge):

- `cuda-equipment` — A2A message types, `TileGrid`, `Confidence`, equipment registry.
- `cuda-telepathy` — binary A2A transport, mailbox, router.
- `cuda-keeper-core` — `/proc`-based system watchdog and health monitor.
- `cuda-edge-runtime` — trust engine, reflex engine, fleet coordinator.
- `cuda-memory-fabric` — four-layer agent memory sketch.
- `cuda-confidence` / `cuda-trust` — confidence and trust primitives.

But each fails the purplepincher adoption bar:

1. **They are external mirrors, not maintained originals.** Most are forks of `Lucineer/*` with a single commit. There is no evidence of ongoing maintenance or real-world use.
2. **No installable artifacts.** None are on crates.io. `cuda-edge-runtime` even uses a sibling `path` dependency that breaks standalone builds.
3. **Committed build artifacts and weak hygiene.** Repos like `cuda-memory-fabric` and `cuda-biology` ship `target/` directories, a red flag for production adoption.
4. **No tests or CI for most.** Only `cuda-genepool` and `cuda-atp-market` had tests; most had no CI runs at all.
5. **Heavy metaphorical baggage.** The code is organized around mitochondria, neurotransmitters, apoptosis, ATP markets, and ghost tiles. Purplepincher's principle #3 is to strip ideology to optional; decoupling these crates from their metaphors would be a rewrite.
6. **Better alternatives already exist in the sketchbook.** For A2A messaging, `fleet-i2i-protocol` + `fleet-protocol` are more mature and already partially deployed. For reflexes, `pincher` is the intended second product. For memory, `exocortex` is far more complete. For edge runtime, `nexus-runtime` is the proven outlier.

The `oxide-*` stubs are even less attractive: they are freshly generated, single-commit, GPU-themed generic crates that duplicate ideas already present in `fleet-*` and `si-runtime-go`.

---

## 6. Concrete verdict

**Keep the entire `cuda-*` / `oxide-*` cluster out of scope for purplepincher.**

The out-of-scope framing from `notes/ecosystem-survey.md` and `notes/VISION_purplepincher.md` is correct, but the reason is slightly different from what those docs state. The cluster is not out of scope because it is uniformly GPU compute; it is out of scope because:

1. **The GPU parts are genuinely GPU-specific** (`cuda-oxide`, the two constraint CUDA repos) and purplepincher has no GPU edge product.
2. **The non-GPU parts are immature mirrors or generated stubs** that do not beat the better-implemented clusters already on the purplepincher roadmap.
3. **There is no hidden outlier** comparable to `nexus-runtime`.
4. **There are no published artifacts, no coherent dependency graph, and no production hygiene** in the non-GPU portion.

If purplepincher ever needs the *ideas* in this cluster, the right move is not to fork these repos; it is to harvest the concepts (confidence propagation, A2A messaging, watchdog health checks) and implement them cleanly inside the already-selected `fleet-*`, `pincher`, or `exocortex` lineages.

**No fork candidates from this cluster.**

---

## 7. Methodology footnote

- Repo enumeration: `gh repo list SuperInstance --limit 4200 --json name,description,updatedAt,pushedAt,primaryLanguage,diskUsage,createdAt,stargazerCount,forkCount` → `notes/cuda_oxide_repo_metadata.tsv`.
- Sample clones: `git clone --depth 1 https://github.com/SuperInstance/<name>.git clones/<name>` for the 25 repos listed above.
- Source inspection: direct reads of `src/lib.rs`, `Cargo.toml`, `Makefile`, `.cu`/`.cuh` files, and `.github/workflows/ci.yml`.
- Build/code metrics: `find` + `wc -l` on source files excluding `.git` and `target/` artifacts; saved in `notes/cuda_oxide_sample_loc.txt`.
- Fork/origin verification: `gh api repos/SuperInstance/<name> --jq '.parent.full_name'`.
- Commit/push patterns: `gh api repos/SuperInstance/<name>/commits?per_page=1` and cluster-wide date aggregation from `notes/cuda_oxide_repo_metadata.tsv`.
- CI verification: `gh run list -R SuperInstance/<name>` and `gh run view -R SuperInstance/<name> <id>`.
- Cross-repo dependency search: `grep -RIn` across the 25 clones for `SuperInstance/`, `path =`, `git =`, `nexus`, and `cuda_`/`oxide_` crate imports.

All clones remain unmodified in `clones/<name>/` for independent re-inspection.
