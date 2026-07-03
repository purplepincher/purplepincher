# Deep Dive: `exocortex` + the `exocortex-*` / `cortex-*` family

**Scope:** Verify the ecosystem survey's Tier-1 #3 ranking of `exocortex`
("distributed cognitive memory substrate … S3-compatible storage … ESP32
support … genuine multi-repo client family") and the Vision's Watchlist caveat
("the current core is mocked — random embeddings, sleep-and-report-random-
accuracy training"), against the actual source of every `exocortex-*` and
related `cortex-*` repo, cloned fresh (`--depth 1`) into `clones/`.

**Method:** Read every non-binary file in the 15 repos. Compiled and ran the C
test suite locally with `gcc -std=c11`. Pulled GitHub Actions logs via `gh api`
for every repo that has a workflow (independent third-party evidence, not
self-report). No pip was available in this sandbox, so Python test execution
was verified via CI logs rather than local runs — same approach the
`nexus-cluster-deep-dive.md` used. CI was inspected for **all 15** repos, not
just the ones that look real.

---

## 0. Headline finding

The Vision's "core is mocked" caveat is **confirmed and is, if anything,
understated.** Three things are worse than the standing verdict knew, and two
are better. Net: the survey's Tier-1 #3 ranking does **not** survive contact
with the code; the Vision's Watchlist placement does, and is if anything
generous.

**Worse than known:**

1. **The headline "S3-compatible memory" claim is fabricated.** A grep for
   `s3|boto|bucket|object.store|minio` across all 15 clones returns exactly
   **two** hits: the GitHub repo *description* and a copy of it in `AGENT.md`.
   There is **zero** S3 code anywhere. The actual backends are (a) in-process
   Python dicts and (b) a SurrealDB backend (`src/memory/surrealdb_backend.py`)
   — SurrealDB is a document/graph DB, not object storage, and is not
   S3-compatible. This is a feature claim with no implementation behind it,
   advertised at the description layer where skimmers see it.

2. **The test suite for the core repo has never collected a single test, ever.
   The green CI check is a lie.** The workflow is the canonical
   `pytest || true` anti-pattern (`ci.yml:19`) that this research set has
   flagged before, but it's worse than that: pulling the actual CI log for the
   *one and only* run (`run 27128216832`, 2026-06-08) shows
   `collected 0 items / 2 errors` — both test files fail at import with
   `ModuleNotFoundError: No module named 'src'` (a `src/`-layout package that
   is never `pip install -e .`'d and has no `conftest.py` or
   `[tool.pytest.ini_options]` to fix `sys.path`). pytest reports
   `2 errors in 0.44s` and `|| true` turns that 0-tests-run outcome into a green
   check. **The badge claims tests exist; GitHub's own runner proves they have
   never executed.**

3. **One of the six "client family" repos targets endpoints that do not exist
   on the server.** `exocortex-tiny-py` (the CircuitPython ESP32 client —
   squarely the most on-mission of the clients) posts every operation to
   `/api/a2a` and health-checks `/health`
   (`exocortex_tiny/client.py:145,181,215,249,281,313,321`). The Python
   exocortex server exposes exactly **12** routes — eight `/api/v1/*` and four
   `/tap/*` (`src/protocols/__init__.py:75-228`). **There is no `/api/a2a` and
   no `/health` route anywhere in the server.** `Protocol.A2A = "a2a"` exists
   as an enum value in `core/types.py` but is never implemented as an endpoint.
   The tiny-py client cannot talk to its own server. Its 83 tests pass only
   because they're mocked end-to-end with `unittest.mock` (CI log confirms
   `83 passed in 0.56s`), so the mismatch was never caught.

**Better than known (and the survey missed entirely):**

4. **The survey undercounted the family by half.** It listed 6 client repos;
   there are actually **12 `exocortex-*`** repos plus **3 `cortex-*` Rust
   crates** whose descriptions literally read "Rust exocortex crate." The 5
   `exocortex-*` repos the survey never mentioned (`kernel-c`, `embed-mojo`,
   `fleet-chapel`, `memory-zig`, `ast-cpp`) are **real implementations, not
   stubs** — and `exocortex-kernel-c` is the single most credible artifact in
   the entire family for the embedded-C angle purplepincher cares about.

5. **`exocortex-mcp-ts` is a genuine, tested MCP server** — real JSON-RPC 2.0
   over stdio (`src/mcp/server.ts`), real backpropagation
   (`src/compute/micro-nn.ts:72-130`, forward + softmax/cross-entropy gradient
   + ReLU derivative, 100 epochs), real logistic regression (500 epochs), and
   **87/87 tests passing across Node 18/20/22** (CI log, `run 27129328391`).
   This is the only repo in the family whose cognitive core is not mocked. It
   is categorically different from the Python `exocortex`. It is also,
   however, a *standalone reimplementation*, not a client of the Python server
   — see §4.

---

## 1. The real, complete repo list (15, not 6)

```
gh repo list SuperInstance --limit 4200 --json name | jq -r '.[].name' | grep -E '^exocortex|^cortex-'
```

**12 `exocortex-*`** + **3 `cortex-*`** = 15 repos. The survey enumerated 6
(`exocortex`, `exocortex-esp32`, `exocortex-tiny-py`, `exocortex-clients`,
`exocortex-mcp-ts`, `exocortex-wasm-runtime`, `exocortex-script-lua` — actually
7, but described as "surrounded by real client repos" without an explicit
count). **Nine repos were missed**, including the entire `cortex-*` Rust
sub-family and the C/Mojo/Chapel/Zig/C++ reimplementations.

Engagement signal: **0 stars, 0 forks, 0 open issues on all 15 repos.** Every
repo was pushed in a single 21-minute burst on **2026-06-08 between 09:24:46Z
and 09:45:33Z** — the same bulk-generation signature the ecosystem survey
flagged for the wider account. None has been touched since. Sizes are small:
the headline `exocortex` repo is 103 KB on disk; `exocortex-esp32` is 5 KB.

| Repo | Lang | Size | LOC (real code) | CI | Tests (verified) | Survey saw it? |
|---|---|---:|---:|---|---|---|
| `exocortex` | Python | 103 KB | ~2,000 | `pytest \|\| true` | **0 collected** (CI log) | ✓ |
| `exocortex-kernel-c` | C | 31 KB | ~1,068 | `echo "No CI"` | **16/16 pass** (I compiled + ran) | ✗ missed |
| `exocortex-mcp-ts` | TypeScript | 68 KB | ~3,500 | `npm ci && npm test` | **87/87 pass** (CI, 3 Node versions) | ✓ |
| `exocortex-tiny-py` | Python | 31 KB | ~1,500 | `pytest \|\| true` | 83/83 pass (CI) — but mocked end-to-end | ✓ |
| `exocortex-embed-mojo` | Mojo | 27 KB | ~1,100 | `echo "No CI"` | never run | ✗ missed |
| `exocortex-fleet-chapel` | Chapel | 27 KB | ~1,100 | `echo "No CI"` | never run | ✗ missed |
| `exocortex-memory-zig` | Zig | 30 KB | ~1,500 | `echo "No CI"` | never run | ✗ missed |
| `exocortex-esp32` | C++ | 5 KB | 333 | `echo "No CI"` | none | ✓ |
| `exocortex-ast-cpp` | C++ | 9 KB | 884 | `echo "No CI"` | none | ✗ missed |
| `exocortex-wasm-runtime` | C | 10 KB | 341 (+ binary) | `echo "No CI"` | has harness, never run in CI | ✓ |
| `exocortex-script-lua` | Lua | 6 KB | 486 | `echo "No CI"` | none | ✓ |
| `exocortex-clients` | C++/JS | 9 KB | 602 | `echo "No CI"` | none | ✓ |
| `cortex-toml` | Rust | 12 KB | ~250 | `cargo check/test/clippy -D warnings` | pass (CI) | ✗ missed |
| `cortex-bus-protocol` | Rust | 10 KB | ~300 | `cargo check/test/clippy -D warnings` | pass (CI) | ✗ missed |
| `cortex-bus` | Rust | 8.8 MB* | 566 | `cargo check/test/clippy -D warnings` | pass (CI) | ✗ missed |

*\*`cortex-bus`'s 8.8 MB GitHub size is misleading — the working tree is
~36 KB (566-line `src/lib.rs`, a 7-line `Cargo.lock` with zero deps). The
inflation is in git objects from the shallow clone, not vendored code.*

Every repo has an identical structural template: `AGENT.md` (a first-person
"ensign" persona, e.g. *"I watch over exocortex … This is my room"*),
`memory/JOURNAL.md` (a single 14-line "First Watch — Ensign Takes Post" entry
dated 2026-06-08, ending *"Status: Operational · Next duty: Awaiting
instructions"*), `.github/workflows/ci.yml`, and MIT license. This is the
fleet's "agent ensign per repo" convention applied uniformly — useful to know
because it means the presence of AGENT.md/JOURNAL.md is **not** evidence of
maintenance or per-repo attention; it's a template stamp.

---

## 2. Verifying the "mocked core" claim — confirmed, with file:line evidence

The Vision says "random embeddings, sleep-and-report-random-accuracy training."
Direct read of `src/compute/__init__.py` confirms both, exactly as described:

**Embeddings are Gaussian noise** (`compute/__init__.py:113-117`):
```python
if op == Operation.EMBED:
    # Placeholder: in production, call embedding model
    dims = payload.get("dims", 384)
    embedding = [random.gauss(0, 1) for _ in range(dims)]
    # Normalize
    mag = math.sqrt(sum(x * x for x in embedding))
    embedding = [x / mag for x in embedding]
    return {"embedding": embedding, "dims": dims}
```
Every `/api/v1/embed`, `/api/v1/remember`, `/api/v1/recall` call routes here.
Recall then does cosine similarity against these noise vectors — so the
"semantic recall" advertised in the README and demo.py is **mathematically
meaningless**: it returns whichever random-noise memory happened to land
nearby in 384-d space. The `/tap/sense` endpoint is even more candid: it
stores sensor readings with `embedding=[0.0]*384` (`protocols/__init__.py:250`)
— a zero vector — so sensor data can **never** be found by similarity search
at all.

**Training is `sleep(0.1)` + a random accuracy report**
(`compute/__init__.py:148-159`):
```python
if op == Operation.TRAIN:
    # Simulate training
    await asyncio.sleep(0.1)  # placeholder
    ...
    model.trained = True
    model.epochs = epochs
    model.accuracy = random.uniform(0.85, 0.96)
    ...
    return {"model": model_name, "epochs": epochs,
            "accuracy": model.accuracy, "trained": True}
```
`model.trained = True` is a flag flip on a freshly-instantiated `MicroNN`
whose weights are still the Xavier-init random values from `__init__`. The
`MicroNN` class *does* contain a real `forward`/`predict` (softmax + argmax)
and is correctly written — but **`train` is never called on it.** No
backpropagation, no gradient descent, no weight update occurs anywhere in the
Python codebase. `predict` then runs the *untrained* (random-weight) network,
so predictions are noise labeled with a confidence score.

The demo script (`demo.py:50-57`) prints `accuracy: {data.get('accuracy',
0):.1%}` after "training" — which will always print a random value between
85% and 96%. The demo literally demonstrates the fake.

### The tests assert the mock's shape, not its semantics — and don't run anyway

Even setting aside the CI collection failure, reading `tests/test_core.py` in
full shows the tests are **tautological against the mock**:

- `test_embed` asserts `len(result["embedding"]) == 384` — passes because
  `random.gauss` produces 384 floats. Does not verify the embedding is
  semantically meaningful.
- `test_train` asserts `result["trained"] is True` — passes because the mock
  sets the flag. Does not verify any learning occurred.
- `test_predict` trains-then-predicts and asserts `"label" in result` — passes
  because the mock returns the key. The prediction comes from random weights.
- `test_remember_and_recall` stores and recalls with the **same** `[0.1]*384`
  vector — cosine similarity of identical vectors is trivially 1.0. Validates
  the data structure, not retrieval quality.

`tests/test_phase2.py` is more revealing still: **every** SurrealDB test runs
in "fallback in-memory mode" (the file's own header comment says so,
`test_phase2.py:16`). `test_surrealdb_connect_failure` even asserts that
connecting to port 99999 fails. **No test in the suite has ever connected to a
real SurrealDB instance.** The KNN vector SQL, the schema DDL, the tier-
transition queries — all untested against a real backend.

### What *is* genuinely real in the Python core

To be precise rather than dismissive — three things in `exocortex` itself are
real, working, and well-written:

1. **The reflex anomaly detector** (`compute/__init__.py:188-243`,
   `reflex_check`). Real Welford-style running mean/variance, 3σ z-score
   detection. The `test_reflex_arc` test validates actual logic (feed ~20°C
   baseline, fire 100°C, get an anomaly). This is the one algorithm in the
   compute engine that is neither mocked nor tautologically tested.
2. **The `DreamCycle` k-means + the `ResonanceEngine`** (`compute/dream.py`,
   `core/resonance.py`). Real Lloyd's-algorithm k-means with k-means++ init
   (`dream.py:67-158`), real cosine-similarity cross-agent resonance
   detection, half-life decay, pruning. Genuinely thoughtful. **Neither is
   wired into the running server** — `main.py` never instantiates either, and
   no route calls them. They exist as tested-but-unplugged libraries.
3. **The SurrealDB backend** (`memory/surrealdb_backend.py`, 495 lines). Real
   schema DDL, real KNN vector queries with decay-adjusted confidence in
   SurrealQL, graceful in-memory fallback. **Also not wired in** — `main.py`
   instantiates `MemoryLayer()` (the in-memory stub), never
   `SurrealDBMemoryLayer`, and the `config.memory_backend` field that would
   select it is parsed but never read. The optional `surrealdb` package isn't
   in `pyproject.toml` dependencies.

The "shadow rendering" pipeline (`src/shadows/__init__.py`, 176 lines) is also
real — a 6-stage filter→classify→compress→color→render pipeline with templated
glyphs and color-by-confidence classification, driving a Textual TUI. It's
cosmetically the strongest part of the repo. But it renders *the mock's
output* as colored emoji, which arguably makes the oversell worse, not better:
the user sees polished `🏋️ model v2 100ep, 80%→92%` shadows for training that
slept 100ms and rolled a random accuracy.

---

## 3. CI audit — the pattern across all 15 repos

This matters because the Vision's adoption bar explicitly bans
`pytest || true`. Checking every workflow:

| CI pattern | Count | Repos |
|---|---:|---|
| `run: pytest \|\| true` (Python, broken collection) | 1 | `exocortex` |
| `run: pytest \|\| true` (Python, tests actually pass) | 1 | `exocortex-tiny-py` |
| `run: echo "No CI configured — customize per project requirements"` | **9** | `kernel-c`, `embed-mojo`, `fleet-chapel`, `memory-zig`, `esp32`, `ast-cpp`, `wasm-runtime`, `script-lua`, `clients` |
| `npm ci && npm test --if-present` (real, matrix Node 18/20/22) | 1 | `mcp-ts` |
| `cargo check && cargo test && cargo clippy -- -D warnings` (real) | 3 | `cortex-toml`, `cortex-bus-protocol`, `cortex-bus` |

**9 of 15 repos — including the headline `exocortex-esp32` client, the entire
`exocortex-clients` SDK repo, and all five "hidden" reimplementations — have a
CI workflow that literally prints "No CI configured" and exits.** The big
adjectives in their descriptions ("SIMD-accelerated embedding operations,"
"comptime-verified semantic memory store," "PGAS distributed fleet
coordination," "header-only AST decomposition engine") describe code that has
never been compiled or tested by any CI system. Whether they work is a matter
of faith; I verified one of them (kernel-c, below) and it does — but the
advertising vastly outruns the verification.

The `exocortex` core's `pytest || true` is the most damaging instance,
because it produces a green check for a suite that collects nothing. Compare
to `exocortex-tiny-py`, which has the same `|| true` anti-pattern but whose
tests *do* actually collect and pass (83/83, per CI log) — because its
package layout (`exocortex_tiny/` at repo root) is importable without
installation, whereas `exocortex`'s `src/` layout is not. Same bad pattern,
opposite outcomes — which is exactly why `|| true` is banned: it makes CI
outcome meaningless as a signal either way.

---

## 4. Integration audit — the "client family" is mostly independent sketches

The survey's strongest claim about exocortex was "a genuine multi-repo client
family." Mapping what each repo actually targets (read from source, not
READMEs):

| Repo | Talks to the Python `exocortex` server? | Evidence |
|---|---|---|
| `exocortex-esp32` | **Yes** — via `/tap/sense`, `/tap/recall`, `/tap/predict` | `src/main.cpp:75,95,114` |
| `exocortex-clients` (JS) | **Yes** — via `/api/v1/*` and `/tap/*` | `js/exocortex.js:17,26,35,44,53,82,85,86,128,133,143,149` |
| `exocortex-clients` (C++) | **Yes** — same routes as JS | `cpp/exocortex.hpp:87,94,100,106,112,123,131,134,135,165,172,181,187` |
| `exocortex-tiny-py` | **No — targets nonexistent endpoints** | Posts to `/api/a2a` (`client.py:145,181,215,249,281,313`) and `/health` (`:321`); **neither route exists** in `src/protocols/__init__.py` (12 routes, all `/api/v1/*` or `/tap/*`) |
| `exocortex-mcp-ts` | **No — is itself a standalone server** | Own in-process `InsightStore` + `EmbeddingIndex`; never imports or calls the Python server |
| `exocortex-wasm-runtime` | No — standalone C TAP library | In-process test harness only |
| `exocortex-script-lua` | No — standalone in-process Lua modules | `memory_store` is a local Lua table |
| `exocortex-kernel-c` | No — standalone C compute library | No network code at all |
| `exocortex-memory-zig` | No — standalone Zig memory store | In-process allocators |
| `exocortex-embed-mojo` | No — standalone Mojo vector/index lib | No network code |
| `exocortex-fleet-chapel` | No — standalone Chapel fleet sim | PGAS-local |
| `exocortex-ast-cpp` | No — **topically unrelated** | A C++17 recursive-descent parser/AST decomposer; has nothing to do with memory, embeddings, or agents |
| `cortex-toml` / `cortex-bus-protocol` / `cortex-bus` | No — standalone Rust crates | Not in `exocortex`'s dependency tree (confirmed: `grep cortex-bus clones/exocortex` → empty) |

**3 of 12 exocortex-* repos actually interoperate with the Python server.**
1 (`tiny-py`) thinks it does but targets routes that don't exist. **8 are
independent reimplementations** of the same conceptual primitives (memory
store, embeddings, compute kernels) in different languages — the same
"name-a-spec-per-language" pattern the `nexus-cluster-deep-dive.md` found in
the nexus family. `exocortex-ast-cpp` isn't even on-topic.

This is the same finding this research set keeps arriving at: a coherent
*paper architecture* (the docs describe a distributed memory system with a
polyglot client family) spread across repos that are independent one-shot
pushes, never reconciled with each other or with a working integration. The
"client family" framing is accurate if you read only the names; it dissolves
on the grep.

---

## 5. Per-repo character (the parts the survey missed or got wrong)

### `exocortex-kernel-c` — **the strongest single artifact, and the survey never saw it**

1,068 lines of C across 7 `.c` files + 7 headers. Implements micro-NN
(`micro_nn.c`, 203 lines), logistic regression (`logistic.c`, 78), k-means
(`kmeans.c`, 138), **isolation forest** (`isolation.c`, 250 — an anomaly
detector none of the other language ports have), matrix ops, stats, and a
unified `ComputeKernel` union interface (`compute.c`). Includes 5 test files.

I compiled it myself and ran the suite:
```
gcc -std=c11 -Wall -Wextra -Iinclude src/*.c tests/test_*.c -o test -lm
→ exit 0, no warnings
./test → Results: 16 / 16 tests passed
```
The `OK` list covers logistic (AND/bounds/separation), k-means (2-cluster,
predict, 3-cluster), and isolation forest (score range, anomaly detection,
error handling). **This is real, working, dependency-free C that compiles
clean under `-Wall -Wextra`.** Its CI is `echo "No CI configured"` and the
survey never enumerated it. Of everything in the 15-repo family, this is the
one I'd most confidently reuse — and the one whose existence most contradicts
the "client family" framing (it's a peer implementation, not a client).

### `exocortex-mcp-ts` — **real, but a parallel implementation, not a client**

3,500 lines of TypeScript. The MCP server itself is correct JSON-RPC 2.0
(`mcp/server.ts:14-42` handles `initialize`/`tools/list`/`tools/call`/`ping`
with proper `protocolVersion: 2024-11-05`). The `MicroNN` here
(`compute/micro-nn.ts:33-147`) is the **real** version of what the Python
core fakes — actual backpropagation: forward pass, softmax + cross-entropy
output gradient (`outputGrad[k] = probs[k] - (k === targetIdx ? 1 : 0)`,
`:101`), ReLU derivative backprop (`:121`), weight updates over 100 epochs.
`LogisticRegression` does real sigmoid + GD over 500 epochs. The embedding is
hash-based random projection (`memory/embedding.ts:34-56`) — a toy, but
**honestly disclosed** ("fast, deterministic, and requires no ML model") and
deterministic, unlike the Python core's `random.gauss`. CI:
`npm ci && npm test`, matrix Node 18/20/22, **87/87 tests pass**
(CI log `run 27129328391`, 7m31s).

It is the only repo in the family that clears the Vision's adoption bar on
its own terms (verified-working, real CI, honest about its embedding). But:
it has its own `InsightStore`/`EmbeddingIndex`, never calls the Python
`exocortex` server, exposes a different operation set (`notebook_query`/
`notebook_cluster`/etc. vs the Python `embed`/`recall`/`train`), and is
described in its own description as *"the web-native interface to the
notebook runtime"* — referencing a "notebook" concept that doesn't exist in
the Python repo. It's a second product wearing the same name.

### `exocortex-tiny-py` — **real code, real tests, wrong API**

The 841-line README is philosophy-heavy (PLATO terminal metaphor, Brooks
subsumption, Friston active inference, Landauer's principle, a "$3 research
lab democratizes intelligent sensing" pitch, a comparison table with
fabricated-looking AWS/Azure pricing) for ~1,500 lines of code. The client
itself (`client.py`, 322 lines) is genuinely well-written: three-tier HTTP
backend detection (urequests → urllib → requests), "never raises on network
failure — returns None," proper JSON-RPC A2A envelope construction. CI shows
83/83 tests pass.

But every operation posts to `/api/a2a` and the health check hits `/health`,
and **neither endpoint is implemented by the Python server it claims to
target.** The tests don't catch this because they mock the network
end-to-end. The "Sense-Think-Act Loop" architecture diagrammed in the README
cannot actually run against a stock `exocortex` server — the first request
gets a 404. The diagram also assumes a "GitHub Codespaces proxy" hosting
model (`base_url="https://your-codespace.github.dev"`) that the Python
server's own `.cortex.toml` and Dockerfile don't reflect.

This is the most damaging single integration gap, because tiny-py is the
ESP32-tier client purplepincher most cares about, and because the gap is
invisible without reading both sides' source.

### `exocortex-esp32` and `exocortex-clients` — **real, thin, working against the (mocked) server**

- **`exocortex-esp32`** (`src/main.cpp`, 197 lines): real Arduino/ESP32 code.
  Correct TMP36 conversion (ADC → voltage → `tempC = (voltage - 0.5) * 100`),
  correct capacitive soil moisture mapping, real WiFi reconnection logic,
  URL-encoder. Hits the three `/tap/*` endpoints the server actually exposes.
  Has a wiring diagram and both config methods. `platformio.ini` has a
  malformed `lib_deps = linklib = stdc++` line (looks like two keys
  concatenated) but it's not load-bearing. **This client works against the
  server** — but the server's `/tap/recall` returns results from random-noise
  embeddings, and `/tap/predict` runs an untrained network, so "works" means
  "HTTP round-trips succeed," not "the device gets useful intelligence."
- **`exocortex-clients`** (JS 177 lines, C++ 205 lines): clean, idiomatic,
  modern HTTP clients. JS uses private `#post`/`#get` methods and `fetch`;
  C++ is header-only with RAII `CurlHandle`, libcurl + nlohmann/json. Both
  map exactly to the server's `/api/v1/*` and `/tap/*` routes. These are the
  best "if you just want to talk to the API from language X" starting points
  in the family — about 150-200 LOC each, regenerable from the route table in
  an afternoon, but well-done as written.

### `exocortex-ast-cpp` — **off-topic for this family**

538-line header-only C++17 recursive-descent tokenizer + parser producing a
typed AST with source ranges. It's a competent standalone library. But it
has nothing to do with memory, embeddings, agents, or the exocortex concept —
it's been labeled `exocortex-ast-cpp` to fit the family naming convention
without fitting the family's purpose. Treat as unrelated.

### The five language ports (`mojo`/`chapel`/`zig`/`wasm-runtime`/`script-lua`)

Each is a real, standalone implementation of the same primitives in its
language — vector ops + scalar quantizer + random projection in Mojo;
FleetNode/Topology/Consensus/TaskQueue/Heartbeat modules in Chapel;
comptime-generic `TernaryVector(Dim)` + Insight struct + store in Zig (the
comptime-verified claim is legit: `Dim` is a comptime int); a zero-heap TAP
protocol test harness in C (`wasm-runtime/main.c`, 111 lines, with a
committed `tap_test` binary); in-process Lua modules. None is a stub. None
has CI. None talks to anything else. They read as a polyglot exercise —
"implement the same idea in N languages" — which is interesting as portfolio
work and useless as a system, because a polyglot set of standalone libraries
with no shared protocol or integration is N independent codebases to maintain,
not one distributed memory substrate.

---

## 6. Fork/polish recommendations, prioritized

**1. `exocortex-mcp-ts` — fork as a standalone MCP server, decoupled from the
exocortex brand.** It's the only repo in the family with a real, tested,
non-mocked cognitive core and real CI. Gaps before calling it a product:
publish to npm (it's not), the REST server path (`src/rest/`) and MCP-stdio
path can't actually run simultaneously (the `index.ts` "both" mode prints a
notice and only starts REST), and the `notebook_*` tool naming should be
genericized. But this is a real, working, MCP-protocol-correct server with
real backprop. It is what the Python `exocortex` pretends to be.

**2. `exocortex-kernel-c` — fork as a standalone embedded-C compute library.**
16/16 tests pass under `-Wall -Wextra`, zero dependencies, the isolation
forest is unique. Gaps: add actual CI (the `echo` placeholder is a no-op),
pull it out of the `exocortex` namespace (it has no exocortex-specific logic —
it's a clean micro-ML library), document the `ComputeKernel` union interface.
For purplepincher's edge tier this is the most credible building block in the
whole family.

**3. `exocortex-esp32` + `exocortex-clients` (JS/C++) — fork only as a
reference for the `/tap/*` and `/api/v1/*` wire format**, not as maintained
clients. They're ~150-200 LOC each; regenerate from the route table rather
than carry the clones forward. They're honest, working code, but they're
worth more as a spec than as a dependency.

**4. The Python `exocortex` core — do NOT fork as-is.** The mock problem is
real and deeper than the Vision knew: not just mocked embeddings/training,
but a fabricated "S3-compatible" claim, a test suite that has never collected
a single test, a `pytest || true` CI that reports 0-tests-run as green, a
SurrealDB backend that's well-written but unwired and untested against a real
DB, and one client (tiny-py) that targets endpoints that don't exist. If the
**architecture** is wanted — the tiered memory model, the Cortical Bus event
spine, the shadow-rendering pipeline, the dream-cycle consolidation, the
resonance engine — those ideas are genuine and the *designs* (in
`shadows/__init__.py`, `compute/dream.py`, `core/resonance.py`,
`memory/surrealdb_backend.py`) are competent. But adopting the architecture
means re-implementing it on top of a real embedding model and a real storage
backend, not forking this code. The honest framing is: this is a high-quality
**specification with executable mock implementations**, not a working memory
substrate.

**5. Explicit non-recommendations.**
- **`exocortex-tiny-py`** — do not fork until either the client is rewritten
  to hit the server's actual routes, or the server grows an `/api/a2a`
  endpoint that matches what the client sends. Today the two cannot
  interoperate; the 841-line README oversells what the code can do by an order
  of magnitude.
- **`exocortex-ast-cpp`** — off-topic; if wanted, fork as a generic C++ AST
  library under a different name.
- **The five language ports** (`embed-mojo`/`fleet-chapel`/`memory-zig`/
  `wasm-runtime`/`script-lua`) — interesting portfolio pieces, zero CI, no
  integration, no shared protocol. Cherry-pick `exocortex-memory-zig`'s
  comptime-verified `TernaryVector(Dim)` design as a *pattern reference* if
  building a Zig memory store; do not adopt any of them as maintained code.
- **The three `cortex-*` Rust crates** — real cargo CI, standalone, **not in
  `exocortex`'s dependency tree** (confirmed by grep). They're tidy but small
  (250-566 LOC each, zero deps). The "Rust exocortex crate" framing in their
  descriptions is aspirational labeling. Evaluate on their own merits as
  generic Rust libraries; don't expect them to integrate with anything
  exocortex-branded.

---

## 7. Verdict — correcting the standing claims

**Survey (`ecosystem-survey.md` §3, Tier-1 #3):** *"exocortex — distributed
cognitive memory substrate (Python). Persistent multi-agent memory,
S3-compatible storage … ESP32 support … a genuine distributed-edge-memory
system with an embedded tier … Tier 1 — Strong, concrete fork candidate."*

**This verdict does not hold up.** Three of its four headline claims are
false or unverified at the code level: storage is **not** S3-compatible
(zero S3 code; the real backend is in-memory dicts with an untested
SurrealDB fallback), the cognitive core is **mocked** (random embeddings,
fake training), and the "genuine multi-repo client family" is **3 working
clients + 1 broken-target client + 8 standalone reimplementations + 1
off-topic repo**, not an integrated system. Only the "ESP32 support" claim is
defensible, and only at the protocol level (the `/tap/*` endpoints are real
and one ESP32 client genuinely targets them) — the cognition behind them is
the mock. **Recommend demotion from Tier 1.**

**Vision (`VISION_purplepincher.md`, Watchlist):** *"exocortex — distributed
agent memory with an ESP32 tier. The architecture is genuinely on-mission;
the current core is mocked (random embeddings, sleep-and-report-random-
accuracy training). Re-evaluate when the sketchbook versions stop being
placeholders, or adopt architecture-only if we build memory infrastructure
ourselves."*

**This verdict holds up and is if anything understated.** The "mocked core"
characterization is exactly right and is confirmed by direct code read
(`compute/__init__.py:113-117, 148-159`). The "architecture is genuinely
on-mission" half is also right — the tiered-memory/decay/resonance/dream-
cycle design is a sensible distributed-edge-memory architecture, and pieces
of it (the SurrealDB schema, the k-means dream cycle, the resonance engine)
are implemented competently as libraries — they're just not wired into the
running server and not tested against real backends. The Vision's escape
clauses ("re-evaluate when placeholders stop" / "adopt architecture-only if
we build it ourselves") are both still the correct framings.

**Two corrections to add to the Watchlist entry:**

1. The single strongest exocortex-family artifact for purplepincher is **not
   the Python `exocortex` repo at all** — it's **`exocortex-mcp-ts`** (real
   backprop, 87 passing tests, real MCP) or **`exocortex-kernel-c`** (real C,
   16 passing tests I verified myself, isolation forest). The Watchlist
   should name these specifically rather than treating "exocortex" as a
   monolith, because the MCP-ts server is already past the bar the Python
   core fails.
2. The "S3-compatible" framing should be **explicitly marked false** wherever
   it's been carried forward, so it doesn't propagate into purplepincher docs.
   The real storage story is "in-memory dicts with an optional, untested,
   unwired SurrealDB backend" — which is fine for a sketch but is the inverse
   of "S3-compatible."

**Bottom line:** exocortex is the most *interesting* failed candidate in this
research set — a genuinely thoughtful architecture wrapped in a fabricated
feature claim (S3), a non-running test suite advertised as passing, and a
"client family" that's mostly independent reimplementations plus one client
targeting endpoints that don't exist. The Vision's instinct to keep it on the
Watchlist was right; the survey's instinct to rank it Tier-1 was wrong; and
the real fork value, if any, lives in two repos (`mcp-ts`, `kernel-c`) that
the survey didn't even enumerate.

---

### Methodology footnote

- Repo list: `gh repo list SuperInstance --limit 4200` filtered for
  `^exocortex|^cortex-` → 15 repos, cross-checked against the 6 the survey
  enumerated (9 missed).
- All 15 cloned (`--depth 1`) into `clones/`; every non-binary file read.
- Per-repo stats: `gh api repos/SuperInstance/<name>` for size/stars/forks/
  pushed_at; all 15 confirmed 0 stars / 0 forks / single-burst push
  2026-06-08T09:24–09:45Z.
- CI audit: read `.github/workflows/ci.yml` in all 15; pulled run logs via
  `gh api repos/.../actions/runs/<id>/logs` for `exocortex` (run 27128216832),
  `exocortex-mcp-ts` (27129328391), `exocortex-tiny-py` (27129321240), and the
  three `cortex-*` Rust crates. The 9 `echo "No CI"` repos have no run logs to
  pull — confirmed by `gh run list`.
- Local compilation: `gcc 13.3.0`, `exocortex-kernel-c` tests built with
  `gcc -std=c11 -Wall -Wextra -Iinclude src/*.c tests/test_*.c -lm` → 16/16
  pass.
- No pip available in sandbox (same constraint as `nexus-cluster-deep-dive.md`),
  so Python test claims verified via CI logs rather than local `pytest` runs.
- Integration map: read `client.py`/`main.cpp`/`exocortex.js`/`exocortex.hpp`
  in full and compared URL paths against `src/protocols/__init__.py`'s 12
  `@app.get/post` decorators (grep-confirmed line numbers in §4).
- S3 claim: `grep -rin 's3\|boto\|bucket\|minio\|object.store'` across all 15
  clones → 2 hits, both the repo description copied into AGENT.md.
- No 11-named-repo content was read; no edits or pushes made to any repo.
