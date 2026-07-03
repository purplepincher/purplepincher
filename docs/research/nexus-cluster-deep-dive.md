# nexus-* cluster deep dive

Follow-up to the ecosystem survey's light pass on the 18-repo `nexus-*` cluster. That
survey flagged it as "conceptually a textbook distributed-edge stack" but caveated
"most are ~3 KB and were bulk-pushed 2026-04-13 ... treat as a specification to
re-implement, not code to fork verbatim" without reading the code. This pass clones
all 18, reads the actual source (not just READMEs), runs what can be run, and checks
GitHub Actions history where relevant.

All clones live in `/tmp/claude-1000/-home-eileen/c25f18c4-8752-45a5-bacc-1c3e70a86ab7/scratchpad/superinstance-research/clones/nexus-*`
(shallow, `--depth 1`, unmodified).

## 0. Headline finding

The survey's "mostly ~3KB stubs, treat as spec not code" verdict is **correct for 17
of the 18 repos** — confirmed with line counts, and confirmed the "3f+1 byzantine
consensus," "MQTT bridge," "WAL persistence," etc. are all in-memory toy simulations
with no real networking/disk I/O behind the docstrings.

**One repo breaks the pattern entirely: `nexus-runtime`.** It is not a stub. It's a
~9.5 MB, ~840-file, ~190K-line-of-Python (+ 7K lines of C firmware) monorepo with a
working bytecode VM, a real COBS/CRC16 wire protocol, 259 test files, and (verified
independently via `gh run view` against GitHub's own CI logs, not self-reported) **2,559
passing pytest tests** and a **52/52-passing C firmware test suite** that I compiled
and ran myself. This one repo is worth a much closer look than the survey gave it,
and it is a materially different fork candidate than "the nexus-* cluster."

## 1. The complete repo list (18, confirmed complete)

```
nexus-runtime            nexus-aab-protocol       nexus-mission
nexus-data-pipeline      nexus-simulation         nexus-learning
nexus-comms              nexus-security            nexus-integration-paper
nexus-persistence        nexus-node-registry       nexus-hardware
nexus-swarm               nexus-git-agent           nexus-explainability
nexus-energy              nexus-edge-runtime        nexus-fracture-sim
```
Verified via `gh repo list SuperInstance --limit 4200 --json name --jq '.[].name' | grep '^nexus-'`
— exactly 18, matching the survey's implied count. The survey's six named examples
(`nexus-comms`, `nexus-persistence`, `nexus-security`, `nexus-energy`, `nexus-swarm`,
`nexus-runtime`) are all present; the other 12 were not previously enumerated.

Note: all 18 mirror from `github.com/Lucineer/<name>` (visible in the clone output —
`gh repo clone` prints `From https://github.com/Lucineer/nexus-comms` etc.), i.e.
SuperInstance is a fork/mirror host of a `Lucineer` (DiGennaro et al. / "Cocapn
fleet") source-of-truth. Several READMEs (`nexus-fracture-sim`, `nexus-integration-paper`)
self-attribute to "Superinstance and Lucineer (DiGennaro et al.)" and link to a live
Cloudflare Worker at `casey-digennaro.workers.dev` — this is the same "Cocapn/The
Fleet" ecosystem referenced elsewhere in this research, not an independent
SuperInstance-native project.

What each repo actually contains (read from source, not README):

| Repo | What's actually there |
|---|---|
| **nexus-runtime** | Full monorepo: ESP32 firmware (C, FreeRTOS/ESP-IDF layout), a Jetson-side Python agent stack (~50 subpackages: mqtt_bridge, byzantine detection, trust engine, MPC, RL, vision, digital twin, marketplace, compliance/COLREGS, etc.), 50+ hardware board configs, 259 test files, CI workflows. See §3. |
| nexus-data-pipeline | 88-line single-file toy: `StreamProcessor`/`Aggregator`-style in-memory pipeline, `demo()` at bottom. |
| nexus-comms | 77-line single-file toy: `MeshNetwork`, `RelayRouter`, `TopicBridge` — all in-process dicts, no `paho-mqtt`, no sockets. See §4. |
| nexus-aab-protocol | 83-line single-file toy: agent-to-agent behavior/role protocol sketch. |
| nexus-simulation | 69-line single-file toy: generic simulation loop skeleton. |
| nexus-security | 73-line single-file: real stdlib `hmac`/`sha256` MAC + token auth (legit, if minimal), plus a simplistic vote-counting "Byzantine detector" (accusation-threshold heuristic, not a consensus protocol). |
| nexus-persistence | 80-line single-file: `WriteAheadLog`/`SnapshotStore`/`StateStore` — entirely in-memory Python lists/dicts. No file writes, no disk I/O at all despite being called a WAL. |
| nexus-node-registry | 92-line single-file toy: node registration/lookup table. |
| nexus-swarm | 279-line single-file: pheromone field, Reynolds flocking, a `ConsensusType` enum (`BYZANTINE_PAXOS`, `CRDT_MERGE` — enum values only, not implementations). Most substantial of the "toy" tier. |
| nexus-mission | 328-line single-file: mission-phase state machine, contingency/retry logic. Second-most substantial toy. |
| nexus-learning | 91-line single-file: tabular Q-learning (`QTable`) — topically unrelated to nexus-runtime's own `jetson/learning/` (which is about observation recording + A/B testing, no Q-learning). Confirms independent authorship, not a shared module. |
| nexus-integration-paper | No code. A markdown "paper" (`PAPER.md`) plus README. Positioned as a research-paper writeup of the nexus-runtime architecture. |
| nexus-hardware | 71-line single-file toy: generic board-capability descriptor stub — unrelated in content to nexus-runtime's real 50+ board config modules. |
| nexus-git-agent | The one non-Python repo: a 206-line Cloudflare Worker (`src/worker.ts`) with `wrangler.toml` (real `account_id`, real KV binding), a trust-score updater, and a DeepSeek LLM call for "compileReflex." Small but genuinely functional Worker code, no tests. |
| nexus-explainability | 69-line single-file toy: decision-attribution/counterfactual sketch. |
| nexus-energy | 176-line single-file: battery model (temp coefficient, charge/discharge efficiency), solar irradiance approximation, priority-based power budget allocator. Reasonably well-implemented physics-lite model, still single-file/no tests. |
| nexus-edge-runtime | 10 files, ~3K LOC total, one file per subsystem (vm.py, trust engine, wire protocol, safety, fleet coordination, perception fusion, digital twin, navigation, reflex compiler). Explicitly says in its own README: "Covers the same architectural patterns as nexus-runtime but generalized." A condensed rewrite/distillation, not a shared dependency. No tests. |
| nexus-fracture-sim | **2 files total: LICENSE + README.** Zero code. README describes a Cloudflare-Workers-hosted "multi-model swarm research platform" with a live demo URL — none of that is in the repo; it's a spec/pitch document only. |

## 2. Maturity numbers (corrects/confirms the survey's "~3KB" characterization)

| Repo | Files | LOC (approx) | Tests | CI | Last real activity |
|---|---:|---:|---|---|---|
| nexus-runtime | 840 | ~190K Python + 7.2K C | 259 test files, 2,559 pytest + 52 C tests, **externally verified passing** | Yes (3 jobs: pytest ✓, ruff ✗ style-only, firmware-build ✗ CI wiring bug) | Squashed to 1 commit 2026-04-15, but CI run history (`gh run list`) shows real commits back to 2026-04-06 — ~9 days of iteration before the squash |
| nexus-edge-runtime | 10 | ~3.0K | 0 | none | single commit 2026-04-13 |
| nexus-mission | 3 | 328 | 0 | none | single commit 2026-04-13 |
| nexus-swarm | 3 | 279 | 0 | none | single commit 2026-04-13 |
| nexus-energy | 3 | 176 | 0 | none | single commit 2026-04-13 |
| nexus-node-registry | 3 | 92 | 0 | none | single commit 2026-04-13 |
| nexus-learning | 3 | 91 | 0 | none | single commit 2026-04-13 |
| nexus-aab-protocol | 3 | 83 | 0 | none | single commit 2026-04-13 |
| nexus-persistence | 3 | 80 | 0 | none | single commit 2026-04-13 |
| nexus-data-pipeline | 3 | 88 | 0 | none | single commit 2026-04-13 |
| nexus-comms | 3 | 77 | 0 | none | single commit 2026-04-13 |
| nexus-security | 3 | 73 | 0 | none | single commit 2026-04-13 |
| nexus-hardware | 3 | 71 | 0 | none | single commit 2026-04-13 |
| nexus-simulation | 3 | 69 | 0 | none | single commit 2026-04-13 |
| nexus-explainability | 3 | 69 | 0 | none | single commit 2026-04-13 |
| nexus-git-agent | 7 | 206 (TS) | 0 | none | single commit 2026-04-13 |
| nexus-integration-paper | 3 | 0 code (paper only) | n/a | none | single commit 2026-04-13 |
| nexus-fracture-sim | 2 | 0 | n/a | none | single commit 2026-04-13 |

So: the survey's "~3KB, bulk-pushed 2026-04-13" description is accurate for **16 of
18** repos (the 17th, nexus-edge-runtime, is a bit bigger at ~180KB/3K LOC but still
untested and single-commit). **nexus-runtime is the sole outlier**, ~50-250x larger
than every sibling, with a genuinely different development history (iterated over
~9 days per its own CI run log, then squashed) and the only repo in the cluster with
any test coverage or CI at all.

Stub-vs-real-logic ratio in nexus-runtime itself: grepped for `TODO`/`FIXME`/
`NotImplementedError` across all ~190K LOC — **19 hits total**, and every one is at
a physical-hardware boundary that can't be honestly implemented without real
hardware (HIL timing tests explicitly marked `# TODO: Implement HIL ...`, actual
serial deployment to hardware marked TODO, one intentional `NotImplementedError` in
an MPC solver branch that has a matching `pytest.raises(NotImplementedError)` test).
No TODOs found in core logic. This is an unusually honest ratio for this research
set — most other SuperInstance repos surveyed so far bury broken/absent logic behind
confident docstrings; nexus-runtime instead marks the actual gaps.

## 3. Verification performed on nexus-runtime (not just reading)

- **VM executor** (`nexus/vm/executor.py`, 465 lines): loaded and ran a hand-written
  bytecode program (`LOAD_CONST R0,10; LOAD_CONST R1,20; ADD R2,R0,R1; HALT`)
  directly via `python3 -c`. Result: `R2 = 30`, correct.
- **Firmware C code**: compiled `firmware/src/core/wire_protocol/cobs.c` standalone
  with `gcc` and ran a COBS encode/decode roundtrip by hand — byte-correct output,
  matches the reference algorithm exactly.
- **Full C firmware test suite**: the repo's own `tests/firmware/` uses a
  self-contained header-only Unity-style macro harness (not the real Unity library —
  no external dependency needed). Manually reconstructed the two-binary link the
  CMake file specifies (`test_vm_opcodes` has its own `main()`; `test_main.c` links
  the other four test files as externs) and compiled/ran with plain `gcc -std=c11`:
  - `test_vm_opcodes`: **52/52 passed**
  - `test_main` (COBS + CRC16 + frame + HAL integration): **all passed, "All tests
    complete"**
- **Python test suite**: could not run locally (sandbox has no `pip`/network
  package install available), so pulled GitHub's own CI logs instead
  (`gh api repos/SuperInstance/nexus-runtime/actions/jobs/<id>/logs`) for the most
  recent CI run (24434805216, 2026-04-15): **`python-tests` job passed, "2559 passed
  in 3.73s."** This is independent, third-party evidence (GitHub's own runner), not
  a self-report.
- **CI is not fully green**, and it's worth being precise about why:
  - `lint` job fails on `ruff check` — 7,241 style violations (mostly `List` vs
    `list`, line-length, quoted type annotations under `from __future__ import
    annotations`). Cosmetic, not correctness bugs.
  - `firmware-tests` job fails, but for a **CI wiring bug**, not a code bug: the
    workflow passes `-DCMAKE_SOURCE_DIR=$GITHUB_WORKSPACE` to `cmake`, which does
    not override CMake's built-in `CMAKE_SOURCE_DIR` the way the author intended;
    CMake resolves source paths relative to `tests/firmware` instead of repo root
    and fails to find `firmware/src/core/wire_protocol/cobs.c`. I proved the actual
    C code and tests are fine by compiling/linking them myself with the correct
    include paths (above) — the underlying code is not the problem.
  - Net: the one job that tests the actual most-substantial logic (Python) is
    green and externally verified; the two red jobs are lint noise and a build
    script bug, not evidence of broken functionality.
- README accuracy check: nexus-runtime's README claims "2,287 passing tests" via a
  badge; actual CI shows 2,559. Close, and in the *understating* direction rather
  than the overselling pattern seen elsewhere in this research — the README is
  stale, not fabricated.

Given this research's repeated pattern of docs overselling nonexistent code, this
is a rare case where the code substantially exceeds what a skim of the README would
suggest, and where I could independently verify function via compilation/execution
rather than trusting either the README or my own read of the source.

## 4. nexus-comms verdict specifically

The prior survey called this "the best single target if you want one" for MQTT +
mesh. That verdict does **not hold up** for the standalone `nexus-comms` repo:

- 77 lines, one file, no dependency on `paho-mqtt` or any MQTT library, no sockets,
  no `import socket`/`import ssl`, nothing that touches a network.
- `TopicBridge` (the "MQTT bridge") is an in-process Python dict: `subscribe()`
  appends to a list, `publish()` appends to another list and returns the subscriber
  list. No broker, no transport, no serialization format, no QoS.
- `MeshNetwork`/`RelayRouter` implement flood-routing and static-table lookup over
  an in-memory `neighbors: Dict[str, float]` — a reasonable *algorithm sketch* for
  flood routing and next-hop routing, but with no radio/link-layer abstraction, no
  packet loss modeling beyond a bare TTL counter, no persistence.
- Zero tests, zero error handling beyond the happy path, single `demo()` function
  at the bottom that just prints to stdout.

If instead you look at **nexus-runtime's** `jetson/agent/mqtt_bridge/` (572-line
`client.py` + `connection_manager.py`, `message_router.py`, `telemetry_bridge.py`,
`topics.py`, 5 test files) — which is a materially more serious piece of software,
with an `abc.ABC` interface (`MQTTClientInterface`), QoS levels, connection state
machine, and reconnection handling — it is still, by its own docstring, **"Abstract
interface and mock implementation... MockMQTTClient: In-memory mock for testing (no
network required)."** Grepped the entire `mqtt_bridge/` directory for `paho`,
`socket.`, `ssl.` — the only hit is the docstring comment noting that a concrete
implementation "can be paho-mqtt, pure-socket, or the mock client." No `paho-mqtt`
dependency is declared anywhere in `jetson/requirements.txt` or `pyproject.toml`.
**Even the most serious version of the MQTT bridge in this entire cluster has never
been wired to an actual MQTT broker.** It's a well-designed, well-tested interface
+ mock — genuinely useful as an architecture reference, not usable as a working
MQTT client without writing the concrete `paho-mqtt`-backed implementation from
scratch.

Verdict: **nexus-comms (the standalone repo) is not a fork target.** If you want
the MQTT/mesh *concept* in usable form, nexus-runtime's `mqtt_bridge/` package is a
better-designed reference to build the real client against (interface, tests, QoS
model already thought through) — but it is still "spec + mock," not working
MQTT/mesh code, matching the prior survey's caveat almost exactly, just at a higher
tier of polish.

## 5. Is this an integrated stack, or independent sketches?

**Independent sketches, confirmed at the code level, not just by inspection.**

- Grepped all 17 non-nexus-runtime repos for `import nexus` / `from nexus` /
  `import nexus_*` / `from nexus_*` — **zero cross-repo imports found anywhere.**
  Each repo's `src/` only imports its own module plus stdlib (`dataclasses`,
  `typing`, `enum`, `hashlib`, etc.) and, in a couple of cases, `random`/`math`.
- No shared package registry entries, no `pip install nexus-comms` style
  dependency declared by any sibling repo.
- Direct content comparison: `nexus-learning` (standalone, 91-line Q-learning
  sketch) vs. `nexus-runtime/jetson/learning/` (1,203-line observation recorder +
  3,274-line A/B-testing framework) — **completely different topics**, sharing
  nothing but the word "learning." Same pattern would likely hold for the other
  name-overlaps (not exhaustively diffed, but the naming/content mismatch here is
  representative of the pattern already seen elsewhere in this research: names
  chosen for a spec, code written independently and later, no reconciliation).
- The clearest documented evidence of *intent* to integrate — but not actual
  integration — is `nexus-edge-runtime`'s own README, which explicitly says "vs
  SuperInstance/nexus-runtime: covers the same architectural patterns... but
  generalized" and then lists a "Next: Additional Modules" section whose bullet
  points (`src/comms/` — MQTT bridge, mesh networking; `src/energy/` — power
  management; `src/security/` — byzantine fault detection; `src/swarm/` — swarm
  behaviors; `src/learning/`; `src/explainability/`; `src/data_pipeline/`) map
  almost one-to-one onto the other 16 repo names in this cluster. This strongly
  suggests the 17 small repos are placeholder/name-reservation sketches for
  modules that `nexus-edge-runtime` (itself unbuilt beyond 10 files) planned to
  eventually contain — a roadmap expressed as repo names, not a built system.
  nexus-runtime's own internal planning doc (`docs/planning/HIGH_LEVEL_SCHEMA.md`,
  §5.4 "Plugin Ecosystem") independently corroborates this: it lists a *future*
  plugin architecture (`nexus-learning/`, `nexus-vision/`, `nexus-voice/`,
  `nexus-nav/`, `nexus-consensus/`, `nexus-crdt/`, `nexus-skill-loader/`) as
  aspirational — and notably, most of those planned plugin names (`nexus-vision`,
  `nexus-voice`, `nexus-nav`, `nexus-consensus`, `nexus-crdt`, `nexus-skill-loader`)
  don't even correspond to any of the 18 actual repos, further indicating the repo
  names and the architecture docs were never reconciled with each other, let alone
  with working code.
- nexus-git-agent (the Cloudflare Worker) is architecturally the *cloud-side*
  bridge nexus-runtime's docs describe (`docs/planning` mentions a "git-agent
  bridge" under a hypothetical `nexus-fleet/` package), but there is no code-level
  link — no shared types, no API calls from nexus-runtime into nexus-git-agent's
  KV store, nothing importable.

Same pattern this research has found in other SuperInstance clusters: a shared
naming convention and a genuinely coherent *paper architecture* (the docs describe
a real, sensible distributed-edge design), but the repos under that name are
independent one-shot pushes, not a working integrated codebase.

## 6. Overall recommendation

The prior survey's verdict — **"treat as a specification to re-implement, not code
to fork verbatim"** — is **correct for 17 of the 18 repos**, and I'd say it more
bluntly than the survey did: these are single-file demo scripts (69–328 lines each,
zero tests, zero CI, all in-memory, no I/O, no networking) that happen to have
well-chosen names and competent docstrings. `nexus-swarm` and `nexus-mission` are
the most substantial of this tier (279 and 328 lines) and read as reasonable
algorithm sketches (pheromone fields / Reynolds flocking; phase-based mission state
machine) if you specifically want a starting *reference* for those two ideas, but
"fork this" isn't really the operative verb for a 300-line un-tested single file —
you'd rewrite it in the process of using it.

**`nexus-runtime` is a genuine exception and deserves separate treatment from the
rest of the cluster.** It is a substantial, mostly-working, independently-verified
(via GitHub's own CI logs, not self-report) codebase: a real bytecode VM (compiles,
runs, byte-exact), a real COBS/CRC16 wire protocol (compiles, roundtrips correctly),
2,559 passing Python tests and 52 passing C firmware tests, and an honest TODO
footprint concentrated exactly at the hardware boundary you'd expect (HIL tests,
physical serial deployment) rather than scattered through core logic. Its own
"MQTT bridge" is still mock-only (no paho-mqtt wiring), and its CI has two red jobs
(lint-only and a CMake path bug, not a functional break), so it is not
"production-ready" out of the box — but it is a real, working codebase you could
plausibly build on, which is categorically different from forking a 77-line demo
script.

If `purplepincher` wants something from this cluster: **`nexus-runtime` is the only
candidate worth actually cloning as code**, and even then, treat it as "clone and
harden" (fix the CI wiring bug, wire a real MQTT client behind the existing
interface, decide whether the maritime-specific domain logic — COLREGS, sonar,
AIS — is wanted baggage or can be stripped) rather than "clone and ship." Everything
else in the 18-repo cluster — including the specific `nexus-comms` repo the earlier
survey called out as "the best single target" — is confirmed, by reading the code,
to be spec-grade sketches with no networking, no persistence, and no tests behind
the docstrings. The earlier survey's instinct to be skeptical of the cluster as a
whole was right; it just hadn't found the one repo where that skepticism doesn't
apply.
