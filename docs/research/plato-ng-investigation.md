# plato-ng Investigation — Honest Verdict

**Date:** 2026-07-03  
**Investigated:** `SuperInstance/plato-ng` cloned at `clones/plato-ng/` (depth-1, commit `6d78f41b`).  
**Context:** `plato-vessel-core/README.md` (line 156-158) points readers to `SuperInstance/plato-ng` as the "full framework" whose Loop Room architecture and conservation law "govern all room state."

---

## 1. What plato-ng says it is

`clones/plato-ng/README.md` (39 lines) markets itself as:

> "Next-generation PLATO: Loop Room architecture with the conservation law at its core."

Key claims:
- Conservation law **γ + H = 1.283 - 0.159·log(V)** experimentally verified, integrated into gate pipeline, memory, Refiner, event bus.
- Loop Room architecture: everything is a loop or a single run; three room types (algorithmic, agentic, refiner).
- Running services on ports 8847 (PLATO) and 7777 (MUD), event bus, governance, agent twin memory.
- Connected to sibling repos: platoclaw, fleet-math, fleet-router, plato-mcp, fleet-stack, etc.

The `docs/wheel/plato-ng.md` wheel note (an internal archaeology doc) is more candid:

> "There's a gap between what's documented and what's implemented."
> **EXISTS:** MUD server, conservation law, event bus, governance, expert rooms, game rooms, perpetual daemon, loop room spec.
> **DOES NOT YET EXIST:** Gleam BEAM migration, TIC-80/SCUMMVM adapters, Tripartite Agent system, Hardware Agent chip, federated coupling protocol.

---

## 2. Source structure and actual contents

`clones/plato-ng/` has 809 files in the single visible commit. Top-level layout:

```
core/          conservation.py, gate_pipeline.py, fleet_math/, neural_kernel.py, etc.
docs/          ~120 Markdown/Gleam docs, specs, roadmaps, wheel archaeology notes
deployments/   generated/boilerplate stubs (autoresearch, clay, go, llama.cpp, neovim, numpy, redis, ruff)
expertise/     expert_daemon.py, expert_rooms.py, SPREADER-TOOL.md
games/         checkers_room.py, connect_four_room.py, othello_room.py, tic_tac_toe_room.py
harness/       __init__.py (p,G,K,M harness standard)
lib/           plato_client.py, plato_connect.py, game_base.py, a2ui.py
native/        refiner_room_nif/ (Rust NIF stubs with Cargo.lock + pre-built target/)
prm/           __init__.py (process reward model scoring)
refiner/       __init__.py (trajectory analysis)
research/      Markdown + test_signed_laplacian.py (numpy experiment)
results/       JSON dumps from refiner/git-agent batches
rooms/         sqlite-tile-store.py (SQLite-backed tile database room)
scripts/       fleet-heartbeat.py, git_agent.py, perpetual-daemon-v2.py, startup-daemon.py
services/      28 Python files: MUD, MCP server, bridges, rooms, monitors
src/           refiner_room.gleam (Gleam GenServer stub)
static/        HTML demos, terrain scene JSONs, Penrose browser
```

**Critical observation:** plato-ng contains **client libraries and services that call a PLATO server**, but it does **not contain a PLATO room server**. There is no `plato-room-server.py` or equivalent HTTP server in the repo. Every service hard-codes an external endpoint:

- `lib/plato_client.py:7` → `PLATO_URL = os.environ.get("PLATO_URL", "https://localhost:8847")`
- `services/pubsub.py:20` → `PLATO = "https://localhost:8847"`
- `services/mud_telnet.py:5-7` imports `equipment.plato.PlatoClient` and `equipment.models.FleetModelClient` from a path **outside** the repo (`FLEET_LIB` = parent of repo + `/equipment/`), and expects `repos/plato-mud-server/src` to exist as a sibling.
- `rooms/sqlite-tile-store.py:44` → `PLATO_URL = os.environ.get("PLATO_URL", "http://localhost:8847")`

By contrast, `clones/plato-vessel-core/server/plato-room-server.py` **is** a complete Python HTTP server (`BaseHTTPRequestHandler`, `/submit`, `/room/<name>`, `/retract`, `/supersede`, WAL, Lamport clocks, tile lifecycle) on port 8847.

---

## 3. Git history

The clone was made with `--depth 1`, so history is truncated. The only visible commit is:

```
6d78f41b67d8baa220cb5f4545ea31b059a0ecba
Author: Cocapn Fleet <fleet@cocapn.ai>
Date:   Mon May 18 23:18:27 2026 +0000
    doc: add MIT LICENSE
```

This commit added all 809 files at once. There are no tags. Only branch `main`. Because of the shallow clone, we cannot tell whether this was a true single-commit repo or simply the latest squashed import. The commit message is misleading for an initial-content dump.

---

## 4. Tests and runnable state

- No `requirements.txt`, `setup.py`, or `pyproject.toml` exists, despite `.github/workflows/python-ci.yml` checking for them.
- Only one test-style file: `research/test_signed_laplacian.py` (numpy experiment, not pytest).
- `pytest` is not installed in the environment; `python3 -m pytest --collect-only` fails immediately.
- Core modules do import cleanly: `core.conservation`, `core.gate_pipeline`, `lib.plato_client`, `harness`, `prm`, `refiner`.
- Several services will not run standalone because they import external packages (`equipment.plato`, `services.plato_mud_server` from outside repo) or assume a running PLATO server.

---

## 5. Comparison with plato-vessel-core, bare-metal-plato, plato-serialize

### 5.1 plato-vessel-core vs bare-metal-plato

`clones/bare-metal-plato/` and `clones/plato-vessel-core/` share identical core files:

```bash
diff -q bare-metal-plato/plato_client.c plato-vessel-core/plato_client.c   # identical
diff -q bare-metal-plato/plato_client.h plato-vessel-core/plato_client.h   # identical
diff -q bare-metal-plato/plato_mcp.c    plato-vessel-core/plato_mcp.c      # identical
diff -q bare-metal-plato/plato_mcp.h    plato-vessel-core/plato_mcp.h      # identical
diff -q bare-metal-plato/EMBODIMENT-PROTOCOL.md plato-vessel-core/EMBODIMENT-PROTOCOL.md  # identical
```

They are effectively the same repository content under two names. plato-vessel-core adds `server/plato-room-server.py`, `EDUCATIONAL-VISION.md`, `examples/`, and chapters.

### 5.2 plato-ng vs the C/embedded stack

plato-ng has **zero** ESP32/RP2040/C/MCP-on-device code. Searching the repo for the vessel keywords:

- `turbo-shell` appears only in `docs/wheel/polyformalism-turbo-shell.md` and `docs/wheel/plato-vessel-core.md`.
- `ESP32` / `RP2040` appear only in docs (`docs/for-engineers.md`, `docs/research/HARDWARE-AGENT-CHIP.md`, `docs/wheel/oracle1-vessel.md`).
- The 5-level embodiment protocol (Discover → Assess → Bridge → Confirm → Upgrade) is documented in `docs/wheel/plato-vessel-core.md` but is **not implemented anywhere in plato-ng source**.

### 5.3 plato-serialize

`clones/plato-serialize/` is a Rust crate (`Cargo.toml`, `src/lib.rs`, `src/lib_tests.rs`) implementing a compact binary tile format for low-bandwidth ESP32↔coordinator links. Its tile schema is incompatible with both plato-vessel-core and plato-ng:

```rust
pub struct Tile {
    pub id: Uuid,
    pub tile_type: TileType,   // Sensor/Actuator/Virtual/Composite
    pub value: ValueType,      // Float(f64) | Int(i32)
    pub confidence: f64,
    pub timestamp: u64,
    pub layer: u8,
}
```

plato-vessel-core and plato-ng use JSON tiles with `{domain, question, answer, tags, source, confidence}`. plato-serialize is a sibling experiment, not a serialization layer used by either.

---

## 6. Internal inconsistencies in plato-ng

The flagship conservation law is not even consistent across plato-ng's own docs:

| Location | Formula |
|----------|---------|
| `README.md:8`, `core/conservation.py:6` | `γ + H = 1.283 - 0.159·log(V)` |
| `docs/UNIFIED-THEORY.md:5,10` | `γ + H = C(V) = 0.870 - 0.232/log(V)` |
| `docs/roadmap/conservation-shared.md:12` | "canonical source: `core/conservation.py`" |
| `docs/wheel/plato-ng.md:9` | `γ+H = 1.283 - 0.159·log(V)` |
| `static/terrain-scenes/fleet-health.json` / MUD room description | `gamma + H = 1.364 - 0.159 log(V)` |

So plato-ng itself ships at least three different constants (1.283, 1.364, 0.870) and two different functional forms (`-0.159·log(V)` vs `-0.232/log(V)`).

---

## 7. Honest verdict

**plato-ng is not a canonical superset of plato-vessel-core / bare-metal-plato / plato-serialize. It is not a strict successor that contains and extends them.**

What it actually is:

1. **A partially-implemented Python application framework** built *on top of* an external PLATO server. It assumes a server at `:8847` but does not ship one. The real server implementation lives in `plato-vessel-core/server/plato-room-server.py`.
2. **A collection of docs and aspirational architecture** (Loop Rooms, Gleam/BEAM migration, conservation law, MUD-as-face, agent twins) that is substantially ahead of its runnable code. The wheel notes repeatedly admit what is spec-only.
3. **A sibling repo, not a parent repo**, to plato-vessel-core. It references the vessel work as a separate wheel entry (`docs/wheel/plato-vessel-core.md`) and does not import or embed the C client, the embodiment protocol, or the ESP32/RP2040 examples.
4. **Disconnected from plato-serialize**. The Rust serializer has its own incompatible tile schema and is not wired into plato-ng's client libraries or services.

The relationship is better described as: **plato-ng is a speculative, documentation-heavy, partially-working Python services layer that conceptually aspires to unify the fleet, while plato-vessel-core is the repo that actually contains the runnable C client and PLATO room server.**

If someone wanted a "canonical superset" that includes the embedded stack, the room server, the serialization, and the loop-room framework, that repo does not currently exist in the SuperInstance organization.

---

## 8. Evidence summary (real paths)

| Claim | Evidence |
|-------|----------|
| No PLATO server in plato-ng | `find clones/plato-ng -name "*server*.py"` returns only `plato_mud_server.py`, `plato_mud_bridge.py`, `plato_mcp_server.py`; none implement `/submit` HTTP room server. |
| External PLATO dependency | `clones/plato-ng/lib/plato_client.py:7`, `clones/plato-ng/services/pubsub.py:20`, `clones/plato-ng/rooms/sqlite-tile-store.py:44` |
| plato-vessel-core has the server | `clones/plato-vessel-core/server/plato-room-server.py` (608 lines, full HTTP API) |
| bare-metal-plato == plato-vessel-core core | `diff -q` confirmed identical `plato_client.c/h`, `plato_mcp.c/h`, `EMBODIMENT-PROTOCOL.md` |
| plato-ng docs admit gaps | `clones/plato-ng/docs/wheel/plato-ng.md` "EXISTS / DOES NOT YET EXIST" table |
| Conservation law inconsistent | `clones/plato-ng/core/conservation.py:6` vs `clones/plato-ng/docs/UNIFIED-THEORY.md:5,10` vs MUD room description in `clones/plato-ng/services/mud_telnet.py:53` |
| Single-commit history | `git log --oneline` in shallow clone shows only `6d78f41 doc: add MIT LICENSE` |
| No requirements/tests | no `requirements.txt`; only `research/test_signed_laplacian.py` |
