# ActiveLog schema foundation — one envelope, one connector record, eight skins

**Status:** synthesis / decision-support, drafted 2026-07-09 on branch
`activelog-schema-foundation-2026-07-09`, for review before merge. This
document reconciles three currently-separate schema efforts — the
`activelog-spec` event envelope, the Phase 1 `LogEntry` extraction now in
flight on `purplepincher/activelog` branch `activelog-phase1-2026-07-09`, and
the room/capability manifest direction (`docs/ROOMS.md` +
`docs/research/ROOM_MANIFEST_PRECEDENTS.md`) — into one coherent data-shape
foundation. It is subordinate to `ROADMAP.md` (the decision record) and to
`docs/ACTIVELOG_ARCHITECTURE.md` (the seam map); where it proposes anything,
it proposes **schema shape only** — no new infrastructure, no new repos, no
new adopted dependencies, and no change to any gate. The room-manifest
material in §2 inherits ROOMS.md's own status ("this paper decides nothing")
— §2 fixes the *shape* the direction would use, not its schedule.

**Why this file, and why here.** The three efforts live in three repos: the
canonical spec is mirrored in
`deckboss/docs/cocapn-foundation-mirror/activelog-spec/`, the Phase 1 build
lives in `purplepincher/activelog`, and the manifest research lives in this
meta-repo's `docs/research/`. The only shelf all three already share is this
meta-repo's `docs/`, beside `ACTIVELOG_ARCHITECTURE.md` and
`ACTIVELOG_FIRST_SLICE.md`, which this document sits exactly between: the
architecture doc owns the seams, the first-slice doc owns the sequence, and
this doc owns the *data shapes* both of them reference. Putting it in the
`activelog` repo instead would invert the dependency (seven unbuilt skins and
the room direction would then cite an implementation repo for their
foundation), and putting it under `docs/research/` would misfile it — it is
synthesis across the org's own artifacts, not verification of anything
external, the same distinction ROOMS.md §5 draws for itself.

Honesty markers per `ACTIVELOG_ARCHITECTURE.md`'s convention:
**✅ real today** (verifiable in a shipped sibling or a fetched primary
source), **⚠️ real but conditional** (works with a stated tradeoff or is
spec-defined but unimplemented), **🔮 later phase** (buildable, deliberately
not now, never labeled "coming soon").

Every claim below about what currently exists is cited to a file read in
full during this pass, at its current tip: `purplepincher/purplepincher`
`main`, `purplepincher/deckboss` `main`,
`purplepincher/activelog` branch `activelog-phase1-2026-07-09` (tip commit
`18a0458`, "chore: bootstrap the Phase 1 app scaffold"), and
`purplepincher/pincher` `main`.

---

## 0. The three schema efforts, as they verifiably exist right now

### 0.1 The ancestor: the `activelog-spec` envelope ✅ (spec real; implementation partial)

`deckboss/docs/cocapn-foundation-mirror/activelog-spec/README.md` defines the
envelope every event carries:

```json
{
  "alv": 1,
  "dev": "phone-a1b2c3",
  "seq": 4021,
  "ts": 1782000123456,
  "mono": 74520331,
  "type": "speech.segment",
  "fix": { "lat": 55.3421, "lon": -131.6461, "sog": 2.4, "cog": 291 },
  "body": { }
}
```

with the merge rule quoted verbatim throughout the org's docs: *"a log set is
merged by set-union on `(dev, seq)`, sorted by `(ts, dev, seq)`. Append-only
+ unique keys = conflict-free by construction. No CRDTs, no vector clocks, no
server authority."* The machine-checkable half is
`activelog-spec/schema/event.schema.json`: required
`["alv","dev","seq","ts","mono","type","body"]`, `additionalProperties:
false` at the envelope level, `ts` an **integer** (UTC epoch ms), `type`
matching `^[a-z]+\.[a-z_]+$`, `fix` optional with required `lat`/`lon` only,
and an optional `prev` sha-256 per-device hash chain. The README's v1 type
table ships nine types (`speech.segment`, `helm.command`, `helm.event`,
`fix.track`, `media.frame`/`media.clip`, `catch.assertion`, `chat.exchange`,
`mark.note`, `session.meta`) plus the two correction types
(`correction.retract`, `correction.amend` referencing `{dev, seq}` of the
target), and two load-bearing extensibility rules: *"unknown types must be
preserved by all tools"* and *"No schema for fish species, ports, gear
(domain vocabularies are versioned data packages, like autopilot profiles —
not spec)."*

**What "partial implementation" means, precisely:** no code anywhere in the
org emits this envelope byte-for-byte today. DeckBoss implements the
envelope's *logical model* (append-only, additive corrections, read-time
fold) in a different wire shape (§0.2), and the `activelog` repo's own
`docs/product-status.md` states it for the landing page: the envelope is
"⚠️ a design described on the page; **no code for it exists in this repo**."

### 0.2 The shipped realization: DeckBoss's `LogEntry` ✅ — which does **not** carry `(dev, seq)`

`deckboss/src/core/types/log-entry.ts` is the field-tested markdown-on-disk
realization. Its exact shape (`logEntryShape`, lines 168–180):

```ts
{
  id: uuidV4,
  timestamp: isoDateString,   // capture time — the primary key
  gps: GPSReadingSchema.nullable(),
  audio: AudioMetaSchema.nullable(),
  transcript: TranscriptResultSchema.nullable(),
  entities: z.array(EntitySchema),   // fishing-flavored enum: gear/depth/species/…
  tags: z.array(z.string()),
  source: EntrySourceSchema,         // "voice" | "text" | "import"
  thread_id: uuidV4,
  version: z.string(),               // SCHEMA_VERSION = "1.0" (common.ts:13)
  corrections: z.array(CorrectionSchema),
}
```

Three properties of this file matter for the reconciliation:

- **Its key is `id` (uuid) + `timestamp` (ISO string) — there is no `dev`,
  `seq`, `mono`, or `alv` field anywhere in the type.** The only device
  identity in the file is `Correction.deviceId` (optional, uuid, stamped on
  new corrections). This is the single largest gap between the shipped code
  and the spec it cites, and it is the subject of §4.
- **The additive-corrections model is the spec's correction model,
  aggregated per entry**: `retracted`/amend live as `Correction` objects in
  the entry's own `corrections` array, folded at read time by
  `applyCorrections()` — the file's header comment quotes the spec's "No
  editing. Ever." verbatim. `entry-serializer.ts` writes the complete
  machine record as YAML frontmatter; "the body is cosmetic —
  entry-parser.ts reads frontmatter only."
- **Forward compatibility is already engineered in**: `LogEntrySchema` is
  `.passthrough()` so "a future version's unknown fields still survive a
  round-trip through an older client" — the file's own comment names this as
  mirroring "activelog-spec's forward-compatibility rule." And
  `tensor-log/invariants.ts` enforces immutability with a compile-time
  coverage guard over `IMMUTABLE_FIELDS`
  (`id, timestamp, gps, audio, source, thread_id, version, transcript,
  entities, tags`).

### 0.3 The build in flight: Phase 1 on `activelog` ⚠️ (scaffold only — no schema code has landed)

Verified at branch tip `18a0458` (authored 2026-07-09): the commit adds
`app/` (Vite + React + TS building into the existing Worker's `public/`
assets dir) and nothing else; `app/src/App.tsx` is, in full,
`"ActiveLog Phase 1 — scaffold placeholder, not yet wired."` The commit
message says the storage/capture extraction "happens next, per
docs/ACTIVELOG_FIRST_SLICE.md §7." So as of this synthesis, **Phase 1 has
made no schema choice yet** — every recommendation in §1 and §4 lands before
any code has to be corrected. What Phase 1 is *committed to* on paper
(`ACTIVELOG_FIRST_SLICE.md` §4, decision D3): "markdown-frontmatter +
manifest + `(dev,seq)` merge rule. Envelope design deferred to architecture
doc" — and §7 item 1: "a trimmed `LogEntry` (entities reduced to a neutral
`tag` set; GPS optional/nullable; additive `corrections` and `thread_id`
kept verbatim), **keyed by `dev` + `seq`**."

### 0.4 The later-phase event producers the architecture doc already names

`docs/ACTIVELOG_ARCHITECTURE.md` commits future phases to three more event
families: BYOK chat turns recorded as the spec's `chat.exchange`
("`body: { role, text, model, key_owner: "byok" }`" — §5, with `key_owner:
"on-device"` for the Chrome Prompt API path), the §5.5 connector-dispatcher
("a connector is anything wake-word-addressable that can receive a request
and optionally return a result," with pincher's
`Allow`/`Deny`/`RequireConfirmation` veto shape), and the parked hardware
tier (ESP32/autopilot I/O — §7, 🔮, landing in the `cocapn.ai` skin). §6
already makes the key banking decision this doc builds on: "**store the
`(dev, seq)` keys and the `mono` monotonic clock because they're free and
make merge deterministic; do not enable the `prev` hash chain until a real
evidentiary use case appears.**"

### 0.5 The manifest effort

`docs/ROOMS.md` (direction paper; its own bottom line: "Nothing proposed in
this document exists") sketches a three-layer room convention and identifies
pincher's skill files as the lever-declaration ancestor.
`docs/research/ROOM_MANIFEST_PRECEDENTS.md` then did the field-level
comparative research across eight real systems and ranked them; its own
one-line takeaway: *"The smallest agent-first manifest that real precedent
supports is a fusion of three fields this org can already name: an
MCP-style invocation block …, a PlatformIO-style compatibility block …, and
an npm-style flat entry-point/command field — with trust deferred to
per-discovery re-verification."* §2 executes that pick.

---

## 1. The event schema: one envelope, many event types — not many envelopes

### 1.1 The verdict, stated first

**Everything the org has planned — Phase 1 voice entries, BYOK chat turns,
connector call/response pairs, hardware I/O — fits the one `activelog-spec`
envelope as distinct event *types* with typed bodies. No second envelope is
needed, and none should be created.** The spec was explicitly designed for
this: the envelope carries only capture facts every event has (`who
(dev)`, `when (ts, mono)`, `where (fix, optional)`, `what kind (type)`) and
pushes everything type-specific into `body`, with unknown types preserved.
The architecture doc already behaves as if this were decided — it reuses
`chat.exchange` from the spec rather than inventing a chat record (§5,
"Chat provenance in the log — reuse the spec's event"), and the spec already
contains the hardware family (`helm.command`, `helm.event`) the 🔮 phases
need. What has *not* been done before this doc is (a) stating the
LogEntry↔envelope correspondence at field level so "compatible by
construction" (`ACTIVELOG_FIRST_SLICE.md` §4) is a checkable table instead
of an assertion, and (b) reserving the connector-dispatcher event types so
§5.5's day-one "build chat *as* the first connector" discipline has a log
shape waiting for it.

The one honest subtlety: DeckBoss's `LogEntry` is not an event — it is an
**aggregate** (one capture plus all its corrections, in one file). That is a
difference of *wire representation*, not of model. Stated as the invariant
this whole section defends:

> **There is one logical model** — append-only records, globally keyed by
> `(dev, seq)`, merged by set-union, mutated only by additive correction
> records, folded at read time — **and two sanctioned wire representations
> of it:**
>
> - **R1 — markdown-file-per-entry** (Phase 1; DeckBoss-proven): the entry
>   file's frontmatter carries the capture record *and* its corrections
>   array. Two devices' corrections arrays union on sync
>   (`deckboss/src/core/sync/conflict-resolver.ts`), which is exactly the
>   spec's event set-union restricted to one entry's correction events.
> - **R2 — JSONL event stream** (the spec's canonical form:
>   `{dev}/{yyyy-mm-dd}.alog.jsonl`; later phases): one line per event,
>   corrections as standalone `correction.*` events referencing their
>   target's `(dev, seq)`.
>
> R1 → R2 must be a **mechanical, lossless export** (§1.3's mapping table is
> the definition of "mechanical"), and R2 → R1 a mechanical fold. Any schema
> choice that would break either direction is a breaking change and needs
> this doc amended first.

### 1.2 The Phase 1 `LogEntry`, concretely — the trimmed shape this doc proposes

This is `ACTIVELOG_FIRST_SLICE.md` §7 item 1 made field-precise. Deltas from
`deckboss/src/core/types/log-entry.ts` are marked; everything unmarked is
inherited verbatim, including the zod idioms (`.passthrough()` on the
parsing schema only; the `invariants.ts` compile-time `IMMUTABLE_FIELDS`
guard, updated for the field set below).

```ts
// activelog src/core/types/log-entry.ts — proposed Phase 1 shape
const logEntryShape = {
  id: uuidV4,                        // kept: filename & thread anchor
  dev: z.string().regex(/^[a-z0-9][a-z0-9-]{2,63}$/),  // ← ADDED (spec `dev`)
  seq: z.number().int().nonnegative(),                  // ← ADDED (spec `seq`)
  timestamp: isoDateString,          // capture wall time (↔ spec `ts`, see §1.3)
  mono: z.number().int().nonnegative().nullable(),      // ← ADDED (spec `mono`; ⚠️ see below)
  gps: GPSReadingSchema.nullable(),  // kept; OPTIONAL in the neutral core
  audio: AudioMetaSchema.nullable(),
  transcript: TranscriptResultSchema.nullable(),
  tags: z.array(z.string()),         // ← REPLACES deckboss `entities` (see §1.4)
  source: EntrySourceSchema,         // "voice" | "text" | "import"
  thread_id: uuidV4,
  version: z.string(),               // LogEntry schema version (distinct from alv)
  corrections: z.array(CorrectionSchema),  // kept; new corrections also stamp dev+seq (§1.3)
};
```

- `dev` is generated once per install and stored local-only (the same
  lifecycle as DeckBoss's `Correction.deviceId`, promoted to the entry
  itself); `seq` is a per-device counter that never rewinds. Together they
  are the merge key `ACTIVELOG_FIRST_SLICE.md` §1 calls "the one
  forward-compatibility guarantee Phase 1 must not skip."
- `mono` ⚠️ **honest browser caveat:** the spec's `mono` is a device
  monotonic clock; the browser's `performance.now()` is *session*-monotonic
  (it resets per page load). Phase 1 should record it (rounded ms,
  nullable when unavailable) and the R2 export should emit it as-is — it
  still buys the spec's stated purpose ("order and measure intervals even
  when `ts` was wrong at capture") *within* a session, and ordering across
  sessions is carried by `ts` + `seq`. Do not claim device-boot monotonicity
  the platform can't provide.
- `EditableFieldsSchema` (what a `Correction.amend` may overlay) trims to
  `{ transcript, tags }` — `entities` leaves the neutral core (§1.4). The
  `IMMUTABLE_FIELDS` list becomes
  `id, dev, seq, timestamp, mono, gps, audio, source, thread_id, version,
  transcript, tags` with the same compile-time coverage guard, so the drift
  bug that list already caught once in DeckBoss cannot recur here.

### 1.3 The R1 ↔ R2 mapping table (this is the "compatible by construction" claim, made checkable)

One Phase 1 `LogEntry` ≡ one capture event plus zero-or-more correction
events in R2:

| `LogEntry` (R1, frontmatter) | Envelope event (R2, JSONL) | Notes |
|---|---|---|
| `dev` | `dev` | identical |
| `seq` | `seq` | identical |
| `timestamp` (ISO 8601) | `ts` (integer UTC epoch ms) | `Date.parse()` / `toISOString()`; lossless at ms precision both ways. R1 keeps ISO because frontmatter is the human-browseable surface (DeckBoss convention); R2 keeps the integer because the spec schema requires it (`event.schema.json`: `"ts": {"type": "integer"}`) |
| `mono` | `mono` | spec requires it on every event; export `0` when the R1 value is null, with `body.mono_absent: true` so the fact isn't invented ⚠️ |
| — | `alv: 1` | R2-only; R1's spec lineage is carried by this doc + `version` |
| `version` | `body.schema_version` | LogEntry schema version ≠ envelope version; both survive round-trip |
| `id` | `body.id` | uuid retained for filenames/threads; **not** the merge key in either representation |
| `gps` (full `GPSReading`) | `fix` (coarse) **+ `body.gps`** (full) | `fix` gets `lat ← latitude`, `lon ← longitude`, `cog ← heading`, `sog ← speed` converted m/s → knots; the full reading (incl. `accuracy` meters, `altitude`, `source`, its own `timestamp`) goes in `body.gps` intact. `accuracy` is **not** mapped to `hdop` — meters and a dilution ratio are different quantities, and faking one from the other would be a small lie in every exported event |
| `source`, `audio`, `transcript`, `tags`, `thread_id` | `body.{source, audio, transcript, tags, thread_id}` of a `speech.segment` event (`text ← transcript.text`, `dur_ms ← audio.duration_ms`, `conf ← transcript.confidence`, `mode: "dictation"`) | the spec's `speech.segment` body highlights (`text`, `dur_ms`, `mode`, `conf`, optional `audio` ref) map one-to-one |
| `corrections[i]` | one `correction.amend` / `correction.retract` event, `body.target: {dev, seq}` of the entry | **requires corrections to carry their own `(dev, seq)`** — so Phase 1 stamps `dev` and `seq` on every new `Correction` alongside the `deviceId` DeckBoss already stamps. Cheap now; a migration later |

Everything in the table is additive on top of DeckBoss's proven serializer
(`entry-serializer.ts` dumps the whole record as YAML; extra fields ride
along for free, and DeckBoss's own `.passthrough()` idiom means a DeckBoss
client reading an ActiveLog file would preserve the new fields untouched).

### 1.4 Where domain entities went — the spec already answered this

DeckBoss's `EntityTypeSchema` is a closed, fishing-flavored enum
(`gear, quantity, location_relative, depth, species, weather, person,
measurement, other` — `log-entry.ts`). The FIRST_SLICE trims it to neutral
`tags`, which raises the obvious question: where do `businesslog`'s
line-items or `cocapn`'s gear events live, if not in a per-entry entity
array that every skin would have to extend?

The spec already made this call, and this doc adopts it as the standing
rule: **domain structure lives in namespaced event *types* with typed
bodies, defined by versioned vocabulary packages — never in the core
entry/envelope schema.** The spec's own words: "No schema for fish species,
ports, gear (domain vocabularies are versioned data packages, like autopilot
profiles — not spec)," and its `catch.assertion` type is the worked example:
a *parsed domain claim* (`species`, `container`, `count?`) carrying its
provenance (`raw` — the sentence it was parsed from — and `parse_conf`).
The pattern every skin repeats: the raw `speech.segment` is always recorded
(capture truth), and a domain parse, when one exists, is a **separate
assertion event** referencing the same moment — never a mutation of the
capture record. DeckBoss's in-entry `entities` array was the single-domain
shortcut for this; the multi-skin system uses the spec's shape.

### 1.5 The event-type registry, consolidated with markers

Rules first (all three are spec text, restated as normative for every skin):
types are `namespace.noun` matching `^[a-z]+\.[a-z_]+$`; unknown types MUST
be preserved by every tool; a skin owns one or more namespaces and defines
its bodies in a versioned vocabulary package.

| Type | Body (highlights) | Status |
|---|---|---|
| `speech.segment` | `text, dur_ms, mode (dictation/command), conf, audio?, speaker?` + Phase 1 fields per §1.3 | ✅ spec v1; Phase 1 realizes it as R1 |
| `correction.amend` / `correction.retract` | `target {dev,seq}`, overlay fields / reason | ✅ spec v1; shipped in R1 form in DeckBoss |
| `mark.note`, `session.meta` | explicit marks; day/session boundaries | ✅ spec v1; ⚠️ not yet emitted by any Phase 1–3 plan |
| `chat.exchange` | `role, text, model, key_owner: "byok" \| "on-device"` | ⚠️ spec v1 + architecture §5; implemented in Phase 3a |
| `fix.track`, `media.frame`/`media.clip`, `clock.beacon` | breadcrumbs; content-addressed media (`sha256`, `uri`, source device); cross-device clock skew | ✅ spec v1; 🔮 for the ActiveLog core (multi-device tier) |
| `helm.command`, `helm.event` | `profile, action, class (C0–C3), result, confirmed_by`; overrides/watchdog | ✅ spec v1; 🔮 `cocapn.ai` skin only (architecture §7 parks the hardware tier) |
| `catch.assertion` | `species, container, count?, raw, parse_conf` | ✅ spec v1; the **model** for every skin's assertion types (§1.4, §3) |
| **`connector.call`** *(reserved here)* | `connector` (ConnectorDecl id, §2), `corr` (correlation uuid), `request` (summary or ref), `capabilities` (aggregate required), `decision: "allow"\|"confirm"\|"deny"`, `confirmed_by?` | 🔮 reserved now, emitted when §5.5's dispatcher exists |
| **`connector.result`** *(reserved here)* | `corr`, `connector`, `ok`, `result` (summary or `media.*`/blob ref), `dur_ms` | 🔮 same |

Why reserve `connector.call`/`connector.result` now rather than when the
dispatcher ships: architecture §5.5's whole point is that the day-one chat
path is "structured so that 'chat provider' is one implementation of a
`Connector` interface rather than a hardcoded special case." The log-side
twin of that discipline is that `chat.exchange` is understood from day one
as the *degenerate case* of a connector call/response pair (one where the
request and result are both chat text, so the pair collapses into one
event). Reserving the general pair costs nothing (unknown types are
preserved by rule) and prevents the later phases from minting an
incompatible RPC-log shape. Note the deliberate echo of the spec's own
safety vocabulary: `helm.command.class` (C0–C3, defined in
`deckboss/docs/cocapn-foundation-mirror/SAFETY.md`: C0 info → C3
propulsion/irreversible) is the same ladder §2's lever `safety.class` uses,
so a `connector.call` recording a hardware lever invocation and the
`helm.command` it caused speak one classification.

### 1.6 Two live inconsistencies found during this pass (fix cheaply, on the record)

1. **The deployed `activelog.ai` page's envelope example contradicts the
   spec schema.** `activelog/public/index.html` line 316 shows
   `"ts": "2025-02-14T12:00:00Z"` (ISO string); `event.schema.json` requires
   `ts` to be an integer (epoch ms). The same page (line 300) says the
   envelope "already powers DeckBoss's real, shipped logbook" — which
   `activelog/docs/product-status.md` already flags, and which §0.2 makes
   precise: DeckBoss ships the *model*, not the envelope (no
   `alv`/`dev`/`seq`/`mono` anywhere in `log-entry.ts`). Both belong in the
   page's next content pass — the example corrected to an integer `ts`, the
   claim softened to the pattern/lineage statement product-status.md
   already words correctly. Neither blocks Phase 1.
2. **The extraction source lacks the required key.** Covered fully in §4;
   listed here because it is the same class of drift: a doc asserting a
   property (`(dev,seq)` keying) that the named source file does not have.

---

## 2. The room / capability manifest — the pick, the schema, and the connector-declaration resolution

Everything in this section is 🔮 with respect to *building* (ROOMS.md's
gates stand untouched; "Nothing proposed in this document exists" remains
true). What this section changes is that the *shape* is now picked rather
than open — so that Phase 1–3 code (specifically the `Connector` interface
architecture §5.5 requires the day-one chat panel to be built against) can
be written compatible with it, instead of guessing.

### 2.1 The pick: MCP `server.json` spine + pincher skill levers + PlatformIO compatibility block

`ROOM_MANIFEST_PRECEDENTS.md` did the ranking and this doc adopts its
conclusion as the decision: of the eight systems, "only **MCP** fills all
three of *invocation*, *compatibility*, and *integrity*" (its summary
table), PlatformIO is "the model for the *compatibility* axis," and the
research's one-line takeaway names the fusion outright. The synthesis here
is that fusion plus the one ingredient the external precedents can't
supply and the org already ships: **the lever record is pincher's skill
shape.** A shipped `pincher/skills/docker-ops.toml` already declares
identity (`[skill]` name/version/description/tags), a voice-first intent
surface (`[skill.intent]` canonical + variations), a permission surface
(`[skill.capabilities]` required/optional), and an executable shape with a
trust seed (`[skill.reflex]` action_template + confidence_seed) — which is
precisely the per-lever record ROOMS.md layer two describes, and it is
verified real code in a graduated repo, not precedent-by-analogy.

What is explicitly **not** adopted, with the reason on the record:

- **MCP's operated registry and namespace-auth trust layer** — violates the
  no-operated-backend constraint (the precedents doc flags this gap
  itself). The manifest's `name` is an identity *claim*, checked by the
  visiting agent's own intake (ROOMS.md §2: "The manifest is a claim. …
  The intake is the check."), never a credential.
- **MCP's runtime-only capability discovery** — MCP declares how to reach a
  running server and leaves "what can it do" to a post-launch handshake
  (the precedents doc's caveat (a)). A room's levers must be **statically
  declared in the manifest**, because the cold agent's whole workflow —
  read, intake-check, import the intent surface, *then* maybe run — happens
  before anything executes. This is the one place the room design is
  deliberately more static than its best precedent.
- **Any field that carries trust across the org boundary.** ROOMS.md's
  rule, kept verbatim: "The declaration travels; **the confidence does
  not.**" `confidence_seed` is a floor suggestion; a visiting agent clamps
  every foreign lever to its confirm tier regardless of the seed.

### 2.2 The concrete schema: `room.json` at the repo root

One JSON file (JSON over TOML for the document, because every precedent the
cold-agent research scored highly is JSON and the reading agent is the
first-class consumer; pincher's TOML skills remain the *in-repo authoring*
format pincher itself uses — a room built on pincher can generate its
`levers` from its `skills/*.toml` mechanically). This supersedes the
illustrative `compatibility.json` sketch by inclusion: every field of that
sketch appears below (ROOMS.md layer one: "a room-ready repo is a strict
superset of a reuse-ready one").

```json
{
  "room": 1,
  "name": "github.com/<owner>/<repo>",
  "version": "1.0.0",
  "description": "Cabin lights controller for ESP32 — voice-built; timing and safeties from the log.",

  "compat": {
    "frameworks": ["cocapn"],
    "platforms": ["esp32"],
    "min_agent": "0.3.0"
  },

  "run": [
    {
      "kind": "esptool",
      "artifact": "firmware/helm-light-switch.bin",
      "offset": "0x0",
      "sha256": "<64 hex>",
      "env": []
    }
  ],

  "levers": [
    {
      "id": "lights.set",
      "intent": {
        "canonical": "turn the cabin lights on",
        "variations": ["cabin lights on", "kill the cabin lights", "lights at half"]
      },
      "capabilities": { "required": ["network"], "optional": [] },
      "invoke": { "type": "http", "method": "POST", "path": "/lights", "params": { "level": "0..100" } },
      "safety": { "class": "C1", "default": "confirm" },
      "confidence_seed": 0.5
    }
  ],

  "manual": "AGENT.md",
  "repository": { "url": "https://github.com/<owner>/<repo>", "id": "<forge repo id>" }
}
```

Field rules, each traced to its precedent:

| Field | Rule | Source of the rule |
|---|---|---|
| `room` | integer schema version, the manifest's `alv` | `activelog-spec` (`alv` on every event so streams are self-describing) |
| `name` | globally unique identity **claim**; `version` pinned, **ranges rejected** | MCP `server.json` (`name` reverse-DNS, "version ranges are rejected"); trust demoted to claim per §2.1 |
| `compat.*` | **advisory by default**; the consumer decides strictness | PlatformIO (`frameworks`/`platforms` "advisory … strict only when `libCompatMode = strict`"), which the precedents doc calls "almost a one-line specification of this org's own re-verify-at-read-time stance" |
| `run[]` | how to obtain/stand up the room's runnable half: kind + artifact/identifier + args/env (secrets flagged `isSecret` when env entries exist) + `sha256` integrity | MCP `Package` (`registryType`/`identifier`/`transport`/`environmentVariables`/`fileSha256` — "clients must validate the downloaded file matches the hash before running") |
| `levers[]` | the statically-declared capability surface; **this is the ConnectorDecl record, §2.3** | pincher `skills/*.toml` (shipped shape), ROOMS.md layer two |
| `levers[].safety.class` | C0–C3, the same ladder as `helm.command.class` | `SAFETY.md` (C0 info / C1 trim / C2 mode / C3 irreversible) |
| `levers[].safety.default` | `"allow" \| "confirm" \| "deny"` — where the dispatcher starts | pincher `VetoDecision { Allow, Deny, RequireConfirmation }` via architecture §5.5 |
| `manual` | path to the agent-addressed prose manual | ROOMS.md layer three (the `AGENT.md` kernel) |
| `repository.id` | forge repo id, resurrection-attack tripwire | MCP (`repository.id` "should change" if a repo is deleted and recreated) |

### 2.3 Connector declaration vs. room manifest: **one record schema, two documents**

The question the architecture doc leaves implicit: §5.5's connector
("anything wake-word-addressable that can receive a request and optionally
return a result") and a room's lever are plausibly the same concept wearing
two names. Resolution: **they are the same record type — call it
`ConnectorDecl`, the object in `levers[]` above — living in two different
documents with different lifecycles:**

- **Local connector config** (day one): a `ConnectorDecl[]` list inside
  `AppConfig` — IndexedDB-only, never crossing a `StorageAdapter` boundary,
  because connectors carry credentials-adjacent settings (base URLs, model
  names; the keys themselves stay in the existing `chat` section per
  architecture §5). The §5 BYOK chat provider is, concretely,
  `{ id: "chat.byok", intent: { canonical: "ask the log", variations: [wake
  phrases] }, capabilities: { required: ["network"] }, invoke: { type:
  "openai-compat", baseUrl, model }, safety: { class: "C0", default:
  "allow" } }`.
- **`room.json` `levers[]`** (🔮): the same record, published, versioned in
  git, credential-free by construction (a published lever declares *shape*,
  never a key — the risk-tiering rule of §5.5 stated as a schema property).

Why one record and not two related schemas: the dispatcher is one code
path. Architecture §5.5 already mandates that the chat panel be "one
implementation of a `Connector` interface … rather than a hardcoded special
case that later has to be untangled"; two divergent declaration schemas —
one for local connectors, one for discovered levers — would be exactly that
untangling, moved into the data layer. With one record type, "import a
room's levers" (🔮) is *append vetted `ConnectorDecl`s to the same list the
local ones live in, clamped to `confirm`* — no adapter layer, no second
matcher. The differences between the two homes are lifecycle facts
(local/mutable/credentialed vs. published/versioned/credential-free), not
shape facts, and lifecycle is exactly what ROOMS.md already says must not
travel (the confidence rule). One schema, two documents, one non-negotiable
asymmetry: **a `ConnectorDecl` arriving from a manifest can never carry a
credential field or an effective confidence — both are local-only by
construction.**

The log linkage that closes the loop: every dispatch through the connector
seam eventually records `connector.call`/`connector.result` (§1.5) with the
`ConnectorDecl.id` and the veto `decision` in the body — the same
provenance-honesty rule `chat.exchange` already enforces for models
(`model` + `key_owner`, never an anonymous string), generalized to every
connector.

---

## 3. The eight-skin walkthrough — does this foundation hold without breaking changes?

`ROADMAP.md` Step 5 names the family: `activelog.ai` (the core) plus
`personallog.ai`, `businesslog.ai`, `activeledger.ai`, `fishinglog.ai`,
`cocapn.ai`/`cocapn.com`, `deckboss.ai`/`deckboss.net`, `capitaine.ai`. The
three walked in depth below are the three with the most divergent needs;
the others are covered by strictly weaker versions of the same arguments
(`personallog` is Phase 1's neutral core as-is; `fishinglog`/`deckboss.ai`
are DeckBoss's existing vocabulary re-expressed as a `catch.*`/`mark.*`
vocabulary package; `capitaine` is `fix.track` + `mark.note` + a chart
renderer, which is a *consumer* of the log, not a schema demand — its hard
part, per architecture §7, is vocabulary training, which is §3.3's
mechanism). Everything in this section is 🔮 (the skinning gate stands);
the exercise is purely "would the schema above need a breaking change," and
the standard for "no" is: no envelope field changes, no LogEntry field
changes, no R1↔R2 mapping changes — only new namespaced types, vocabulary
packages, and per-skin policy.

### 3.1 `businesslog.ai` — teams, spreadsheets, financial paper trails

- **Paper trail**: the append-only, corrections-only model *is* a paper
  trail — this is the one skin where the core invariant is the product
  feature, unmodified. ✅ by construction.
- **Tabular/spreadsheet data**: a `sheet.*` namespace, e.g. `sheet.cell`
  `{ sheet, ref: "B7", value, fmt? }` — one event per cell write, exactly
  the additive shape. Attachments (a real `.xlsx` someone imported) are the
  spec's content-addressed media (`media.clip`-style `sha256` + blob path),
  already designed. A *rendered spreadsheet* is a read-time fold over
  `sheet.cell` events — the same `applyCorrections()` move at a different
  granularity — and an export to `.csv`/`.xlsx` is a derived artifact, the
  spec's label-export pattern ("Exporters live downstream; the spec only
  guarantees the timestamps are trustworthy").
- **Teams — the honest gap, named rather than papered over**: two people
  editing cell `B7` offline produce two `sheet.cell` events; set-union
  keeps both; the fold orders by `(ts, dev, seq)` and the later one wins
  *deterministically*. That is **convergence, not intent-merging** — the
  spec excludes CRDTs on purpose, so concurrent spreadsheet editing will
  never be Google-Sheets-live under this schema, and the skin must say so.
  **No schema addition is needed** — the losing value is not lost (it is in
  the log; surfacing "B7 was also set to X by dev-…" is the corrections-UI
  pattern DeckBoss already ships, generalized), and real-time co-editing,
  if ever wanted, would be a product decision requiring an operated or
  peer-to-peer sync layer — an infrastructure question this doc is
  explicitly not allowed to propose, and honestly outside what
  "conflict-free by construction" ever promised. Verdict: **fits; one
  documented semantic limitation, zero schema changes.**

### 3.2 `activeledger.ai` — pure finance / trading

- **Event types**: a `ledger.*`/`trade.*` vocabulary — `ledger.txn`
  `{ amount, ccy, account, counterparty, memo }`, `trade.order`,
  `trade.fill` — pure `catch.assertion`-pattern typed bodies. An
  append-only, immutable, corrections-only trade log is the *natural*
  fit; nothing bends.
- **The one real gap: numeric precision.** JSON numbers are IEEE floats;
  `0.1 + 0.2` finance is not acceptable. **Smallest addition (and it is a
  vocabulary-package rule, not an envelope change): monetary/quantity
  fields in `ledger.*`/`trade.*` bodies are decimal *strings* with an
  explicit `ccy`/unit field.** The envelope never sees these fields (they
  are `body` content), so nothing about the core schema changes; the rule
  lives in the skin's versioned vocabulary package, which is where the spec
  says domain rules live.
- **Tamper-evidence**: the one skin with a plausible *real* use case for
  the spec's optional `prev` hash chain — and the envelope **already
  carries the field** (`event.schema.json`: optional `prev`, sha-256 of the
  device's previous event). Enabling it is per-skin policy, exactly the
  deferral architecture §6 banked ("do not enable the `prev` hash chain
  until a real evidentiary use case appears with a real regulator
  attached"). ⚠️ policy decision then; zero schema work now. Verdict:
  **fits; one body-level convention, zero schema changes.**

### 3.3 `cocapn.ai` — hardware control, vessel voice backbone

- **Hardware I/O**: `helm.command` (with `class` C0–C3, `result`,
  `confirmed_by`), `helm.event`, `fix.track`, `media.frame`,
  `clock.beacon` — all already ✅ *defined* in spec v1, because the spec was
  written for this skin first. The multi-device story (phone + helm unit +
  deck camera, each its own `dev`, skew measured via `clock.beacon`) is the
  spec's own design center. Nothing to add.
- **The connector dispatcher**: `connector.call`/`connector.result`
  (§1.5) carry the provenance; the local-broker/high-stakes-credential
  boundary is architecture §5.5's risk-tiering, which §2.3's "published
  levers are credential-free by construction" restates as a schema
  property. The `ConnectorDecl.safety.class` sharing `helm.command`'s
  C0–C3 ladder means the veto layer and the log speak one vocabulary from
  declaration to execution to record.
- **Vessel vocabulary** ("dropped the gear in" = "set the net"): a
  versioned vocabulary package mapping learned phrases to typed assertion
  events — the mechanism is `catch.assertion`'s two-record pattern (§1.4):
  the raw `speech.segment` is always kept verbatim (capture truth,
  training provenance — the spec's stated reason corrections preserve
  history), and the parse lands as a separate assertion event with `raw`
  and `parse_conf`. Teaching a new phrase *appends to the vocabulary
  package* (which is data, versioned, per-vessel) — it never touches
  schema. GPS-required is per-skin **policy** (the envelope's `fix` stays
  optional; a skin demanding it is a stricter producer, which breaks no
  consumer). Verdict: **fits; this is the skin the ancestor spec was
  designed around, and the only additions it needs are the two reserved
  connector types §1.5 already reserves.**

**Summary:** all three divergent skins land on the same result — new
namespaced types + vocabulary packages + per-skin policy, **zero changes to
the envelope, the LogEntry, or the R1↔R2 mapping**. The two additions this
walkthrough surfaced are already folded into §1.5 (`connector.*` reserved
types) and recorded as vocabulary-package rules (decimal-string money;
phrase-mapping packages). The one honest limitation worth repeating is
§3.1's: set-union gives deterministic convergence, not collaborative-intent
merging, and any skin whose product story implies live co-editing must
either say so or take on an infrastructure conversation that is out of this
doc's scope.

---

## 4. The live flag for the Phase 1 build — one latent conflict, correct it now (cheap)

**Finding:** as of `activelog` branch `activelog-phase1-2026-07-09` tip
`18a0458`, no schema code has landed (scaffold + placeholder `App.tsx`
only), so there is **no live conflict yet**. But there is a latent one,
sitting in the plan's own text, that the very next commit will trip over if
it follows the extraction instructions literally:

`ACTIVELOG_FIRST_SLICE.md` §7 item 1 specifies the trimmed `LogEntry`
"keyed by `dev` + `seq`," and §1 calls the `(dev, seq)` merge rule "the one
forward-compatibility guarantee Phase 1 must not skip." But the named
extraction source — `deckboss/src/core/types/log-entry.ts`, which §7's
table lists to be "copied near-verbatim" — **contains no `dev` or `seq`
field** (§0.2; its key is `id` + `timestamp`, and the only device identity
is the optional `Correction.deviceId`). A faithful near-verbatim copy
therefore ships Phase 1 *without* the one guarantee the plan says it must
not skip, and nothing in the build would fail loudly — entries would write,
persist, and read back fine on one device, and the gap would surface only
at the first second-device merge in Phase 2, as a migration.

**Recommendation: correct at extraction time — this week, in the same PR
that lands `log-entry.ts`.** Concretely (all from §1.2/§1.3):

1. Add `dev`, `seq`, and nullable `mono` to `logEntryShape`; generate `dev`
   once (local-only, same lifecycle as the existing `deviceId` pattern) and
   keep `seq` as a per-device counter in the local store.
2. Add all three to `IMMUTABLE_FIELDS` and let the existing compile-time
   coverage guard enforce it.
3. Stamp `dev` + `seq` on new `Correction`s (alongside DeckBoss's
   `deviceId`), so the R1→R2 correction-event export (§1.3, last row) needs
   no backfill.

Cost: a few dozen lines, zero migration (no user data exists on any
branch). Cost of accepting instead and adjusting later: a real migration at
the first multi-device/second-backend moment, plus a Phase 1 that quietly
violates its own plan's only non-negotiable schema requirement. This is the
easy case of the correct-now-vs-adjust-later fork: still in flight, nothing
shipped, correct now.

One scope note so this flag isn't over-read: **DeckBoss itself is not
touched.** Its `LogEntry` staying `(id, timestamp)`-keyed is fine — it is a
single-product realization whose sync already works, and re-pointing the
flagship at the extracted core is explicitly deferred
(`ACTIVELOG_FIRST_SLICE.md` D7). The `(dev, seq)` requirement binds the
*neutral core* going forward, not the flagship retroactively.

---

## 5. What this document decides, in one table

| # | Decision | Scope | Marker |
|---|---|---|---|
| S1 | One envelope (`activelog-spec` v1, unchanged), many namespaced event types — never a second envelope | all phases, all skins | ✅ adopts existing spec |
| S2 | One logical model, two wire representations (R1 markdown-per-entry, R2 JSONL), with the §1.3 field mapping as the normative, lossless bridge | Phase 1 ↔ later phases | ✅ makes an existing claim checkable |
| S3 | Phase 1 `LogEntry` = DeckBoss's shape trimmed to neutral **plus `dev`, `seq`, nullable `mono`**, corrections stamped with `dev`+`seq` | Phase 1, this week | ⚠️ action required (§4) |
| S4 | Domain structure = namespaced assertion events + versioned vocabulary packages (the `catch.assertion` pattern); never core-schema entity enums | all skins | ✅ adopts existing spec rule |
| S5 | `connector.call` / `connector.result` reserved as the dispatcher's provenance pair; `chat.exchange` understood as its degenerate case | Phase 3+ / 🔮 | 🔮 reserved, zero code now |
| S6 | Room manifest = `room.json`: MCP-spine (identity/run/integrity) + PlatformIO compat block (advisory) + pincher-skill levers; no registry, no traveling trust | rooms direction | 🔮 shape fixed, schedule unchanged |
| S7 | `ConnectorDecl` is **one record schema, two documents** (local AppConfig connector list; published `levers[]`) — published instances credential-free and confidence-clamped by construction | Phase 3 seam ↔ rooms | 🔮 with a ✅ day-one consequence (build the chat provider as a `ConnectorDecl`) |
| S8 | Skin rule: new namespaces + vocabulary packages + policy only; body-level conventions (decimal-string money) live in packages; `prev` hash chain stays off until a skin's policy turns it on | eight skins | 🔮 |
| S9 | Fix the live `activelog.ai` page drift (`ts` string example; "already powers DeckBoss" phrasing) at the next content pass | activelog repo | ⚠️ doc-truth item |

---

*Synthesis document. Subordinate to `ROADMAP.md`; the seam map is
`docs/ACTIVELOG_ARCHITECTURE.md`; the build sequence is
`docs/ACTIVELOG_FIRST_SLICE.md`; the rooms direction remains
`docs/ROOMS.md`-status (not started, no engineering time claimed). Sibling
citations resolve in `purplepincher/deckboss` (`src/...`,
`docs/cocapn-foundation-mirror/...`), `purplepincher/activelog` (branch
`activelog-phase1-2026-07-09`), and `purplepincher/pincher`
(`skills/...`). If Phase 1 lands a shape that diverges from §1.2, fix this
document or the code in the same pass — never let the two drift silently,
per `ROADMAP.md`'s own update rule.*
