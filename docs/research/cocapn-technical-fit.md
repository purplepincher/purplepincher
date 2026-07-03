# Cocapn Technical Fit Assessment

**Scope:** Do SuperInstance's embedded/hardware/vessel-flavored repositories contain real, reusable code that could implement the `cocapn-foundation` vision, or is the connection mostly naming/aspiration?

**Method:** Read `cocapn-foundation` in full, then read the actual code (not just READMEs) of the named vessel/embedded repos, and search for explicit cross-references to cocapn / DeckBoss / ActiveLog.

---

## 1. What cocapn-foundation actually needs technically

`cocapn-foundation` (reference: `clones/cocapn-foundation/project/foundation/`) is a design brief for a voice-first helm assistant and timestamped logger for working fishing vessels. Its concrete technical requirements are:

| Layer | What it actually needs |
|-------|------------------------|
| **ActiveLog substrate** | Append-only JSONL event streams keyed by `(dev, seq)`, merge = set union, media anchored by timestamp, corrections as new events (`activelog-spec/README.md`, `schema/event.schema.json`, `examples/troll-day.jsonl`). |
| **Phone / laptop app** | Wake word "cocapn", on-device command STT against a closed profile grammar, dictation mode with VAD pause segmentation + GPS stamping, profile-generated autopilot GUI (`repos/cocapn-app/README.md`, `snippets/voice_state_machine.ts`). |
| **Helm unit firmware** | ESP32-S3 acting as a *profile interpreter* for autopilot remote ports: contact closures / resistor ladders / NMEA, BLE+WiFi secure link, 500 ms command TTL, hardware watchdog, override detect, momentary-only outputs, C0–C3 command classes (`repos/cocapn-helm-firmware/README.md`, `snippets/autopilot_hal.h`, `snippets/watchdog.cpp`). |
| **Autopilot profiles** | JSON data files (not code) per device describing electrical interface, named actions, spoken grammar, and GUI layout (`repos/autopilot-profiles/README.md`, `comnav-p-series.json`). |
| **Provisioner** | Dockside installer that flashes firmware, guides meter-verified wiring discovery, and runs a SAFETY.md sea trial (`repos/cocapn-provisioner/README.md`). |
| **Sync (optional)** | BYO Cloudflare Workers+R2+D1, append-only log exchange (`repos/activelog-sync/README.md`). |
| **Intelligence** | BYOK chat, label export from time-anchored media, conversational profile drafting (`VISION.md`, `ARCHITECTURE.md`). |

Key non-negotiables: **offline-first**, **ActiveLog event schema**, **profile interpreter**, **safety supervisor with TTL/watchdog/override**, **momentary fail-safe outputs**, **two-mode setup** (dockside / sea).

---

## 2. Per-repo technical fit

### `Edge-Native`

**What it actually contains:** A massive specification- and knowledge-base repo for a general "NEXUS" edge-AI platform. It has a real ESP-IDF firmware tree (`firmware/nexus_vm/`, `firmware/wire_protocol/`, `firmware/safety/`) with a bytecode VM, COBS/CRC-16 wire protocol, watchdog/heartbeat/E-stop code, plus a Python Jetson SDK skeleton. It also has a detailed marine autopilot PID engineering doc and a runnable Python simulation (`autopilot/06_pid_simulation.py`).

**Verdict on cocapn fit:** *Interesting but not there yet — parallel universe.*

- It targets marine autonomy (`vessel-platform/`, `autopilot/01_marine_pid_engineering.txt`), ESP32-S3 reflex nodes, Jetson cognitive layer, and a trust-gated autonomy model. That *sounds* like it overlaps with cocapn's helm/brain stack.
- However, the data and control model is totally different: NEXUS uses a 32-opcode bytecode VM, RS-422 COBS/CRC-16 links, and an INCREMENTS trust algorithm. cocapn-foundation needs an ActiveLog event substrate, a JSON profile interpreter for autopilot remote ports, and a phone-driven BLE/WiFi voice pipeline.
- There is **no ActiveLog implementation**, no profile interpreter, no closed grammar voice pipeline, no provisioner, and no momentary-output autopilot HAL.
- The only explicit cocapn tie is one line in `CHARTER.md`: "Part of the Cocapn Fleet ecosystem".
- The repo is ~99% documentation/specs; the actual firmware is scaffold-level and not tied to cocapn's event or safety semantics.

### `openmind-esp32-bridge`

**What it actually contains:** A Rust **host-side** RPC library (`src/lib.rs`, `src/serial.rs`, `src/websocket.rs`, `src/framing.rs`, `src/registry.rs`) that speaks a CRC-8 framed protocol to an ESP32 over serial or WebSocket. It exposes generic chords: `gpio_read/write`, `adc_read`, `pwm_set`, `i2c_*`, `spi_transfer`, `uart_write`, `mqtt_*`. There is **no ESP32 firmware in the repo**.

**Verdict on cocapn fit:** *Naming coincidence / not actually related.*

- It is a generic remote-GPIO bridge, not a marine autopilot remote emulator.
- No mention of cocapn, DeckBoss, ActiveLog, profiles, or safety supervisor.
- The transport framing is a custom CRC-8 protocol, not ActiveLog.
- Could theoretically be used to toggle relays from a host, but it provides none of the cocapn-specific safety architecture (TTL, watchdog, override, command classes).

### `ESP32-Plane-Radar`

**What it actually contains:** A real, working ESP32-C3 Arduino project (`src/main.cpp`, `src/services/adsb_client.cpp`, `src/ui/radar_display.cpp`, `include/config.h`, `platformio.ini`) that connects to WiFi, fetches aircraft data from `adsb.fi`, and renders it on a round GC9A01 LCD.

**Verdict on cocapn fit:** *Naming coincidence / not actually related.*

- It is an air-traffic display gadget, not a marine system.
- No mention of cocapn / DeckBoss / ActiveLog.
- The only overlap with cocapn is "ESP32 + WiFi + display" — too generic to count.

### `cocapn-forth`

**What it actually contains:** A gforth test suite (`autopilot.fs`, `nmea.fs`, `devices.fs`, `deadband.fs`, `escalation.fs`, `cocapn-test.fs`) implementing a PID autopilot, a partial NMEA 0183 parser, device capability flags, and a 4-tier escalation chain (Reflex → Backbone → Cortex → Cloud). The README explicitly frames it as "CoCapn in Forth" and "Layer 0".

**Verdict on cocapn fit:** *Interesting but not there yet — conceptual exercise.*

- It is the only repo besides the plato-* ones that uses the actual "CoCapn" name.
- The autopilot PID (`autopilot.fs:31-80`) is real and includes heading wraparound and rudder clamping.
- However, it runs under gforth on a desktop (`Makefile`), not on an ESP32. There is no I/O, no BLE/WiFi, no profile interpreter, no ActiveLog.
- The NMEA parser is stubbed: `parse-gga` (`nmea.fs:45-47`) just drops its inputs and returns zeros.
- Verdict: a philosophical proof-of-concept for the *shape* of the lowest layer, not a building block you could flash today.

### `plato-vessel-core` / `bare-metal-plato`

**What it actually contains:** A small, real C library (`plato_client.c`, `plato_client.h`, `plato_mcp.c`, `plato_mcp.h`) for ESP32 and RP2040 that does HTTP to a PLATO server, plus a Python PLATO room server (`server/plato-room-server.py`) with WAL, Lamport clocks, tile lifecycle, and tests. The README (`plato-vessel-core/README.md:24`) explicitly shows a diagram where `plato-vessel-technician` is the "Deckboss" node and uses `fleet.cocapn.ai` as the example server.

**Verdict on cocapn fit:** *Real building block worth forking toward purplepincher — but significant divergence remains.*

- **Why it fits:** It is actual, flashable embedded C with a documented agent-to-device handshake ("turbo-shell" levels 0–4), MCP tool registry, and a server that agents can discover devices through. The repo directly references Deckboss and `cocapn.ai`.
- **Why it's not a drop-in:** The data model is PLATO *tiles* (`room/domain/question/answer`), not ActiveLog events. There is no append-only `(dev, seq)` event log, no media anchoring by timestamp, no voice pipeline, no closed grammar, no autopilot profile interpreter, and no safety supervisor (TTL/watchdog/override).
- `plato_mcp_handle_command` (`plato_mcp.c:88-140`) "upgrades" a device by storing a raw intelligence string and bumping a level counter — it does not interpret or safely sandbox that intelligence.
- The server is more advanced than cocapn-foundation's current repos (WAL, causal clocks, lifecycle states), so the *server-side persistence layer* could be a useful starting point for ActiveLog sync, but the schema would need to be rebuilt.

### `plato-vessel-technician`

**What it actually contains:** Markdown documentation for "Deckboss": voice command reference (`VOICE-INTERFACE.md`), hardware setup (`DECKBOSS-HARDWARE.md`), fail-safe design (`FAILSAFE-DESIGN.md`), and marine workflow (`MARINE-WORKFLOW.md`). It describes a Jetson/RPi brain + multiple ESP32 nodes (rudder, throttle, engine, battery, bilge) with mechanical overrides.

**Verdict on cocapn fit:** *Interesting but not there yet — aspirational docs.*

- The *vision* is almost identical to cocapn-foundation Phase 4 (voice helm, ESP32 nodes, fail-safe). The workflow even creates a room named `cocapn-vessel` (`MARINE-WORKFLOW.md:54`).
- But this repo contains **zero code**. It is a specification of desired behavior, not an implementation.
- Many of its safety ideas (mechanical override cables, break-away shear pins, dual bilge floats) are excellent and align with `SAFETY.md`, but they are not software.

### `plato-serialize`

**What it actually contains:** A Rust `no_std`-capable binary serializer for PLATO tiles: fixed-size 43-byte compact tile encoding, CRC-16, batch encoding (`src/lib.rs`).

**Verdict on cocapn fit:** *Interesting but not directly applicable.*

- Well-engineered for low-bandwidth ESP32 ↔ coordinator links.
- But it serializes PLATO tiles (UUID, tile_type, value, confidence, timestamp, layer), not ActiveLog events (`alv/dev/seq/ts/mono/type/fix/body`).
- No cocapn references.

### `grand-pattern-embedded`

**What it actually contains:** A `no_std` Rust library (`src/lib.rs`, `src/graph.rs`, `src/jepa.rs`, `src/gossip.rs`, `src/transport.rs`) for "mono-vibe" propagation: rooms, JEPA predictors, 32-byte "murmur" gossip messages, and transport stubs for WiFi/BLE/UART/I2C.

**Verdict on cocapn fit:** *Naming coincidence / not related.*

- Generic embedded AI-agent gossip framework; no marine semantics.
- No cocapn / DeckBoss / ActiveLog mentions.
- Transport implementations are stubs (`transport.rs:44-107` just set `ready = true`).

---

## 3. Cross-reference summary

| Repo | Mentions cocapn/DeckBoss/ActiveLog? | Evidence |
|------|-------------------------------------|----------|
| `Edge-Native` | Weak | `CHARTER.md:4` — "Part of the Cocapn Fleet ecosystem" only. |
| `openmind-esp32-bridge` | No | Searched all files; none. |
| `ESP32-Plane-Radar` | No | Searched all files; none. |
| `cocapn-forth` | Yes | README, file names, comments throughout. |
| `plato-vessel-core` / `bare-metal-plato` | Yes | README references Deckboss and `fleet.cocapn.ai`; example code uses `fleet.cocapn.ai:8847`. |
| `plato-vessel-technician` | Yes | Entire repo is Deckboss docs; uses "cocapn-vessel" room name. |
| `plato-serialize` | No | Searched all files; none. |
| `grand-pattern-embedded` | No | Searched all files; none. |

Only three of the eight repos explicitly reference cocapn/DeckBoss. Two of those (`cocapn-forth`, `plato-vessel-technician`) are documentation or desktop exercises; only `plato-vessel-core` has real embedded C code with an explicit cocapn tie.

---

## 4. Overall verdict

**Is there a real, buildable bridge between SuperInstance's scattered embedded work and the cocapn-foundation vision?**

No — not yet.

- **The cocapn-foundation vision is far more mature than the code.** The foundation repo has detailed specs, a safety model, an event schema, and a phased roadmap. The embedded repos are mostly sketches, documentation, or generic libraries.
- **The closest concrete assets are:**
  1. `plato-vessel-core` — real ESP32/RP2040 C client + Python PLATO server that already names Deckboss and `cocapn.ai`. This is the only repo you could realistically fork toward a purplepincher-style prototype, but you would have to replace the PLATO tile model with ActiveLog events and add the voice/profile/safety layers.
  2. `plato-vessel-technician` — a well-aligned Deckboss *requirements* document that could inform cocapn-foundation's Phase 4 expansion, but it has no implementation.
  3. `cocapn-forth` — a clean Forth sketch of the lowest-layer control logic, useful as a teaching/reference artifact, not as firmware.
- **The other five repos do not materially advance cocapn-foundation today.** `Edge-Native` is a parallel mega-spec with different primitives; `openmind-esp32-bridge`, `ESP32-Plane-Radar`, `plato-serialize`, and `grand-pattern-embedded` are either generic or in a different domain.

**Honest bottom line:** The naming theme is real in a few places, but the *technical substrate* required by cocapn-foundation — ActiveLog, the profile interpreter, the safety supervisor, the voice pipeline — is not implemented in any of these repos. The best path would be to treat `plato-vessel-core`'s embedded HTTP/MCP scaffolding and server as a starting point, graft the ActiveLog schema onto it, and then port the safety ideas from `cocapn-foundation/project/foundation/SAFETY.md` and the mechanical-override concepts from `plato-vessel-technician`. That is a substantial fork, not a ready-made bridge.
