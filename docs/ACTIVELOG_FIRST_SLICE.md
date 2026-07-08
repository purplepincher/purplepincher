# ActiveLog — first slice: an executable sequence

This is the **build plan** for Step 5 of `ROADMAP.md` ("ActiveLog: one shared
core, then skins"). It is not the deep technical design — a parallel
`docs/ACTIVELOG_ARCHITECTURE.md` owns the schema/envelope deep-dive. This doc
exists to turn the vision into a sequence an agent team (opencode/GLM, aider on
deepseek, mmx) can open PRs against, where **each phase is independently real
and shippable**, not one all-or-nothing build.

It is written to the bar `ROADMAP.md` Step 5 sets: every capability shown must
*actually work when a stranger visits `activelog.ai`* — no "coming soon" labels,
no greyed-out panels for unbuilt features. Anything not buildable to that bar
this phase is a later phase, full stop.

Read alongside: `ROADMAP.md` Step 5 (the brief), and `deckboss`'s
`ARCHITECTURE.md` + the `cocapn-foundation-mirror/activelog-spec/` it carries
(the canonical schema ancestor).

---

## 0. The load-bearing premise: this is an extraction, not greenfield

The single most important fact for sequencing: **deckboss already ships the
entire ActiveLog core, domain-skinned for fishing.** ActiveLog is not a new
design — it is the domain-neutral extraction of a core deckboss has proven in
the field. The first slice is "pull the core out and stand it up under a neutral
name," not "invent voice-to-markdown." Every Phase 1 component below has a
working, tested precedent in `deckboss-ref/` with a named file an implementer
can read today:

| ActiveLog Phase 1 piece | Proven precedent in `deckboss-ref/` |
|---|---|
| Mic capture (`MediaRecorder`, auto-stop, permission errors) | `src/core/audio/recorder.ts` |
| Zero-setup transcription (no key, works on first tap) | `src/services/webspeech.ts` |
| Opt-in key-based transcription (BYOK, direct to provider) | `src/services/whisper.ts` |
| Markdown wire format (frontmatter + body, human-browseable) | `src/core/tensor-log/entry-serializer.ts` + `entry-parser.ts` |
| Additive-corrections log entry (merge-safe by construction) | `src/core/types/log-entry.ts` |
| `StorageAdapter` contract (swap backends, change one line) | `src/core/storage/interface.ts` |
| The zero-auth storage backend (in-memory + zip) | `src/core/storage/adapters/local-zip.ts` |
| The S3-compatible backend (R2/Oracle, BYO credentials) | `src/core/storage/adapters/s3-compatible.ts` |
| Local-first store + offline sync (the one code path) | `src/core/storage/local-db.ts`, `src/core/sync/sync-engine.ts` |
| Local-only diagnostics counters (the field-support export) | `src/core/diagnostics.ts` |
| Local-only config that must never cross a storage boundary | `src/config/schema.ts` |

The canonical schema ancestor is **`activelog-spec`** (mirrored at
`deckboss-ref/docs/cocapn-foundation-mirror/activelog-spec/`): an append-only
event log keyed by `(dev, seq)`, merged by **set-union** — "No CRDTs, no vector
clocks, no server authority." DeckBoss's `LogEntry` is a scoped-down,
markdown-on-disk realization of that spec; its additive `corrections` and
`thread_id` already embody the merge-safe invariant. ActiveLog inherits both.

**Implication for sequencing:** because ~80% of Phase 1 is proven code, the
honest first slice is small (sized in §7). The risk is *discipline* (don't pull
in the nine-domain vision, don't touch the flagship mid-beta), not *invention*.

### 0.1 What every phase leaves out, stated up front

- **No re-skinning in parallel.** The eight domain skins (`fishinglog.ai`,
  `personallog.ai`, `businesslog.ai`, etc.) do not start until this core is real
  by ROADMAP Step 5's own "skinning gate." Not a Phase 1–3 item.
- **No PurplePincher-operated server sits between a user and the user's own
  log data.** This is the line `README.md`, `PARADIGM.md`, and `ROADMAP.md`'s
  sharing chapter all draw identically. The one place this line is genuinely
  tested is the Phase 3 free-tier chat allowance — that fork is named explicitly
  in §5, not silently crossed.
- **No placeholder UI.** A phase ships exactly the capabilities that work in
  it; unbuilt capabilities are absent from the UI, not greyed out behind a
  "coming soon."
- **DeckBoss is not destabilized.** Per `ROADMAP.md` Step 1, deckboss's field
  beta is the org's center of gravity. Re-pointing deckboss at the extracted
  core is explicitly **not** in Phase 1–3. The core is extracted *alongside*
  the flagship; the flagship consumes it later, on its own schedule.

---

## 1. Phase 1 — storage picker + mic-to-markdown (nothing else)

### What ships, exactly

A new repo `purplepincher/activelog` (does not exist yet; this doc lives in the
meta-repo as the plan) — a Vite + React + TypeScript PWA deployed to
`activelog.ai`, using the same dependency set deckboss already validates
(`zod`, `idb-keyval`, `js-yaml`; `MediaRecorder`, Web Speech API). One screen.
No chat. No wake-word. No hardware. The screen does exactly three things:

1. **Pick a storage location.** A single backend ships first: the **File System
   Access API** (`window.showDirectoryPicker()`). The user picks a folder on
   their own machine; ActiveLog writes real files into it.
2. **Mic-to-markdown.** Tap to record; `MediaRecorder` captures audio while the
   Web Speech API transcribes live; tap to stop; one structured markdown file is
   written to the picked folder and to the local store.
3. **Read back.** The screen lists entries written so far (from the local
   store), each linking to its on-disk file. Reload the page; the entries
   persist.

**On-disk shape** (one file per entry, human-browseable in the picked folder,
matching deckboss's proven layout):

```
<chosen-folder>/ActiveLog/
  2026/07/08/<uuid>.md        ← frontmatter + transcript body
  .activelog/manifest.json    ← file index + hashes
```

Each entry's YAML frontmatter carries the full machine record; the body
duplicates the transcript text for readability when opened in a text editor —
exactly `entry-serializer.ts`'s shape. The entry is keyed by a stable `dev`
(device id, generated once, local-only) + monotonic `seq`, so the
**`activelog-spec` merge rule (set-union on `(dev, seq)`) holds from day one**,
even though only one device and one backend exist yet. This is the one
forward-compatibility guarantee Phase 1 must not skip: a second backend/device
in Phase 2 must merge without a migration.

**Transcription default: Web Speech API**, not a local model. Inherited
deliberately from deckboss (`config/schema.ts`'s
`defaultAppConfig().transcription.engine = "webspeech"`, with the rationale in
`ARCHITECTURE.md` "Deliberate divergences" §2): requiring a key before a
stranger's first recording is too high a friction bar, and Web Speech works the
moment the mic permission is granted. The honest caveat — stated in the UI, not
hidden — is the one deckboss's `webspeech.ts` documents: most browsers' Web
Speech is **network-backed, not on-device**, so "no signal" and "no transcript"
are the same failure. The local/offline STT upgrade is a Phase 2+ open question
(§4), not a Phase 1 blocker.

### What "actually works" means — the falsifiable test

A skeptic who has never seen this org, on a clean Chrome/Edge desktop profile,
can do all of the following with no account anywhere and no key from anyone:

1. Open `activelog.ai`; grant the mic permission.
2. Tap **Pick folder**, choose a folder; tap **Record**, speak a sentence, tap
   **Stop**.
3. Open the chosen folder in their OS file browser and observe a real file at
   `ActiveLog/2026/07/08/<id>.md` whose YAML frontmatter contains the
   transcribed text and whose body shows it human-readably.
4. Reload `activelog.ai`; the entry is still listed (read from local store),
   and a second recording appends a second file.
5. Open the manifest at `ActiveLog/.activelog/manifest.json` and find both
   entries indexed with hashes.

If any of those steps silently no-ops or shows a placeholder, Phase 1 has not
shipped — same bar as adoption-bar item 1 in `ROADMAP.md` ("not 'has tests'").

### Why File System Access API is the first backend (the named decision)

The brief asks the first backend to be stated and justified. The call: **File
System Access API**, because it is the only candidate that satisfies all four of
Phase 1's real constraints at once:

- **No credentials, no account, no operated endpoint.** A stranger can use it
  on first visit. This is the literal Phase 1 bar. Every other candidate
  (Cloudflare R2, Google Drive, a DB, Oracle) requires the stranger to
  provision something and enter secrets first.
- **It is the example `ROADMAP.md` Step 5 itself names** for "actually
  persist[s] somewhere real (a local folder via the File System Access
  API…)."
- **It is genuinely the fastest to build *honestly*.** The surrounding pipeline
  (recorder, transcriber, serializer, manifest, entry-builder) is proven in
  deckboss. The *only* net-new adapter is the FS Access one — roughly one
  screen of UI plus a ~100-line adapter implementing the same `StorageAdapter`
  contract as `local-zip.ts`. Nothing else in the path is new.
- **The output is a real, human-browseable folder of markdown files** — exactly
  the "structured markdown log" the vision asks for, on disk, owned by the user.

**Honest caveat that ships with it:** the File System Access API is not
available in Safari or Firefox (Chrome/Edge desktop and recent Android Chrome
only). Phase 1 handles this honestly, not silently: where `showDirectoryPicker`
is absent, ActiveLog falls back to an **IndexedDB capture store** (always
available) with **zip export/import** for persistence — the exact shape of
deckboss's `local-zip.ts` — and the UI says plainly "your browser can't write to
a folder you choose; entries are saved locally and you can export a zip." No
fake folder picker.

**The second backend (fast-follow, not Phase 1):** Cloudflare R2 via the
existing `s3-compatible.ts` pattern. It is the natural second backend because
the org already operates Workers (`plato-semantic-search`, `vessel-tuner`) and
deckboss already syncs to R2 — but it requires the stranger to provision R2 and
enter keys, so it is not the *first-visit* backend. It lands once the core is
extracted and the picker offers a second choice.

### Ready to start immediately?

**Yes.** No open technical question blocks Phase 1. The transcription choice
(Web Speech) and the backend choice (FS Access API) are both settled above and
both have working precedent. The only design work is *extraction discipline*:
lift the four core modules (recorder, webspeech, serializer/parser, the
`StorageAdapter` + one adapter) into `purplepincher/activelog` without pulling
deckboss's fishing-specific layer (entities like `species`/`gear`/`depth`,
GPS-as-required, the whisper opt-in) into the neutral core. GPS, in particular,
is **optional** in the neutral core (a personallog user is not on a boat) — it
attaches when present, `null` otherwise, exactly as `LogEntrySchema` already
allows.

---

## 2. Phase 2 — pause/wake-word-driven organization

The vision: organize a continuous dictation into a *structured* log via pause
and wake-word detection — i.e. turn one long mic stream into discrete, typed
entries/sessions, not one unbounded blob. This phase bundles two capabilities of
**very different readiness**, so it is split here so neither stalls the other:

### Phase 2a — pause/silence-driven segmentation (ships; ready now)

- Tap once to start a *session*; ActiveLog keeps the mic open and segments on
  silence (a configurable `autoStopSilenceMs`, which deckboss's `recorder.ts`
  already implements) into discrete entries, each its own markdown file sharing
  a `thread_id` (the session). A long pause ends an entry; the next speech
  starts a new one. This is the "structured markdown log" becoming *structured*,
  and it is trivially real — the segmentation primitive already exists.
- Adds entry `tags`/`mode` (dictation vs. command) to frontmatter, surfacing
  the `activelog-spec` `speech.segment { mode }` distinction without requiring
  the full envelope yet.

**Falsifiable test:** start a session, speak three sentences with 2-second gaps
between each; observe three separate `.md` files sharing one `thread_id`, in
order; speak nothing for the silence window and confirm the session closes
cleanly.

### Phase 2b — always-listening wake-word (real open fork — decision required first)

This is the genuinely unsettled part, and the honest finding from the precedent
review: **`pincher` is a text-intent reflex engine, not an audio wake-word
detector.** It has no microphone, keyword-spotting, or transcription code — it
matches *text* intents against learned reflexes and fires known actions locally
without an LLM call (`pincher-ref/ARCHITECTURE.md`). So pincher is the natural
*consumer* once a wake signal has fired (transcribed phrase → local reflex, no
cloud round-trip — exactly the vision's "wake-word-triggered local actions"), but
**it does not produce that signal.** The browser has no built-in keyword
spotting. The open question, named explicitly:

> **Which wake-word mechanism ships, given that every option trades off against
> the org's offline-first thesis?**

Candidates, with the real trade-off each carries:

1. **Web Speech API continuous mode + transcript text-pattern match** (e.g. fire
   when the phrase "hey log" appears in the live transcript). Cheapest; reuses
   the Phase 1 transcriber. But Web Speech is network-backed in most browsers
   (§1), so the wake-word *itself* stops working offline — directly undermining
   the "wake-word-triggered local actions without a cloud round-trip" premise
   this feature exists for. Honest only if labeled "online wake-word."
2. **A local keyword-spotting model in WASM** (e.g. a transformers.js tiny
   model, vosk-wasm, or a porcupine-style TFLite). Works offline, true to the
   thesis. Real cost: a model download (tens of MB), non-trivial integration,
   CPU/battery on a phone, and picking a stack that is honestly maintainable.
3. **Defer always-listening; ship explicit activation only** (push-to-talk /
   tap-to-wake). Zero new tech; the wake "word" is a physical gesture. Lowest
   honesty risk, weakest "magic."

**Decision needed before Phase 2b starts:** pick one — and the deciding
criterion is whether "works offline" is a *Phase-2 requirement* or a
*Phase-3+ requirement*. The recommendation, on record: **ship 2a + option 3
(explicit activation) as Phase 2's shippable slice**, and treat option 2 (local
WASM keyword-spotting) as its own scoped spike *only after* a stranger has
confirmed the pause-segmented core is actually useful. Option 1 (online
wake-word) is rejected for v1 because it silently breaks the property the
feature is named for. This recommendation is itself a fork a reviewer should
confirm, not silently execute.

**Phase 2's "actually works" test (2a):** a stranger dictates a multi-topic
session and gets discrete, ordered, tagged markdown files — not one blob — with
no manual "new entry" action beyond pausing.

### Ready to start immediately?

- **Phase 2a: yes.**
- **Phase 2b: no** — blocked on the wake-word-mechanism decision above.

---

## 3. Phase 3 — the BYOK chat panel

A chat panel that talks *to the user's own log*, using a model the user
chooses and keys they supply. The `activelog-spec` already defines the record:
`chat.exchange { role, text, model, key_owner: "byok" }`.

### Phase 3a — BYOK-only chat (ships; ready now)

- A provider picker: **Ollama (localhost), DeepSeek, OpenAI, OpenRouter,
  DeepInfra.** These are all OpenAI-compatible `/chat/completions` endpoints, so
  one client parameterized by `{ baseURL, apiKey, model }` covers all five — the
  same shape as deckboss's `whisper.ts` (direct browser→provider, no
  PurplePincher server in between).
- The key is stored in **local-only config that never crosses a storage
  boundary** — exactly the invariant `deckboss`'s `config/schema.ts` enforces
  (the one schema "that must NEVER be written through a StorageAdapter").
- The panel injects the user's recent log entries as context (read from the
  local store), so "ask your log" works against entries the user just dictated.
- Chat turns are persisted as additive entries (`chat.exchange`) in the same
  merge-safe log, keyed by `(dev, seq)`.

**Falsifiable test:** a stranger enters their own OpenAI (or DeepSeek, or local
Ollama) key, asks "what did I log today?", and gets a grounded answer citing
entries dictated in Phase 1/2; the key appears nowhere in any file under
`ActiveLog/`; clearing the key disables the panel cleanly.

### Phase 3b — the free-tier chat allowance (the one decision that tests the org's core line)

The vision includes "a small free allowance on a cheap model" so a stranger can
try the chat without a key. This is the single decision in the whole plan that
genuinely risks the line every strategy doc in this org draws: **no
PurplePincher-operated server sits between a user and the user's own data.** The
open question, named explicitly:

> **Can the free allowance be delivered without operating a server at all — and
> if a server is unavoidable, is it a server the org's own rule permits?**

The honest analysis, because hand-waving here is exactly the failure mode
`ROADMAP.md`'s "how this roadmap gets updated" section exists to prevent:

- **Pure client-side call with a PP-funded key embedded in the bundle** is not
  real — the key ships in the JS and is exfiltrated in minutes. Rejected; do not
  ship this even as a "quick v1."
- **A Cloudflare Worker proxy** that holds the PP-funded key, rate-limits per
  client (by IP or an anonymous client id), and relays to the cheap provider.
  This *is* an operated endpoint. The distinction that decides whether the org's
  rule permits it: **the Worker relays chat tokens to a model provider; it never
  touches, stores, or proxies the user's log data.** It sits between the user
  and *the model*, not between the user and *their log*. The log never routes
  through it unless the user explicitly sends a snippet as a chat prompt (which
  is the user's own action, identical to BYOK).
- This is a *different line* than the one the fleet-learning rejection
  (`ROADMAP.md` "The sharing chapter") drew — fleet learning was rejected
  because it required a DeckBoss-operated server *that aggregates users' log
  data*. A token-relay proxy aggregates no log data. But because the two look
  superficially similar ("PurplePincher runs a server"), **this decision must be
  made on the record and confirmed by a reviewer, not slid in as an
  implementation detail.**

**Decision needed before Phase 3b starts:** (i) confirm the token-relay Worker
is permitted under the org's line (the argument above), or (ii) decide the free
allowance is dropped entirely and Phase 3 ships BYOK-only (the strictly safer
call, and a perfectly real Phase 3 on its own). If (i) is confirmed, the Worker
is small (the org already operates `plato-semantic-search` and `vessel-tuner`)
and the rate-limit is the only real design work.

**Recommendation on record:** Phase 3 ships **3a (BYOK-only) as the real
slice**; 3b (free allowance) proceeds only after the proxy decision is made
explicitly. Do not let 3b gate 3a — BYOK-only is a complete, honest chat panel.

### Ready to start immediately?

- **Phase 3a: yes.**
- **Phase 3b: no** — blocked on the proxy-permissibility decision above.

---

## 4. The schema fork — what Phase 1 decides vs. what the architecture doc owns

To keep this plan executable without duplicating the parallel
`docs/ACTIVELOG_ARCHITECTURE.md`, the split is:

- **Phase 1 commits to** (and these are enough to ship): markdown-with-YAML-
  frontmatter, one file per entry, human-browseable; a `manifest.json` index
  with hashes; the additive-corrections invariant; and the **`(dev, seq)`
  append-only merge rule** (set-union, sorted by timestamp). DeckBoss already
  proves all of this. Phase 1 must not ship without the `(dev, seq)` keying —
  it is cheap and it is what makes a second backend/device merge without a
  migration.
- **The architecture doc owns** the deeper `activelog-spec` envelope (JSONL
  `speech.segment` / `chat.exchange` typed bodies, the `mono` monotonic clock,
  optional per-device hash chains for tamper-evidence, the full media-anchoring
  model). Phase 1's markdown format and the envelope are *compatible by
  construction* — a later move to JSONL event streams is an additive
  representation change that preserves the same merge rule, not a breaking
  rewrite. Phase 1 does not need the envelope to be real.

The one cross-cutting Phase-1 invariant, restated because it is load-bearing:
**history is immutable; corrections are additive events; "delete" sets
`retracted`, "edit" appends a `Correction`.** This is what makes offline sync
trivially safe (two devices' corrections union, they never conflict —
`deckboss-ref/src/core/sync/conflict-resolver.ts`) and it is non-negotiable from
day one. Inheriting deckboss's `log-entry.ts` + `invariants.ts` as-is preserves
it for free.

---

## 5. Open decisions register (consolidated)

Every fork in this plan, in one place, so nothing is hand-waved:

| # | Decision | Blocks | Status / recommendation |
|---|---|---|---|
| D1 | First storage backend | Phase 1 | **Decided:** File System Access API (+ IndexedDB/zip fallback). §1. |
| D2 | First transcription engine | Phase 1 | **Decided:** Web Speech API (network-backed caveat shown in UI). §1. |
| D3 | Schema format for Phase 1 | Phase 1 | **Decided:** markdown-frontmatter + manifest + `(dev,seq)` merge rule. Envelope design deferred to architecture doc. §4. |
| D4 | Wake-word mechanism | Phase 2b | **Open.** Recommend: ship 2a + explicit-activation; spike local WASM keyword-spotting later; reject online wake-word for v1. §2. |
| D5 | Local/offline STT upgrade | Phase 2+ | **Open, non-blocking.** transformers.js whisper-tiny vs. whisper.cpp WASM vs. vosk-wasm. Not required to ship honestly. §1, §2. |
| D6 | Free-tier chat allowance: backend or not | Phase 3b | **Open.** Token-relay Worker is *probably* permitted (relays model tokens, never log data) but must be confirmed on the record. Recommend: ship Phase 3 BYOK-only; do not let 3b gate 3a. §3. |
| D7 | Re-point deckboss at the extracted core | post-Phase-3 | **Deferred deliberately** — do not destabilize the flagship during its field beta (`ROADMAP.md` Step 1). §0.1. |

---

## 6. Explicitly future — not planned here, per ROADMAP.md Step 5's own text

Called out only so the doc doesn't read as if they're forgotten. Each is named
in Step 5 as "a later phase, not a 'coming soon' label on day one":

- **I/O panel for hardware** (ESP32 autopilot, camera feeds) — the ESP32
  chapter. `cocapn-foundation`'s `SAFETY.md` five-layer model and
  device-profiles-as-data pattern are the design ancestors (mirrored in
  `deckboss-ref/docs/cocapn-foundation-mirror/`); `plato-engine-block-c` is the
  org's existing embedded-C proof.
- **ESP32 voice-to-firmware** (describe → generate → upload over USB) — stated
  direction in `README.md` and `PARADIGM.md`, not code today.
- **`capitaine.ai` chart-drawing skin** with vessel-specific vocabulary
  training — a skin; gated behind the skinning gate, which opens only after the
  core is real.
- **The full nine-domain skinning pass** — gated behind the same skinning gate.

None of these get engineering time in Phase 1–3.

---

## 7. What ships in Phase 1, literally, this week

The smallest real thing that can go live on `activelog.ai` and pass the
"stranger can actually use it" bar. Sized for a first PR, not a multi-week arc.

**The PR creates `purplepincher/activelog`** (new repo), bootstrapped from the
same Vite/React/TS/zod stack deckboss uses, containing exactly:

1. `src/core/types/log-entry.ts` — a trimmed `LogEntry` (entities reduced to a
   neutral `tag` set; GPS optional/nullable; additive `corrections` and
   `thread_id` kept verbatim), keyed by `dev` + `seq`.
2. `src/core/storage/interface.ts` — the `StorageAdapter` contract, copied
   near-verbatim from deckboss (the one proven seam).
3. `src/core/storage/adapters/file-system-access.ts` — **the one net-new
   adapter** (`showDirectoryPicker` → `writeFile`/`readFile`/`listFiles` against
   a real `FileSystemDirectoryHandle`, plus manifest read/write). ~100 lines.
4. `src/core/storage/adapters/indexed-db.ts` — the always-available capture
   store + fallback (deckboss's `local-db.ts` shape), so Safari/Firefox aren't
   silently broken.
5. `src/core/storage/adapters/local-zip.ts` — zip export/import for the
   fallback case (deckboss's, near-verbatim).
6. `src/services/webspeech.ts` — copied verbatim (default transcriber).
7. `src/core/audio/recorder.ts` — copied verbatim (`MediaRecorder` wrapper,
   auto-stop, permission errors).
8. `src/core/tensor-log/{entry-builder,entry-serializer,entry-parser}.ts` —
   copied, markdown wire format.
9. `src/core/diagnostics.ts` — copied (local-only counters; the field-support
   path, day one).
10. **One screen** (`App.tsx`): pick folder → record → stop → see the written
    entry → reload → still there. No chat. No wake-word. No hardware. No
    placeholder UI for any of those.
11. A README whose every claim is literally true and a one-line deploy to
    `activelog.ai` (Cloudflare Pages, the org's existing pattern).

**The acceptance check a reviewer runs** (this is the gate, not a checklist):
on a clean Chrome desktop profile, no account anywhere, open the deployed URL,
pick a folder, dictate, and find a real markdown file on disk with the
transcript — and reload to confirm it persisted. If that works, Phase 1 is real.
If any step is a placeholder, it isn't, and the PR doesn't merge.

**What is explicitly *not* in this PR:** chat, wake-word, a second storage
backend, GPS-as-required, the fishing entity set, any skin, any re-pointing of
deckboss. Each of those is a later phase per §§2–6.

---

*This is the executable sequence for Step 5. The deep schema/envelope design
lives in `docs/ACTIVELOG_ARCHITECTURE.md`; the decision record for the step
lives in `ROADMAP.md`. If a phase's premise turns out wrong on contact with the
build, fix this document rather than working around it silently — same rule
`ROADMAP.md`'s "how this roadmap gets updated" section applies to itself.*
