# `deckboss-*` family vs. `purplepincher/deckboss`: a real diff

**Date:** 2026-07-03
**Method:** shallow-cloned (`--depth 1`) every SuperInstance repo matching
`deckboss`/`cocapn-foundation`, plus the upstream `Lucineer` repos several of
them fork from, plus one more fishing-specific sibling (`Lucineer/fishinglog-ai`)
discovered while tracing lineage. Read READMEs, source trees, representative
source files, and `gh api` metadata (created/pushed/size/fork-parent) for each.
Compared against `purplepincher/deckboss` as it exists in this environment
(README.md, ARCHITECTURE.md, ROADMAP.md, `src/core/types/log-entry.ts`, git log).

**Headline finding, up front:** the `deckboss-*` name in SuperInstance is a
**false-friend cluster** — most repos named `deckboss-*` are *not* about fishing
logbooks at all (they're a Jetson/RPi onboarding CLI, a hardware sales page, a
vendor marketplace, an AI spreadsheet engine, and an unrelated "Agent Edge OS").
The actual design ancestor of `purplepincher/deckboss` is **`cocapn-foundation`**
— and `purplepincher/deckboss`'s own README says so explicitly, in writing, citing
it as a "sibling repo." This is not a guess; it's confirmed by direct textual
quotation (below).

---

## 1. The real, complete list — what each repo actually is

Found via `gh repo list SuperInstance --limit 4200 --json name --jq '.[].name' | grep -iE 'deckboss|cocapn-foundation'`. Eleven repos:

| Repo | Created → last push | Fork of | What it actually is (read, not guessed) |
|---|---|---|---|
| **`cocapn-foundation`** | 2026-07-02 → 2026-07-02 (same day) | — (original) | **The real ancestor.** A "Claude Design" handoff bundle: `VISION.md`, `ARCHITECTURE.md`, `SAFETY.md`, `ROADMAP.md` for "Cocapn / ActiveLog — fishing vessel voice assistant," plus a spec repo (`activelog-spec` — JSON Schema + example JSONL), and README-only stub sub-repos for the pieces not yet built (`cocapn-app` phone app, `cocapn-helm-firmware` ESP32 firmware, `autopilot-profiles`, `cocapn-desktop`, `cocapn-provisioner`). Design docs only — **no working app code**, but very thorough design. |
| `deckboss-net` | 2026-06-10 → 2026-06-14 | — | A single Cloudflare Worker (`src/worker.ts`, 9 KB repo) that returns one hardcoded HTML string — a fake "fleet ops dashboard" mockup (hardcoded vessel "F/V Trailblazer," fake GPS/fuel/catch numbers). README describes a Rust reliable-messaging library ("BBR-inspired congestion control for maritime links") that **does not exist as code** — the "Quick Start" is literally `fn main() { println!(...) }`. Pure vaporware/landing page. |
| `deckboss-hardware` | 2026-04-10 (fork) → 2026-04-13 | `Lucineer/deckboss-hardware` | A marketing landing page (Cloudflare Worker, static HTML) selling "preloaded Jetson/RPi units" ($299–$1,499 tiers) with Whisper/Piper/Phi-3-mini preloaded. No firmware, no provisioning code — pure pricing page. |
| `deckboss-marketplace` | 2026-04-10 (fork) → 2026-04-13 | `Lucineer/deckboss-marketplace` | A real, self-contained Python module (`src/marketplace.py`, ~230 LOC): vendor tiers, platform-fee calculator, vendor sort/ranking, BOM generator. Generic e-commerce logic for a hardware-vendor marketplace — **not fishing-specific**, and not wired to anything (no server, no persistence, no tests). |
| `deckboss-fab` | 2026-04-10 (fork) → 2026-04-13 | `Lucineer/deckboss-fab` | A real, working-looking Python module (`src/fab.py`, ~260 LOC): generates parametric OpenSCAD source for Jetson/RPi carrier cases (ports, standoffs, fan cutouts), wraps `openscad`/slicer/Gazebo CLIs. CAD tooling for the hardware line above — not fishing-specific. |
| `deckboss-ai` | 2026-04-10 (fork) → 2026-04-13 | `Lucineer/deckboss-ai` | **Not fishing at all.** A large (10.8 MB) TypeScript **AI-powered spreadsheet engine** — `spreadsheet-engine.ts`, `formula-engine.ts`, `cell-protocol.ts`, a Univer bridge. Its own vision doc (`docs/DECKBOSS-VISION.md`) frames "Deckboss" as "The Agent's Nervous System" — a hub-and-spoke agent/spreadsheet UI, only tangentially nautical ("Captain Mode (Voice)" is one bubble-routing metaphor, not a fishing feature). |
| `deckboss-agent` | 2026-05-03 → 2026-05-04 | — | A ~40-line Python demo script (`agent.py`) that logs "deck operations" (containers loaded/unloaded, crane maintenance) to a "PLATO tile server" at a hardcoded IP. Cargo/shipping-deck metaphor, not fishing-vessel logbook. Toy demo, not a product. |
| `deckboss-1` | 2026-04-10 (fork) → 2026-05-26 | `Lucineer/deckboss` | **Fork of the true original `Lucineer/deckboss`.** This is a CLI onboarding wizard for Jetson/RPi devices — "clone this repo onto a Jetson or RPi, run one command, and start talking to your hardware." Six-step onboarding (hardware detection, model engine choice, API keys, I/O setup, mic/speaker, git-agent profile), produces a `character.yaml` "character sheet." It is explicitly the **generic dev-tooling layer** underneath the whole Lucineer/Cocapn app family, not a fishing app itself (see §2). |
| `deckboss-ai-pages` / `deckboss-net-pages` | May 2026 | — | Trivial GitHub Pages HTML shells (branding image + link), no logic. |
| `DeckBoss` (capitalized) | 2026-02-28 → 2026-06-08 | — | **Unrelated product, same name collision.** "The Agent Edge OS" — a Cloudflare Durable-Objects-based persistent agent orchestration platform (MCP server, "squadrons," background missions). Largest repo in the family (12.3 MB), 2 stars (highest engagement of the eleven), but has nothing to do with fishing or logbooks — it belongs to the `agent-*`/`fleet-*` cluster the ecosystem survey already covers. |

**Also found, one hop outside the literal name match (necessary for an honest
diff):**

| Repo | What it is | Why it matters here |
|---|---|---|
| `Lucineer/deckboss` | The **true original** of `deckboss-1`/`deckboss-ai`/`deckboss-hardware`/`deckboss-fab`/`deckboss-marketplace` (all five are literal GitHub forks of `Lucineer/*`, created 2026-04-08, bulk-forked into SuperInstance 2026-04-10, bulk-pushed once more 2026-04-13, then abandoned). Confirms the "deckboss" *name* originates as generic Jetson/RPi onboarding tooling, not a fishing app. |
| `Lucineer/fishinglog-ai` | **The actual closest functional prior-art** to `purplepincher/deckboss`. A deployed (`fishinglog-ai.casey-digennaro.workers.dev`) Cloudflare Worker with a detailed `docs/ARCHITECTURE.md` ("FishingLog.ai — A Co-Captain AI for Commercial Fishing Vessels"): voice pipeline (wake word "Hey Cap", Whisper-tiny, RNNoise), vision pipeline (YOLOv8-nano + cloud Megadetector species ID), Jetson/GPS/NMEA 2000 hardware target, GitHub-repo-as-storage-backend model. **Every function is a `TODO` stub** — `transcribeAudio()`, wake-word detection, and the TensorRT inference calls are all unimplemented placeholders; `src/simulation/demo-mode.ts` (363 LOC) is the only thing that actually runs. 23 commits, all March 31–Apr 8 2026, single burst, dead since. |
| `Lucineer` (user account) | A second ~632-repo personal GitHub account using identical naming conventions to SuperInstance (`vessel-*`, `cocapn-*`, `JetsonClaw1-vessel` — matching the exact machine name `JetsonClaw1` the ecosystem survey identified as one of Casey DiGennaro's physical boxes). Near-certainly the same author's second account, not a different person's independent project. |

`cocapn-foundation`'s bundle also documents a shared `lib/` of TypeScript
utilities (`byok.ts`, `confidence-tracker.ts`, `deadband.ts`, `model-router.ts`,
etc.) that turned out, on diff, to be **byte-for-byte identical** between
`deckboss-ai` (spreadsheet engine) and `Lucineer/fishinglog-ai` (fishing app) —
confirming these are copy-pasted generic scaffolding (an LLM-provider-BYOK
helper, not a storage-BYOK helper — see §3), not fishing-specific design, despite
living in a fishing-named repo.

---

## 2. Verdict: independent build, with an explicit, textual acknowledgment of its real ancestor

**`purplepincher/deckboss` is not a fork, and not descended in git history from
any `SuperInstance/deckboss-*` repo.** Its first commit (`3df1a94`, "DeckBoss
Phase 1: voice-first digital fishing logbook (MVP)") is dated 2026-07-02, the repo
has no shared git ancestry with any SuperInstance/Lucineer repo, and its
`git remote -v` points only at `purplepincher/deckboss`. It is an independent,
from-scratch codebase (React/TypeScript PWA, IndexedDB + Zod schemas, Vite).

**But it is a knowing, deliberate implementation of `cocapn-foundation`'s
design**, not a coincidental namesake. Evidence, not inference:

1. **`purplepincher/deckboss/README.md` says so directly:** *"Part of the
   PurplePincher Vessel-as-a-Robot family... and [SAFETY.md] (in the sibling
   `cocapn-foundation` repo) for the longer-horizon autopilot/steering research
   this project deliberately keeps out of scope."* "Vessel-as-a-Robot" is a term
   coined verbatim in `cocapn-foundation/project/foundation/VISION.md` §4 ("VaaR
   is a recognized category and we named it").
2. **Verbatim design-principle quotation.** `purplepincher/deckboss`'s
   `src/core/types/log-entry.ts` doc comment cites *"activelog-spec: 'No editing.
   Ever. Corrections are events, history is never rewritten — vital for training
   data provenance and trust'"* — this is a direct quote of
   `cocapn-foundation/project/foundation/repos/activelog-spec/README.md`'s "What's
   deliberately NOT in the spec" section ("No editing. Ever.") combined with its
   "Corrections" paragraph.
3. **Identical core architecture pattern**, independently re-derived in code:
   append-only entries, `correction.retract`/`correction.amend` events (ActiveLog
   spec) vs. `retracted`/`corrections[]` with `amend`/`retract` types (DeckBoss
   `log-entry.ts`); "merge = set union, conflict-free by construction" (ActiveLog)
   vs. "a state-based CRDT — a grow-only set of corrections merged by union"
   (DeckBoss README). Same idea, same terminology, independently typed out.
4. **`purplepincher/deckboss/ROADMAP.md`'s "Fleet learning: off the roadmap"
   entry is a direct, named rebuttal of a `cocapn-foundation` ambition** — see
   §3 below. You don't write a rejection that specific without having read the
   source.

**So the relationship is: two siblings under one author, at different maturity
levels, with the newer one explicitly reading and selectively adopting the
older one's design docs while consciously scoping down.** `cocapn-foundation` is
the shared design ancestor (a "Claude Design" export, i.e., a mockup/spec
handoff, not runnable code); `purplepincher/deckboss` is the first member of the
family to actually ship a working, tested product. None of the `SuperInstance
deckboss-*` code repos (net/hardware/marketplace/fab/ai/agent/1) are ancestors —
they're a different, non-fishing sub-family that happens to share the "deckboss"
brand name because it was originally a generic Jetson/RPi onboarding tool
(`Lucineer/deckboss`), not a fishing app.

---

## 3. Ideas worth absorbing vs. ideas already superseded

### Genuinely new scope/ideas in the SuperInstance/Lucineer side, absent from `purplepincher/deckboss`

These are real, specific, and `purplepincher/deckboss` currently has none of
them:

1. **The Helm hardware layer (ESP32 remote-emulator + five-layer safety
   architecture).** `cocapn-foundation/project/foundation/SAFETY.md` is a
   genuinely rigorous safety spec for voice-actuated steering: electrical
   (parallel wiring, momentary-only, de-energized-safe), firmware (hardware
   watchdog + 500 ms command TTL), protocol (single helm-token master, replay
   protection), voice (C0–C3 command classes with escalating confirmation —
   "belay"/"cancel" always wins), and human (mandatory sea-trial checklist).
   `purplepincher/deckboss` deliberately excludes this — see below, it's a
   considered exclusion, not a gap — but if the project ever *does* pursue
   hardware, this document is the one worth pulling wholesale rather than
   re-deriving.
2. **Device profiles as data, not code** (`autopilot-profiles/`,
   `cocapn-helm-firmware/README.md`). "One JSON file defines the wires, the
   words, and the screen" — new device support ships as a reviewable data file,
   never a firmware fork. This is a clean pattern that would generalize well if
   DeckBoss ever adds device integrations (NMEA 2000 gauges, AIS, etc.) even
   without literally doing the Helm/autopilot part.
3. **Media anchoring by time overlap, not by reference** (ActiveLog `ARCHITECTURE.md`
   Layer 3, `activelog-spec/README.md`). Camera frames and speech segments are
   linked purely by overlapping timestamps — no explicit foreign key needed at
   capture time. `purplepincher/deckboss` has no camera/vision feature at all
   yet; if it ever adds photo attachment to catches, this timestamp-correlation
   pattern (plus periodic `clock.beacon` events for cross-device clock-skew
   correction) is a specific, reusable idea.
4. **Structured `catch.assertion` extraction with `parse_conf`**
   (`activelog-spec` event types table + `troll-day.jsonl` example): a small
   deterministic parser turns "putting lingcod in the port tote" into
   `{species: "lingcod", container: "tote-port", parse_conf: 0.9}`, always keeping
   the raw sentence. `purplepincher/deckboss` has `entity-extractor.ts` (per
   `ARCHITECTURE.md`'s module graph) which is conceptually similar but — per its
   own ROADMAP — the fishing-vocabulary split into a dedicated `domain-packs/fishing/`
   is still just "opportunistic," not done. The ActiveLog spec's specific
   `species`/`container`/`count`/`raw`/`parse_conf` shape is a good concrete
   target for that extraction.
5. **Per-device hash chain for tamper-evidence** (`activelog-spec/schema/event.schema.json`'s
   `prev` field: "sha256 of this device's previous event line... tamper-evidence
   for logs used as legal/regulatory records"). `purplepincher/deckboss` explicitly
   disclaims regulatory/evidentiary use (README: "not a substitute for official
   catch reporting... hasn't been reviewed or certified as evidence-grade"), so
   this is currently irrelevant by design — but it's a specific, cheap idea
   (one extra hash field) to bank if that stance ever changes.
6. **BYOK *chat* with log-query tools** (`cocapn-app/README.md`: a `ChatProvider`
   interface where "the chat gets tools to query the local log
   (`events_between(t0,t1)`, `assertions(species=...)`)" — answering "when did we
   hit that pinnacle?"). `purplepincher/deckboss`'s "Ask-Your-Log" feature (README:
   "In progress, ahead of the beta") is a **local keyword/date/species parser,
   explicitly no LLM, no API key** — a narrower, more conservative version of the
   same idea. The BYOK-chat-with-tool-calling version is a plausible v2 upgrade
   path already spec'd out on the SuperInstance/cocapn side.
7. **Connectivity-tier framing (T0–T3)** (`ARCHITECTURE.md`'s table: T0 no
   connectivity ever / T1 dock WiFi only / T2 intermittent cell/Starlink / T3
   always on, "design target is T0, everything above is progressive
   enhancement"). `purplepincher/deckboss` has the *substance* of this (offline-first,
   sync queue, offline detection) but not this explicit tiering as a documented
   design/test framework — worth adopting just as a testing/prioritization
   checklist, cheap to add.
8. **`deckboss-marketplace`'s vendor/fee-tier model and `deckboss-fab`'s
   parametric OpenSCAD generator** are real, working, self-contained code (not
   vaporware) — but they belong to the hardware-sales side of the ecosystem
   (Helm units, Jetson cases), which is out of scope for `purplepincher/deckboss`
   as a software-only PWA. Not applicable unless the hardware line is revived.

### Ideas already superseded — `purplepincher/deckboss` has solved these better

1. **Sync correctness.** Nothing on the SuperInstance/Lucineer side has a
   working sync implementation at all — `deckboss-net`'s "reliable messaging"
   is a README with no code; `fishinglog-ai`'s storage model (commit-to-your-own-GitHub-repo
   via a `GITHUB_TOKEN` Worker secret) is implemented but crude compared to
   `purplepincher/deckboss`'s multi-backend `StorageAdapter` (Google Drive, R2,
   Oracle Object Storage, local ZIP) with property-based tests proving
   convergence and no-silent-drop guarantees under randomized concurrent
   scenarios, and a real fixed bug on record ("Fix critical bug: cloud sync
   silently no-op'd for every network backend"). The ActiveLog *spec* for merge-by-union
   is sound design (§3 above gives it credit), but `purplepincher/deckboss` is the
   only one of the two that actually built and tested a working sync engine —
   it just built it around Markdown files instead of JSONL event streams.
2. **Additive corrections — implemented vs. specified.** ActiveLog specifies
   `correction.retract`/`correction.amend`; DeckBoss ships them, with a documented
   invariant enforcer (`invariants.ts`), a CODEOWNERS lock on the audit-log core,
   and an explicit non-freezing-retraction semantics decision (`ARCHITECTURE.md`:
   "Retraction does not freeze field values") that the spec doesn't even address.
3. **BYOK storage vs. BYOK LLM key.** Worth flagging as a false-friend: the
   `byok.ts` shared across `deckboss-ai`/`fishinglog-ai` is about *LLM provider*
   keys (DeepSeek/OpenAI routing), not *storage* credentials. `purplepincher/deckboss`'s
   BYOK is about never holding the user's *data* (their own Drive/R2/Oracle
   account) — a different and, for a logbook app, more important problem. Nobody
   on the SuperInstance/Lucineer side solved storage-BYOK; `fishinglog-ai`'s
   "commit to your GitHub fork" approach is a real but much more awkward answer
   to the same problem (requires a GitHub token with `repo` scope stored in a
   Worker secret, coupling data ownership to git literacy).
4. **Honesty about maturity.** Every SuperInstance/Lucineer sibling *oversells*
   in its README relative to what's in the code (marketing pages presented as
   products, `TODO`-stub pipelines presented as "the co-captain AI"). `purplepincher/deckboss`'s
   README is unusually calibrated in the other direction — it explicitly
   disclaims regulatory/evidentiary status and states what's shipped vs. "in
   progress, ahead of the beta" with a specific feature list. This isn't a
   feature to "absorb," but it's the single biggest quality gap between the two
   sides and worth naming as the actual differentiator.
5. **Deliberate scope discipline.** `purplepincher/deckboss/ROADMAP.md` explicitly
   rejects two ambitions that `cocapn-foundation/VISION.md` treats as inevitable
   long-term: **"Fleet learning: off the roadmap"** (cross-vessel data sharing
   requires a DeckBoss-operated backend, which breaks the local-first trust
   promise — "removed as a planned phase," not deferred) and **"Multi-trade /
   'substrate' ambition: not now"** (VISION.md's studylog.ai/reallog.ai/playerlog.ai/businesslog.ai
   generalization — DeckBoss's ROADMAP says stay fishing-specific for 6+ months,
   "substrate plays get earned by dominating one vertical, not chosen in advance
   of any user validation"). Both are reasoned, explicit course corrections
   against the more grandiose SuperInstance-side vision, not oversights.

---

## 4. Recommendation

**Archive/deprecate now, no cherry-pick needed:**
- `deckboss-net`, `deckboss-agent`, `deckboss-ai-pages`, `deckboss-net-pages` —
  marketing pages and toy demos with no real logic. Archive; nothing to carry
  forward.
- `deckboss-1` — a stale (last push 2026-05-26), unmaintained fork of `Lucineer/deckboss`.
  Its "character sheet" hardware-profiling concept (`character_sheet.py`) is
  mildly interesting for a future *hardware companion app* but is speculative and
  low-value for the current software-only PWA. Archive.
- `deckboss-ai`, `DeckBoss` (capitalized) — genuinely different products (AI
  spreadsheet engine; agent orchestration OS) wearing the same name by
  coincidence. Not wrong to keep them alive as their own thing, but they should
  be **explicitly delinked from the fishing-logbook narrative** — right now
  anyone browsing SuperInstance by name would reasonably assume they're related
  to `purplepincher/deckboss` and they are not. At minimum, rename or add a
  one-line disambiguation to each README.

**Keep, but relabel as "hardware roadmap, not current product":**
- `deckboss-hardware`, `deckboss-marketplace`, `deckboss-fab` — these three form
  a coherent (if unimplemented-beyond-pricing-page/calculator) hardware-sales
  and CAD-generation story for a future Helm/Jetson product line. They're not
  useful to `purplepincher/deckboss` *today* (software-only PWA, no hardware),
  but they're the most concrete existing draft of what a hardware phase would
  look like if `cocapn-foundation`'s VaaR vision is ever revived. Don't archive;
  label them clearly as "Phase 3+ hardware speculation, not active."

**The one repo worth actually reading closely and cross-referencing from
`purplepincher/deckboss`'s own docs: `cocapn-foundation`.** It already is
referenced (the README cites it as the sibling holding `SAFETY.md`), but the
citation currently points at design documents that live only in a raw HTML
export bundle inside a repo whose own README says "these are prototypes, not
production code... a handoff bundle from Claude Design." That's a fragile
cross-reference for a repo `purplepincher/deckboss` leans on for its safety
story. Concretely worth doing: **promote `activelog-spec`'s schema/event
taxonomy and `SAFETY.md`'s five-layer defense model into a stable, versioned
document that `purplepincher/deckboss` can cite without depending on a
"design handoff bundle" repo staying put** — either mirror the two documents
directly into `purplepincher/deckboss/docs/`, or fork just `activelog-spec`
under `purplepincher` (it's the one sub-repo in the bundle that's an actual
spec with a JSON Schema and conformance-test intent, not a stub README).

**Not worth cherry-picking:** `Lucineer/fishinglog-ai`'s code itself (all
`TODO` stubs — the value is entirely in its `docs/ARCHITECTURE.md`, which is
useful *reading* for the eventual vision/hardware pipeline but has zero
reusable implementation), and the `deckboss-ai`/`fishinglog-ai` shared `lib/`
scaffold (`byok.ts` et al. — generic LLM-key boilerplate, solves a different,
smaller problem than DeckBoss's storage-BYOK, and DeckBoss's actual
`StorageAdapter` implementation is more relevant to its own needs than this
scaffold is).
