# SuperInstance Ecosystem Survey (everything beyond the 11 named repos)

**Author:** GLM survey pass · **Date:** 2026-07-03
**Scope:** The 4,095-public-repo `SuperInstance` GitHub *user* account, as it
relates to `purplepincher`'s stated mission ("production-ready tools for
modular and distributed edge development"). The 11 repos in
`KIMI_TASK_11_REPOS.md` are deliberately **not** re-covered here.

---

## TL;DR (read this first)

1. **SuperInstance is one person's (Casey DiGennaro) personal R&D sketchbook,
   not a product portfolio.** The "fleet" is literally a handful of physical
   machines he owns — `Oracle1`, `Oracle2` (ARM64 Oracle free-tier),
   `Forgemaster` (build box), `Pi`, `JetsonClaw1`, a WSL2/RTX-4050 box —
   dressed up in the docs as a "planet-scale fleet." Treat it accordingly.

2. **The 4,095 number is inflated by bulk generation.** Of 4,200 repos
   sampled, **1,423 were last pushed on a single date (2026-06-08)**, 507 on
   2026-06-13, 457 on 2026-04-13. **Nothing is older than ~90 days.** A large
   fraction are scaffolding stubs ("A Rust library for X", <60 LOC). The
   ecosystem is real and creative, but `updatedAt` measures *bulk-push
   bursts*, not maintenance.

3. **The docs oversell dramatically and contradict each other.** README.md
   pitches a Python SDK (`pip install superinstance`), ONBOARDING.md pitches
   a Node/Tminus dispatcher (`npm i @superinstance/tminus-client`), PROFILE.md
   pitches `@superinstance/core/fleet/cli`. I verified publish status:
   - **npm:** only `@superinstance/tminus-client` (1.0.1),
     `@superinstance/tminus-dispatcher` (1.0.1), `@superinstance/sdk` (0.3.0)
     actually exist. `core`/`fleet`/`cli`/`miab`/`ensign` are **not published**.
   - **crates.io:** `cargo install pincher/flux-core/cuda-oxide/cudaclaw/open-parallel`
     (all claimed in ROADMAP.md) — **none are published.**
   - **PyPI:** `superinstance` (the headline install) is **not published**;
     only `cocapn` (0.3.0) and `si-superinstance` (0.1.1) are.
   - **Engagement is near-zero:** ~1 star, **0 forks** on almost every repo,
     including the "flagships."

4. **There is still a small, genuinely-real, purplepincher-relevant core.**
   The best fork candidates are `pincher`, `cocapn`, `exocortex`,
   `git-native-agents`, the `nexus`/`edge`/`vessel` clusters, and the
   `deckboss-*` family (which is already SuperInstance's prototype of
   purplepincher's exact vertical). Detailed shortlist in §3.

---

## 1. What the meta-repo (`SuperInstance/SuperInstance`) actually reveals

### 1a. The navigation map

`SuperInstance/SuperInstance` is the index repo. Its root is the map. Useful
files, in order of value:

| File | What it is | Reliability |
|------|------------|-------------|
| `ONBOARDING.md` | Node.js "hello-agent" walkthrough using `@superinstance/tminus-client` + a WebSocket dispatcher on `ws://localhost:8765`. Defines the protocol lifecycle: **register → subscribe → dispatch → complete → result**. | Honest but narrow — only the Tminus/Node surface. |
| `README.md` | Pitches a *different* product: a Python SDK with `Agent`/`Fleet`/`AgentMemory`, markdown-file memory in `~/.superinstance/agents/`. Says `fleet-metrics` is "Alpha," ternary stack "in active R&D — not yet in this repo." | Honest, modest. |
| `PROFILE.md` | Pitches a *third* product (`npm i @superinstance/core @superinstance/fleet @superinstance/cli`), claims "1,012 crates indexed, 20+ repos, 500+ tests." | Numbers fiction. |
| `ROADMAP.md` | The most informative single doc. Lists "**8 core fleet apps**" (see below) with concrete install commands, and the "365 ternary crates" thesis. Aspirational on packaging, accurate on *intent*. | Use for intent, not for "is it shipped." |
| `VECTOR-INDEX.md` | A subagent-generated "9-channel polyformalism" scoring of ~6 repos. Pseudoscientific framing (γ+η=C "conservation law"), but the *recommendations* (archive Zed/Weaviate forks, split plato-portal) are sane. | Skim for opinions, ignore the math. |
| `KILLER-APPS.md` | 5 cross-cutting demo ideas (Fleet Vital Signs, Bottle Beach, etc.). Good signal for what the author thinks the ecosystem *does*. | Concept only. |
| `CATALOG.md` (623 KB) | Auto-generated catalog, **capped at 2000 repos**, regenerated daily, tags **every** repo 🟢active and groups by "Vessel" (= which physical machine hosts it). | Useless for maturity; useful for the Vessel taxonomy. |
| `INDEXES/{TYPE,LANGUAGE,TOPIC,REALM,CONCEPTS}.md` | Auto-generated cross-indexes (~285 KB each). TYPE (Cli/Library/Framework/…) and REALM are the most useful for discovery. | Auto-gen, good coverage. |

**The "8 core fleet apps" from ROADMAP.md** (the concrete product claims worth
remembering): `tminus-dispatcher`, `message-in-a-bottle`, `repo-ensign`,
`pincher`, `flux-core`, `cuda-oxide`, `cudaclaw`, `open-parallel`. Of these,
`repo-ensign` does **not exist as a repo**; the other 7 do but none are
published to a package registry under their claimed name.

### 1b. How an agent is *meant* to navigate it

The intended workflow, reconstructed from ONBOARDING + VECTOR-INDEX + the repo
names, is:

1. **Install surface = Tminus.** Node client + WebSocket dispatcher. This is
   the only part with a coherent "hello world" and published npm packages.
2. **Agent runtime/memory = Cocapn or SuperInstance-Python or git-native.**
   Three competing, overlapping answers exist (see §1c).
3. **Discovery = the Fleet Vector API**
   (`POST https://fleet-vector-api.casey-digennaro.workers.dev/search`). It
   returns `{name, score, loc, description, github_url, domain, wave}`.
   **Important caveats I verified:** the index is **not** SuperInstance-only —
   it also contains Casey's personal `casey-digennaro/*` repos, and it is
   **polluted with scaffolding stubs** ("A Rust library for Crdt Gset",
   `loc=43`; "A Rust library for Adc Sensor", `loc=1`). **Always filter vector
   results by `loc` (treat <~100 as probable stub) and check `github_url`.**
4. **Indexing = auto-generated markdown.** The 5 `INDEXES/*.md` files and
   `CATALOG.md` are regenerated by fleet CI and are the only realistic way to
   browse 4k repos without API paging. They cap at 2000 (the `gh repo list`
   default) so they undercount.

### 1c. The central contradiction: three different "what is SuperInstance"

The single most important thing to know before forking anything:

- **README.md** = *Python agent-memory SDK* (markdown-file memory, `Agent.ask()`).
- **ONBOARDING.md** = *Node.js Tminus WebSocket fleet* (register/dispatch/complete).
- **PROFILE.md / ROADMAP.md** = *npm `@superinstance/*` package family* (core/fleet/cli + 365 ternary crates).

These are **not** three views of one system — they are three partially-built
products that share a brand. An agent navigating by any single doc gets a
wrong mental model. The numbers (20+ / 1,012 / 1,200+ / 4,095 repos; 500 /
14,000 tests) never reconcile and are auto-generated by different subagents on
different days.

---

## 2. Cluster breakdown (by first token of repo name)

Grouping the 4,200 sampled repos by first `-`-delimited token. "Activity" =
spread of `updatedAt`; remember the whole ecosystem is ≤90 days old, so
"active" here means "touched in the June burst," and only a handful
(`plato-portal`, `plato-dojo`, `cocapn-foundation`, `si`) are touched in the
last 2 weeks.

| Cluster | Count | What it is | Purplepincher fit | Activity |
|---|---:|---|---|---|
| `ternary-*` | 370 | The "ternary computing" library {-1,0,+1}: vectors, tensors, search, sort, PID, ZKP. Mostly Rust. The ecosystem's "mathematical DNA." | Low (research) — exceptions: ternary-search-rs, ternary-pid, ternary-mesh | Very high (core) |
| `lau-*` | 333 | Speculative applied-differential-geometry-for-agents (Kähler/Morse/TQFT/songlines/anyons). **55% have empty descriptions; 312 are Rust stubs.** | None — academic, low-LOC | High volume, thin |
| `fleet-*` | 320 | Orchestration/ops surface: conductor, coordinator, harbor, metrics, dashboard-api, registry-worker, build, scanner, scanner, oracle2. Mixed real (fleet-build, fleet-metrics) + stubs. | Medium (fleet-build/scanner/metrics) | High |
| `plato-*` | 276 | The "Plato" agent/application framework (rooms, tiles, dojos, portal monorepo). Includes 4 of the 11 named repos. | Medium (plato-dojo, plato-portal) | Highest (most recent commits) |
| `flux-*` | 214 | "FLUX" bytecode IR / constraint runtime: flux-core (VM), flux-engine-c (single-header), flux-realm, flux-swarm (Go), flux-bridge. | Medium (portable runtime, but ternary-tied) | High |
| `cuda-*` / `oxide-*` | 143 / 30 | GPU compute: cuda-oxide (Flux→PTX), constraint-checker, sensor-agent; oxide-crdt/workflow/checkpoint. | Low (GPU-specific) | High |
| `agent-*` | 80 | Agent infra: harness-generator, dream-cycle, template, workspace-template, orchestration, groove, semiosis. | Medium (orchestration, templates) | High |
| `cocapn-*` | 77 | The "Cocapn fleet" brand: cocapn (Python agent, **on PyPI**), cocapn-zig (bare metal), cocapn-wasm (browser), foundation, dashboard, cli, sdk, explain, health-rs. | **High** — the agent layer + deckboss link | High (cocapn-foundation Jul 2) |
| `conservation-*` | 58 | The γ+η=C "conservation law" stack in 9+ languages, CLI, explorer, conformance. | Low (signature idea, niche) | High |
| `constraint-*` | 46 | Eisenstein-integer constraint theory: engines in Rust/C/C++/Mojo/MLIR, papers, MCP server, substrate. | Low (research) | High |
| `si-*` / `si` | 45 | SuperInstance-* tooling: `si` dev CLI, si-superinstance (**on PyPI**), si-runtime-go (real Go + tests), si-wasserstein-fleet, si-sheaf-gossip. | Medium (runtime-go, CLI) | Medium-high |
| `grand-*` | 42 | "Grand Pattern" cellular-graph intelligence in Zig/Rust/C/OpenCL/WASM + grand-pattern-embedded (named repo). | Low-medium | Medium |
| `superinstance-*` | 29 | Meta/tooling (harness, sdk, etc.) | Low | Medium |
| `openconstruct-*` | 22 | Multi-language **agent-onboarding SDKs** (Rust/Swift/Ruby/Zig/Python/Jupyter/C-ABI) + catalog. | **Medium-high** (modular onboarding) | Medium |
| `forge-*` | 20 | Tile decomposition for Plato (audio/text/subtitle/sensor → tiles), forge-cli, a2a. | Low (Plato-specific) | Medium |
| `nexus-*` | 18 | **Python edge/IoT stack:** comms (MQTT/mesh), persistence (WAL), security (byzantine), energy (power budget), swarm (consensus), runtime. Mostly 3 KB stubs. | **High intent, low maturity** | Low (Apr 13 burst) |
| `git-*` | 18 | Git-native tooling: git-native-agents, git-storage, git-agent-system, git-graph-rs, git-cuda-agent, git-agentlog. | **High** (git-storage, git-native-agents) | Medium-high |
| `nexus`/`holodeck`/`exocortex`/`vessel`/`edge`/`deckboss`/`crdt`/`snapkit` | 9–12 each | See shortlist. Several directly edge/distributed-relevant. | **High** for deckboss/vessel/edge/crdt/exocortex | Mixed |
| `craftmind-*` | 8 | Minecraft AI games. | None | Low |
| `sketch-*` | 10 | Disposable notes/logs (explicitly documented as such in `sketch-workspace-sketchbook-pattern`). | **None — do not fork.** | High (recent) |

---

## 3. Ranked shortlist for purplepincher fork candidacy

Outside the 11 named repos. Tiered by a combination of (a) fits "modular /
distributed edge," (b) actually has real content vs. stub, (c) isn't a
`sketch-*` note. I read the README + file tree (and for the top few, the
README body) of each.

### Tier 1 — Strong, concrete fork candidates

**1. `pincher` — reflex engine (Rust). ⭐ top pick.**
A "shell" that sits in front of an LLM: embeds every intent into 384-d (via
`sqlite-vec`), fires known reflexes in <50 ms with **no LLM**, asks
confirmation for semi-known, and compiles new reflexes via the LLM only on
miss. Includes a veto/sandbox engine and a portable `.nail` agent-state
bundle. Real Rust project (1.7 MB, has `benches/`, `Cargo.toml`, full docs:
AGENT/API_REFERENCE/ARCHITECTURE/GETTING_STARTED). This is **the namesake of
"purple*pincher*"** and uses the identical hermit-crab/shell metaphor — almost
certainly the intended seed. Polish-to-MVP gaps: (1) publish to crates.io
(ROADMAP claims `cargo install pincher` but it's not there), (2) the README is
philosophy-heavy and example-light — needs a 60-second demo, (3) extract the
veto/sandbox as a pluggable policy trait, (4) document the `.nail` format
spec.

**2. `cocapn` — repo-first agent framework (Python, on PyPI 0.3.0).**
Tiles (atomic Q&A knowledge) → Rooms (self-training collections) → Flywheel
(compounding engine). Pure Python, JSONL storage, zero infra. `pip install
cocapn` **actually works**. Most-engaged repo in the entire ecosystem (3
stars, 1 fork, 4 open issues — tiny in absolute terms but outliers here).
Family: cocapn-sdk (model-agnostic LLM), cocapn-explain, cocapn-health-rs,
cocapn-lessons, plus `cocapn-zig` (bare-metal) and `cocapn-wasm` (browser)
showing a credible multi-target story. The `cocapn-foundation` repo (pushed
Jul 2) is literally "fishing vessel voice assistant design export" — the
**deckboss/purplepincher vertical already prototyped inside SuperInstance**.
Polish gaps: (1) split the sprawling family into a documented core + optional
extras, (2) `cocapn-cli` is referenced but **not on PyPI** — publish it,
(3) add persistence backends beyond flat JSONL, (4) real test coverage
numbers.

**3. `exocortex` — distributed cognitive memory substrate (Python).**
Persistent multi-agent memory, **S3-compatible** storage, "shadow rendering,
tiered compute, **ESP32 support**." Structured Python project (`src/`,
`tests/`, `demo.py`, `docker-compose.yml`, `Dockerfile`, `.cortex.toml`).
Surrounded by real client repos: `exocortex-esp32` (sensor node), 
`exocortex-tiny-py` (CircuitPython/ESP32), `exocortex-clients` (C++/JS SDKs),
`exocortex-mcp-ts` (MCP server + REST), `exocortex-wasm-runtime`,
`exocortex-script-lua`. This is a genuine **distributed-edge-memory** system
with an embedded tier — squarely purplepincher. Polish gaps: (1) README is a
stub (badges + related-repos list, no real usage docs), (2) needs a "deploy
one server + one ESP32 node" quickstart, (3) formalize the S3 compatibility
surface.

**4. `git-native-agents` — zero-dependency fleet orchestration (POSIX shell).**
Multi-agent coordination built **entirely on git primitives**: each agent is
a repo, messages are files committed to the recipient's `inbox/`, memory is
`memory/*.txt` + git tags, "thought branches" for speculative exploration,
merge-based consensus. No broker, no DB, no scheduler — git is the infra.
Explicitly scoped to 5–50 agents (then O(N²) routing → move to a broker).
This is the cleanest expression of "modular + distributed + auditable" in the
whole ecosystem, and it's small enough to harden fast. Polish gaps: (1)
re-implement `orchestrator.sh` in Rust/Go for portability + safety, (2) add
concurrency/locking (concurrent `tick` races are likely today), (3) auth
model for cross-machine agents, (4) tests.

### Tier 2 — Strong clusters / investigate-before-fork

**5. `deckboss-*` family (9 repos) — the existing purplepincher vertical.**
`deckboss-net` (commercial fishing ops: vessel tracking, fuel monitoring — TS,
Jun 14), `deckboss-hardware` ("preloaded Jetson units for the Cocapn
ecosystem"), `deckboss-agent`, `deckboss-marketplace`, `deckboss-ai`,
`cocapn-foundation` (voice-assistant design). This is **SuperInstance's own
prototype of what purplepincher is meant to become.** First thing to do:
diff `SuperInstance/deckboss-*` against the real `purplepincher/deckboss` to
see what's already been carried over and what hasn't. Likely the highest-
signal cluster for purplepincher specifically.

**6. `nexus-*` cluster (18, Python) — edge/IoT stack by intent.**
`nexus-comms` (MQTT bridge + mesh networking), `nexus-persistence` (WAL +
snapshots), `nexus-security` (byzantine fault detection, encryption),
`nexus-energy` (battery + solar + power-budget), `nexus-swarm` (consensus,
pheromone, emergence), `nexus-runtime`, `nexus-comms`. **Conceptually a
textbook distributed-edge stack** — exactly purplepincher's territory.
**Caveat: maturity is low** — most are ~3 KB and were bulk-pushed 2026-04-13.
Treat as a *specification* to re-implement, not code to fork verbatim. Best
single target if you want one: `nexus-comms` (MQTT + mesh).

**7. `edge-*` cluster (10) — edge-tier runtimes.**
`edge-conservation-rs` (**`no_std`, small binary** for edge), 
`edge-conservation-worker` + `edge-benchmark` (**Cloudflare Workers**
compatible), `edge-llama` (C shared lib wrapping **llama.cpp for Jetson**),
`edge-compiler` (compile/optimize models for hardware targets),
`edge-relay-agent`. Several are **Cloudflare-Workers-aligned**, which is a
clean edge story. Caveat: the "conservation law" branding is baggage; the
`no_std`/Workers/Jetson framing is the valuable part. Pick `edge-conservation-rs`
(no_std edge verifier) and `edge-llama` (Jetson inference) as the two most
concrete.

**8. `vessel-bridge` + `vessel` — hardware abstraction + ops console.**
Note these are **two different things**: `vessel-bridge` is the actual
**hardware HAL** ("ESP32 to Jetson to Cloud — unified sensor" layer, 13 KB,
thin) — the modular-edge abstraction purplepincher wants; the `vessel` meta
repo is a charming MUD-style **interactive ops console** ("walk into the
engine room, grab the throttle"). The HAL is the fork target; the console is
a nice-to-have UX layer. Also see `vessel-equipment-agent-skills` (a
"vessel(hardware) + equipment(input code) + agent" 4-layer model — good
architecture doc even if code is thin).

**9. `fleet-build` + `fleet-scanner` — real dev CLI tools (Rust).**
`fleet-build`: automated Rust-crate build/test/fix/push CLI (312 LOC, has
`tests/`); `fleet-scanner`: scans a directory of git repos and emits a health
report. Both are practical, tested, and squarely "modular tooling." Easy wins.

**10. `agent-orchestration` — fleet role/section assignment (Rust).**
"Orchestration for agent fleets — role assignment, section balancing," 347
LOC, `src/` + `tests/` + `memory/`. Real, small, on-mission.

**11. `openconstruct` — multi-language agent-onboarding SDKs (22 repos).**
Rust/Swift/Ruby/Zig/Python/Jupyter SDKs + a **C ABI** ("any language that can
call C can onboard") + a tech/module **catalog**. The mission is
"plug-and-play shell commands to create fully functional agent workspaces."
This is literally a **modular onboarding standard** — strong fit. Caveat:
`openconstruct-rust` is 36 MB (bloated, probably vendored assets/binaries) —
audit before adopting.

### Tier 3 — The math foundation (R&D; a few reusable primitives)

**12. Selected `ternary-*` primitives.** The 370-crate ternary library is
mostly research, but a few are genuinely reusable building blocks:
`ternary-search-rs` (axum + rayon vector-search server — a real service),
`ternary-pid` (PID controller with ternary output — **directly relevant to
edge robotics/vessel control**), `ternary-mesh` (dynamic agent mesh
networking). Fork these *individually*, decoupled from the ternary ideology.

**13. `flux-core` — portable bytecode runtime (Rust, 197 KB).**
VM + assembler + disassembler + A2A. A portable "agent IR." Real and
structured. Relevant as a **modular portable runtime**, but tightly coupled
to the FLUX/ternary paradigm — only adopt if you buy that paradigm.

**14. `si-runtime-go` — Go fleet runtime (real, with tests).**
`agent.go`, `capability.go`, `cell.go`, `conservation.go`, `fleet.go`, each
with `_test.go`. A second-language reference implementation of the fleet
runtime. Useful as a clean-room spec; the `si` shell CLI itself is a thin
(`si.sh`, 10 KB) wrapper.

**15. `fleet-metrics` + the `*-guardian`/`*-budget` pattern.**
`fleet-metrics` (TypeScript, real-time metrics) plus a practical sub-pattern
across the catalog: **budget/token guardians for AI coding CLIs** —
`codex-budget-guard`, `build-guardian`, `dify-budget-watchdog`,
`conservation-guardian`, `cache-guardian-c`. These are *real, useful,
production-minded dev tools* (enforce token/time/build budgets on Codex/Dify
workflows). Understated and genuinely good — a coherent mini-product family
worth pulling out.

### Explicit non-recommendations (save the effort)

- **All `sketch-*` (10):** disposable notes; `sketch-workspace-sketchbook-pattern`
  documents this explicitly. Don't fork.
- **Most `lau-*` (333):** 55% empty-description, speculative differential-
  geometry/topology (Kähler manifolds, Morse theory, TQFT anyons, songlines).
  Research poetry, not tooling.
- **Auto-generated stubs** ("A Rust library for Crdt Gset", loc 39–60):
  `crdt-gset/gcounter/gset/orset/pnvector/lwwreg`, the `plugin-*` stubs,
  `adc-sensor`/`spi-device`/`uart-bridge` sensor stubs. Scaffolding only.
  Note: the **real** CRDT work is `crdt-sync` (TS, repo-native fleet sync) and
  `crdt-core` — fork those, not the named-type stubs.
- **`craftmind-*` (8):** Minecraft AI games. Out of scope.
- **Most `constraint-*`/`conservation-*` math (100+):** research, not edge
  tools (exceptions called out in Tier 3).
- **`cuda-*`/`oxide-*` (170+):** GPU-specific. Skip unless purplepincher does
  GPU edge compute.

---

## 4. What surprised me / changed my understanding

1. **The "4,095 repos" headline is misleading by ~1 order of magnitude.**
   The bulk-push distribution (1,423 repos on one date, ~3,400 across ~12
   dates, nothing older than 90 days) plus the stub pattern means the *real*
   actively-shaped surface is maybe a few hundred repos, and the genuinely
   shippable core is a few dozen. Plan effort against the few dozen, not 4k.

2. **The published-package reality is the inverse of the docs' claims.** The
   ROADMAP/PROFILE loudly advertise `@superinstance/core`, `cargo install
   pincher`, `pip install superinstance` — almost none of which exist on a
   registry. The *actually installable* artifacts (`@superinstance/tminus-*`,
   `@superinstance/sdk`, PyPI `cocapn`, PyPI `si-superinstance`) are the quiet
   ones. **Verify every "install" claim against the registry before believing
   it.**

3. **"Vessels" are physical machines, not abstractions.** Oracle1/Oracle2/
   Forgemaster/Pi/JetsonClaw1 are Casey's boxes (Oracle = ARM64 free-tier,
   Forgemaster = build box, Jetson = edge GPU). The "planet-scale fleet" is a
   home lab. This is fine — but it reframes "fleet coordination" from
   "distributed-systems infrastructure" to "one person's multi-machine task
   routing." The reusable ideas (git-native coordination, conservation
   budgeting, reflex shells) survive the reframe; the grand scale does not.

4. **The center of gravity is math/physics research, not edge tooling.**
   Clusters `ternary`(370) + `lau`(333) + `conservation`(58) +
   `constraint`(46) + `spectral`(24) + the `si-*` math(45) ≈ **876 repos
   (~21%) are research** — Eisenstein lattices, sheaf cohomology, γ+η=C
   conservation laws, tropical geometry. The edge/distributed tooling
   purplepincher wants (`nexus`, `vessel`, `edge`, `deckboss`, `exocortex`,
   `crdt`, `git-native`, `fleet` tooling) is real but is a *minority* of the
   surface. purplepincher will be cherry-picking from a research sketchbook,
   not adopting a platform.

5. **The ecosystem is one fishing-vertical deep.** The `deckboss-*`/`cocapn-*`
   family (fishing fleet ops, vessel tracking, fuel monitoring, Jetson
   hardware units, voice assistant) shows the author has *already* prototyped
   purplepincher's exact intended vertical. The highest-leverage first step may
   not be "find new repos to fork" but "diff what's already in
   `SuperInstance/deckboss-*` + `cocapn-foundation` against
   `purplepincher/deckboss` and consolidate."

6. **The Fleet Vector API is a good but *dirty* tool.** It mixes
   SuperInstance repos with `casey-digennaro/*` personal repos and is
   polluted with scaffolding stubs. It's the right discovery mechanism — but
   always filter by `loc` and sanity-check `github_url`, or you'll "discover"
   a 43-line `crdt-gset` placeholder as a top result.

7. **A quiet, underrated sub-product: the budget-guardian family.**
   `codex-budget-guard`, `build-guardian`, `dify-budget-watchdog`,
   `conservation-guardian`, `cache-guardian-c` — practical tools that enforce
   token/build/time budgets on AI-coding CLIs. These are the most
   *production-minded* things in the ecosystem and nobody (including the
   docs) frames them as a product. Worth pulling into purplepincher as a
   coherent "edge/agent budgeting" toolset.

---

### Methodology footnote

- Repo list: `gh repo list SuperInstance --limit 4200 --json name,description,updatedAt,primaryLanguage`
  (4,200 rows; the account advertises 4,095 — close enough; the extra ~100 are
  the delta since the task doc was written).
- Cluster counts: first `-`-token grouping in Python; samples + activity taken
  per cluster.
- Publish status: direct registry queries to `registry.npmjs.org`,
  `crates.io/api`, `pypi.org/pypi/*/json` on 2026-07-03.
- Per-repo stats (size/stars/forks/pushed): `gh api repos/SuperInstance/<name>`.
- Maturity: `gh api repos/.../contents` for file trees + README body for the
  top candidates. `pincher`, `cocapn`, `exocortex`, `vessel`,
  `git-native-agents` READMEs were read in full.
- Fleet Vector API: 8 semantic queries; results filtered by `loc` and
  `github_url` to suppress stubs.
- No 11-named-repo content was read beyond what was needed to *avoid* them.
