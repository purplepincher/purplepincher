# SuperInstance `superinstance-*` Cluster Survey

## Bottom line

The `superinstance-*` cluster is **the namesake family of the account, and it is also one of its weakest**. Out of 29 current repos, 26 were sampled directly. The pattern already observed across the broader SuperInstance mission holds here in an even more concentrated form:

- **READMEs oversell; code undersells.** Nearly every repo claims fleet-wide, foundational, or production-grade status, while the implementation is a thin sketch, a single-file worker, or a documentation-only repo.
- **Nothing is actually consumed as shared infrastructure.** No sampled repo has a proven downstream code dependency inside the SuperInstance org. Cross-references exist almost entirely in Markdown indexes, wikis, and CROSS-POLLINATION.md files, not in `Cargo.toml`, `package.json`, `pyproject.toml`, or source imports.
- **CI is either absent or gamed.** Many repos have no CI. Where CI exists, it is often a no-op (`echo "No CI configured"`) or uses `pytest || true` / `npm test --if-present` to mask real failures.
- **The cluster has no `nexus-runtime`-scale outlier.** The largest bodies of real code (`superinstance-ffi`, `superinstance-fleet-proto`, `superinstance-math`) are either mismatched in scope, unpublished, or have no consumers. The rest are static sites, documentation repos, or single-file prototypes.

**Verdict: nothing here meets the adoption bar for `purplepincher`.** A well-evidenced "nothing new here" is the correct finding.

---

## Method and scope

1. Fetched the authoritative repo list with `gh repo list SuperInstance --limit 4200` and filtered to the `superinstance-*` prefix.
2. Cloned and read code for 26 of 29 repos; checked real CI logs with `gh api repos/SuperInstance/<repo>/actions/runs`; attempted builds/tests where applicable.
3. Searched for cross-repo usage via local grep across clones and `gh search code` where rate limits allowed.
4. Explicitly compared `superinstance-cocapn` against the previously surveyed `cocapn-*` family.

**Current cluster size:** 29 repos (not the 41 in the older snapshot). The prefix dropped from the older snapshot because several repos were renamed, deleted, or never existed.

---

## Cluster-wide evidence table

| Repo | Lang | Claims | Reality | Build/test | CI | Downstream usage | Verdict |
|---|---|---|---|---|---|---|---|
| `superinstance-core` | Rust | "Foundation of fleet entity management" | 263-line HashMap ECS; no archetypes, no systems | `cargo test` ✅ 5 pass; `cargo clippy` ❌ fails; CI red | Latest run `27587671023` failed on clippy | None found | Thin, broken at lint gate, no consumers |
| `superinstance-protocol` | Rust | "Universal wire format for fleet communication" | 345-line `Bottle` serde crate; `bottle_integration.rs` is dead code | `cargo test` ✅ 7 pass; `cargo fmt` ❌ fails | **No CI** (0 workflows) | None found | Functional but aspirational; no fleet adoption |
| `superinstance-runtime` | Python | "Universal event bus; 141 proven regime transitions" | 242-line Python plugin skeleton; not on PyPI | `uv build` ✅; `pytest` ✅ 20 pass | Only Dependabot graph update | None found | Clean but tiny; production claim unsupported |
| `superinstance-cli` | Python | "Fleet CLI: `si init`, `si dev`, `si deploy`" | 379-line Click demo with math/rigidity commands; no packaging | `pytest` ✅ 32 pass | Only Dependabot graph update | None found | Toy CLI, no package, no fleet commands |
| `superinstance-ffi` | Rust + C/Zig | "Focused math primitives for Cocapn fleet" | 952-line Rust FFI plus 5.8 kLoC unrelated audio/world-music code; stale header | `cargo test` ✅ 57 pass; C headers ✅; CI green (11 runs) | All 11 runs success | Zero reverse deps; no org consumers | Build green but scope-mismatched and unconsumed |
| `superinstance-cocapn` | Rust | "Captain-level fleet coordination" | 584-line in-memory fleet coordinator with conservation audit | `cargo test` ✅ 9 pass | **No CI** | None found | Clean but isolated; no cocapn namespace collision |
| `superinstance-agent` | TypeScript | "RAG agent for 1,600+ Rust crates" | 214-line Cloudflare Worker; no tests | `tsc` ✅, `wrangler --dry-run` ✅ | **No CI** | None found | Thin wrapper, untested, undepended-on |
| `superinstance-agent-trait` | Rust | "Core Agent trait every agent implements" | 584-line trait + `Forgemaster` example | `cargo test` ✅ 8 pass | **No CI** | None found | Isolated design sketch, no consumers |
| `superinstance-harness` | Rust + TS | "Core optimization engine of SuperInstance" | 630-line Rust crate with **broken test module**; TS worker reimplements it | `cargo test` ❌ parse error | **No CI** | None found | Broken, no CI, no integration |
| `superinstance-flux-runtime-ruby` | Ruby | "Full FLUX ISA v3.0 VM" | 2,161-line partial VM; missing opcode dispatch, no Gemfile | Could not run locally; CI ❌ 2 failures | 2 runs, both failed on `rspec` not found | None found | Partial sketch; CI red |
| `superinstance-gpu-compute` | Rust | "Shared CUDA harness for fleet GPU agents" | 960-line `cudarc` wrapper; default build broken | `cargo check` ❌; `cargo check --features gpu` ❌ | **No CI** | None found | Broken boilerplate, no GPU infra |
| `superinstance-mcp` | TypeScript | MCP server for 10 fleet crates | 6 of 10 claimed crates are 404; 4 are unrelated existing crates | `npm run build` ✅; no tests | **No CI** | None found | Misleading crate registry; prior issues unchanged |
| `superinstance-math` | Python | "Pure-Python mathematical foundations for ML" | 1.2 kLoC pure-Python math modules; not on PyPI | 94 tests pass with numpy; 2 fail without | CI green via `pytest || true` without install | None found | Mostly functional but CI gamed, unpublished |
| `superinstance-spreadsheet` | HTML/JS + Python | "750M-agent GPU spreadsheet" | Browser demo + 2 Python scripts; no manifest | JS syntax OK; Python needs numpy/torch | 3 failed runs; workflow deleted | None found | Demo/prototype, no tests, license mismatch |
| `superinstance-ternary-wavelet` | Rust | "Ternary wavelet + Bottle integration" | 642-line Haar wavelet in one file; no Bottle integration | `cargo test` ✅ 14 pass | **No CI** | None found | Functional but SPEC claims unimplemented |
| `superinstance-vectorize` | TypeScript | "Semantic search across 560+ repos" | 177-line Cloudflare Worker with hand-coded heuristic vectors | `wrangler --dry-run` ✅; `tsc` fails (no tsconfig) | 1 run, failed | Only website health-check polls it | README endpoints don't match code |
| `superinstance-embedder` | Rust | "32-dim crate DNA embeddings seed Vectorize" | 357-line heuristic encoder; no network code | `cargo test` ✅ 11 pass; clippy/fmt ❌ | 5 runs, all failure | None found; `superinstance-vectorize` reimplements it | Unconsumed duplicate |
| `superinstance-knowledge` | Markdown | "Knowledge mine with formal proofs" | 353 files of LLM-generated essays; no build | Some Python scripts run; no tests | 1 run, no-op `pytest || true` | None found | Documentation-only, metrics inflated |
| `superinstance-live` | Python | "DAW session controller with CRDT sync" | 1 kLoC asyncio transport + OSC bridge; no CRDTs | 168 tests pass after installing 3 undeclared deps | Green via `pytest || true` despite collection errors | None found | Undeclared deps, overstated scope |
| `superinstance-ecosystem` | Python/docs | "Agent operating system — four layers" | 252-line YAML bottle protocol + code generators; no OS | `bottle_protocol.py` works; integration tests fail | 1 run, `pytest || true` masked `No module named yaml` | None found | Meta-repo, not a platform |
| `superinstance-design` | CSS/JS | "Shared design system for all web properties" | 1,319-line static theme | Static files work | 12 runs, all no-op echo | Consumed by `superinstance-ai-pages` only | Usable but not fleet-wide |
| `superinstance-ci` | Shell/YAML | "Shared CI templates for fleet Rust repos" | 653-line template pack pointing to a missing reusable workflow | Script syntax OK; installs broken wrapper | **No CI** | None found | Non-functional shared infra |
| `superinstance-fleet-proto` | Rust | "Shared PLATO client and I2I protocol" | 1,681-line Rust lib, well-tested | 64 tests + 5 doc-tests pass (needs pkg-config) | **No CI** | None found | Best Rust lib in cluster, but orphan |
| `superinstance-hdc-core` | Rust | "AVX-512 HDC, Python bindings, no-std" | 3.6 kLoC scalar HDC; SIMD/Python/no-std all broken | `cargo test` ❌ 26 errors | 5 runs, all failed | None found | Scaffold with broken headline features |
| `superinstance-index` | Markdown | "Auto-generated map of 598 repos" | 1,025-line hand-written catalog | Nothing to build | **No CI** | Referenced by wiki indexes only | Documentation directory only |
| `superinstance-architecture` | Markdown/docx | "145k-line architecture / shared I2I vessel" | 18.8 kLoC docs/diagrams, no code | Nothing to build | 1 no-op run | None found | Pure documentation |
| `superinstance-website` | HTML/JS/JSON | Main portal | Static site; builds on Pages | Static deploy ✅ | 18 runs, green | Catalogs other repos as data | Healthy static portal |
| `superinstance-wiki` | Python/Markdown | Wiki engine + docs | Packaged wiki engine; `pytest` has numpy collection error | Wheel builds; tests fail without numpy | Green via `pytest || true` | No code consumers | Docs/wiki engine with gamed CI |
| `superinstance-ai-pages` | HTML/CSS | GitHub Pages landing | Static site consuming `superinstance-design` | Static deploy ✅ | 13 runs, green | Consumes `superinstance-design` | Healthy static landing page |

---

## Detailed findings

### Foundational/shared-infrastructure repos (the ones most likely to matter)

#### `superinstance-core`
- **Claim:** "SuperInstance Core is the foundation of fleet entity management."
- **Reality:** A single `src/lib.rs` (263 lines) implementing a HashMap-based ECS. The README admits cache locality is "poor (HashMap)". No archetypal storage despite doc comments.
- **Build:** `cargo test` passes (5 tests); `cargo clippy` fails on three trivial errors (never-read field, missing `is_empty`, missing `Default`).
- **CI:** Latest run `27587671023` (2026-06-16) failed on the clippy step.
- **Consumers:** `CROSS-POLLINATION.md` claims use by `ternary-fleet`, `ternary-search-rs`, and `superinstance-protocol`. None of those repos list `superinstance-core` in their manifests or import it.
- **Verdict:** Toy ECS, broken at the lint gate, no downstream users.

#### `superinstance-protocol`
- **Claim:** "Universal wire format for fleet communication," used by `superinstance-core`, `ternary-fleet`, `conservation-action`.
- **Reality:** A 345-line Rust crate with a `Bottle` serde envelope (JSON/msgpack). `src/bottle_integration.rs` (252 lines) is **not wired into the crate** (`mod bottle_integration;` is missing), so its 6 tests never run. `types.ts` is orphan TypeScript.
- **Build:** `cargo test` passes 7 tests; `cargo fmt` fails.
- **CI:** **No workflows; 0 runs.**
- **Consumers:** No manifest or source imports found in other SuperInstance repos.
- **Verdict:** Functional serialization crate with dead integration code and no fleet adoption.

#### `superinstance-runtime`
- **Claim:** "Universal event bus … 141 proven regime transitions."
- **Reality:** 242-line Python plugin skeleton with a `COLLECT → SELECT → COMPILE` pipeline. No persistence, no serialization, no concurrency.
- **Build:** `uv build` succeeds; `pytest` passes 20 tests.
- **CI:** Only a Dependabot dependency-graph run.
- **Consumers:** None. Not on PyPI despite README's `pip install superinstance-runtime`.
- **Verdict:** Clean but tiny; production claims are fiction.

#### `superinstance-cli`
- **Claim:** Fleet CLI with `si init`, `si dev`, `si test --watch`, `si deploy`, `si docs generate`, `si debug` (from `SuperInstance-papers/docs/DEVELOPER_EXPERIENCE.md`).
- **Reality:** 379-line Click CLI with commands like `check`, `edges`, `rigid`, `norm`, `encode`, `filter`, `benchmark`, `status`. No `si` entry point, no packaging metadata (`pyproject.toml`/`setup.py`), no README.
- **Build:** 32/32 `pytest` tests pass.
- **CI:** Only a Dependabot graph update.
- **Consumers:** None.
- **Verdict:** Toy math utility CLI, not an ecosystem CLI.

#### `superinstance-ffi`
- **Claim:** "Unified C FFI and WASM bindings for SuperInstance math primitives" used across the Cocapn fleet.
- **Reality:** 952-line Rust FFI exports math functions **and** a full spline wavetable synthesizer (8 undocumented `si_spline_wavetable_*` functions). Ships 5.8 kLoC of unrelated audio/world-music Zig + C headers that are not wired into the Cargo build. The committed `superinstance-ffi.h` is stale; running `cargo build --release` regenerates it with the missing wavetable API.
- **Build:** `cargo test` passes 57 tests; C header tests pass; CI is green (11 runs).
- **Consumers:** Zero crates.io reverse dependencies; no SuperInstance repo imports it.
- **Verdict:** Build green, crate published (16 downloads), but the scope is a personal audio/math experiment packaged as fleet math. Not a focused foundation.

---

### Agent/orchestration repos

#### `superinstance-agent`
- **Claim:** RAG AI agent for 1,600+ Rust crates.
- **Reality:** 214-line Cloudflare Worker calling two managed AI bindings and a Vectorize index. No tests, no CI, not published to npm.
- **CI:** 0 runs.
- **Consumers:** None. Even the repo description contradicts the README.
- **Verdict:** Thin wrapper.

#### `superinstance-agent-trait`
- **Claim:** "Core Agent trait for the SuperInstance ecosystem — every agent in the fleet implements it."
- **Reality:** 584-line Rust crate defining a 2-method `Agent` trait and a `Forgemaster` example. Depends on `superinstance-protocol` via path.
- **Build:** 8 tests pass.
- **CI:** 0 runs.
- **Consumers:** None. `superinstance-agent` (TypeScript) and `superinstance-harness` do not use it.
- **Verdict:** Isolated design sketch.

#### `superinstance-harness`
- **Claim:** "Core optimization engine of SuperInstance" with Cloudflare Worker REST API, D1/KV persistence, and integration with `fleet-vector-api`, `fleet-metrics-cron`, `knowledge-cron`.
- **Reality:** 630-line Rust crate with a syntactically broken test module (`unexpected closing delimiter: }` at line 630). The TypeScript worker reimplements the algorithm instead of consuming the Rust crate.
- **Build:** `cargo test` fails to parse.
- **CI:** 0 runs.
- **Consumers:** None.
- **Verdict:** Broken, no CI, no integration.

---

### Specialized compute/repos

#### `superinstance-gpu-compute`
- **Claim:** "Shared CUDA harness for the SuperInstance fleet GPU agents."
- **Reality:** 960-line `cudarc` wrapper.
- **Build:** `cargo check --no-default-features` fails because `bench.rs` calls a non-existent `launch` method on `PtxKernel`. `cargo check --features gpu` fails because CUDA headers are missing and `GpuError::Kernel` is undefined.
- **CI:** 0 runs.
- **Consumers:** None.
- **Verdict:** Broken boilerplate.

#### `superinstance-hdc-core`
- **Claim:** AVX-512 HDC, pip-installable Python bindings, `no-std` support.
- **Reality:** 3.6 kLoC scalar hyperdimensional computing code. SIMD files have intrinsic-name errors and wrong cfg gating; Python bindings have invalid Rust syntax (`m.add_class::<<PySramImage>>()?`); `no-std` fails because `std::fs`/`std::path` are used unconditionally.
- **Build:** `cargo test` fails with 26 errors in `simd_avx512.rs`.
- **CI:** 5 runs, all failed.
- **Consumers:** None.
- **Verdict:** Scaffold with broken headline features.

#### `superinstance-flux-runtime-ruby`
- **Claim:** Full FLUX ISA v3.0 VM, assembler, disassembler, CLI, published as `superinstance-flux-runtime` on RubyGems.
- **Reality:** 2,161-line partial Ruby VM. `execute_opcode` omits control-flow opcodes `0x03`–`0x07` and A2A opcodes `0x80`–`0x89`. Runtime opcode registration is mis-wired. No `bin/flux` executable. No `Gemfile`, so CI cannot install dependencies. Gemspec homepage points to a non-existent repo.
- **Build:** Could not run locally (no Ruby). CI logs show `rspec: command not found`.
- **CI:** 2 runs, both failed.
- **Consumers:** None.
- **Verdict:** Partial, untested sketch.

---

### `superinstance-cocapn` and the cocapn family

The mission's earlier `cocapn-family-deep-dive.md` found a PyPI namespace collision: `SuperInstance/cocapn` and `SuperInstance/cocapn-py` both claimed the bare `cocapn` package name.

- `superinstance-cocapn` **does not repeat that collision**. Its Cargo name is `superinstance-cocapn`; it makes no PyPI/crates.io claim on bare `cocapn`.
- It is a 584-line Rust fleet-coordination crate (vessel registry, conservation audit, load routing). `cargo test` passes 9 tests. No CI. No downstream consumers.
- It shares **no code** with `cocapn-foundation` (a Claude Design handoff bundle of HTML/CSS/JS prototypes for a fishing-vessel voice assistant). The only overlap is the nautical metaphor.

**Conclusion:** `superinstance-cocapn` is clean on the namespace issue but is otherwise another isolated orphan.

---

### MCP / embedder / vectorize / knowledge

#### `superinstance-mcp`
A prior survey flagged three issues. All persist:
1. **Fabricated crate list:** `src/index.ts` lists 10 crates. Six return 404 from crates.io; the four that exist (`shoal`, `openagent`, `wavefront`, `ternary`) are unrelated pre-existing crates owned by other users.
2. **Hardcoded path:** `.mcp.json:5` still contains `"args": ["tsx", "/home/phoenix/repos/superinstance-mcp/src/index.ts"]`.
3. **Unconditional install success:** `install.sh:39` uses `timeout 3 npx tsx ... || true` and prints "✅ Server test complete" regardless.
- **Build:** `npm run build` passes; no tests; no CI.
- **Verdict:** Unchanged, untrustworthy.

#### `superinstance-embedder` vs `superinstance-vectorize`
- `superinstance-embedder` is a 357-line Rust heuristic encoder (one-hot domain + scalar ratios). It has no Cloudflare client and no consumer.
- `superinstance-vectorize` is a 177-line TypeScript Cloudflare Worker that **reimplements** the same heuristic idea; it does not consume `superinstance-embedder`.
- Both claim to power semantic search across 560+ repos. Both are hand-coded string matchers, not learned embeddings. Both have no downstream consumers.

#### `superinstance-knowledge`
- **Claim:** "Knowledge mine and research repository … formal proofs, experiment logs."
- **Reality:** 353 Markdown files (~117 k lines) of LLM-generated speculative essays. One Lean file is pseudocode. The single CI run used `pytest || true`, which masked "no tests collected." The workflow file has since been deleted.
- **Verdict:** Documentation/wiki only, not engineering infrastructure.

---

### Ecosystem / meta / website repos

#### `superinstance-ecosystem`
- **Claim:** "The agent operating system — four layers that compose," 327+ tests across repos, cross-language validation.
- **Reality:** 252-line `bottle_protocol.py` (YAML message envelope) plus a `validate_all.py` that generates Rust/C/JS/CUDA test files but does not compile or run them. No `pyproject.toml`. Integration tests hardcode `~/repos/{lever-runner,pincherOS,zeroclaw,...}` and fail when those paths do not exist.
- **CI:** 1 run, `pytest || true`, masked `ModuleNotFoundError: No module named 'yaml'`.
- **Verdict:** Design/meta-repo, not a working OS.

#### `superinstance-ci`
- **Claim:** Central shared CI templates for the fleet.
- **Reality:** Template wrappers reference `SuperInstance/.github/.github/workflows/rust-template.yml@main`, which returns 404. The `SuperInstance/.github` repo only contains a no-op placeholder workflow.
- **CI:** 0 runs on the repo itself.
- **Consumers:** None. The six claimed downstream repos all use their own workflows or have none.
- **Verdict:** Non-functional shared infrastructure.

#### `superinstance-design`
- **Claim:** "Shared design system for all SuperInstance web properties."
- **Reality:** 1,319-line static CSS/JS theme. GitHub Pages deploy works.
- **CI:** 12 runs, all no-op (`echo "No CI configured"`).
- **Consumers:** Only `superinstance-ai-pages` loads it. `superinstance-website` and `superinstance-wiki` use their own styles.
- **Verdict:** Usable but not fleet-wide.

#### `superinstance-website`, `superinstance-wiki`, `superinstance-ai-pages`
- `superinstance-website`: Healthy static portal; CI deploys to Pages. Catalogs other repos as JSON data.
- `superinstance-wiki`: Packaged Python wiki engine; wheel builds; `pytest` has a numpy collection error masked by `|| true` in CI.
- `superinstance-ai-pages`: Healthy static Pages site; consumes `superinstance-design`.
- These are presentation/documentation properties, not fork candidates.

---

### The closest thing to an outlier

#### `superinstance-fleet-proto`
- **Reality:** 1,681-line Rust library with a `PlatoClient`, custom I2I message/bottle format, `FleetAgent` async trait, capability validation, and wiremock-based tests. 64 tests + 5 doc-tests pass.
- **Build:** Passes after setting `OPENSSL_DIR`/`OPENSSL_LIB_DIR`/`OPENSSL_INCLUDE_DIR` (the environment lacks `pkg-config` metadata).
- **CI:** 0 runs; no workflows.
- **Consumers:** None. Wiki indexes label it "Functional / Orphan / REVIEW."
- **Verdict:** The most substantial Rust implementation in the cluster, but it has no CI, is not published, and has no proven consumers. It is an orphan, not a shared foundation.

---

## Cross-cutting observations

1. **No proven dependency graph.** Despite names like `core`, `protocol`, `runtime`, `cli`, `ffi`, `agent-trait`, `harness`, `fleet-proto`, none of these repos are actually depended on by other SuperInstance repos in code. The "ecosystem" is a set of parallel monoliths linked only by Markdown.

2. **CI is largely decorative.** Repos either have no CI or have CI configured to always pass (`pytest || true`, `npm test --if-present`, `echo "No CI configured"`). Real test failures are common but hidden.

3. **Publication claims are false.** `superinstance-runtime`, `superinstance-cli`, `superinstance-math`, `superinstance-embedder`, `superinstance-hdc-core`, `superinstance-flux-runtime-ruby`, `superinstance-fleet-proto` all claim or imply publication; none are actually installable from PyPI/crates.io/RubyGems except `superinstance-ffi` (published but unconsumed).

4. **Documentation inflation.** READMEs, ARCHITECTURE.md, and CROSS-POLLINATION.md describe fleets, operating systems, hundreds of tests, and cross-language validation. The code does not back these claims.

5. **License mismatches are routine.** Multiple repos (`superinstance-spreadsheet`, `superinstance-design`, `superinstance-knowledge`, `superinstance-architecture`, `superinstance-flux-runtime-ruby`) claim MIT in README while the `LICENSE` file is Apache-2.0, or vice versa.

---

## Final verdict for `purplepincher`

**Do not fork anything from the `superinstance-*` cluster.**

The cluster matches the account's dominant pattern exactly: ambitious READMEs, thin or broken implementations, absent or gamed CI, no real adoption, and no hidden outliers on the scale of `nexus-runtime` or `plato-semantic-search`. Even the best repo, `superinstance-fleet-proto`, is an unpublished, CI-less orphan. `superinstance-ffi` builds and passes tests but is scope-mismatched and unconsumed. Everything else is static sites, documentation, or single-file prototypes.

The survey covered 26 of 29 current repos directly and gathered enough evidence on the remaining 3 website/wiki properties. The finding is: **nothing here is worth real fork consideration.**

---

## Appendices

### A. Authoritative repo list (current)

Fetched 2026-07-05 from `gh repo list SuperInstance --limit 4200`:

```
SuperInstance/superinstance-agent
SuperInstance/superinstance-agent-trait
SuperInstance/superinstance-ai-pages
SuperInstance/superinstance-architecture
SuperInstance/superinstance-ci
SuperInstance/superinstance-cli
SuperInstance/superinstance-cocapn
SuperInstance/superinstance-core
SuperInstance/superinstance-design
SuperInstance/superinstance-ecosystem
SuperInstance/superinstance-embedder
SuperInstance/superinstance-ffi
SuperInstance/superinstance-fleet-proto
SuperInstance/superinstance-flux-runtime-ruby
SuperInstance/superinstance-gpu-compute
SuperInstance/superinstance-harness
SuperInstance/superinstance-hdc-core
SuperInstance/superinstance-index
SuperInstance/superinstance-knowledge
SuperInstance/superinstance-live
SuperInstance/superinstance-math
SuperInstance/superinstance-mcp
SuperInstance/superinstance-protocol
SuperInstance/superinstance-runtime
SuperInstance/superinstance-spreadsheet
SuperInstance/superinstance-ternary-wavelet
SuperInstance/superinstance-vectorize
SuperInstance/superinstance-website
SuperInstance/superinstance-wiki
```

(29 repos; `SuperInstance/SuperInstance` meta-repo and `SuperInstance-papers` excluded per task instructions; `SuperInstance-Starter-Agent` no longer exists under the prefix.)

### B. Survey clones location

All repos were cloned under:

```
/tmp/claude-1000/-home-eileen/c25f18c4-8752-45a5-bacc-1c3e70a86ab7/scratchpad/superinstance-cluster-kimi/scratch/superinstance-clones/
```

Build artifacts, virtual environments, and `target/` directories were left in place for verification; no source files were modified.
