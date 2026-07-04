# Deep Dive: the remaining 9 `vessel-*` repos

**Scope:** The 9 of 11 `vessel-*` repos that prior passes never opened — `vessel-constellation`, `vessel-template`, `vessel-room-navigator`, `vessel-prototype`, `vessel-equipment-agent-skills`, `vessel-coordination-protocol`, `vessel-tuner`, `vessel-spec`, `vessel-sandbox`. (`vessel` and `vessel-bridge` were covered in `edge-vessel-bridge-deep-dive.md` and `ecosystem-survey.md` and are *not* re-examined here except as cross-reference targets.)

**Method:** All 9 cloned `--depth 1` into `clones/` (`gh repo clone SuperInstance/<name>`). Every non-binary file read in full. Toolchain available in this environment: Python 3.12, Node 22 (no Rust/Cargo, no wrangler deploy). Tests run where a runnable toolchain existed (`vessel-template`'s unittest, `vessel-prototype`'s `agent.py`, live `curl` against the deployed `*.casey-digennaro.workers.dev` URLs, registry queries against crates.io / npm / PyPI). Git history inspected per-repo including the `Lucineer` upstream remotes that `gh repo clone` surfaced. No edits, no pushes.

**Prior context relied on:** `cocapn-technical-fit.md` (cocapn-foundation's actual requirements + the "docs oversell code" pattern), `edge-vessel-bridge-deep-dive.md` (vessel-bridge is an inert HAL; `📝 Upgrade README to textbook quality` commit = docs rewritten as marketing), `ecosystem-survey.md` (the "fleet" is Casey's home lab; bulk-push bursts on specific dates; published-package reality is the inverse of docs' claims).

---

## TL;DR

1. **None of the 9 advance cocapn-foundation's technical needs** (ActiveLog, profile interpreter, safety supervisor, voice pipeline, helm firmware, momentary fail-safe outputs). This is the same "naming/aspiration theme-park around the home lab" verdict `cocapn-technical-fit.md` and `edge-vessel-bridge-deep-dive.md` already returned for the rest of the vessel/embedded cluster. Carrying any of these forward as cocapn substrate would be a net negative.

2. **Only one of the 9 is genuinely shippable as-is: `vessel-tuner`** — a real, live (HTTP 200), iteratively-developed (8 upstream commits) Cloudflare Worker that scans a fleet of CF Workers for health/latency/size/spec/security and scores them. Even it has a real fallback-fetch bug and three README overclaims (see §3). It is *generic ops tooling*, not cocapn substrate.

3. **Two of the 9 are deployed and currently returning HTTP 500 to end users** because of the same trivial bug: `vessel-spec` and `vessel-sandbox` both `return new Response(html, …)` where `html` is never defined anywhere in the file. This was caught in this pass by static reading and confirmed by `curl` — it is not theoretical.

4. **`vessel-prototype` has never run on any Python version** but its CI is green. Cause: a classic dataclass field/method name-collision (`host: str` field + `def host(self, …)` method) that throws `TypeError: non-default argument 'os' follows default argument` at class-definition time. The repo's `ci-python.yml` ends `pytest … || true`, which swallows the crash. Trivial one-rename fix, but it means the "Agent/Vessel Separation Architecture" demo has been dead-on-arrival since the single commit that created it.

5. **Two of the 9 are pure README+LICENSE** (`vessel-coordination-protocol`, `vessel-equipment-agent-skills`). Both are well-written essays. `vessel-equipment-agent-skills` README claims to be "A reference implementation used in production across the Cocapn Fleet" — that is false; there is no implementation, only a 48-line architecture blog post.

6. **All publish claims are false**, matching the cluster-wide pattern. `vessel-constellation` README's `vessel-constellation = "0.1"` → not on crates.io. `vessel-prototype`'s pyproject `name = "vessel-prototype"` v1.0.0 → not on PyPI (and could not be — it doesn't import).

7. **Zero real code-level cross-dependencies among the 9**, or to `vessel` / `vessel-bridge`. constellation's only external dep is `serde`; prototype and template are stdlib-only; the four CF Workers have no `package.json` and import nothing; room-navigator pulls Three.js from a CDN via importmap. All "cocapn"/"fleet" textual cross-refs are branding footers, not imports.

---

## 1. Repo inventory and bulk-generation signatures

Confirmed `gh repo list SuperInstance … | grep '^vessel'` returns exactly **11** repos. The 9 in scope, with `updatedAt`, primary language, and git-history shape (from `git log --all` across both `origin` and the `Lucineer` upstream that several repos carry):

| Repo | Lang | updatedAt | Commits (origin+upstream) | Author pattern | Bulk-gen signature |
|---|---|---|---|---|---|
| `vessel-constellation` | Rust | 2026-06-11 | **1** | `SuperInstance` (squashed) | **Yes** — single squashed commit on the Jun-11 burst |
| `vessel-coordination-protocol` | none | 2026-04-29 | 3 | `Lucineer` / `Casey Digennaro` | No — organic |
| `vessel-equipment-agent-skills` | none | 2026-05-03 | 4 | `CedarBeach2019` / `Lucineer` / `Cocapn Fleet` | No — organic |
| `vessel-prototype` | Python | 2026-06-11 | **1** | `Casey Digennaro` (squashed) | **Yes** — single squashed commit |
| `vessel-room-navigator` | HTML | 2026-06-11 | **1** | `Forgemaster` (squashed) | **Yes** — single squashed commit; "Forgemaster" is the build-box persona |
| `vessel-sandbox` | TS | 2026-04-29 | 2 | `CedarBeach2019` / `Casey Digennaro` | No — fork from Lucineer |
| `vessel-spec` | TS | 2026-04-29 | 2 | `CedarBeach2019` / `Casey Digennaro` | No — fork from Lucineer |
| `vessel-template` | Python | 2026-05-16 | **1** | `Cocapn Fleet` (squashed) | **Yes** — single squashed commit; "Cocapn Fleet" persona |
| `vessel-tuner` | TS | 2026-04-29 | **8** | `CedarBeach2019` / `Lucineer` | **No** — the only repo with real iterative history |

**Reading the table:** the four single-commit repos (constellation, prototype, room-navigator, template) were all squash-pushed on the bulk-burst dates the ecosystem survey identified, under author personas (`SuperInstance`, `Forgemaster`, `Cocapn Fleet`) rather than a human name. The five multi-commit repos forked from `Lucineer/<name>` and retain real upstream history; `vessel-tuner` is the standout — 8 commits with messages like *"Fix case-insensitive CSP and X-Frame-Options detection"*, *"Fix scanner — use GitHub raw as primary (CF 1042 workaround)"*, *"Remove phantom repos from scan list, keep deployed vessels only"*, i.e. actual debugging iteration, the signature of code that has been run against the real world.

The bulk-generation signal correlates with code quality in a useful way: of the four single-commit repos, one is non-running (prototype), two are art/portfolio pieces whose correctness is aesthetic (constellation, room-navigator), and only one (template) actually works. Of the five multi-commit repos, one works well (tuner), two are broken (sandbox, spec), and two are README-only (coordination-protocol, equipment-agent-skills).

---

## 2. Per-repo findings

### `vessel-constellation` — real, correct physics; metaphor art with a false crates.io claim

**What it actually is:** ~1,100 lines of Rust across `src/{lib,vessel,orbit,gravity,conservation,perturbation,constellation}.rs` plus inline `#[cfg(test)]` modules. The README's "N-body gravitational simulation of a fleet of vessels and their orbiting repositories" is accurate *as a description of the code*. The physics is genuinely correct, not hand-waved:

- Velocity-Verlet leapfrog (kick-drift-kick) implemented correctly (`constellation.rs:47-84`), with a `step_euler` for comparison and a test asserting leapfrog's energy drift is ≤ 1/5 of Euler's over 200 steps (`constellation.rs:175-194`) — a real, meaningful test.
- `F = G·m₁·m₂/r²` with proper vector direction normalization and a `< 1e-10` singularity guard (`gravity.rs:23-35`).
- Kepler's third law `ω = √(μ/r³)` correctly derived from the orbital radius (`orbit.rs:30-34`).
- Conservation state computes total energy (KE+PE) and angular momentum `Σ mᵢ(rᵢ × vᵢ)` with a cross-product helper that correctly generalizes 2D inputs to a z-component (`vessel.rs:94-106`), and a relative-tolerance `energy_conserved`/`angular_momentum_conserved` check (`conservation.rs`).
- `is_circular_orbit` checks `v² = GM/r`; `is_lagrange_triangle` checks equilateral within 10% tolerance. Both are tested.

No Rust toolchain in this environment, so I could not `cargo test` — but the math reads correctly and the test assertions match the README's conservation-drift table. CI (`.github/workflows/ci.yml`) runs `cargo check/test/clippy/fmt --all-features` on every push.

**The "fleet" it simulates is Casey's four physical machines** (`initial_fleet()`, `constellation.rs:125-133`): Forgemaster (mass 330 = repo count), CCC (116), JetsonClaw1 (76), Oracle (43). This is the same `ecosystem-survey.md` finding — "vessels are physical machines, not abstractions" — dressed in a solar-system metaphor. It is the conservation-law ideology (γ+η=C) from the `conservation-*` cluster rebadged with vessels as the narrative device.

**Defects / overclaims:**

- README §Installation: `vessel-constellation = "0.1"`. Verified **not on crates.io** (`curl https://crates.io/api/v1/crates/vessel-constellation` → not found). Same publish-claim-vs-reality inversion the ecosystem survey flags for the whole account.
- The repo has no connection to anything cocapn, ActiveLog, profiles, safety, voice, or hardware. It is a self-contained physics-sim teaching artifact whose only "fleet" semantics are four hardcoded machine names.

### `vessel-prototype` — never ran on any Python; CI hides it

**What it actually is:** a single 219-line `agent.py` plus a 113-byte `__main__.py` and a `pyproject.toml` claiming `name = "vessel-prototype"` v1.0.0. The code defines three dataclasses (`Capability`, `AgentSoul`, `Vessel`) and a `FleetScheduler` class, then a `demo()` that prints a fleet-scheduling scenario over hardcoded IPs for Oracle1/JetsonClaw1/Forgemaster (Casey's machines — `147.224.38.131`, `192.168.1.100`, `10.0.0.5`). The docstring says "Built for PurplePincher."

**It does not run.** `python3 agent.py` and `python3 -c "import agent"` both throw on Python 3.12 (the environment default):

```
TypeError: non-default argument 'os' follows default argument
  at @dataclass (line 53), decorating class Vessel
```

I bisected this precisely. Root cause is a **field/method name collision** in the `Vessel` dataclass (`agent.py:54-84`): the class declares a field `host: str` (line 57) *and* a method `def host(self, soul: AgentSoul) -> bool:` (line 76). Python's `@dataclass` treats any annotated name that also has a value in the class namespace as a field *with that value as its default*. The method definition binds the name `host` to a function object in the class namespace, so dataclass sees field `host` as having a default (the function). The subsequent field `os: str` has no default, so it "follows a default" → TypeError. Confirmed with a 4-line minimal repro:

```python
from dataclasses import dataclass
@dataclass
class A:
    name: str
    host: str        # field
    os: str          # ← dataclass sees this as non-default following the defaulted 'host'
    def host(self):  # method collides with field name
        return 1
# → TypeError: non-default argument 'os' follows default argument
```

Renaming either the field or the method fixes it (verified). This is not a Python-3.12 regression; the dataclass field-default detection has worked this way since dataclasses shipped in 3.7, so **the repo has been non-importable on every Python version since the day it was committed.**

**Why CI is green anyway:** `.github/workflows/ci-python.yml` line 34 is `python -m pytest --import-mode=importlib -x -v || true`. The `|| true` swallows any failure — including an import-time `TypeError`. There are also no test files (`find . -name 'test_*.py' -o -name '*_test.py'` returns nothing), so even if `agent.py` imported cleanly there is nothing for pytest to run. This is the same "CI is theatre" pattern `edge-vessel-bridge-deep-dive.md` found in `edge-llama`'s `pytest || true`.

**Other defects:** `import hashlib, os, time` at line 17 — all three are dead imports (never referenced; the collision above is the only thing `os` does in this file). `pyproject.toml` claims an installable package and a `vessel_prototype = "agent:main"` console script, but there is no package directory (no `__init__.py`), so `pip install -e .` would also fail. README's `python3 -m vessel-prototype` usage would fail even after a rename (Python `-m` needs a package, and `vessel-prototype` is hyphenated anyway). Verified `vessel-prototype` is **not on PyPI**.

### `vessel-room-navigator` — real Three.js web app; "live data" and "ESP32 firmware" are fiction

**What it actually is:** a 664-line single-page `index.html` (+ a duplicate under `docs/` for GitHub Pages), a 215-line `rooms-config.json` defining room topology (wheelhouse/galley/foredeck/aft_cockpit/engine_room/wheelhouse_roof/crow_nest + virtual alarm_center), and ~7 MB of AI-generated equirectangular panoramas (the README credits FLUX-1-schnell at 1792×1024). It loads Three.js r170 from `cdn.jsdelivr.net` via an importmap, renders the active panorama on the inside of a `SphereGeometry(50,64,64)` with `BackSide`, and lets the user click doors to "walk" or lightning-bolt icons to "warp." That part is genuine and works (live at `https://fleet.cocapn.ai/`, HTTP 200).

**What it is not, despite the README:**

- *No live data.* README: "Live engine, nav, and monitor gauges," "Trigger alarms → click to auto-warp to problem room." `grep -nE 'fetch\(|WebSocket|EventSource|wss?:|api/|/health' index.html` returns **zero** network calls. The "ENGINE" dashboard (`index.html:241`) is four hardcoded display values: `data-b="1800"` (RPM), `data-b="72"` (FUEL), `data-b="185"` (TEMP), `data-b="3"` (TRIM). There is no data source behind them. The "alarms" are UI state, not signal-driven.
- *No ESP32/camera firmware.* README §Tech Stack: "Camera agents: ESP32-S3 firmware (C++, JSON/WebSocket, ESP-NOW)." `find . \( -name '*.cpp' -o -name '*.ino' -o -name '*.h' -o -name '*.c' -o -name 'platformio*' \)` returns **nothing**. The "cameras" are static JPGs (`textures/cam_thermal.jpg`, `cam_gunnery.jpg`, `cam_radar.jpg`). The ESP32 story exists only as research markdown under `research/vessel-room-esp32-agent.md`.
- *No chat/voice.* README: "Chat: Talk to the room agent," "Chat/voice: Gemini Nano + PLATO tiles." No Gemini, no PLATO, no WebSocket — pure front-end, no backend of any kind.

**What's salvageable:** the 360°-panorama-walk + room-graph-warp UX is a clean ~660-line Three.js reference (importmap-based, zero-build, CDN-only). If purplepincher ever wants a "walk the boat" operator UI, this is a legitimate starting point — but it would need to be paired with a real telemetry backend (the exact thing none of the 11 vessel-* repos provide) and stripped of the false live-data/firmware/agent claims.

**Bulk-gen signal:** single squashed commit by `Forgemaster` (the build-box persona) on 2026-05-13, with `docs/` being a byte-identical duplicate of root `research/` purely to satisfy GitHub Pages. The 16 research markdown docs (~240 KB) are idea-dense but, as above, describe capabilities the code does not have.

### `vessel-sandbox` — broken; deployed URL returns HTTP 500

**What it actually is:** `src/worker.ts` is 50 lines. The first 48 lines define a plausible-looking sandbox abstraction: an `Env` interface declaring `SESSIONS`/`USAGE`/`VESSEL_TEMPLATES` KV+DO bindings, `Session`/`LaunchRequest` interfaces, and a `UsageLimiter` Durable Object class implementing a 10-requests/day-per-IP limiter via DO storage. Then line 50 is the entire actual entry point:

```ts
const sh = {…CSP/X-Frame-Options…};
export default { async fetch(r: Request) {
  const u = new URL(r.url);
  if (u.pathname==='/health') return new Response(JSON.stringify({status:'ok'}), …);
  return new Response(html,{headers:{'Content-Type':'text/html;charset=UTF-8',…sh}});
}};
```

`html` is **never defined anywhere in the file.** Every non-`/health` request throws `ReferenceError: html is not defined` at runtime. Confirmed live: `curl https://vessel-sandbox.casey-digennaro.workers.dev/` → **HTTP 500**. (`/health` returns 200, which is presumably why no one noticed.)

Three additional problems: (1) the `UsageLimiter` DO class is defined but **never exported and never wired** in `wrangler.toml` — dead code. (2) `wrangler.toml` declares no KV or DO bindings at all, so even the `Env` interface's `SESSIONS`/`USAGE`/`VESSEL_TEMPLATES` would be `undefined` at runtime if any code path tried to use them. (3) The README is 10 lines and just says "Try any vessel in an isolated sandbox before forking" — the sandbox feature itself is **not implemented**. The README, the interfaces, and the DO class describe an application that does not exist behind the deployed Worker.

### `vessel-spec` — broken in the identical way; "authoritative guide" has no guide

**What it actually is:** 5-line `src/worker.ts`:

```ts
interface Env { /* No environment variables needed for this example */ }
const sh = {…CSP/X-Frame-Options…};
export default { async fetch(r: Request) {
  const u = new URL(r.url);
  if (u.pathname==='/health') return new Response(JSON.stringify({status:'ok'}), …);
  return new Response(html,{headers:{'Content-Type':'text/html;charset=UTF-8',…sh}});
}};
```

Same `html`-is-not-defined bug as `vessel-sandbox`. Live: `curl https://vessel-spec.casey-digennaro.workers.dev/` → **HTTP 500**. README pitches this as "Vessel specification — the authoritative guide to building Cocapn vessels." There is no guide, no schema, no spec document anywhere in the repo (5 files total: the broken worker, a 10-line README, a 4-line `wrangler.toml`, LICENSE, `.gitignore`). It is a deployed-but-crashing placeholder.

### `vessel-tuner` — real, live, the only one with iterative history; README overclaims

**What it actually is:** 230-line `src/worker.ts` + a 98-line `CLAUDE.md` (agent-operating manual) + `vessel.json` self-description. It is a fleet scanner: a hardcoded array of 64 vessel names (the comment says 90 — off by ~30) under `Lucineer/<name>`, and for each it does a 5-stage check (repo existence on GitHub raw → bundle size → `vessel.json` presence → CSP/X-Frame-Options text scan → weighted scoring 0–100), batches 5 at a time, stores the result in KV, and serves a self-contained HTML dashboard with sorting and top-issues aggregation. Live and working: `curl https://vessel-tuner.casey-digennaro.workers.dev/health` → `{"status":"ok","vessel":"vessel-tuner"}`. The vessels it scans are real deployed Workers (spot-checked `cocapn-ai`, `the-fleet`, `deckboss-ai`, `dmlog-ai` — all HTTP 200 on `/health`). So the "fleet" it tunes is a genuine (if home-lab-scale) collection of ~64 small CF Workers.

This is the **only one of the 9 with a real iterative commit history** (8 commits, oldest 2026-04-07), with messages showing actual debugging: *"Fix case-insensitive CSP and X-Frame-Options detection"*, *"Fix scanner — use GitHub raw as primary (CF 1042 workaround)"*, *"Remove phantom repos from scan list, keep deployed vessels only."* The `CLAUDE.md` is a genuine shipwright/specialist operating manual with deploy commands, refactoring rules ("NEVER change the JSON helper function name — breaks all endpoints"), and before/after checklists — the kind of doc that only exists if a real agent has been actively maintaining the thing.

**Defects / overclaims:**

- *README lie #1 — "Scan History: KV storage retains the last 100 scan results for simple trend observation."* False. The code does `await env.TUNER_KV.put('latest_scan', JSON.stringify(scan))` (`worker.ts:216`) — a single key that **overwrites** the previous scan. There is no array, no rotation, no history. The dashboard's `scanRecent` button fetches `latest_scan` (singular). Retained scan count: 1, not 100. No trend viewing is possible.
- *README lie #2 — "presence of key security headers (CSP, HSTS, etc.)."* The code checks `content-security-policy` and `x-frame-options`/`frame-ancestors` only (`worker.ts:77-80`). HSTS is never checked.
- *README lie #3 — "Core logic is under 150 lines of TypeScript."* Actual: 230 lines in `worker.ts` (CLAUDE.md honestly says "~230 lines," contradicting its own README).
- *Real bug — discarded fallback fetch.* The fallback-fetch pattern at `worker.ts:47-48` and again at `72-75` is:
  ```ts
  const resp = await fetch(`…master/worker.ts`, …);
  if (!resp.ok) await fetch(`…main/worker.ts`, …);   // result thrown away
  const body = await resp.text();                     // reads the FAILED master response
  ```
  When the `master` fetch fails, the `main` fallback is performed but its response is not assigned to anything, so `body`/`size`/CSP detection operate on the original failed response. On repos whose default branch is `main` (which is most of them under `SuperInstance/`), size and security checks will silently produce wrong/empty results. The repo's own commit *"Fix scanner — use GitHub raw as primary (CF 1042 workaround)"* shows the author has fought this code path before and it is still not right.
- *Inefficiency, not a bug:* `checkVessel` fetches the same `worker.ts` three separate times per vessel (existence, size, CSP scan). Functionally OK on a 64-vessel list batched 5-at-a-time, but wasteful.

**What's salvageable:** with the three README corrections, the fallback-fetch bug fixed, and the `latest_scan`-only honesty restored (or actually implement the 100-scan rotation), this is a clean, small, zero-dependency, fork-and-deploy CF Worker fleet scanner. It is the only one of the 9 that does what it claims, is live, and would be useful to someone other than its author — *as a generic ops tool*. It still has nothing to do with cocapn-foundation's substrate needs.

### `vessel-template` — real, tested, runs; generates org-mythology markdown

**What it actually is:** a single 299-line `template.py` containing a `VesselConfig` dataclass, an `AgentType` enum (`LIGHTHOUSE`/`VESSEL`/`SCOUT`/`BARNACLE` with auto-assigned fleet ranks 2/3/4/5), a `TEMPLATE_FILES` dict of 8 markdown templates (`CHARTER.md`, `IDENTITY.md`, `MANIFEST.md`, `TASKBOARD.md`, `FENCE-BOARD.md`, `CAREER.md`, `DIARY/README.md`, `KNOWLEDGE/public/README.md`), and a `generate_vessel()` function that renders them into a directory. **13 embedded unittest tests, all passing** (verified locally — `python3 -m unittest template -v` → `Ran 13 tests in 0.062s OK`, matching the README's "13 tests passing" exactly).

This is the only repo in the 9 where the README's test claim is precisely accurate and the code runs first try. CI is reasonable: `clean.yml` rejects `__pycache__`/`.pyc`/`.whl` commits, and `ci-python.yml` runs flake8 + pytest (this one without `|| true`, though there's still no test discovery issue since the tests are inline).

**What it generates is the question.** The templates encode Casey's personal agent-org mythology: the "Tom Sawyer Protocol" (work posted as puzzles on a "fence board" for others to claim), a "merit badge sash," a five-stage career ladder (`FRESHMATE` → `HAND` → `CRAFTER` → `ARCHITECT` → `TOM_SAWYER`), and a fleet hierarchy `Captain Casey → Lighthouse → Vessel → Scout → Barnacle`. The generated files are organizational scaffolding markdown, not executable systems. `CHARTER.md` is literally described in-template as "the agent's constitution."

**Verdict:** genuinely good small Python (clean dataclasses, real tests, runs, packaged as both a library and a CLI-ish `__main__`), but its output is only useful if you have adopted the Cocapn/Tom-Sawyer/barnacle agent-org framework. If purplepincher wants *that* social model, this is the scaffolder for it. If not, the code quality doesn't redeem the domain-specific payload.

### `vessel-coordination-protocol` — README only; the spec admits its own non-implementation

**What it actually is:** 2 files: a 166-line `README.md` and a LICENSE. The README is a genuinely well-thought-out git-native coordination protocol ("VCP"): three phases (Discovery via `vessel.json` + `ASSOCIATES.md`, Handshake via I2I commit messages, Coordination via task/status/dispute/energy message types), trust-score mechanics, and a clear separation of Vessel/Lighthouse/Captain roles.

**It contains no code.** Its own `## Status` section is honest about this:

```
- [x] Spec written
- [x] vessel.json format defined
- [x] I2I message types defined
- [x] Trust propagation rules defined
- [ ] Implementation in Rust (cuda-fleet-coordination)
- [ ] Integration with HAV vocabulary
- [ ] Integration with FLUX runtime
- [ ] Integration with cuda-energy ATP system
- [ ] Testing with Oracle1
```

Every implementation checkbox is unchecked, and the referenced implementation repo (`cuda-fleet-coordination`) doesn't exist in the 11-repo vessel cluster. As a *specification* it overlaps heavily with the (also codeless) `plato-vessel-technician` requirements docs covered in `cocapn-technical-fit.md` and is conceptually adjacent to the (real, shell-script) `git-native-agents` from the ecosystem survey. Verdict: keep the spec as a reference if implementing git-native fleet coordination; do not treat as code.

### `vessel-equipment-agent-skills` — README only; explicitly false "production" claim

**What it actually is:** 2 files: a 48-line `README.md` and a LICENSE. The README describes a clean four-layer agent architecture (Vessel = runtime / Equipment = data+tools / Agent = reasoning / Skills = prompts+behavior) and pitches it as *"A reference implementation used in production across the Cocapn Fleet. It runs on Cloudflare Workers and has zero dependencies."*

That claim is **false**. There is no implementation. There is no `src/`, no `worker.ts`, no code of any kind. The "Quick Start: Fork → Deploy → Edit" instructions have nothing to fork-and-deploy. The linked "Live URL" is `https://the-fleet.casey-digennaro.workers.dev` (a different repo, the fleet index — not a four-layer reference implementation). The architecture essay itself is reasonable (it is essentially the standard runtime/data/model/prompt separation dressed in nautical nouns), but calling a 48-line blog post "a reference implementation used in production" is the clearest single instance of the docs-oversell-code pattern in this whole 9-repo batch.

---

## 3. Cross-repo dependency map

Verified by reading every `Cargo.toml`, `pyproject.toml`, `package.json`, `wrangler.toml`, and import statement across the 9:

| Repo | External deps | Depends on another vessel-* repo? |
|---|---|---|
| `vessel-constellation` | `serde` only (+ `serde_json` dev) | No |
| `vessel-prototype` | none (stdlib) | No |
| `vessel-template` | none (stdlib) | No |
| `vessel-room-navigator` | Three.js r170 via CDN importmap (no build, no package.json) | No |
| `vessel-sandbox` | none declared (no package.json) | No |
| `vessel-spec` | none declared (no package.json) | No |
| `vessel-tuner` | none declared (no package.json) | No |
| `vessel-coordination-protocol` | n/a — no code | No |
| `vessel-equipment-agent-skills` | n/a — no code | No |

The `grep` for textual cross-refs (`vessel-bridge`, `cocapn`, `plato`, `deckboss`, `activelog`, etc.) returns hits in every repo, but they are **branding footers and self-references only** — "Part of the Cocapn fleet," `the-fleet.casey-digennaro.workers.dev` links, the repo's own name. There is no `import`, no `use`, no `from … import`, no git submodule, no `dependencies`/`dev-dependencies` entry that crosses between any two of these repos, or to `vessel` / `vessel-bridge`. The 11-repo "vessel cluster" is 11 independent artifacts that share a naming theme and a footer link, not a system.

The one genuine external integration any of them has is `vessel-tuner`'s runtime `fetch()` to `raw.githubusercontent.com/Lucineer/<vessel>` and `https://<vessel>.casey-digennaro.workers.dev` — i.e. it reads the *deployed artifacts* of other fleet Workers, not their code.

---

## 4. cocapn-foundation fit

Run against the same requirements table `cocapn-technical-fit.md` and `edge-vessel-bridge-deep-dive.md` use. None of the 9 implement any of the non-negotiables.

| cocapn-foundation requirement | Present in any of the 9? | Closest artifact |
|---|---|---|
| ActiveLog substrate (append-only JSONL, `(dev,seq)` keyed, set-union merge) | **No** | None — no event log anywhere in the 9 |
| Profile interpreter (JSON data describing electrical interface + actions + grammar + GUI) | **No** | `vessel-room-navigator/rooms-config.json` is the *only* JSON-driven config in the 9, but it describes a room graph for a 3D walkthrough, not an autopilot profile |
| Safety supervisor (500 ms TTL, watchdog, override, momentary outputs, C0–C3) | **No** | None — no safety code of any kind |
| Voice pipeline (wake word, closed-grammar STT, VAD) | **No** | None |
| Helm firmware (ESP32-S3, BLE+WiFi, contact/resistor/NMEA) | **No** | `vessel-room-navigator` README *claims* ESP32-S3 camera-agent firmware; **zero** `.cpp`/`.ino`/`.h`/`platformio.ini` files exist |
| Offline-first | Partial | The Workers are inherently online; `room-navigator` is offline-capable after first load |

This does not change `cocapn-technical-fit.md`'s bottom line. The single best embedded-C building block for cocapn-foundation remains `plato-vessel-core` (real flashable ESP32/RP2040 C + Python PLATO server with WAL and Lamport clocks), and the single best wire-protocol artifact to cherry-pick remains `vessel-bridge`'s `ESP32Protocol` CRC-8 framing (per `edge-vessel-bridge-deep-dive.md` §3). **None of the 9 repos reviewed here displaces either of those picks.**

---

## 5. Fork / polish recommendations

Tiered by (a) does it run, (b) is it what its README claims, (c) is it useful for anything outside Casey's home lab, (d) does it advance cocapn-foundation.

### Tier 1 — fork with bounded cleanup

1. **`vessel-tuner`** — the only unambiguous yes. Real, live, iteratively developed, genuinely useful as a generic Cloudflare-Worker fleet health/latency/spec/security scanner. Before relying on it: (a) fix the discarded-fallback-fetch bug at `worker.ts:47-48` and `72-75` (assign the `main`-branch response, don't throw it away), (b) either implement the "last 100 scans" rotation the README promises or delete that claim and the `scanRecent` trend language, (c) delete the "HSTS" mention (it isn't checked) and the "150 lines" claim (it's 230), (d) parameterize the hardcoded `VESSELS` array and `Lucineer/<name>` org path so it can scan someone else's fleet. After that: a clean, small, zero-dependency ops tool. **No relevance to cocapn-foundation's substrate needs** — keep it as a fleet-ops utility, not a cocapn building block.

### Tier 2 — fork only under specific conditions

2. **`vessel-template`** — fork *only if* purplepincher has decided to adopt the Cocapn "Captain → Lighthouse → Vessel → Scout → Barnacle" hierarchy, the "Tom Sawyer Protocol" fence-board, and the merit-badge/career-ladder social model. Under that condition it is a clean, tested (13/13), runnable scaffolder for new agent repos. Without that condition, the code quality (which is the best of any Python in the 9) does not justify importing the domain payload. Verify first that the generated markdown structure is what you actually want agents to live inside.

3. **`vessel-room-navigator`** — fork *only as a UX starting point* for a "walk the boat" operator UI, accepting that ~90% of the README's feature claims do not exist in code. Before building on it: delete the "live gauges," "alarms auto-warp," "chat/voice," "ESP32 camera agents," "GPU vector DB," and "PLATO tiles" claims from the README and docs (none are implemented), accept that the dashboards are static `data-b="1800"` cosplays that need a real telemetry backend (the actual cocapn substrate), and treat the 16 `research/*.md` docs as brainstorm notes, not architecture. The genuinely reusable core is ~660 lines of importmap-based, zero-build Three.js for panorama-sphere room navigation — small and clean once the aspiration is stripped out.

4. **`vessel-constellation`** — fork *only as a physics-sim teaching artifact*, with the README's `vessel-constellation = "0.1"` crates.io claim removed (it is not published and probably shouldn't be under that name). The code is genuinely correct (velocity-Verlet, F=Gm₁m₂/r², Kepler, Lagrange detection, real tests) and would be a respectable small example of symplectic-integration + conservation-law checking — but its only "fleet" semantics are four hardcoded home-machine names, and it has zero connection to cocapn, ActiveLog, profiles, safety, voice, or hardware. Do not adopt the "fleet as solar system" framing as if it were a coordination product; it is a metaphor over `Vec<Vessel>`.

### Tier 3 — do not fork

5. **`vessel-prototype`** — broken since the day it was committed (dataclass field/method `host` collision), CI green only because of `pytest || true`, no tests exist anyway, not on PyPI despite claiming v1.0.0, single squashed commit, hardcoded home-lab IPs, dead imports. The 130 lines of actual scheduling logic (FleetScheduler.find_best_vessel/migrate) are reasonable but trivial; if you want agent/vessel separation, reimplement from scratch rather than salvage. If you do salvage: rename the `host` method (or the `host` field) first — it is a one-token fix that makes the file importable for the first time in its life.

6. **`vessel-sandbox`** — deployed and returning HTTP 500 to users today (`html` is not defined). The `UsageLimiter` Durable Object is dead code (defined, never exported, never bound in `wrangler.toml`). The `Env` interface declares KV/DO bindings that `wrangler.toml` does not create. The README's "isolated sandbox" feature is unimplemented. Do not fork; if you want a try-before-fork sandbox, this contributes ~0 lines of working code to start from.

7. **`vessel-spec`** — same `html`-not-defined crash, same HTTP 500 in production. The README's "authoritative guide to building Cocapn vessels" does not exist as any document in the repo. Five files, none of them the spec. Do not fork.

8. **`vessel-coordination-protocol`** — no code. Keep the README as a reference specification if implementing git-native fleet coordination (it is conceptually adjacent to the real `git-native-agents` repo from the ecosystem survey). Do not fork as a code starting point.

9. **`vessel-equipment-agent-skills`** — no code, and its README's "reference implementation used in production across the Cocapn Fleet" is false. The four-layer (Vessel/Equipment/Agent/Skills) architecture essay is fine as a blog post. Do not fork.

### Cluster-wide verdict

The remaining 9 vessel-* repos do not change the earlier verdicts about the vessel cluster. `vessel` is a MUD-style ops console and `vessel-bridge` is an inert HAL (per `edge-vessel-bridge-deep-dive.md`); of the 9 examined here, only `vessel-tuner` is a genuinely shippable, currently-working, externally-useful artifact — and it is generic fleet-ops tooling, not cocapn substrate. Two repos (`vessel-sandbox`, `vessel-spec`) are deployed in a crashing state. One (`vessel-prototype`) has never imported on any Python. Two more (`vessel-coordination-protocol`, `vessel-equipment-agent-skills`) are README-only. The remaining four (`vessel-constellation`, `vessel-template`, `vessel-room-navigator`, plus `vessel-tuner` already counted) are real code of varying quality and zero cocapn relevance.

**Net effect on the cocapn-foundation fork shortlist:** none. The picks from `cocapn-technical-fit.md` stand unchanged — `plato-vessel-core` for the embedded-C + server substrate, `vessel-bridge`'s `ESP32Protocol` narrowly for the wire framing, the safety ideas from `cocapn-foundation/project/foundation/SAFETY.md` and `plato-vessel-technician` for the supervisor layer. If purplepincher additionally wants a small fleet-ops tool alongside the cocapn work, `vessel-tuner` (after the four cleanup items above) is a clean pick — but frame it as ops tooling, not as part of the cocapn foundation.
