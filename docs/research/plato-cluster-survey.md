# Plato Cluster Survey

**Date:** 2026-07-04  
**Scope:** all 278 SuperInstance repos whose names start with `plato` (`notes/plato_all_repo_names.txt`), excluding deep re-coverage of the four already-surveyed repos (`plato-ng`, `plato-vessel-core`, `plato-vessel-technician`, `plato-serialize`).  
**Method:** sampled deliberately by naming sub-family, read code/READMEs/manifests from treeless clones (`clones/`) and extracted samples (`notes/plato_samples/`), checked git metadata (`notes/plato_cluster_metadata.jsonl`), grepped for cross-repo imports, and ran whatever tests the local toolchain allowed.  
**Local constraints:** Python 3 and Node are available; `pip`, `cargo`, `rustc`, `zig`, `elixir`, and `gleam` are not. This limits Python test runs to stdlib-only modules and Rust/Elixir/Zig/Gleam repos to source inspection.

---

## 1. Cluster metadata — bulk-generation signature

- **278 repos** total in the `plato-*` naming family.
- **Primary languages:** Rust 136, Python 110, TypeScript 2, C 2, plus single Elixir/Zig/Gleam/Chapel/Ruby/PHP/JS/HTML/Cuda repos (`notes/plato_cluster_metadata.jsonl` language counts).
- **Created:** almost all in April–June 2026 (`2026-04`: 130, `2026-05`: 128, `2026-06`: 19).
- **Last-push bulk dates:** 120 repos pushed on `2026-05-08`, 58 on `2026-06-08`, 20 on `2026-05-29`. This is exactly the bulk-generation pattern seen in other SuperInstance clusters.
- **Shallow history:** every sampled repo is a single visible commit because the clones were made with `--depth 1`; the push-date spikes are the strongest evidence of mass repo creation.
- **Disk vs. source:** many repos look large on disk (e.g. `plato-engine` 788 MB, `plato-room-nav` 253 MB) because `target/debug/` build artifacts are committed. The actual source in most repos is tiny — often 100–400 LOC.

---

## 2. Sub-cluster breakdown

| Sub-cluster | Count | What it actually is | Maturity read |
|---|---|---|---|
| `plato-tile-*` + `plato-tiles`/`plato-tiling`/`plato-meta-tiles`/`plato-genepool-tile` | ~40 | A cloud of noun/verb microservice *names* (import, query, split, store, scorer, bridge, etc.) with small Python/Rust stubs. Most define their own ad-hoc tile struct and have no persistence or network layer. | Thin sketches; no shared library or wire format. |
| `plato-room-*` + `plato-room`/`plato-rooms`/`plato-live-room` | ~23 | Same pattern for rooms: analytics, configs, engine, musician, nav, phi, server, wasm, etc. Most are 100–400 LOC self-contained stubs. | Thin sketches; the exception is `plato-room-configs`. |
| `plato-engine-block-*` | 5 | Language ports (Rust, C, Elixir, Gleam, Zig) of a sensor/actuator/alarm room runtime with a text protocol and ternary state. | Clean ports; the C implementation is buildable and fully tested here. |
| `plato-forge-*` + `plato-forge` | 8 | Two real Rust crates (`forge-bridge`, `forge-pipeline`), one Python training daemon, and four Python repos whose only source lives inside pre-built wheels. | Fragmented; not wired together. |
| `plato-jepa*` (`plato-jepa`, `-jepa-dual`, `-vision-jepa`, `-audio-jepa`) | 4 | Genuine JEPA-themed Rust primitives: embeddings, predictors, dual-vector spaces, vision/audio perception. | Well-scoped library sketches with tests and CI, but isolated. |
| `plato-vessel-*` | 4 | One working C/Python system (`plato-vessel-core`, already surveyed) and three documentation-only repos. | See prior surveys. |
| Standalone / core (`plato`, `plato-core`, `platoclaw`, `plato-mud`, `plato-mud-server`, `platonic-randomness`) | 6 | Mixed bag: a full telnet/web MUD app (`plato`), a clean foundational package (`plato-core`), a real HTTP server (`platoclaw`), an archived MUD server, and a tiny randomness repo. | `plato` and `plato-core` are the most complete. |
| Other outliers | ~202 | Everything else: training frameworks, dashboards, bridges, kernels, MCP stubs, etc. Mostly thin or doc-heavy; a few are real (see §4). | Mostly spec/placeholder; the clear verified outlier is `plato-semantic-search`. |

### 2.1 `plato-tile-*` sample details

Read: `plato-tile-spec`, `plato-tile-query`, `plato-tile-split`, `plato-tile-import`, `plato-tile-client`, `plato-tile-store`, `plato-tile-room-bridge`, `plato-tile-library`.

- `plato-tile-spec` advertises a canonical `TileSpec`/`TileDomain` format, but `src/plato_tile_spec/__init__.py:3` imports `TileDomain` and `TileSpecValidator` that do **not** exist in `spec.py`, so the package is import-broken.
- `plato-tile-client` (`src/plato_tile_client/client.py:109-114`) has `get/post/put/delete` methods whose `_request()` returns a simulated `{"ok": true}` unless a test handler is registered — it never opens a socket.
- `plato-tile-room-bridge` is just an in-memory association between two local structs (`src/lib.rs:115-204`), not a network bridge.
- `plato-tile-library` contains no code; the working tree is a set of large JSON tile dumps.
- `plato-tile-import`, `-query`, `-split`, and `-store` are small single-file Rust or Python modules with no execution engine, no shared types, and no tests in most cases.

### 2.2 `plato-room-*` sample details

Read: `plato-room-configs`, `plato-room-wasm`, `plato-room-phi`, `plato-room-nav`, `plato-room-musician`, `plato-room-server`, `plato-room-analytics`, `plato-room-engine`.

- `plato-room-configs` is the standout: a Rust crate (`Cargo.toml:11-17`) that loads/validates JSON room configs with `serde`, `jsonschema`, `thiserror`, and `glob`. It has four test files under `tests/`, JSON schemas under `schema/`, and 15+ example room configs under `configs/`.
- `plato-room-musician` is a real creative pipeline (~1,400 LOC Python) that fetches PLATO rooms and renders them as MIDI, but it requires `mido`, `numpy`, and `flux-tensor-midi`, so it could not be run here.
- `plato-room-phi` has a test/code mismatch: `tests/test_phi.py` calls `compute_phi()`, but the class only exposes `compute_prii()` (`plato_room_phi/__init__.py:106`).
- `plato-room-server` is a routing stub with no socket layer (`src/plato_room_server/server.py:38-46`).
- `plato-room-engine` mentions `plato-address`/`plato-hooks`/`plato-tile-spec`/`plato-tile-scorer` in doc comments (`src/lib.rs:8-11`) but imports only `std::collections::HashMap`.

### 2.3 `plato-engine-block-*`

All five are ports of the same small sensor/actuator/alarm room runtime.

- `plato-engine-block-c` is the most mature: ~2 kLOC C99, single-header core, `poll()`-based TCP server on port 7070, examples, and 35 tests. `make test` in `clones/plato-engine-block-c` passes 35/35.
- `plato-engine-block` (Rust) has a real Tokio TCP server and CI, but `cargo` is unavailable here.
- `plato-engine-block-elixir`, `-gleam`, `-zig` are clean language sketches; no cross-repo deps.

### 2.4 `plato-forge-*`

- `plato-forge-bridge` and `plato-forge-pipeline` are real, small Rust crates (~700 LOC each) with inline tests and CI.
- `plato-forge-daemon` is a Python training demo that runs DistilGPT-2 steps on synthetic fleet tiles, but it needs `torch`/`transformers`.
- `plato-forge-buffer`, `-emitter`, `-listener`, `-trainer` have `pyproject.toml` + README in version control, but the actual source only exists inside pre-built wheels in `dist/`. This is a red flag: the published artifact is ahead of the repo.

### 2.5 `plato-jepa*`

Four Rust crates implementing JEPA-style primitives: `TileEmbedding`, `Predictor`, `LatentSpace`, dual-vector `PerceptionDB`/`PredictionDB`, 16-dim vision/audio room states. They have tests and CI, depend only on `serde`/`uuid`, and do not import each other. The ML is intentionally tiny (linear models, simple DFT), so they are pedagogical primitives rather than a production perception stack.

### 2.6 `plato-vessel-*`

Already covered in `notes/11-named-repos.md` and `notes/plato-ng-investigation.md`. In short: `plato-vessel-core` is real (C client + Python room server with 75 passing tests); the other three are documentation-only.

### 2.7 Standalone / core

- `plato` (the root repo) is a runnable telnet + web MUD-style PLATO application with ~6,400 LOC, room templates, tile persistence, onboarding, and NPCs. No tests were found in the sample, but the code matches the README.
- `plato-core` is a clean foundational package: `TrainingTile`, `TileLifecycle`, `LamportClock`, and a `MeshRegistry` that discovers ecosystem plugins via `importlib.metadata.entry_points()` (`plato_core/registry.py:46-48`). It has tests and CI.
- `platoclaw` is a real Python HTTP server with `/submit`, `/rooms`, `/room/{id}/history`, and a hardcoded model router; it imports `fleet_router`, so it is not self-contained.
- `plato-mud-server` is an archived class-based MUD with a 346-line pytest suite.
- `plato-mud` is a single-file asyncio workspace MUD; no tests.

---

## 3. Are `plato-tile-*` and `plato-room-*` a coherent microservice architecture?

**No. They are independent sketches sharing a naming convention and a vague conceptual model (rooms contain tiles).**

Evidence:

1. **Zero cross-repo package/crate dependencies.** No `Cargo.toml`, `pyproject.toml`, or `package.json` in the sampled tile/room repos declares a dependency on another `plato-*` or `fleet-*` package. Dependencies are limited to generic third-party crates (`serde`, `jsonschema`, `wasm-bindgen`) or Python packages (`requests`, `mido`, `numpy`).
2. **Zero cross-repo imports in source.** A recursive grep of all sampled `*.py` and `*.rs` for `import plato_` / `from plato_` / `use plato_` returned only intra-package imports (e.g. `from plato_room_musician.score import ...`) and self-references. `plato-room-engine/src/lib.rs:8-11` names sister crates in doc comments but never imports them.
3. **No shared tile schema.** `plato-tile-store`, `plato-tile-import`, `plato-tile-room-bridge`, and `plato-room-wasm` each define their own tile struct; `plato-tile-spec` is import-broken and is not consumed by any other sampled repo.
4. **No inter-service clients.** `plato-tile-client` simulates HTTP. `plato-room-phi` and `plato-room-musician` call a single hard-coded PLATO gateway (`http://localhost:8847` or `http://147.224.38.131:8847`), not each other. `plato-tile-room-bridge` is an in-process data-structure helper.
5. **No service discovery or deployment wiring.** No `docker-compose.yml`, Kubernetes manifests, gRPC/proto, or service mesh configs link the repos.

The names suggest a sharded microservice pipeline, but the code is a collection of parallel, non-composing stubs.

---

## 4. Outlier hunt

The task asked specifically for a `nexus-runtime` / `exocortex-mcp-ts`-scale outlier — a real, substantial, well-tested implementation hiding among thin siblings. The honest answer is that **nothing in the previously unsurveyed 274 repos clearly beats the already-known `plato-vessel-core`**, but several repos are credible enough to be worth noting.

### Verified outliers (tests actually run here)

1. **`plato-semantic-search`** — Cloudflare Worker, 67 passing tests.
   - `src/index.ts:31-69` dispatches `/health`, `/stats`, `/search`, `/upsert`, `/delete`.
   - `wrangler.toml:5-10` binds Workers AI and Vectorize.
   - `npm install && npm test` in `notes/plato_samples/plato-semantic-search` produced:
     ```
     Test Files  2 passed (2)
          Tests  67 passed (67)
     ```
   - Caveat: small (~1,200 LOC), Cloudflare-specific, and depends on external AI/Vectorize bindings. It is production-shaped but not a large standalone runtime.

2. **`plato-engine-block-c`** — C99 room engine, 35 passing tests.
   - `include/plato_engine.h` is a single-header engine; `src/server.c:62-247` is a real `poll()`-based TCP server on port 7070.
   - `make test` in `clones/plato-engine-block-c` produced:
     ```
     === Engine Tests === 14/14 passed
     === Protocol Tests === 14/14 passed
     === History Buffer Tests === 7/7 passed
     ```
   - Caveat: it is a compact embedded runtime (~2 kLOC), not a distributed framework.

3. **`plato-room-configs`** — Rust config loader/validator, tests present but unverified.
   - `Cargo.toml:11-17` uses `serde`, `serde_json`, `jsonschema`, `thiserror`, `glob`.
   - Four test files under `tests/` exercise loading, validation, manifests, and the bundled example configs.
   - Could not run `cargo test` because `cargo` is not installed, but the code is well-structured and the docs match the implementation.

4. **`plato-core`** — clean foundational Python package.
   - `plato_core/types.py` defines `TrainingTile`, `TileLifecycle`, `LamportClock`, etc.
   - `plato_core/registry.py:46-48` auto-discovers plugins via `importlib.metadata.entry_points(group="superinstance.plugins")`.
   - `PYTHONPATH=. python3 -c "from plato_core import ..."` succeeds; tests are simple pytest-style functions but could not be executed without `pytest`.

### Large but unverified / suspect candidates

- **`plato-training`** is the biggest Python repo by source (38k+ LOC, 44 test files). It has a full `TrainingTile` lifecycle, GPT-2 trainers, LoRA factory, fleet miners, and data pipelines. However, `pyproject.toml:12` declares **zero runtime dependencies** while the code imports `torch`, `transformers`, `datasets`, `sklearn`, etc. That mismatch means the package is not actually installable/runnable as documented without manually installing undeclared deps. It looks like a generated scaffolding dump rather than a working product.
- **`plato-portal`** is 271 MB on disk but mostly markdown, images, and catalog/index scripts; the Python/JS glue is thin.
- **`plato-tour-guide`** is large and multi-language (Python + Rust + CUDA), but its README references `room.py` and `cascade.py` that do not exist in the repo, so the docs already oversell the code.

### Previously known outlier

- **`plato-vessel-core`** remains the strongest working system in the whole cluster: C HTTP/MCP client for microcontrollers, Python room server with Lamport clocks, WAL crash recovery, tile lifecycle, and 75/75 passing tests. This was already established by earlier surveys and is not re-covered here.

---

## 5. Concrete fork/polish shortlist

If the goal is to find something worth polishing or building on, the list is short and prioritized by evidence:

1. **`plato-semantic-search`** — already passes 67 tests, is Cloudflare-deployable, and has a clear, narrow scope. Polish: add real Vectorize index seeding, end-to-end integration test against a real Workers AI account, and auth-secret documentation.
2. **`plato-engine-block-c`** — the only newly surveyed repo that builds and passes a full test suite in this environment. Polish: wire CI to actually run `make test` (currently the workflow just echoes), add a client example, and tighten the protocol spec.
3. **`plato-room-configs`** — the most mature repo in the tile/room family. Needs only `cargo test` verification (not possible here) and perhaps a small CLI/example binary to load a manifest.
4. **`plato-core`** — the cleanest shared-foundation candidate. Polish: make the tests runnable without `pytest` (or add a `pytest` runner), and publish it as the canonical tile-type package that the rest of the cluster currently lacks.
5. **`plato-vessel-core`** — already known; if consolidating the cluster, this is still the most sensible base because it actually runs end-to-end.

**Honest negative finding:** the other ~270 repos are largely spec-heavy, single-commit sketches with oversold READMEs. They do not compose into a working PLATO ecosystem, and none of them currently rivals `plato-vessel-core`, `Edge-Native`, or `exocortex` in runnable substance.

---

## 6. Verification summary

| What was checked | Result |
|---|---|
| `python3 -m py_compile` on all sampled Python files | Passed |
| `npm install && npm test` in `plato-semantic-search` | **67/67 passed** |
| `make test` in `plato-engine-block-c` | **35/35 passed** |
| `python3 -m unittest test_plato_v3` in `plato-vessel-core/server` (via symlink) | **75/75 passed** (re-confirms prior survey) |
| `PYTHONPATH=. python3 -c "from plato_core import ..."` | Succeeded |
| Rust/Elixir/Zig/Gleam tests | Could **not** run — toolchains missing |
| Python repos requiring `torch`/`pytest`/`numpy`/`mido` | Could **not** run — `pip` missing |

---

## 7. Bottom line

The `plato-*` cluster is the largest naming family in the account, but it is mostly **speculative mass-produced scaffolding**. The tile/room families in particular are a naming-convention brainstorm, not a microservice architecture. The best new finds are small, well-bounded implementations (`plato-semantic-search`, `plato-engine-block-c`, `plato-room-configs`, `plato-core`); the biggest repos (`plato-training`, `plato-portal`, `plato-tour-guide`) are either un-runnable due to missing/undeclared dependencies or are mostly documentation and generated artifacts.

If forced to pick one repo from the previously unsurveyed 274 to fork, **start with `plato-semantic-search`** because it is the only one that both runs all of its tests right now and matches its README, or with **`plato-engine-block-c`** if the goal is an embedded C runtime. Otherwise, the data supports the negative finding: this cluster does not hide a large, well-tested, ready-to-polish framework beyond what was already found in `plato-vessel-core`.
