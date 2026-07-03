# Deep Research: 11 Named SuperInstance Repos

## Cross-cutting summary

Across all 11 repos, the dominant pattern is **spec-heavy, single-commit sketches** rather than a coherent, working product family. Every repo was created recently (April–June 2026), and all but one have **exactly one commit** in their shallow history. The code that does exist is generally readable, but it is surrounded by far more documentation, vision prose, and AI-generated synthesis artifacts than runnable software.

There is **no shared, coherent "Plato" or vessel/edge framework** binding the four `plato-*` repos together. `plato-vessel-core` and `bare-metal-plato` are effectively byte-for-byte duplicates of the same C HTTP client/MCP stubs; `plato-serialize` defines a binary tile format that nobody consumes; and `plato-vessel-technician` is a documentation-only pitch with zero code. The other seven repos are similarly independent experiments: a Rust bridge to nowhere (`openmind-esp32-bridge`), a `no_std` Rust graph-diffusion toy with stub transports (`grand-pattern-embedded`), a desktop Forth autopilot sketch (`cocapn-forth`), an ESP32 aircraft-display gadget (`ESP32-Plane-Radar`), a Python agent-memory scaffold with mocked ML (`exocortex`), a documentation shell with empty manifests (`openconstruct-hub`), and a large spec warehouse anchored by a small but real C bytecode VM (`Edge-Native`).

The honest conclusion is that these are **not** 11 modules of one product family. They are a collection of concept demos and design briefs that share SuperInstance branding and a fondness for words like "vessel," "edge," "fleet," "room," and "agent." The only pieces with enough working code to be worth follow-up are `Edge-Native` (the VM core), `plato-vessel-core` (the Python room server), and `ESP32-Plane-Radar` (a self-contained Arduino display app). Everything else is either too early or literally empty.

---

## The Plato cluster: `plato-vessel-core`, `bare-metal-plato`, `plato-serialize`, `plato-vessel-technician`

### What it actually is

The four `plato-*` repos look like they should be one subsystem, but they are not. They are four independent snapshots that happen to share the word "Plato."

- **`plato-vessel-core`** is the most complete. It contains a small C HTTP client (`plato_client.c/h`), an MCP-style tool registry (`plato_mcp.c/h`), two ESP32/RP2040 examples, a Python agent script, and a Python "PLATO room server" (`server/plato-room-server.py`) that stores JSON "tiles" with Lamport clocks and a write-ahead log. The C command dispatcher is a stub: it stores an incoming "intelligence" string and increments a level counter, but never executes anything. The server has 75 tests, but the test file imports `plato_room_server` while the source file is named `plato-room-server.py` (hyphen), so tests fail out of the box until the file is renamed.

- **`bare-metal-plato`** is nearly identical to `plato-vessel-core` at the C/Python example level — same file names, same contents, same docs. It lacks the Python server and test suite. It also has committed `__pycache__` artifacts, no `LICENSE` file despite the README claiming MIT, and a CI workflow that runs `make || true` when no `Makefile` exists.

- **`plato-serialize`** is a standalone Rust crate that defines a binary serialization format for "PLATO tiles" (`Tile { id, tile_type, value, confidence, timestamp, layer }`), a 6-byte wire header, and CRC-16. It has 20 unit tests but is not `no_std`, has no C bindings, and is not imported or referenced by either C client or the Python server. Its tile schema does not match the `room/domain/question/answer` JSON tiles used by `plato-vessel-core`.

- **`plato-vessel-technician`** is a pure documentation repo pitching "Deckboss," a voice-first marine technician agent. It describes ESP32 nodes, Jetson hardware, NMEA gateways, PID tuning, and mechanical fail-safes, but it contains zero source files, zero tests, and no dependency manifest.

### Maturity assessment

**Prototype/snapshot tier, with one working server core and three supporting artifacts that do not connect to it.**

- `plato-vessel-core`: the Python server is the most mature piece — it starts, serves JSON, and 75 tests pass after fixing the module import. The C client compiles without warnings. But the MCP dispatch is a stub, there is no persistence, no TLS, and the repo is a single commit.
- `bare-metal-plato`: same C code as `plato-vessel-core` but without the server/tests; CI is broken; repository hygiene is poor.
- `plato-serialize`: a clean, small Rust crate with decent unit tests, but it is an island — nothing consumes it and it is not embedded-ready.
- `plato-vessel-technician`: a spec document, not software.

### Relationship to DeckBoss / purplepincher's direction

The cluster is **thematically close** to "modular and distributed edge development" — each device as a PLATO room, agents discovering capabilities, sensor-to-coordinator tiles — but the implementation does not yet deliver on the pitch. There is no fleet coordination, no mesh, no OTA, no low-bandwidth wire format in use, and no vessel-specific behavior. The DeckBoss concept in `plato-vessel-technician` is a plausible marine application layer, but it would have to be built from scratch on top of `plato-vessel-core`.

### Concrete fork/polish recommendation

**Needs more investigation before a fork, with a strong bias toward consolidating on `plato-vessel-core`.**

Specific open questions:
1. Is `SuperInstance/plato-ng` real and intended to supersede this work? `plato-vessel-core/README.md` points to it as "the full framework."
2. Which repo is canonical — `plato-vessel-core` or `bare-metal-plato`? They are duplicates; one should probably be archived.
3. Is the binary tile format in `plato-serialize` meant to replace the JSON wire format, or is it an abandoned experiment?

If you still want to polish:
- Rename `server/plato-room-server.py` → `server/plato_room_server.py` so tests run out of the box.
- Add the missing `LICENSE` file to both C repos.
- Merge `bare-metal-plato` into `plato-vessel-core` or archive it.
- Replace the stub MCP dispatch with a real callback/function-pointer table and wire the RP2040 LED handlers generically.
- Implement persistence for capability level and behavior across reboots.
- Decide whether `plato-serialize` becomes the wire format; if yes, make it `no_std`, add a C decoder, and teach the Python server to read it.

Do **not** fork `plato-vessel-technician` as-is — there is no code to polish.

---

## `Edge-Native`

### What it actually is

`Edge-Native` is a large specification-and-research seed repo for a distributed edge-robotics platform called NEXUS. The only executable substance is a small, well-structured C bytecode VM under `firmware/nexus_vm/` (~1,400 lines of firmware C), plus COBS/CRC framing and host unit tests. The VM is a static-memory stack machine with 32 opcodes, PID syscalls, and cycle-budget enforcement. Everything else — wire-protocol frame handling, safety state machine, watchdog/heartbeat, sensor/actuator drivers, and the entire Jetson Python SDK — is a TODO placeholder or absent. The repo also contains ~167 Markdown files, a dissertation, simulations, PDFs, JSON schemas, and archived prior work.

### Maturity assessment

**Early prototype / Sprint 0.1 — not yet a working system.**

Evidence:
- 55 host C unit tests pass for the VM and wire-protocol primitives.
- `firmware/main/app_main.c` creates FreeRTOS tasks whose bodies are TODOs.
- 10 firmware source files are placeholder comment headers.
- The Jetson packages are empty one-line `__init__.py` stubs.
- HIL tests are `@pytest.mark.skip` skeletons.
- The shallow clone shows a single commit (2026-04-15), so no iterative development is visible.
- `DOCKSIDE-EXAM.md` leaves every readiness checkbox unchecked.

It is not abandoned — the docs are polished and CI is wired — but it is far from runnable.

### Relationship to DeckBoss / purplepincher's direction

Directly aligned in intent: three-tier edge robotics (ESP32-S3 limbs, Jetson brain, cloud fleet), marine autopilot notes, vessel-platform network docs, and a reflex-bytecode model for deterministic control. But the current code demonstrates none of the integration. It is a research seed for the same problem space.

### Concrete fork/polish recommendation

**Needs more investigation before a production fork; acceptable as a research/seed repo only.**

If you fork, the polish work is specific and large:
1. Finish the firmware runtime: implement `wire_rx.c`, `wire_tx.c`, `msg_dispatch.c`, safety SM, watchdog/heartbeat, E-stop ISR, overcurrent monitor, sensor bus, actuator driver, and I/O poll task.
2. Build out the Jetson SDK: every `jetson/` package is a stub.
3. Replace the empty HIL tests with real hardware-in-the-loop fixtures.
4. Verify an actual ESP32-S3 ↔ Jetson round-trip before claiming the system works.
5. Deepen the clone and inspect full commit/authorship history before committing engineering resources.

The VM core is promising, but the repo is currently a skeleton with a very large README.

---

## `openmind-esp32-bridge`

### What it actually is

A host-side Rust library that encodes typed RPC calls into a small CRC-8-framed binary protocol and sends them over serial (tokio-serial) or WebSocket (tokio-tungstenite). It exposes an `Esp32Bridge::flex()` API, a registry of 10 "chords" (gpio_read/write, i2c_read/write, spi_transfer, pwm_set, adc_read, uart_write, mqtt_publish/subscribe), and ~24 inline unit tests using a mock transport. There is **no ESP32 firmware** — no code that actually toggles GPIO, runs an ADC, or speaks the protocol on a device.

### Maturity assessment

**Prototype / early sketch, not abandoned.**

- Single commit (2026-06-13), so no iteration history.
- The trait design and framing layer are coherent, but several README claims do not match the code: the serial transport uses a fixed retry count with no exponential backoff, and WebSocket "auto-reconnect" is exposed as `reconnect()` but never invoked automatically.
- `usize` serialization silently truncates to `u16`; `String` results are not implemented.
- No serial resynchronization logic; a single noise byte can desynchronize the stream.
- Could not compile locally because Rust is not installed in this environment.

### Relationship to DeckBoss / purplepincher's direction

Directly aimed at the same problem: giving higher-level agents typed access to edge sensors and actuators. As a pure host library without an embedded runtime, it is currently a protocol definition and client SDK, not a working edge node.

### Concrete fork/polish recommendation

**Needs more investigation.** The biggest question is whether the ESP32 firmware side exists in another repo. If yes, this crate could become a useful Rust client. If no, a fork would have to build both halves.

Required polish if you proceed:
1. Build and run `cargo test --all-features` and fix issues.
2. Create or locate the ESP32 firmware counterpart.
3. Fix the `usize` serialization bug and implement `String` results.
4. Add serial resynchronization (drain stale bytes, scan for sync bytes).
5. Reconcile README claims with code (real backoff, auto-reconnect).
6. Add usage examples and integration tests.
7. Resolve the license mismatch (`LICENSE` says MIT; `CONTRIBUTING.md` says MIT OR Apache-2.0).

---

## `grand-pattern-embedded`

### What it actually is

A ~1,300-line Rust `no_std` sketch of a fixed-memory graph-diffusion library. It models a network of "rooms" carrying scalar `f64` "vibes," runs a simple weighted-average predictor per room (misleadingly called "JEPA"), diffuses values along weighted edges, and serializes state into 32-byte "Murmur" messages. All hardware transports (Wi-Fi, BLE, UART, I2C) and storage backends are non-functional stubs. The 29 host-side tests cover graph math and serialization round-trips but do not exercise any hardware.

### Maturity assessment

**Prototype / sketch, very early.**

- Single commit (2026-06-08); no history.
- Every transport and storage backend is a stub.
- Uses `f64` throughout, which is unrealistic for the claimed "Arduino / a few KB RAM" target.
- No message versioning, checksum, or conflict resolution — `process_murmur` blindly overwrites local state.
- No target-specific features or real drivers.
- Could not build locally because Rust is not installed.

### Relationship to DeckBoss / purplepincher's direction

Thematically aligned: fixed-size data structures, no-heap design, small-message gossip, and room/cell topology fit modular edge agents. But only the in-memory graph math exists; the distributed and hardware parts are aspirational.

### Concrete fork/polish recommendation

**Needs more investigation.**

If you proceed:
1. Pick a real target (e.g., `thumbv7em-none-eabihf` or ESP32) and make `cargo build --target …` pass.
2. Replace `f64` with embedded-appropriate types (`f32` or fixed-point).
3. Implement at least one real transport behind the existing trait, or depend on a real crate (`esp-wifi`, `embassy-net`, `arduino-hal`).
4. Add sequence numbers, timestamps, checksums, and merge rules instead of blindly overwriting vibes.
5. Rename or clarify "JEPA" — the current predictor is a weighted moving average.
6. Add target-side examples/tests (QEMU, Wokwi, or real hardware).

Without that work, it is a readable but inert prototype.

---

## `cocapn-forth`

### What it actually is

A very small Forth prototype (~450 lines of Gforth) that implements a handful of control-system primitives under a "CoCapn" / vessel-autopilot framing: a deadband checker, a basic PID heading controller, an integer escalation state machine (`REFLEX → BACKBONE → CORTEX → CLOUD`), a linked-list device registry with capability bitflags, and NMEA 0183 checksum verification. The `parse-gga` word is an empty stub that discards input and returns zeros. There is no embedded code, no timer interrupt, no I/O, and no integration with the PLATO/CoCapn wire protocol.

### Maturity assessment

**Sketch / early prototype.**

- Single commit (2026-06-08); CI literally echoes "No CI configured."
- `make deadband` is documented but the Makefile target does not exist.
- README claims anti-windup, but the PID only clamps final rudder output.
- `parse-gga` is a stub; the test suite only verifies checksums.
- `gforth` is not installed locally, so tests could not be executed.

### Relationship to DeckBoss / purplepincher's direction

The relationship is thematic and naming-only. The repo uses SuperInstance vocabulary (`REFLEX/BACKBONE/CORTEX/CLOUD`, "vessel," "NMEA"), and other repos mention `cocapn`. But `cocapn-forth` itself references no other repo, shares no types, and has no wire protocol or serialization. It is a standalone thought experiment.

### Concrete fork/polish recommendation

**No — not ready.**

It is not a rough implementation; it is a one-commit sketch. Before it could be forked productively, you would need to:
1. Replace the stub `parse-gga` with a real comma-delimited NMEA parser and tests.
2. Add real anti-windup to the PID.
3. Fix the Makefile and wire CI to run `make test` with `gforth`.
4. Port to an actual embedded Forth or clearly label it as a desktop simulation.
5. Add GPIO/timer-driven rudder output, compass/GPS input, and an event loop.
6. Define how it talks to the PLATO/CoCapn fleet.

---

## `ESP32-Plane-Radar`

### What it actually is

An Arduino/PlatformIO firmware for an ESP32-C3 + 1.28" round GC9A01 LCD that displays nearby aircraft on a radar-like screen. The README implies it has an ADS-B radio receiver, but the code actually polls the public `opendata.adsb.fi/api/v3` REST endpoint over HTTPS every 3 seconds using `WiFiClientSecure` with `setInsecure()` (no certificate validation). It projects lat/lon onto a 240×240 round display with range rings, heading triangles, and speed vectors. Range and units are persisted in ESP32 NVS; initial Wi-Fi setup uses a WiFiManager captive portal.

### Maturity assessment

**Prototype / early sketch, close to a single-commit demo.**

- Single commit (2026-06-08); CI is a placeholder that prints "No CI configured."
- No tests.
- README is misleading: no on-device ADS-B receiver; it is a cloud-API display.
- HTTPS is insecure (`setInsecure()`).
- Flat-earth projection becomes inaccurate at larger range presets.
- Could not run a PlatformIO build locally.

The code structure is reasonable (services/UI/hardware separated), but it is a monolithic Arduino sketch with global state and no simulation/stub mode.

### Relationship to DeckBoss / purplepincher's direction

This is a **standalone embedded gadget**, not a modular or distributed system. It has no inter-node communication, no API/RPC/bus, and no connection to the SuperInstance/plato ecosystem beyond a cosmetic `AGENT.md` "fleet neighbors" list. It is edge hardware only in the literal sense.

### Concrete fork/polish recommendation

**Yes — but only as a hobby/polish fork, and only if you correct the product story.**

Specific polish work:
1. Fix the README honesty gap: state clearly that it is a Wi-Fi/cloud-API display, not an ADS-B receiver. If true ADS-B reception is desired, add hardware support (e.g., 1090 MHz SDR or `dump1090-fa` integration).
2. Replace `setInsecure()` with proper TLS certificate pinning or a root CA bundle.
3. Add a real CI build with `pio run` for the `supermini` env.
4. Add off-target unit tests for lat/lon math and a mock HTTP server test for `adsb_client`.
5. Verify display init and show an error screen if the panel is not detected.
6. Improve map projection (local tangent-plane or Haversine).
7. Add a simulation/stub mode so the UI works without internet.

If you need a ready-to-ship project today, the answer is **no**.

---

## `exocortex`

### What it actually is

A Python proof-of-concept / scaffold for an agent memory server. It has a FastAPI facade, an in-memory memory layer with tiered cooling/decay semantics, a toy single-layer neural network stub (`MicroNN`), a Z-score anomaly detector, an asyncio event bus, a Textual TUI, and optional SurrealDB backend code. The "cognitive" features are either mocked or unintegrated: `EMBED` returns random Gaussian vectors, `TRAIN` sleeps 0.1s and reports random accuracy, the `DreamCycle` and `ResonanceEngine` modules are written but not wired into `main.py`, and A2A/MCP toggles in the config have no implementation.

### Maturity assessment

**Prototype / sketch.**

- Single commit (2026-06-08); all files added at once.
- Core ML operations are placeholders.
- Package naming is broken: the source tree is under `src/`, but the Dockerfile and branding refer to `exocortex.main`.
- CI runs `pytest || true` and does not install project dependencies.
- SurrealDB client is imported inside a `try` but not declared as a dependency.
- Tests (`tests/test_core.py`, `tests/test_phase2.py`) look reasonable for the mocked logic but could not be executed locally.

### Relationship to DeckBoss / purplepincher's direction

The repo aspires to fit: it mentions ESP32 support, has a "TAP" (Tiny Agent Protocol) for ESP32, and the fleet catalog places related repos in the Hardware & Edge domain. But this repo has no hardware code, no real edge protocol beyond plain-text HTTP endpoints, and no integration with the local PLATO/vessel/embedded repos. It is a server-side Python sketch that could one day feed an ESP32 client.

### Concrete fork/polish recommendation

**Needs more investigation before forking.**

Required polish if pursued:
1. Replace fake embeddings with a real local embedding model or external embedder.
2. Actually train `MicroNN` on data or remove it.
3. Wire `DreamCycle` and `ResonanceEngine` into `main.py`.
4. Fix packaging: rename `src` to `exocortex`, update `pyproject.toml`, Dockerfile, and CI.
5. Fix CI to install deps, remove `|| true`, add lint/type checks.
6. Declare SurrealDB client as an optional dependency and provide integration tests.
7. Implement or remove A2A/MCP toggles.
8. Add graceful shutdown, optional auth/API key, and configurable CORS.

It is an enthusiastic, well-organized scaffold, but the core functionality is mocked and the runtime is not fully integrated.

---

## `openconstruct-hub`

### What it actually is

A single-commit documentation shell. It contains aspirational Markdown describing OpenConstruct as the "front door" of the SuperInstance ecosystem — a 5-phase agent onboarding system with Shell, Senses, Fleet, Plato rooms, and A2A protocol. There is no implementation: the `Cargo.toml` defines a workspace with all 35 members commented out; `package.json` points to `index.js` and `src/` that do not exist; the `Makefile` delegates to absent submodules; and several "design" files contain raw LLM prompts and tool traces rather than cleaned technical prose.

### Maturity assessment

**Sketch/spec level, not a working prototype.**

- Single commit (2026-05-29) adding only docs.
- Zero implementation files, zero tests, zero CI.
- None of the documented build paths work locally (`cargo install openconstruct`, `pip install openconstruct`, `npm install @superinstance/openconstruct`).

It is closer to a marketing/wiki stub than runnable software.

### Relationship to DeckBoss / purplepincher's direction

The *docs* describe exactly that kind of system: heterogeneous edge fleet with ESP32 sense nodes, Jetson shells, MQTT/gRPC discovery, and Plato deliberation rooms. But the repository itself contains none of that functionality. It is an index and vision document for a broader ecosystem whose actual code lives elsewhere.

### Concrete fork/polish recommendation

**No — do not fork or polish this repo as-is.**

It is not a broken prototype that can be fixed; it is a documentation placeholder with no code. If the goal is to build the OpenConstruct onboarding system, the real work is to create the `openconstruct` crate/package from scratch. If the goal is a documentation hub, strip the LLM-generation artifacts, add real package/workspace structure, add CI, and ensure every doc claim maps to an implemented repo or module.
