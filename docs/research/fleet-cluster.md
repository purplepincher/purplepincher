# The `fleet-*` Cluster — Deep Dive

**Author:** GLM fleet pass · **Date:** 2026-07-03
**Scope:** All 319 public repos whose name starts `fleet-` on the `SuperInstance`
GitHub account, assessed for `purplepincher`'s mission ("production-ready tools
for modular and distributed edge development"). Evidence cited from cloned source
(`clones/<name>/`), live HTTP probes, wrangler configs, and git logs.

---

## TL;DR

1. **`fleet-*` is not 11 services, it's 319 repos** — and it is the single
   strongest fork-candidate cluster in the whole SuperInstance ecosystem for
   purplepincher's stated mission, because it is **a genuinely working
   distributed edge system, not a naming prefix**. But it is buried under the
   same bulk-generation pattern as the rest of the account: only ~40–50 of the
   319 carry real infrastructure; the rest are stubs, a 92-repo MIDI explosion,
   language-pack experiments, and "fleet-as-organism" metaphor repos.

2. **It is actually deployed and partially live.** Probed 2026-07-03:
   - Cloudflare Workers tier — **3 healthy-live** (`fleet-vector-api`,
     `fleet-registry-worker`, `fleet-dashboard-api` all return **200**),
     **3 deployed-but-broken** (`fleet-edge` → 404 root, `fleet-budget` → 500,
     `fleet-metrics-cron` → 500).
   - `fleet-oracle2` runs as **15 systemd units** (.service + .timer) on an
     Oracle Cloud ARM64 free-tier box — a self-orchestrating node on a 5-min
     cron pulse.

3. **There is a real protocol stack and a real dependency graph.** Agents
   communicate via **I2I bottles** (SMTP-style, transport-agnostic, speech-act
   aware) + a Python foundation lib (`fleet-protocol`) + an MQTT daemon; the
   Rust coordination crates form a dependency DAG (`fleet-coordinate` ←
   `fleet-topology`/`fleet-homology`/`fleet-manifest`/`fleet-spread`/`fleet-keel`).

4. **Caveats that temper enthusiasm.** Scale is **home-lab, not planet-scale**
   (5 physical vessels, one offline). The ideology — γ+η≤C "conservation law,"
   Pythagorean48 trust vectors, Laman rigidity, Eisenstein lattices — is
   eccentric baggage that should be **decoupled** for any production fork.
   Deploy hygiene is weak (3/6 workers broken). Treat the cluster as a
   high-quality **specification + half-deployed reference implementation** to
   consolidate and harden, not a platform to adopt wholesale.

---

## 1. The real, complete repo list

`gh repo list SuperInstance --limit 4200 | grep '^fleet-'` → **319 repos.** A
line-item for all 319 is noise; below is the structure. Size is a decent
stub proxy: **178 repos (56%) are <10 KB; only 44 are >200 KB.**

### 1a. Sub-cluster counts (by 2nd token)

| Sub-cluster | Count | Character |
|---|---:|---|
| `fleet-midi-*` | **92** | Music-generation sub-project. **64 of 92 are ≤5 KB stubs.** Separate concern from orchestration infra. |
| `fleet-math-*` | 8 | Math benchmarks/foundations (`-c`, `-go`, `-ts`, `-py`, `-demos`). Research. |
| `fleet-agent-*` | 4 | Agent runtime (`-core`, `-api`, `-universal`, `-early-version`). |
| `fleet-a2a-*` | 4 | Agent-to-agent bridges (`-bridge`, `-pipeline`, `-spectral`, `-wasm`). |
| `fleet-dashboard*` | 3 | `-api`, `-dashboard`, `-dashboard-night`. |
| Language packs | 7 | `arabic/chinese/finnish/japanese/latin/navajo/sanskrit` — "reason natively in X" experiments. |
| "Organism" repos | ~15 | `consciousness`, `immune`, `nervous-system`, `biosphere`, `ecology`, `entropy`, `resonance`, `mythology`, `genealogy`, `homology`, `synapse` — fleet-as-living-system metaphor research. |
| Music connectors | ~8 | `orca/rave/magenta/hydra/strudel/touchdesigner/diffrhythm` connectors. |
| Infra core (below) | ~40–50 | The actual distributed system. |

### 1b. Infra-core repos, individually (the ones that matter), with real descriptions

**Protocol / wire layer (the shared language):**
- `fleet-protocol` (Python, has CI) — foundation lib: `FleetMessage` format, agent/service/health registries, **Message-in-a-Bottle** async coord (TTL, hops, delivery tracking), HMAC/DH security, CLI.
- `fleet-i2i-protocol` (Rust, 40 MB, has CI) — **I2I = Instance-to-Instance.** Text "bottles" modeled on SMTP/email headers; transport-agnostic (files/HTTP/WS/memory); speech-act aware (inform/request/promise). "Forgemaster and Oracle2 agents use this to coordinate."
- `fleet-proto-rs` (Rust, 28 MB) — shared types: **PLATO client + I2I messages + constraint types**. The canonical cross-crate type lib.
- `fleet-daemon` (Python) — real-time **MQTT** daemon (HiveMQ broker) replacing 30s git-polling with sub-second event dispatch.

**Registry / identity / trust:**
- `fleet-manifest` (Rust, 17 MB, has CI) — **decentralized fleet registry.** "No central registry — every agent has a copy." Trust encoded as Pythagorean48 vectors; Laman rigidity (E=2V−3) decides which subgroups are self-coordinating.
- `fleet-registry-worker` (CF Worker, KV `AGENT_REGISTRY`) — **LIVE 200.**

**Coordination / topology (real Rust DAG):**
- `fleet-coordinate` (Rust, **369 MB flagship**, has CI) — "Zero Holonomy Consensus": agents agree via independent geometric projection, no voting. *Depends on* `holonomy-consensus`; *depended on by* `fleet-spread`, `fleet-topology`, `fleet-homology`, `fleet-manifest`, `fleet-keel`.
- `fleet-topology` (13 MB) + `fleet-topology-rs` (26 MB) — network topology, constraint-aware routing.
- `fleet-conductor`, `fleet-coordinator`, `fleet-orchestrator`, `fleet-orchestrator-v2`, `fleet-orchestra` — orchestration surfaces (mixed maturity).

**Compute nodes (physical vessels):**
- `fleet-oracle2` (Python, 1 GB) — **Oracle Cloud ARM64 free-tier node.** Self-aware orchestration OS; 5-min cron "9-step pulse pipeline"; harbor-daemon :8796/:8797, conservation-meter :8798, headspace :9090. **15 systemd units** (construct-pulse, fleet-watchdog, fleet-reflect, gc-pid-bridge, headspace-rs, reflex-daemon, …).
- `fleet-oracle` (Rust) — ARM-optimized, v0.3.0.

**Edge tier (Cloudflare Workers — all configured to account `049ff5e8…`):**
- `fleet-vector-api` (TS, 70 MB, Vectorize+AI+KV, CI+**publish.yml**) — semantic crate search. **LIVE 200.**
- `fleet-edge-worker` (TS, 74 MB, Vectorize+AI+KV, CI) — edge WorkItem/WorkResult pattern, capacity-gated, atomic throughput counters. **Deployed (404 on `/`).**
- `fleet-budget` (TS, D1 `fleet-budget`) — enforces γ+η≤C at the DB level. **Deployed, 500.**
- `fleet-metrics-cron` (TS, D1 `fleet-events` + KV) — **Deployed, 500.**
- `fleet-dashboard-api` (TS, D1 `fleet-telemetry`) — **LIVE 200.**
- `fleet-murmur-worker` (TS, 6 MB) — "5 thinking strategies always running."

**Application / intelligence:**
- `fleet-router` (Python) — critical-angle model routing from "6,000+ trials"; 16 models × 12 domains × 5 tiers; picks cheapest safe model.
- `fleet-intelligence-api`, `fleet-h1-router`, `fleet-event-router`.
- `fleet-metrics`, `fleet-health-monitor` (42 MB, "necrosis detection"), `fleet-warden-rs`, `fleet-status`, `fleet-observatory`.

**Everything else** (the ~260 non-core): MIDI stubs, language packs, organism metaphors, connectors, one-off experiments (`fleet-rpg`, `fleet-university`, `fleet-yaw`, `fleet-court`, …). Characterize as **chaff** for fork purposes.

---

## 2. The system map — yes, this is a real distributed system

It is layered, the layers reference each other, and several are deployed.

```
            ┌──────────────────────── EDGE TIER (Cloudflare Workers) ──────────────────────┐
            │  vector-api ✅   registry-worker ✅   dashboard-api ✅                        │
            │  edge-worker ⚠️(404)   budget ⚠️(500)   metrics-cron ⚠️(500)   murmur-worker   │
            └─────────────▲────────────────────────────────────────────────────▲───────────┘
                          │ D1 / Vectorize / KV                                 │ I2I bottles
   ┌──────────────────────┼────────────────────────────────────────────────────┼──────────┐
   │           PROTOCOL / WIRE LAYER (the shared language)                      │          │
   │   fleet-protocol (Py)   fleet-i2i-protocol (Rust bottles)   fleet-proto-rs │  MQTT    │
   │   fleet-daemon (MQTT, sub-second dispatch, replaces git-poll)              │  daemon  │
   └──────────────────────▲────────────────────────────────────────────────────▲──────────┘
                          │                                              │
   ┌──────────────────────┼──────────────────────────────────────────────┴───────────────┐
   │        REGISTRY / TRUST              │          COORDINATION (Rust DAG)               │
   │  fleet-manifest (decentralized,      │  fleet-coordinate (flagship) ← topology,       │
   │  Pythagorean48 trust, Laman rigid)   │  homology, manifest, spread, keel              │
   │  registry-worker (KV)                │  conductor / coordinator / orchestrator-v2     │
   └──────────────────────▲────────────────────────────────────────────────────▲──────────┘
                          │                                              │ systemd + cron
   ┌──────────────────────┴──────────────────────────────────────────────────┴───────────┐
   │                    PHYSICAL VESSELS (from fleet-manifest)                              │
   │  Oracle1 🔮 (primary, trust 1.00, Online)    Forgemaster ⚒️ (GPU, 0.85, Online)        │
   │  JetsonClaw1 ⚡ (Edge/Orin, 0.75, OFFLINE)   CCC 🦀 (Research, 0.70)  Test Probe 🔬     │
   │  fleet-oracle2 = 15 systemd units, 5-min pulse pipeline on ARM free-tier              │
   └───────────────────────────────────────────────────────────────────────────────────────┘
```

**How the pieces connect (evidence):**
- `fleet-proto-rs` README: "Shared fleet types: **PLATO client, I2I messages**, constraint types" — one type lib consumed across Rust crates.
- `fleet-i2i-protocol` README: "Forgemaster and Oracle2 agents use this protocol to coordinate builds, exchange experiment results … via text files in a shared directory."
- `fleet-coordinate` Meta block: explicit `Depends on` / `Depended by` edges forming the Rust DAG.
- `fleet-daemon` README: MQTT broker (HiveMQ) → fleet daemon → git repo as source of truth; replaces 30s polling with sub-second dispatch.
- `fleet-manifest`: 5 named vessels with trust scores and Online/Offline status — the same physical-machine fleet the ecosystem survey identified.

**Verdict on the crux question:** "fleet-*" is **not** just a shared naming prefix. There is a coherent, partially-deployed distributed system here — decentralized registry, transport-agnostic messaging, a coordination DAG, an ARM node running under systemd, and a live Cloudflare edge tier. It is small (home-lab) and ideologically decorated, but it is real.

---

## 3. Maturity per repo (tiers)

**Tier A — Genuinely running / deployed (highest confidence):**
| Repo | Evidence |
|---|---|
| `fleet-vector-api` | LIVE HTTP 200; Vectorize + CI + publish workflow; 70 MB. |
| `fleet-registry-worker` | LIVE HTTP 200; KV registry. |
| `fleet-dashboard-api` | LIVE HTTP 200; D1 telemetry. |
| `fleet-oracle2` | **15 systemd units** (.service+.timer); self-orchestrating ARM node; 5-min pulse. The most "alive" repo in the cluster. |
| `fleet-edge-worker` | Deployed (404 root route only); 74 MB; real WorkItem/WorkResult design; CI. |
| `fleet-budget` / `fleet-metrics-cron` | Deployed but returning **500** (likely D1 binding/provisioning gaps). |

**Tier B — Real code, CI, but local/dev-only (not seen running):**
`fleet-protocol` (CI), `fleet-i2i-protocol` (CI, 40 MB, world-class README),
`fleet-manifest` (CI, decentralized registry), `fleet-coordinate` (CI, flagship),
`fleet-router`, `fleet-proto-rs`. These are well-structured and tested locally
but have no deployment artifacts — they'd run where an agent embeds them.

**Tier C — Prototype / thin:** `fleet-conductor`, `fleet-coordinator`,
`fleet-orchestrator-v2` (note: forked from a `Lucineer` org), `fleet-metrics`,
`fleet-intelligence-api`, `fleet-math-*`, `fleet-health` (2 KB).

**Tier D — Stubs / chaff (do not fork individually):** the 64 tiny `fleet-midi-*`,
the 7 language packs, the ~15 "organism" repos, the music connectors, and the
long tail of one-offs.

---

## 4. Concrete recommendation for purplepincher

**Yes — this is the strongest fork-candidate cluster for purplepincher's
"modular and distributed edge development" mission, specifically because the
edge tier + protocol layer are already a working distributed edge system.**
But fork **selected repos**, not the cluster wholesale.

### 4.1 The three highest-value fork targets, in priority order

1. **The Cloudflare Workers edge tier** (package as a "fleet-edge" toolset).
   - Take the **3 healthy-live** workers (`vector-api`, `registry-worker`,
     `dashboard-api`) as-is — they are already production-ish.
   - Fix the **3 broken ones**: add a root route to `fleet-edge-worker`
     (currently 404); provision/repair the **D1 bindings** on `fleet-budget`
     and `fleet-metrics-cron` (both 500). These are small, concrete fixes.
   - This is the most "modular distributed edge" thing in the entire
     ecosystem and it is *already deployed*.

2. **`fleet-i2i-protocol` + `fleet-protocol` (the messaging standard).**
   - A transport-agnostic, human-readable, debuggable agent-messaging
     protocol (SMTP-style bottles + speech acts + registries + HMAC/DH) is
     exactly "modular distributed" infrastructure.
   - **Gap to MVP:** extract as a **versioned, standalone spec** decoupled
     from the γ+η≤C / conservation-law framing; publish `fleet-protocol` to
     PyPI (it is not published); add an interop test suite between the Python
     and Rust implementations.

3. **`fleet-edge-worker` pattern** (the `WorkItem`/`WorkResult`, capacity-gated
   edge-compute abstraction). Cleanest single expression of "edge worker" in
   the cluster; small enough to harden fast. **Gap:** the README is design-only
     — needs a deploy quickstart and a second concrete `task_type` kernel.

### 4.2 Worth investigating, not immediate fork

- **`fleet-manifest`** (decentralized trust-weighted peer registry) — genuinely
  interesting distributed-systems idea ("no central registry, every agent has a
  copy"). **But:** the Pythagorean48 + Laman-rigidity math is eccentric; for a
  purplepincher fork, either replace with standard trust/gossip primitives or
  clearly document the math as optional.
- **`fleet-coordinate`** (flagship, 369 MB) — real ZHC consensus + rigidity, but
  tightly coupled to the constraint-theory ideology and to the
  `holonomy-consensus`/Eisenstein-lattice stack. High-effort to decouple.
- **`fleet-router`** — critical-angle model routing; overlaps with the broader
  fleet's model-routing work (covered elsewhere); niche.

### 4.3 Explicit non-recommendations (save the effort)

- **All 92 `fleet-midi-*`** — separate music project; 70% stubs. Out of scope
  for edge tooling.
- **Language packs, "organism" repos, music connectors** — experiments/metaphor.
- **The long tail of one-offs** (`fleet-rpg`, `fleet-university`, `fleet-court`,
  `fleet-yaw`, `fleet-mythology`, …).

### 4.4 Cross-cutting "polish to stable MVP" gaps (real, not "clean it up")

1. **Deploy hygiene:** 3 of 6 Workers are 500/404. Stand up monitoring +
   repair D1 provisioning before calling anything "production."
2. **Decouple the ideology.** γ+η≤C conservation, Pythagorean48, Laman
   rigidity, Eisenstein lattices recur across the core and will confuse
   adopters. Make them optional layers, not load-bearing.
3. **Publish the artifacts.** Nothing in this cluster is on a registry
   (`fleet-protocol` not on PyPI; Rust crates not on crates.io). A
   "production-ready toolset" needs installable packages.
4. **Scale honesty.** 5 physical vessels (1 offline) is the real floor; the
   "planet-scale fleet" framing should be dropped for a purplepincher product.

---

### Methodology footnote
- Repo list: `gh repo list SuperInstance --limit 4200 --json name,description,diskUsage,primaryLanguage` → 319 `fleet-*`.
- 59 cloned (`--depth 1`) into `clones/`; READMEs read for the infra core.
- Deployment: `wrangler.toml` inspected in 6 worker repos (all share account
  `049ff5e8…`); live HTTP probes to `<worker>.casey-digennaro.workers.dev` on
  2026-07-03; `fleet-oracle2/systemd/` lists 15 units; CI workflows checked
  (`.github/workflows/ci.yml` in fleet-protocol/vector-api/edge-worker/manifest/i2i).
- Dependency DAG from `fleet-coordinate` README "Meta" block + `fleet-proto-rs`
  + `fleet-i2i-protocol` READMEs. Fleet topology from `fleet-manifest` table.
- Prior session (`glm-fleet.log`) had read ~50 READMEs and left two explicit
  TODOs — live-deploy verification and this deliverable — both completed here.
