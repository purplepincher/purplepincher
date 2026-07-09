# SuperInstance narrative consistency — survey, vocabulary, and the fixes made

**Status:** survey + executed-fixes record, drafted 2026-07-09 on branch
`narrative-consistency-2026-07-09`, for review before merge. This document is
subordinate to `ROADMAP.md` (the decision record); it makes no architecture or
infrastructure decisions. Its scope is narrative, language, and cross-reference
consistency across the SuperInstance repos verified real this week, measured
against the org's mature understanding as recorded in this repo's `README.md`,
`docs/PARADIGM.md`, `docs/ROOMS.md`, and `docs/ACTIVELOG_SCHEMA_FOUNDATION.md`.

Honesty markers per the standing convention: **✅ real today** /
**⚠️ real but conditional** / **🔮 later phase**.

Every claim below was traced to a file read during this pass, at each repo's
`main` tip on 2026-07-09. Where a fix was executed, it lives on a branch named
`narrative-consistency-2026-07-09` in that repo — nothing was pushed to any
`main`.

---

## 1. The throughline, stated as a factual map (not a pitch)

The one-sentence relationship this org already states on its front door
applies repo-by-repo: *ideas start soft at SuperInstance; a few of them harden
at purplepincher* (`README.md`, closing line). Within the SuperInstance repos
surveyed here, there are **two real lineages, two graduated pairs, one pages
repo — and a set of name collisions that look like connections but are not.**

### Lineage A — Cocapn / ActiveLog (the design-brief line)

| Repo | What it is | Depends on / relates to | Roadmap position |
|---|---|---|---|
| `SuperInstance/cocapn-foundation` | 🔮 A design brief (no runnable code, says so itself): the Cocapn helm-assistant safety case + the **ActiveLog v1 event envelope** (`project/foundation/repos/activelog-spec/`) | Nothing upstream. Downstream: everything in this row's column | The ancestor. `ROADMAP.md` Step 0 identified it as DeckBoss's "real design ancestor," confirmed by direct evidence |
| `purplepincher/deckboss` | ✅ Shipped, field-ready voice logbook PWA | Implements a knowingly scoped-down version of cocapn-foundation's design; mirrors its `SAFETY.md` + `activelog-spec/` into `docs/cocapn-foundation-mirror/` | Step 1 (field beta) — the org's center of gravity |
| `purplepincher/activelog` + this repo's `docs/ACTIVELOG_*` | ⚠️ Phase 1 in flight | The envelope from `activelog-spec` is the adopted one-envelope foundation (`ACTIVELOG_SCHEMA_FOUNDATION.md` S1) | Step 5, core first, skins gated |
| `SuperInstance/cocapn-marine` | ✅ Real, tested Rust library (NMEA 0183, PID autopilot, bathymetry, deadbands; 70/71 tests, CI) | *Designed for* the Cocapn sensor tier; **nothing consumes it yet**, and it stands alone as a general marine library | Not on the roadmap as a step; a real artifact the Step 5 hardware phase (🔮) could someday pull |

### Lineage B — the PLATO tile-server family

PLATO is the sketchbook's shared tile-server memory: a **room** is a named
collection on a server, a **tile** is one JSON record in it (canonical
plain-language definition: `capitaine-agent/README.md`, "Where PLATO fits").
The server itself is **not published in any of the surveyed repos** — every
member of this family is a client defaulting to `http://localhost:8847`.

| Repo | What it is | PLATO relationship |
|---|---|---|
| `SuperInstance/plato-portal` | ✅ Python SDK published as **`superinstance`** on PyPI (agents with markdown memory, in-memory fleet, LRU cache) | SDK itself: none. `scripts/plato-*.py` are clients of an external server |
| `SuperInstance/fishinglog-agent` | ✅ PyPI client: fishing sessions as tiles in room `fishinglog-ai` | Client only (its README already said so — the honesty model for its siblings) |
| `SuperInstance/activeledger-agent` | ✅ PyPI client: activity/investment/trade tiles in room `activeledger-ai`; real `fleet-agent` dependency | Client only |
| `SuperInstance/reallog-agent` | ✅ PyPI client: **text** scene descriptions as tiles in room `reallog-ai`. ⚠️ No camera/vision code exists in the package | Client only |
| `SuperInstance/luciddreamer-agent` | ✅ PyPI, standalone in-memory dream/sleep journal with JSON export | **None in code.** Constructor stores `plato_url` and never uses it (verified: no network code in the package) — name-family membership only |
| `SuperInstance/activelog-agent` | ✅ PyPI, zero-dependency log-**file** monitoring toolkit | None. And it is **unrelated to the ActiveLog event format** despite the name (see §3) |

### Graduated pairs (sketch here, hardened continuation at purplepincher)

- `SuperInstance/git-native-agents` → `purplepincher/git-native-agents`.
  The SI copy's README is honest about its own limits (`index.lock`
  collisions, no tests); the graduated fork fixed the race with `flock` and
  ships `tests/run.sh` + `tests/concurrency.sh` (verified by reading the fork).
- `SuperInstance/sonar-vision` → `purplepincher/sonar-vision`. Identical tree
  at the production-grade merge, then the PP copy got unmasked CI, a docs
  honesty pass, a `CHANGELOG.md`, and more tests (verified: `f964dda` is an
  ancestor of the PP tip; the two PR-merge tips have an empty diff).
  ⚠️ Note the task brief's "verify org" question resolves as: **both exist**;
  purplepincher is the maintained one.

### Pages

- `SuperInstance/superinstance-ai-pages` — static source for superinstance.ai:
  site pages, self-contained demos, and a large set of speculative design
  essays (`FLEET-ARCHITECTURE.md`, `FORMAL-PROOFS.md`, …) that must be read as
  sketchbook writing, not shipped-system documentation.

### The name collisions that are not connections (the map's most important row)

- **`activelog-agent` ≠ ActiveLog.** A log-file monitor vs. the event-log
  format. No shared code, no dependency.
- **`fishinglog-agent` / `activeledger-agent` / `capitaine-agent` ≠ the
  `fishinglog.ai` / `activeledger.ai` / `capitaine.ai` skins.** `ROADMAP.md`
  Step 5 names those domains as gated future *skins of the ActiveLog core*.
  The PyPI packages wearing the same names are earlier, PLATO-tile-or-standalone
  sketches with a different data model. They are not early versions of the
  skins, and no doc should imply they are. (Per this repo's own README: do not
  infer PurplePincher products from SuperInstance repo names.)
- **PLATO tile ≠ ActiveLog event.** Structurally similar (one timestamped
  record in a named stream) but different systems — `ROADMAP.md` Step 4's
  technical-fit note already states the PLATO-tile repos "implement a
  completely different data model … from what cocapn-foundation actually
  needs (ActiveLog events)." The consistency fix is *definitional clarity and
  explicit non-equivalence*, not terminology unification.

---

## 2. Per-repo survey and what was done

Each entry: the story the repo told, the gap, the highest-value fix, and
whether it was executed (all executed fixes are README-only, on branch
`narrative-consistency-2026-07-09` in that repo).

### 2.1 `cocapn-marine` — **executed: the batch's one hard doc-truth bug**

Register was already excellent (markers, verified outputs, limitations
section). But the Install section instructed `cocapn-marine = "0.1"` and **the
crate does not exist on crates.io** (checked via the crates.io API,
2026-07-09: "crate `cocapn-marine` does not exist") — a non-working install
command, the org's most-hunted bug class and adoption-bar item 4 violated in
place. Fixed: git-dependency install + ⚠️ not-published marker + a "does NOT
do yet" entry, matching the pincher precedent ("`cargo install pincher` does
not work yet"). Also softened "It **is** the sensor-and-control layer
underneath the wider Cocapn/ActiveLog system" to "designed as" — nothing
consumes it yet.

### 2.2 `cocapn-foundation` — **executed: connect the brief to its shipped descendants**

The README was honest about being a design brief but never mentioned that the
design *has a shipped descendant*: a reader had to already know that
`purplepincher/deckboss` implements a scoped-down version of it (evidence:
`ROADMAP.md` Step 0; deckboss's `log-entry.ts` quotes "No editing. Ever."
verbatim; `deckboss/docs/cocapn-foundation-mirror/` verified present with
`SAFETY.md` + `activelog-spec/`). Added a "Where this design has gone since"
section: deckboss (✅, model-not-envelope stated precisely per
`ACTIVELOG_SCHEMA_FOUNDATION.md` §0.1–0.2), cocapn-marine (✅, unconsumed),
the schema-foundation doc as the envelope's continuation (⚠️), everything else
still 🔮, plus the `activelog-agent` disambiguation.

### 2.3 `git-native-agents` — **executed: pointer to the hardened fork**

README already exemplary on honesty (explicit "no LLM, no reasoning" section;
✅/⚠️/🔮 markers). Gap: no mention that the graduated
`purplepincher/git-native-agents` fixed the exact concurrency limitation the
SI README documents. Added a "Relationship to purplepincher/git-native-agents"
section; the SI copy's self-description remains true of itself.

### 2.4 `sonar-vision` (SuperInstance copy) — **executed: replace ghost-fleet framing with the real pointer**

"Sonar Vision is part of the SuperInstance fleet ecosystem" said nothing
checkable — the exact framing the PP copy deliberately removed from itself
(PP commit `0e7ed5c`, "remove SuperInstance fleet framing"). Replaced with the
verified maintenance-moved note.

### 2.5 `reallog-agent` — **executed: full honest rewrite (worst offender)**

Old README: 25 lines, pitch register, "Camera events → PLATO → persistent
spatial memory," ungrounded hermit-crab metaphor ("Don the shell"). Code
reality (verified in `reallog_agent/__init__.py`): caller supplies text;
`ask()` is a first-three-words substring filter over the last 50 tiles;
`don_shell()` constructs a new client with a different `camera_id`; no
image/video code exists. Rewritten in the sibling clients' register with
markers, real requirements (`fleet-agent`, `requests`, reachable PLATO
server), and an honest license note — **no LICENSE file or pyproject license
field exists** (PyPI shows `license: None`); the old implied-MIT status needs
a maintainer decision (§4).

### 2.6 `activelog-agent` — **executed: name disambiguation**

Already had markers and honest scope. The one gap was the name: added the
"Naming note" distinguishing it from the ActiveLog event format — the same
move Step 0 item 3 applied to the `deckboss-ai`/`DeckBoss` collisions.

### 2.7 `activeledger-agent` — **executed: define PLATO instead of linking the org**

`[PLATO](https://github.com/SuperInstance)` linked the whole org as a
definition. Replaced with the shared one-line room/tile definition +
client-only caveat + sibling links.

### 2.8 `fishinglog-agent` — **executed: minor; it was already the model**

Recently fixed (its own commit: "honest README (no fabricated hot-spot
detection), drop unused fleet-agent dep") and already carried the "client
only; does not ship or run PLATO itself" line the siblings now share. Added
only the common room/tile definition and sibling links so all three clients
describe PLATO in the same words.

### 2.9 `luciddreamer-agent` — **executed: unused-parameter honesty note**

README was factual, but the module docstring claims "Integrates with the
PLATO memory layer" and the constructor accepts `vessel`/`domain`/`plato_url`
— all stored, never used; the package has no network code at all. Added a
"How it fits (honest scope)" section (✅ in-memory + JSON export; ⚠️ reserved
slots, not a capability). The docstring itself is a code change left as a
recommendation (§4).

### 2.10 `capitaine-agent` — **executed: fleet-neighbor table demoted to personas**

The strongest README in the SuperInstance batch (the PLATO section is the
org-wide model for deflating an aspirational integration). One inconsistency:
its closing "Fleet context" recited `AGENT.md`'s neighbor table
(`tminus-dispatcher`, `fleet-bridge`, …) as context — sketchbook personas
with no code connection (the package has zero runtime dependencies), the
exact "Fleet Neighbors table pointing at sketchbook repos that were never
hardened" pattern `docs/ROOMS.md` names in its opening section. Rewrote that
bullet to say so, applying the README's own PLATO discipline to its fleet
paragraph.

### 2.11 `plato-portal` — **executed: the three-names map**

README substance was already fixed in a prior pass (vision clearly separated
under "Where this is going"). What remained: nothing explained why a repo
called `plato-portal` ships a PyPI package called `superinstance`, or what
PLATO means, while four sibling packages cite "PLATO tiles" as if defined
somewhere. Added an "About the names" section: repo/package mapping, the
room/tile definition, client-only caveat, and the explicit PLATO-tile ≠
ActiveLog-event non-equivalence.

### 2.12 `superinstance-ai-pages` — **executed: factual rewrite**

Old README claimed "The Cocapn Fleet operates 20+ interconnected domains,
each with its own personality" (uncheckable, pitch register) and linked a
"fleet dashboard" at `http://147.224.38.131:4046/` that is unreachable
(verified 2026-07-09). Rewritten: what the repo actually contains (pages,
self-contained demos, speculative essays — now labeled as sketchbook writing),
sibling `*-ai-pages` repos verified to exist, and only links that resolve
(superinstance.ai and cocapn.ai both returned 200).

---

## 3. The vocabulary rules (picked from the mature docs, none invented)

1. **Honesty markers on every capability claim**: ✅ real today / ⚠️ real but
   conditional / 🔮 later phase, never "coming soon"
   (`ACTIVELOG_SCHEMA_FOUNDATION.md` header convention). Now present in every
   surveyed README.
2. **"ActiveLog event"** means exactly one thing: the `activelog-spec` v1
   envelope — `(alv, dev, seq, ts, mono, type, body)`, append-only, set-union
   merge on `(dev, seq)` (`ACTIVELOG_SCHEMA_FOUNDATION.md` §0.1, S1: one
   envelope, many namespaced types, never a second envelope).
3. **"PLATO room" / "tile"** means the tile-server model: room = named
   collection, tile = one JSON record. Always say **"PLATO room"**, never bare
   "room", in cross-repo writing — because
4. **"room" is a three-way collision** in this org's record: (a) a PLATO
   room; (b) `plato-engine-block-c`'s sensor/actuator/alarm physical room;
   (c) `docs/ROOMS.md`'s room-as-repo direction. All three are real usages
   with real ancestors; the rule is qualification, not renaming.
5. **"skin"** is reserved for the gated Step 5 white-labels of the ActiveLog
   core. The SuperInstance `*-agent` PyPI packages are **not** skins and must
   not be described as such.
6. **"fleet"** should mean coordination code that exists (git-native-agents'
   spawn/send/tick; plato-portal's in-memory `Fleet` class) — not a collective
   noun implying that separately-published packages form an operating system
   of agents. The "Cocapn Fleet operates 20+ interconnected domains" framing
   is retired with the superinstance-ai-pages rewrite.
7. **Crab/shell language** stays where it does argumentative work
   (`PARADIGM.md`'s vacancy chain) and goes where it is decoration —
   `ROOMS.md` §4 already states the asymmetry ("strongest where the crab
   biology was doing real argumentative work … weakest where it was persona
   theater"). Applied here: reallog's "don the shell" is now described as
   what it is (a re-scoped client).
8. **Register**: precise factual opener, real usage example, explicit
   limitations/"does NOT do yet" section, no superlatives — the tone the
   most mature SI repos (cocapn-marine, git-native-agents, capitaine-agent)
   already model.

---

## 4. Left as recommendations — judgment calls for Casey, not guessed at

1. **Archive-or-keep for the two stale originals.** `SuperInstance/sonar-vision`
   and `SuperInstance/git-native-agents` now point at their purplepincher
   continuations, but the Step 0 precedent (archiving the `deckboss-*`
   false-friends) suggests archiving may be the cleaner end state. Owner call.
2. **Whether the SI PLATO clients should mention the Step 5 skins at all.**
   I deliberately did **not** add "a future fishinglog.ai skin exists on a
   roadmap" pointers to `fishinglog-agent`/`activeledger-agent`/
   `capitaine-agent`: this repo's README explicitly warns against inferring
   PP products from SI names, and a forward-pointing link could read as a
   promise. If Casey wants the skins acknowledged in those READMEs, it should
   be worded as "an unrelated future project reserves this domain," decided
   deliberately.
3. **`reallog-agent` license**: no LICENSE file, no pyproject license field,
   PyPI shows none. Siblings use MIT; someone with authority should add it.
4. **`luciddreamer-agent` docstring** (code change): remove/soften
   "Integrates with the PLATO memory layer" in
   `luciddreamer_agent/__init__.py` — the README now tells the truth, but the
   module still overclaims to anyone reading `help()`.
5. **`cocapn-marine` publication**: publish to crates.io (making the old
   install line true) or leave the git-dependency instructions standing; and
   add the missing `LICENSE` file its own `GRADUATION_NOTES.md` flags.
6. **The `AGENT.md` ensign files** across SI repos (capitaine-agent,
   sonar-vision, activelog-agent, activeledger-agent, superinstance-ai-pages)
   still carry Fleet Neighbors persona tables. `ROOMS.md` already identifies
   the pattern and the keep-the-kernel move ("I reside in this repository" +
   duty log = real; neighbor tables = theater). A dedicated AGENT.md pass —
   also touching `plato-portal`'s trailing epigraph and similar signature
   lines — should be its own commit series, not smuggled into a README pass.
7. **`fleet-agent`** (PyPI, the base class reallog/activeledger really depend
   on) ships a `fleet_math` module (`EmergenceDetector` "H1 cohomology",
   `HolonomyConsensus`) — speculative-math naming inside a real, installed
   dependency. If the PLATO client family matters going forward, `fleet-agent`
   deserves its own two-layer intake; it was outside this pass's repo list.
8. **The superinstance-ai-pages essays themselves** (FORMAL-PROOFS.md etc.)
   are labeled from the README now, but per-essay banners would be a larger
   editorial decision about how loudly the sketchbook disclaims itself on its
   own pages.

---

*Survey document. Decision record: [`ROADMAP.md`](../ROADMAP.md). Vocabulary
source of truth: [`ACTIVELOG_SCHEMA_FOUNDATION.md`](./ACTIVELOG_SCHEMA_FOUNDATION.md);
register source: [`docs/PARADIGM.md`](./PARADIGM.md) and the org README. The
executed fixes cited in §2 live on branch `narrative-consistency-2026-07-09`
in each named SuperInstance repo, unmerged, awaiting review.*
