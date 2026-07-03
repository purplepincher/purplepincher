# Deep Dive: `edge-*` Cluster and `vessel-bridge`

**Scope:** Verify the ecosystem survey's provisional picks (`edge-conservation-rs`, `edge-llama`) and its "vessel-bridge = real thin HAL" characterization, against the actual source of all 10 `edge-*` repos plus `vessel-bridge`, cloned fresh (`--depth 1`) into `clones/`.

**Method:** Read every file that isn't a binary asset. Ran what could be run locally (Python test suites, a live import of `vessel-bridge`, curl against claimed "live reference" URLs). No Rust or C toolchain was available in this environment, so Rust/C++ build claims are verified by static inspection of `Cargo.toml`/`CMakeLists.txt`/CI workflows rather than by compiling — noted explicitly wherever that matters.

---

## 1. Real, complete repo list

```
gh repo list SuperInstance --limit 4200 --json name --jq '.[].name' | grep -E '^edge-|^vessel-bridge$'
```

11 repos, not 10 — the survey's "10 edge-*" count missed one (`edge-boarding-protocol`, `edge-equipment-catalog`, or `edge-native-paper` — whichever it hadn't enumerated). All are forks of `Lucineer/<name>` (same repo names) with a single `origin` remote already pointed at `SuperInstance/<name>`. Git history is thin (1–3 commits per repo in the depth-1 view), and three READMEs carry the commit message `📝 Upgrade README to textbook quality` as their latest change — i.e., the docs were deliberately rewritten to sound more authoritative *after* the code was written, independent of any code change. That commit-message pattern is itself a signal: treat these READMEs as marketing, not documentation.

| Repo | Contents | LOC (real code) | Verdict |
|---|---|---|---|
| `edge-conservation-rs` | Rust crate, conservation-law math checks | 291 (lib.rs) + 184 (tests.rs) | Real code, **broken `no_std` claim** (see §2) |
| `edge-conservation-worker` | Cloudflare Worker (TS), same math as a Worker | 154 (worker.ts) | Real, small, working demo |
| `edge-benchmark` | Rust crate, benchmarking primitives (`no_std` compatible) | 619 (lib.rs) | Real, `no_std` claim plausible (no libm dependency) |
| `edge-compiler` | Cloudflare Worker (TS), "compile/quantize models for hardware" | 336 (worker.ts) | Real scaffolding, **core function is fake** (see §2) |
| `edge-llama` | C++ Jetson inference server + stray C/Python files | ~4,092 across 20 files | Real GGUF parsing, but **the wired-up build never runs a real transformer forward pass** (see §2) |
| `edge-relay-agent` | Pure-stdlib Python cloud↔edge relay/discovery/bandwidth agent + CLI | ~78KB across 5 modules + 708-line test file | Real, tested, **79/79 tests pass locally** |
| `edge-research-relay` | Earlier/parent version of the above, household-specific | ~50KB + 1,352 lines of tests (pytest-only) | Real but not portable (pytest dependency, no offline run possible here); superseded by `edge-relay-agent` |
| `edge-boarding-protocol` | README + LICENSE only | 0 | **Vaporware** — no `worker.js` exists despite README instructing you to deploy one; "Live Reference Implementation" link 404s |
| `edge-equipment-catalog` | README + LICENSE only | 0 | **Vaporware** — no `schemas/` directory exists; "Live Reference Catalog" link 404s |
| `edge-native-paper` | README + PAPER.md (a "research paper") | 0 | **Vaporware** — a marketing/vision document with fabricated-looking metrics ("70+ autonomous vessels," "99.9% uptime over 30 days" on "10 fishing-log vessels on Alaska fishing boats"), zero code, zero data behind the numbers |
| `vessel-bridge` | Python HAL: sensor/actuator abstraction + ESP32 wire protocol | 465 (bridge.py) | Real, runnable, but **inert** — no transport implementation, no firmware, no safety layer (see §3) |

---

## 2. Verifying the survey's two picks

### `edge-conservation-rs` — "no_std, small binary" claim is **not actually true as shipped**

The README (`README.md:3,11,63-65`) leads with `no_std`-compatible, zero-dependency, embeddable-on-Cortex-M/RISC-V framing, and a "Binary Size" section touting `opt-level = "z"`, LTO, strip, single codegen unit.

Reading `Cargo.toml` and `src/lib.rs` together finds a real defect:

- `Cargo.toml` has an **empty `[dependencies]` block** — nothing is declared.
- `src/lib.rs:263-266`, under `#[cfg(not(feature = "std"))]` (i.e. the actual `no_std` path), calls `libm::log(x)` unconditionally:
  ```rust
  #[cfg(not(feature = "std"))]
  fn ln_f64(x: f64) -> f64 {
      libm::log(x)
  }
  ```
  `libm` is never declared as a dependency anywhere. Building with `--no-default-features` (the only way to actually exercise the `no_std` path, since `default = ["std"]` in `Cargo.toml`) would fail to resolve `extern crate libm` / the `libm::` path — a compile error, not a warning.
- `.github/workflows/ci.yml` confirms this was never caught: every job (`check`, `test`, `clippy`, `fmt`) runs with **`--all-features`**, which always turns `std` back on. The `no_std` build path has *never once been exercised in CI*.
- `Cargo.lock` lists zero packages beyond the crate itself, consistent with `libm` never having been fetched.

**Correction to the survey:** the "no_std, small binary" claim is aspirational, not verified, and is currently **broken** for the entropy/KL-divergence functions in the no_std build. The `std`-mode code (the sum/determinant checks, the `EdgeVerifier` accumulator, the hand-rolled JSON) is real and works — 184 lines of passing-looking unit tests exist for it — but the crate's headline differentiator (embedded-deployable, no_std) does not build today. Fixing it is a one-line `Cargo.toml` addition (`libm = { version = "...", optional = true }` gated behind the `no_std` cfg) plus a CI job that actually runs `cargo check --no-default-features`. Cheap to fix, but as-is it's a false claim.

By contrast, `edge-benchmark` (same author, same day, same README-quality-upgrade commit) makes the identical `no_std` claim and it **is** structurally sound: it never calls any transcendental math function, so there's no `libm` dependency to omit. Its CI has the same gap (never tests `--no-default-features`), but nothing in the no_std path would actually fail to compile. If you want a genuinely verified-no_std building block from this cluster, `edge-benchmark` is safer than `edge-conservation-rs` as shipped.

### `edge-llama` — "wraps llama.cpp" is technically true, but the README's "MVP Complete" framing oversells a scattered, only-partially-wired codebase

The repo contains **two unrelated, non-interoperating implementations**, plus a third abandoned attempt, plus stray files (`flato.c`, `edge_native.py`) referencing a downstream project that isn't part of this repo's build:

1. **`edge-cuda.c` + `edge-cuda-impl.cpp`** (the *only* thing `CMakeLists.txt` actually builds, into `edge_cuda`/`edge_test`): this genuinely calls the real llama.cpp public C API — `llama_model_load_from_file`, `llama_tokenize`, `llama_decode`, `llama_sampler_sample`, etc. (`src/edge-cuda-impl.cpp:28-157`). This is real, correct-looking binding code. It would work — **if** the hardcoded library path resolves:
   ```cmake
   set(GGML_LIB "/home/lucineer/.local/lib/python3.10/site-packages/llama_cpp/lib")
   ```
   (`CMakeLists.txt:9,13`) — a personal machine path from a pip-installed `llama-cpp-python`, not portable to any other machine without editing the CMake file.

2. **`main.cpp` + `gguf_loader.cpp` + `backend_inference.cpp` + `model_qwen2.cpp` + `ggml_inference.cpp` + `engine.cpp` + `ggml_ops.cpp` + `server.cpp`** (≈2,600 lines) — the code the README's "What Works ✅" section describes ("GGUF v3 file loading — full metadata parsing, all 339 tensors loaded," "Full Qwen2 transformer architecture — 28-layer attention + FFN"). **None of these files are referenced anywhere in `CMakeLists.txt`.** The binary the README's Usage section tells you to build and run (`./edge_llama path/to/model.gguf`) is not a build target that exists. If you did compile these files by hand:
   - GGUF parsing (`gguf_loader.cpp`) is real, hand-rolled binary parsing — genuinely reads the format correctly.
   - Tensor loading into `ggml_tensor` structs (`backend_inference.cpp:138-278`) is real.
   - But the function that's actually wired to `main.cpp`'s interactive loop, `BackendInference::generate()` (`backend_inference.cpp:280-319`), does **not** run a forward pass. It literally repeats the last input token/byte back at you:
     ```cpp
     // Simplified generation: decode each token and return
     // (full ggml graph forward pass is the next step)
     for (int step = 0; step < max_tokens && step < 50; step++) {
         result += m.tokenizer.decode(all_tokens.back());
         all_tokens.push_back(all_tokens.back());
         ...
     }
     ```
     There is no attention, no matmul, no logits, no sampling anywhere in this path. Its "tokenizer" (`BackendTokenizer`) is a placeholder that maps raw bytes to token IDs by adding 3 — not a real Qwen2 BPE tokenizer.
   - `main.cpp:84-86`: the `serve` command — the exact command the README's Usage section demonstrates (`./edge_llama path/to/model.gguf serve /tmp/edge-llama.sock`) — prints `"server mode not yet implemented"` and returns.

3. **`full_forward.cpp`** (401 lines) — a *third*, more complete attempt at a real ggml-graph-based forward pass (RMS norm → QKV → RoPE → attention → FFN, all via real ggml ops, not naive loops). This is the most credible-looking of the three inference attempts, but it too is **not referenced in any build file**.

4. CI (`.github/workflows/ci-python.yml`) only lints/tests Python (flake8 + `pytest || true`, so failures are silently swallowed) — there is **no CI job that compiles the C/C++ code at all**. Nothing in this repo has ever been verified to build on a machine other than the original author's Jetson.

**Correction to the survey:** "wraps llama.cpp for Jetson" is accurate only for the small, real `edge-cuda.c`/`edge-cuda-impl.cpp` shim (which would need the CMake path parameterized to be portable). The much larger "MVP Complete" GGUF-loader-plus-Qwen2-architecture story in the README describes ~2,600 lines of orphaned code that isn't built by the repo's own build system, and whose one wired-up entry point (`main.cpp`) is a stub that echoes input rather than performing inference. A committed compiled binary (`test_edge`, ARM aarch64, 13.3KB) proves *something* was run on real Jetson hardware at some point, but it corresponds to the `edge-cuda` path, not the README's headline feature list.

**Net verdict on the two picks:** `edge-conservation-rs` and `edge-llama` are still the most *interesting* of the cluster in terms of ambition, but neither is "concrete" in the sense the survey implied. Both have a real, small, working core (the `std`-mode conservation math; the llama.cpp C-API shim) surrounded by either a broken feature claim or a large body of disconnected/non-functional scaffolding. If forking for `purplepincher`, budget time to (a) fix the `edge-conservation-rs` no_std build and add a real no_std CI job, and (b) for `edge-llama`, throw away everything except `edge-cuda.c`/`edge-cuda-impl.cpp`/`edge-cuda.h`, parameterize the CMake paths, and drop the dead GGUF/Qwen2/full_forward code entirely rather than trying to salvage it.

### `edge-compiler` — the Cloudflare Worker scaffolding is real; the core feature is fabricated

Not one of the survey's two picks, but worth flagging since it's in the same cluster and looked plausible from the README ("compile/optimize models for hardware targets"). `src/worker.ts` has real, well-formed request handling, KV job-status tracking, and R2 model storage (`handleCompile`, `processCompilationJob` at `worker.ts:69-166`). But the actual compilation/quantization step calls:
```ts
const result = await env.AI.run("@cf/onnx", { model: ..., options: ... });
...
const quantized = await env.AI.run("@cf/quantization", { model: ..., options: ... });
```
`@cf/onnx` and `@cf/quantization` are not models in Cloudflare Workers AI's catalog — Workers AI only exposes the fixed set of published models (Llama, Whisper, etc.) under `@cf/<vendor>/<model>` naming; there is no generic "run arbitrary ONNX compile/quantize job" binding. This call would fail at runtime with a model-not-found error. The whole "compile/optimize" feature is well-structured scaffolding around an imaginary AI capability.

---

## 3. `vessel-bridge` vs. cocapn-foundation's actual requirements

`cocapn-technical-fit.md` established that none of the 8 previously-reviewed repos deliver cocapn-foundation's core technical substrate (ActiveLog event schema, profile interpreter, safety supervisor with TTL/watchdog/override, voice pipeline), and that `plato-vessel-core` was the closest real building block. `vessel-bridge` was not in that set. Checked directly against the same requirements table:

**What `vessel-bridge` actually is:** a single 465-line Python file (`src/bridge.py`) — the entire repo is 3 files (`LICENSE`, `README.md`, `src/bridge.py`, ~23KB total, not 13KB as the survey estimated but still genuinely thin). It defines dataclasses for sensors/actuators (`SensorReading`, `ActuatorCommand`, `SensorConfig`, `ActuatorConfig`), a `VesselBridge` class exposing `read_sensor`/`command_actuator`/`get_health`/`get_state_json`, an `ESP32Protocol` class implementing a real CRC-8 binary frame format, and hardcoded domain presets (`MARINE_SENSORS`/`MARINE_ACTUATORS`/`AERIAL_SENSORS`/`AERIAL_ACTUATORS`). It imports and runs cleanly (verified locally: `create_marine_vessel()`, `command_actuator()`, `get_state_json()`, and the `ESP32Protocol.encode_sensor`/`decode_frame` round-trip all work as expected).

| cocapn-foundation requirement | Present in `vessel-bridge`? | Evidence |
|---|---|---|
| ActiveLog substrate (append-only JSONL, `(dev,seq)` keyed, merge=set-union) | **No** | `get_state_json()` (`bridge.py:301-311`) emits a point-in-time snapshot dict, not an append-only event stream. No `dev`/`seq` fields anywhere. |
| Voice pipeline / wake word / closed grammar STT | **No** | Zero mentions of voice, STT, wake word, or grammar in the entire repo. |
| Profile interpreter (JSON data files describing electrical interface + named actions + spoken grammar + GUI layout) | **No** | Sensor/actuator "profiles" are Python literals hardcoded in `bridge.py` (`MARINE_SENSORS` etc.), not data-driven JSON, and carry no named-action or grammar concept at all. |
| Safety supervisor: 500ms command TTL | **No** | No timestamp-based expiry check anywhere in `command_actuator()`. |
| Safety supervisor: hardware watchdog | **No** | No watchdog logic; `ESP32Protocol` defines a `FRAME_HEARTBEAT` frame type constant but nothing sends, receives, or times out on it. |
| Safety supervisor: override detect | **No** | No override concept. |
| Momentary-only outputs (fail-safe) | **No** — actively contradicted | `command_actuator()` clamps a persistent `value` into `[min_value, max_value]` and treats `"set"` as a standing command, not a momentary pulse; there is no fail-safe timeout that reverts an actuator if updates stop. |
| C0–C3 command classes | **No** | Only a generic `ActuatorType` enum (motor/servo/thruster/rudder/etc.), no criticality tiering matching cocapn's C0–C3 model. |

Two additional concrete defects found while checking the "safety" angle specifically:

- **The one safety-adjacent code path is dead.** `command_actuator()` (`bridge.py:245-274`) logs a command to `self._command_log` only `if cmd.qos == QoSLevel.SAFETY_CRITICAL`. But `ActuatorCommand.qos` defaults to `QoSLevel.REALTIME` (`bridge.py:148`), and `command_actuator()` never passes a `qos` argument when constructing `cmd` (`bridge.py:256-263`). Given the current API, `cmd.qos` can never be `SAFETY_CRITICAL` — the safety-logging branch is unreachable. Verified live: `command_actuator('thruster_port', 'set', 0.7)` returns `True` and nothing lands in the log.
- **Commands never actually reach hardware.** `command_actuator()` validates and clamps the value, constructs a command object, and returns `True` — but the actual I/O is a stub: `# TODO: Route to transport driver / # self._route_command(cmd)` (`bridge.py:271-272`). This is a pure software-side data model; no UART/I2C/SPI write ever happens. There is also no ESP32 firmware anywhere in the repo (confirmed: `find . -iname "*.ino" -o -iname "*.c" -o -iname "*.h" -o -iname "platformio*"` returns nothing) — same gap pattern `cocapn-technical-fit.md` found in `openmind-esp32-bridge`: host/edge-side code with no device-side counterpart.
- **A live type-confusion bug**, found by actually running the code: `MARINE_ACTUATORS`'s winch entry passes `ActuatorType.RELAY` where a `TransportType.RELAY` was clearly intended (`bridge.py:429`). Confirmed at runtime — `vessel.actuators['winch'].transport` evaluates to `<ActuatorType.RELAY: 'relay'>`, not a `TransportType` member. Harmless today only because nothing reads `.transport` yet; it's a sign this file has never been exercised against a type checker or a real transport implementation.
- **cocapn reference is a label, not a tie-in.** The only occurrence of "cocapn" anywhere in the repo is one word in the README's architecture diagram: `Cloud (cocapn)` (`README.md:8`). No shared schema, no shared server URL, no code that talks to anything cocapn-related — the same "naming/aspiration only" pattern `cocapn-technical-fit.md` found in `Edge-Native`.

**Does this change the earlier verdict?** No, not even partially. `vessel-bridge`'s wire protocol (`ESP32Protocol`, CRC-8 framing, compact binary sensor encoding) is a genuinely clean, small, reusable piece of design — arguably the best-designed single artifact in this whole cluster — but it is a data-model and framing sketch with the actuator I/O explicitly stubbed out, no safety supervisor, no ActiveLog, no voice, and no firmware. It is **less mature than `plato-vessel-core`**, which at minimum has real flashable ESP32/RP2040 C and a working Python server with WAL and Lamport clocks. `vessel-bridge`'s only advantage over `plato-vessel-core` is that its data model (typed sensors/actuators across five vessel domains) is closer in *shape* to what a profile interpreter would eventually need to describe — but shape isn't substance. If anything is worth carrying forward from `vessel-bridge` into a cocapn-aligned fork, it's narrowly the `ESP32Protocol` CRC-8 frame format and the `SensorReading`/`ActuatorCommand` binary encoding — as a component bolted onto `plato-vessel-core`'s more complete stack, not as a foundation in its own right.

---

## 4. Fork/polish recommendations, prioritized

1. **`edge-relay-agent`** (highest confidence pick in this whole batch). Pure-stdlib Python, zero dependencies, 79/79 tests pass locally with nothing but a stock `python3`, has a real CLI (`onboard`/`status`/`serve`/`discover`/`bandwidth`/`route`), and is explicitly a cleaned-up "standalone extraction" of `edge-research-relay` with the household-specific baggage (`VESSEL-SPECIALIZATION.md`'s "Oracle1"/"JetsonClaw1"/named-family-members org chart) removed. This is the one repo in the cluster that is unambiguously what its README claims. Gap: no packaging (`setup.py`/`pyproject.toml`) — trivial to add before forking.

2. **`edge-conservation-worker`** — small, clean, actually-correct Cloudflare Worker (154 lines, no runtime dependencies beyond CF types). Gap: `wrangler.toml`'s KV binding is `id = "placeholder"` — needs a real namespace before it's deployable, and the `/fleet` endpoint returns hardcoded fake numbers that should be cut or clearly marked as illustrative. Otherwise ready to fork.

3. **`edge-conservation-rs`** — fork, but budget a fix pass before calling it "no_std": add `libm` as an optional dependency gated to the `no_std` path, and add a CI job running `cargo check --no-default-features` / `cargo build --no-default-features --target thumbv7em-none-eabihf` (or similar) so the claim is actually enforced going forward. Without that fix, don't repeat the "no_std, small binary" framing — it's currently false.

4. **`edge-benchmark`** — safer no_std claim than its sibling (no libm dependency to omit), same CI gap (never tests `--no-default-features`) but lower risk of being outright broken. Worth a quick manual no_std smoke build before forking, but likely fine.

5. **`edge-llama`, narrowly scoped** — fork only `edge-cuda.c` / `edge-cuda-impl.cpp` / `edge-cuda.h` (the real llama.cpp C-API shim), parameterize the hardcoded `/home/lucineer/...` CMake paths into a `find_package`/environment-variable lookup, and add a CI job that actually attempts a compile (currently there is none). Discard `main.cpp`, `gguf_loader.cpp`, `backend_inference.cpp`, `model_qwen2.cpp`, `ggml_inference.cpp`, `engine.cpp`, `ggml_ops.cpp`, `server.cpp`, `full_forward.cpp` — ~2,700 lines of unbuilt, partially-fake, or duplicate-effort code that would only mislead future contributors if carried forward under the "MVP Complete" framing.

6. **`vessel-bridge`, as a component not a foundation** — worth cherry-picking `ESP32Protocol` (CRC-8 framing) and the `SensorReading`/`ActuatorCommand` binary encoding into whatever eventually becomes cocapn's ESP32 wire layer, grafted onto `plato-vessel-core`'s more complete embedded C + server stack per the existing `cocapn-technical-fit.md` recommendation. Do not fork `vessel-bridge` standalone expecting a usable HAL — fix the dead safety-logging branch, the winch type-confusion bug, and most importantly implement the `# TODO: Route to transport driver` stub (or remove the illusion of hardware control it currently presents) before it's safe to build on.

7. **Do not fork:** `edge-boarding-protocol`, `edge-equipment-catalog`, `edge-native-paper` — all three are README-only with dead "live reference" links (`the-fleet.casey-digennaro.workers.dev/*` returns 404) and zero code. `edge-compiler` — real scaffolding wrapped around a fabricated Workers AI binding (`@cf/onnx`, `@cf/quantization` don't exist); not worth polishing until/unless there's a real model-compilation backend to call. `edge-research-relay` — superseded by its own cleaner extraction (`edge-relay-agent`); fork the child, not the parent.

**Bottom line on the original survey's picks:** `edge-conservation-rs` and `edge-llama` remain defensible picks only if you go in explicit-eyed about what needs fixing first (a one-line dependency fix for the former; a near-total rewrite scoped down to ~400 lines for the latter). If the goal is "concrete, ready-to-polish today," `edge-relay-agent` and `edge-conservation-worker` are the actually-concrete picks this cluster has to offer, and neither was in the survey's original top two.
