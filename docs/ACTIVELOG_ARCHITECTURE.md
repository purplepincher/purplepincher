# ActiveLog core — technical architecture

**Status:** architecture for the `activelog.ai` reference implementation. Drafted
2026-07-08, on branch `activelog-architecture-2026-07-08`, for independent
review before merge to `main`. This is the technical counterpart to the brief
in [`ROADMAP.md`](../ROADMAP.md) §"Step 5 — ActiveLog: one shared core, then
skins." A parallel effort is drafting `docs/ACTIVELOG_FIRST_SLICE.md` (the
first buildable slice); this document is the architecture, not the slice plan.

*Addendum (2026-07-08, branch `activelog-architecture-addendum-2026-07-08`):
the sections above are reviewed and unchanged. This branch adds four
subsections that fold in design conversations held after the original draft —
§4's "Chrome's on-device Prompt API" accelerator; §5's "Gemini — a sixth
OpenAI-compatible endpoint" and "The on-device escape hatch" for the
free-allowance tension; and a new §5.5 "The connector-dispatcher paradigm"
that generalizes the §5 BYOK provider into a named-connector primitive. All
new claims carry the same ✅/⚠️/🔮 honesty markers as the rest of the doc.*

**The one-line scope, restated so this can't drift into nine shallow skins:**
build ONE real reference implementation of a voice-capture-and-storage core on
`activelog.ai`, real enough that a stranger who arrives with no setup can tap a
button, dictate into a markdown log, and have it persist somewhere they
actually own — and real enough that every capability shown on the page does
exactly what it says. The eight domain skins do not start until that bar is met
(see `ROADMAP.md`'s "Skinning gate").

This document does not design from scratch where a working sibling already
solved the same problem. Two siblings are load-bearing throughout:

- **DeckBoss** (`../deckboss-ref/`, the shipped flagship) — already solved
  offline-first sync to user-owned storage, an additive audit-log, diagnostics
  export, and made a real, documented voice-engine decision. Almost every
  seam below reuses a DeckBoss pattern on purpose, and says where and why.
- **pincher** (`../pincher-ref/`, the namesake reflex engine) — already solved
  "fire a known local action in <50ms with no LLM call, confirm the
  semi-known, compile the novel via an LLM only on a miss." That is exactly
  the shape "a wake-word triggers a local action without a cloud round-trip"
  wants, once ported from a Rust CLI to a browser.

## How to read the honesty markers

Three markers recur, each meaning a specific thing, none of them optional:

- **✅ Real today / buildable to the bar.** Verifiable now, in shipping
  siblings or in today's browsers, with no research breakthrough required.
- **⚠️ Real but conditional.** Works, but only on some browsers, only with a
  user setup step, or only with a stated tradeoff the UI must surface honestly.
- **🔮 Later phase, not day one.** Genuinely buildable eventually, but either
  needs a real model/ classifier in the browser or an org-level decision that
  shouldn't be made silently. Never labeled "coming soon" on the shipped page.

This is the `fishinglog-agent` guardrail made structural: that package's PyPI
copy claimed "hot-spot detection" and shipped no such algorithm — a thin CRUD
client requiring a live server (`ROADMAP.md`, "What we will not fork"). Every
capability below is marked for which bucket it is actually in.

---

## 1. The core seam: storage is a `StorageAdapter`, and the app never knows which one

### What DeckBoss already solved (and we reuse)

DeckBoss defined the exact contract this core needs, and proved it works across
four real backends with one sync engine. The contract lives in
`deckboss-ref/src/core/storage/interface.ts`:

```ts
export interface StorageAdapter {
  readonly id: StorageBackendId;
  isAuthenticated(): Promise<boolean>;
  authenticate(): Promise<void>;
  logout(): Promise<void>;
  readFile(path: string): Promise<string>;
  writeFile(path: string, content: string): Promise<void>;
  deleteFile(path: string): Promise<void>;
  listFiles(prefix: string): Promise<FileMetadata[]>;   // recursive
  readBlob(path: string): Promise<Blob>;                 // audio
  writeBlob(path: string, blob: Blob): Promise<void>;
  getManifest(): Promise<Manifest>;
  writeManifest(manifest: Manifest): Promise<void>;
}
```

Three properties of that contract are the entire reason offline-first is free
here, and we inherit them verbatim:

1. **`sync-engine.ts` is written against `StorageAdapter` only — it never
   imports a concrete adapter.** Swapping backends means swapping which
   adapter `registry.ts` hands the sync engine. (`deckboss-ref/ARCHITECTURE.md`,
   "Why this shape.") The generic ActiveLog core gets this for nothing.
2. **A local-only `AppConfig` is the one schema that never crosses a
   `StorageAdapter` boundary.** Every other schema is meant to land in the
   user's cloud storage in human-readable form; `AppConfig` holds credentials
   and stays in IndexedDB. (`deckboss-ref/src/config/schema.ts` header comment.)
   BYOK depends on this invariant, and it is cheap to keep cheap.
3. **Markdown is a wire format at the edges, not the in-memory
   representation.** `entry-serializer.ts`/`entry-parser.ts` are the only code
   that knows markdown; everything else works with typed `LogEntry` objects
   (`deckboss-ref/ARCHITECTURE.md`). Casey's vision — "dictation software into a
   markdown editor" — is satisfied by making the *on-disk* format markdown
   without making the *in-app* model a raw string.

**Decision: adopt DeckBoss's `StorageAdapter` contract, the path layout, the
manifest, and the additive-corrections `LogEntry` schema as the ActiveLog
core's storage layer, unchanged in shape.** The reason is not laziness — it is
that this contract has already survived the hardest test in the org: a
multi-method QA pass that found and fixed a manifest-destruction bug that
silently erased the only index into a user's archive, an audio-rehydration
gap, and a stale-manifest orphan-scan gap, all verified by an adversarial
restore drill (`deckboss-ref/ROADMAP.md`, "The restore drill"). Reusing it
inherits that hardening. Redesigning it would re-open bugs already closed.

### What diverges, and why

DeckBoss writes under a `STORAGE_ROOT = "DeckBoss"` folder. ActiveLog writes
under a configurable root (default `ActiveLog/`) so nine eventual skins each
get a distinct, human-browseable folder (`FishingLog/`, `BusinessLog/`, …)
without colliding. This is a one-line constant plus a config field, not a
structural change — the contract, the path helpers, and the manifest are all
parameterized by that root already (`interface.ts`:
`entryPath()`/`audioPath()`/`STORAGE_ROOT`).

The bigger, deliberate divergence is the **storage-picker UX on first
arrival**, which the vision makes central ("simply start by designating a
storage location"). DeckBoss defers storage setup; ActiveLog must lead with
it. The picker is described concretely in §2 because it is where the hardest
browser-reality tradeoff lives.

### The five backends, honestly

DeckBoss ships four. ActiveLog adds one (the local folder) and inherits the
other four as-is:

| Backend | Real today? | Source of truth | What the page must say |
|---|---|---|---|
| **Local folder** (File System Access API) | ⚠️ Chromium desktop only | new `LocalFolderAdapter` | "Chrome/Edge only; on Safari/Firefox your folder choice falls back to an in-browser store or a ZIP download." |
| **Google Drive** (OAuth) | ✅ | `deckboss-ref/.../google-drive.ts` | Reuse verbatim. Testing-mode 100-user cap applies (see §2). |
| **Cloudflare R2** (S3-compatible) | ✅ | `deckboss-ref/.../s3-compatible.ts` | Reuse verbatim; user supplies their own R2 token. |
| **Oracle Object Storage** | ✅ | same S3-compatible class | Reuse verbatim; subclass fixes `id` and region default. |
| **Local ZIP export** | ✅ | `deckboss-ref/.../local-zip.ts` | Reuse verbatim; the universal fallback. |

Two of those reuse claims deserve to be made explicit, because they are the
ones a reviewer should be able to check against the sibling code:

- **R2 and Oracle are the same file.** `s3-compatible.ts` is an abstract class
  both backends subclass; the only differences are `id`, display name, and a
  region default (`region: config.region ?? "auto"`, "R2 uses 'auto'; Oracle
  wants a real region string"). Both go **directly browser-to-bucket** over
  CORS-enabled S3 endpoints with `@aws-sdk/client-s3`, no server component
  (`s3-compatible.ts` class doc). ActiveLog adds nothing here.
- **Drive is client-side-only OAuth.** `google-drive.ts` uses Google Identity
  Services' token client with the `drive.file` scope — "the least-privileged
  option: the app can only see files it created, never the user's whole Drive"
  (`google-drive.ts` header comment). No PurplePincher server is in the
  token-exchange path. ActiveLog adds nothing here either, except a different
  OAuth client ID configured under its own origin.

### The one carry-over gotcha worth bank

`GoogleDriveAdapter.listFiles()` historically listed only direct children of a
folder, not descendants — a real bug that broke DeckBoss's cold-start orphan
scan. It is now a real recursive walk (`walkFolderTree()`), and that fix is
load-bearing for any new adapter that wants "point a fresh device at storage
and recover everything" to be true. **Any new `StorageAdapter` ActiveLog ships
must implement `listFiles()` as genuinely recursive**, or it inherits the exact
class of bug the sibling already paid to fix (`deckboss-ref/ROADMAP.md`,
"manifest-fallback-scan"; `google-drive.ts` `listFiles` doc).

---

## 2. Storage abstraction: what "works for everyone" actually means

The vision says: a person "can also click save later, and just have the system
fully functional as a loaded website." That is the key to the honesty claim —
**the core must work with zero storage setup, and storage is always an
upgrade, never a precondition.** DeckBoss already ships this property
(Local-ZIP works the instant the page loads). ActiveLog must keep it as the
floor.

### The File System Access API is Chromium-only — say so on the page

The "designate a local folder" path uses
`window.showDirectoryPicker()` + `FileSystemDirectoryHandle` +
`FileSystemWritableFileStream`. As of 2026-07-08, MDN marks this whole family
**"Limited availability"** and **"Experimental"** — explicitly "not Baseline
because it does not work in some of the most widely-used browsers," requires a
secure context (HTTPS), and requires transient user activation (a real click)
on every grant. In practice: **Chromium desktop only** — Chrome, Edge, Opera.
Not Safari (desktop or iOS), not Firefox.

That single fact drives the storage design, so it is worth being precise about
what it does and does not rule out:

- **`showDirectoryPicker()` / `showSaveFilePicker()` / `showOpenFilePicker()`
  and `FileSystemWritableFileStream`** — the device-facing, user-visible-folder
  API — ⚠️ Chromium desktop only. This is what "pick a local folder" needs.
- **Origin Private File System (OPFS)** — `navigator.storage.getDirectory()`
  and `FileSystemSyncAccessHandle` in a Web Worker — ✅ broadly supported and
  Baseline, including Safari and Firefox. **But it is not user-visible.** OPFS
  is an origin-private sandbox the user cannot browse with a file manager. It
  is excellent for "save later, fully functional website" but it is *not* "a
  local folder."

This maps cleanly onto Casey's two phrases. "Click save later, fully
functional" → OPFS (works everywhere). "Designate a local folder" → File
System Access API (Chromium only). The page must offer both and never conflate
them. Concretely, the storage picker renders:

```
Where should your log live?
  [ ] In this browser, for now        ← OPFS, works on every browser; not a file you can see
  [ ] A folder on this computer       ← File System Access API; Chrome/Edge only; real .md files
  [ ] My Google Drive                 ← OAuth, drive.file scope
  [ ] My Cloudflare R2 / Oracle       ← S3-compatible, your credentials
  [ ] Just give me a .zip sometimes   ← Local ZIP, the universal fallback
```

The `LocalFolderAdapter` (new) implements the same `StorageAdapter` contract
against a stored `FileSystemDirectoryHandle`, serializing the handle into
IndexedDB so it survives reloads (handles are
`structuredClone`-able/`IDB`-storable per the MDN File System API guide). On a
browser without `showDirectoryPicker`, the "folder on this computer" option is
simply absent and the OPFS option is the highlighted default. **No
capability is faked.** A Safari user never sees a "pick a folder" button that
then no-ops.

### Drive/R2/Oracle linking: realistic to build vs. stretch

All three are ✅ **realistic and already built** in the sibling — this is not
OAuth-design from scratch. What "realistic" means, concretely:

- **Drive** uses Google Identity Services' token client with PKCE-equivalent
  flow and `drive.file` scope; the OAuth client ID is a `VITE_GOOGLE_CLIENT_ID`
  build-time env the operator supplies from their own Cloud Console
  (`deckboss-ref/README.md`, "Storage setup (BYOK)"). The one carry-over
  landmine is documented and bounded: **a Drive app in Google "Testing" mode
  caps at 100 users before Google requires verification**
  (`deckboss-ref/ROADMAP.md`, "Known landmine"). Fine for a reference build;
  a real gating item before any skin expects public traffic.
- **R2/Oracle** are pure S3-compatible: the user pastes an endpoint, bucket,
  access key, secret key into a real password-masked form field (the sibling
  moved these out of unmasked `prompt()` dialogs after a security review —
  `deckboss-ref/ROADMAP.md`, "Iteration-1 multi-method beta test"). The
  `connect-src` CSP is deliberately unrestricted because BYOK needs arbitrary
  S3-compatible endpoints (same source).

**Stretch, not day one:** Casey's "database service or cloud instance like
Oracle" and "the user can put the system in their Cloudflare account easily."
The S3 path already covers Oracle Object Storage (a real S3-compatible
endpoint) and R2. "Deploy ActiveLog into the user's own Cloudflare account" is
a real, valuable capability — a static PWA on the user's CF Pages plus an
optional free-token-proxy Worker (see §5) — but the one-click deploy story
("instructed on how to put the system in their cloudflare account easily") is
a documentation/onboarding project, not a missing capability. 🔮 Phase 2
onboarding track, not a day-one architecture item.

### Why this reuses rather than diverges

The stated reason, on the record: **storage-BYOK is the harder, more important
problem this org has already solved once, and solving it twice is exactly the
"pile of repos" failure mode Step 5 exists to prevent** (`ROADMAP.md` §Step 5,
"What 'real' means"). DeckBoss's storage layer is also the single most
battle-tested code in the org (three named data-loss bugs found and fixed in
the sync layer alone). The only honest reason to diverge would be a
capability DeckBoss's contract cannot express — and so far the only candidate
(local folder) is expressible as just another `StorageAdapter`. So we don't
diverge; we add one adapter and parameterize the root.

---

## 3. Voice-capture-to-markdown pipeline

### What DeckBoss decided, and why it transfers to a generic core

DeckBoss made one real, documented voice-engine call that ActiveLog should
inherit wholesale:

> **Web Speech API is the default transcription engine, Whisper is opt-in.**
> The default requires an OpenAI key before a first recording works; flipping
> it removes that friction. (`deckboss-ref/ARCHITECTURE.md` divergence #2;
> `deckboss-ref/src/config/schema.ts` `defaultAppConfig`.)

The reasoning transfers to ActiveLog even more strongly, because ActiveLog is
explicitly the "fully functional the moment they arrive" product. A stranger
should not need an OpenAI account to test the page. So: **Web Speech default,
Whisper opt-in**, identical to the sibling.

But Web Speech has a property DeckBoss discovered the hard way and that the
generic core must inherit the *fix* for, not just the default:

> **Most browsers' Web Speech implementation is network-backed, not on-device.**
> Chrome ships audio to a cloud recognition service. MDN confirms this on the
> `SpeechRecognition` page: "On some browsers, like Chrome, using Speech
> Recognition on a web page involves a server-based recognition engine. Your
> audio is sent to a web service for recognition processing, so it won't work
> offline."

DeckBoss hit this as a live bug: `webspeech.ts`'s error handler treated a
`"network"` failure identically to benign silence, so a user recording with
zero signal got a saved entry that *looked* like a confident empty transcript.
The fix — `WebSpeechTranscriber.hadNetworkError` plus `useRecording` leaving
`transcript` unset on network failure — shipped, with honest UI copy "No
transcript — audio saved" (`deckboss-ref/ROADMAP.md`, "Pre-launch hardening
pass"; `deckboss-ref/src/services/webspeech.ts`).

**ActiveLog inherits `webspeech.ts` and its `hadNetworkError` handling
unchanged.** This is not a nice-to-have; it is the difference between an honest
"no signal, no transcript" and a silent lie. MDN also notes `SpeechRecognition`
itself is "Limited availability" (not Baseline) — it is absent on Firefox, and
where present the network-backed behavior dominates. So:

- **The capture path** (`AudioRecorder` wrapping `MediaRecorder`,
  `deckboss-ref/src/core/audio/recorder.ts`) is engine-agnostic and reusable
  as-is. `MediaRecorder` is supported on Safari since 2021 (the sibling
  verified this against the docs after a fragile WebKit-sandbox false alarm —
  `deckboss-ref/ROADMAP.md`, "Iteration-1"). Audio + timestamp are always
  saved; only the transcript depends on the engine.
- **The transcription path** is a swappable `TranscriptResult`-producing
  function. Two implementations are ✅ real today and inherited:
  - `transcribeWithWebSpeech` (default, free, network-backed on Chrome).
  - `transcribeWithWhisper` (opt-in, BYOK OpenAI key, direct
    `https://api.openai.com/v1/audio/transcriptions`, with
    `WhisperNetworkError` so an offline attempt queues a retry rather than
    failing permanently — `deckboss-ref/src/services/whisper.ts`).

### The offline/local-model honesty tradeoff

Casey's vision implies the page should work fully offline ("download the
entire page with their data to run the whole thing locally"). The capture,
storage, and markdown-serialize paths are genuinely offline (they're the same
local-first code DeckBoss already ships). The transcription path is the one
that is *not* honestly offline today, and the architecture must say which
sub-part is real vs. stretch:

- **Audio capture + markdown write: ✅ fully offline.** `MediaRecorder` →
  `IndexedDB` → `StorageAdapter`. No network needed.
- **Web Speech transcription: ⚠️ online on Chrome, on-device on iOS Safari.**
  MDN documents a new `SpeechRecognition.processLocally` property and
  `SpeechRecognition.available()`/`install()` static methods for on-device
  recognition, plus an `on-device-speech-recognition` Permissions-Policy
  directive — the on-device path is arriving but is not yet a reliable
  cross-browser default. Treat network-backed as the Chrome reality and design
  for "no transcript, audio saved" as the offline outcome (DeckBoss's pattern).
- **A real offline transcription model in the browser: 🔮 later phase.** This
  is buildable — `whisper.cpp` compiled to WASM, or Vosk-WASM, or a
  Transformers.js speech model, all run in-browser today — but it is a real
  artifact with a real cost: a multi-hundred-megabyte model download and
  non-trivial WASM CPU. Shipping it as a *default* would betray the "works the
  moment you arrive" promise. The honest architecture is: **BYOK Whisper for
  online quality now; an opt-in "download the offline model" toggle as a
  Phase 2 feature** that, once downloaded, makes transcription genuinely
  offline. Never claim offline transcription on the landing page until that
  toggle exists and a user has turned it on.

This is the same honesty discipline the org applied to the `fishinglog-agent`
"hot-spot detection" claim: the capability either exists or it's a later
phase, never a "coming soon" banner.

---

## 4. Pause- and wake-word-driven organization

The vision: "backend algorithms to make the organization of the markdown
better just through hearing pauses and wake-up words," and "wake-up words for
different agents and models." This section separates what is implementable
today from what needs a real classifier, because the two are routinely
conflated and that conflation is how "AI organization" becomes a lie.

### Tier A — pause/silence-gap detection (no model, ✅ real today)

Segmenting a continuous mic stream into utterances by detecting silence gaps
needs **no classifier** — it is a solved signal-processing problem with
Baseline browser APIs:

- **Web Audio API `AnalyserNode`** (Baseline, all browsers) on the
  `MediaStream` from `getUserMedia`. Compute per-frame RMS (root-mean-square)
  amplitude; a frame below a threshold for a sustained window (DeckBoss's
  default is 3000ms — `recording.config.autoStopSilenceMs`) marks a silence
  gap; a frame above marks speech onset. This is the same machinery DeckBoss's
  recorder already exposes via the `onDataAvailable` timeslice for a live
  waveform (`recorder.ts`).
- **Web Speech API boundary events** as a second, cheaper signal. MDN lists
  `soundstart`, `soundend`, `speechstart`, `speechend` events on
  `SpeechRecognition` itself — i.e. the recognizer already tells you when
  speech and sound start/stop. These are a free utterance-boundary signal
  wherever Web Speech works, with no extra model.

**What Tier A buys, honestly:** automatic paragraph breaks in the markdown
(one utterance ≈ one paragraph or list item), automatic "end of thought"
segmentation, and the raw timing a higher tier can later build on. **What it
does not buy:** understanding *what* was said, classifying intent, or
recognizing a specific wake-word. Calling silence-gap detection "AI
organization" would be exactly the overclaim this org hardens against. It's
signal processing, and the doc should call it that.

### Tier B — wake-word as exact phrase match in the transcript (no embedding model, ✅ real)

The cheapest "wake-word triggers a local action" is not a keyword-spotting
model at all — it is **literal string/regex match against the live transcript**.
The transcript is already arriving from Web Speech (Tier A's recognizer);
checking whether a segment starts with "hey log" or "deck" is a constant-time
local check with zero ML. This is the browser port of pincher's **Exact** tier.

pincher's matcher (`pincher-ref/pincher-core/src/reflex/matcher.rs`) classifies
a match into three buckets with thresholds calibrated for
`all-MiniLM-L6-v2`:

- **Exact** — cosine similarity ≥ 0.80 → fire immediately, no confirmation.
- **Similar** — 0.55–0.80 → fire but flag as uncertain; surface for review.
- **Novel** — < 0.55 → do nothing; hand back to a smarter layer (an LLM, or a
  human) to decide.

The Exact tier, done as exact-string matching, needs no embedding model and
runs in microseconds. **Tier B in ActiveLog = the Exact tier: a configurable
list of wake-phrases ("hey activelog", "ask deepseek", "note this") each bound
to a local action (start a new entry, open the chat panel, drop a
timestamped mark).** When the transcript contains the phrase, the action fires
with no cloud round-trip. This is genuinely real and genuinely local.

### Tier C — real semantic wake/intent matching (needs a model, 🔮 later phase)

pincher's Similar and Novel tiers need an **embedding model** in the browser —
its Rust runtime uses `all-MiniLM-L6-v2` via ONNX Runtime with a SHA-256
trigram hash fallback when the model is absent
(`pincher-ref/ARCHITECTURE.md`, "Embedding"; `pincher-ref/pincher-core/src/reflex/matcher.rs`).
The browser equivalent is **Transformers.js** (`@huggingface/transformers`,
formerly `@xenova/transformers`), which runs the same `all-MiniLM-L6-v2` model
in-browser via ONNX Runtime Web / WASM, and a small in-browser vector store
(e.g. `sqlite-wasm` or `localStorage`-backed cosine, since there is no
`sqlite-vec` extension in the browser the way there is in pincher's Rust
runtime).

This is buildable, but it carries the same honesty cost as offline
transcription: a ~22MB+ model download and WASM compute. **Tier C is therefore
a Phase 2 capability** ("teach the log your own wake-phrases, matched by
meaning not exact wording"), not a day-one claim. The architecture leaves room
for it by routing every transcript through a single `matchWakePhrase` seam
that starts as exact-match (Tier B) and can later host an embedder (Tier C)
behind the same interface — mirroring pincher's own seam design where the
hash-fallback and the ONNX path sit behind one `Embedder` trait.

### Chrome's on-device Prompt API — an optional accelerator for Tier C (and only when present)

Tier C's embedder is a 🔮 later-phase item because of its model-download/WASM
cost (above). There is a second path to a *local* semantic classifier that is
free in a different sense — free of network and free of cost to the org — but
only on the one browser that ships it: **Chrome's on-device Prompt API (Gemini
Nano)**, reached through Chrome's built-in AI runtime. ⚠️ **Real but
conditional:** as of 2026-07-08 this API is Chromium-only (Chrome/Edge desktop),
origin-trial / flag-gated, requires the browser to have downloaded the on-device
model once, and is explicitly *not* present in Safari, Firefox, or mobile Chrome
in the same form. **Verify the current enablement surface** (origin-trial token
vs. `chrome://flags` vs. general availability) and the exact JS entry point
against Chrome's live built-in-AI docs before wiring — the gating mechanism has
shifted across Chrome versions and must not be assumed from memory. (The
org's own research already encountered this surface in
`docs/research/cocapn-family-deep-dive.md`, "Chrome's built-in Gemini Nano
Prompt API with cloud fallbacks.")

Where it **is** present, it is a strictly better Tier C implementation option
than a downloaded WASM embedder for one job: **wake/intent classification** —
"is this utterance addressed to the log, and to which connector" — decided by a
model that runs on-device, with zero network round-trip and zero cost to the
org. This does **not** replace Tier A/B and does not make Tier C mandatory: the
architecture is *prompt-capable-when-present, exact-match-when-not*.
`matchWakePhrase` (the single seam §4 already defines) gains a third branch —
"if an on-device language model is available, ask it to classify intent;
otherwise fall back to Tier B exact-match (Tier A segmentation runs regardless)"
— behind the same interface, so a Safari or Firefox user never sees a broken
feature, only the Tier B behavior that was always the day-one floor. This is the
same seam discipline pincher uses, where the hash-fallback covers the no-model
case and the ONNX path is an upgrade behind one `Embedder` trait, never a
precondition. It is also the local-classifier input to the connector-routing
decision in §5.5, which is why it appears here rather than only under chat.

**The caveat that matters most, stated so a reader cannot conflate it:**
"Chrome AI" is not one uniformly-local thing. The Prompt API above runs
**on-device**. Chrome's **default `SpeechRecognition`** — which is §3's Web
Speech default transcription engine — does **not**: it ships audio to a
Google server-side recognizer (§3 quotes MDN on this directly, and DeckBoss
shipped `hadNetworkError` precisely because that network path fails). These two
Chrome-shipped AI surfaces have **opposite privacy properties**, and the page
must never let "Chrome has on-device AI" imply that the default voice-capture
path is private by default. Concretely: routing wake-classification through the
on-device Prompt API keeps that classification local; routing *transcription*
through Web Speech does not, on Chrome, until a user opts into a genuinely
on-device recognition path (§3's `processLocally` / `on-device-speech-recognition`
direction, still not a reliable cross-browser default). The two must be
presented as separate choices with separate privacy footprints, never as one
"Chrome AI" toggle.

### What is explicitly not claimed

- **True always-on keyword spotting (KWS)** — the kind that listens
  continuously on a low-power edge device and fires on "hey X" without ever
  transcribing — is **not realistic in a browser at <50ms today**. pincher's
  <50ms Exact claim is for a Rust CLI against a local SQLite, not a browser
  tab that must keep a `MediaRecorder`/`AnalyserNode` running. ActiveLog's
  "wake-word" is, honestly, "wake-phrase detected in the live transcript
  during an active session," not "always-listening hands-free KWS." The page
  copy must say the former, not imply the latter.

---

## 5. BYOK chat panel

The vision: "add a panel for a chatbot with a wake-up word," "the user can
add any api including local ollama, cloud services like deepseek or openai or
even openrouter and deepinfra," and "we could give human visitors a limited
amount of tokens through a cheap model like deepseek flash."

### Key storage — client-side only, never through a server we operate

The storage rule is DeckBoss's, applied to a new credential type:

> **Nothing about storage credentials or API keys ever gets written to a file
> that syncs to the user's cloud storage. `AppConfig` lives in IndexedDB
> only.** (`deckboss-ref/README.md`, "Design principles"; `config/schema.ts`
> header.)

ActiveLog's `AppConfig` gains a `chat` section — `provider`, `baseUrl`,
`apiKey`, `model`, `wakePhrase` — that lives in IndexedDB and **never** crosses
a `StorageAdapter` boundary. The key leaves the browser in exactly one place:
the direct `fetch()` to the provider's API endpoint, the same way
`transcribeWithWhisper` sends the OpenAI key only to `api.openai.com`
(`whisper.ts`: "no PurplePincher server sits in between"). There is no
PurplePincher proxy in the BYOK path. This is the same property, restated for a
second credential type, and it is what makes "BYOK" a real claim rather than a
label.

### Provider routing — one OpenAI-compatible shape, six endpoints

Six of the named providers expose an OpenAI-compatible `/v1/chat/completions`
shape with a `Bearer` key, so a single client covers them:

| Provider | Base URL | Notes |
|---|---|---|
| OpenAI | `https://api.openai.com/v1` | reference impl |
| DeepSeek | `https://api.deepseek.com` (or `/anthropic`) | `deepseek-v4-flash` |
| OpenRouter | `https://openrouter.ai/api/v1` | aggregates many models behind one key |
| DeepInfra | `https://api.deepinfra.com/v1/openai` | OpenAI-compatible |
| Gemini | `https://generativelanguage.googleapis.com/v1beta/openai` | Google's OpenAI-compatible shim; ⚠️ see below |
| Ollama (local) | `http://localhost:11434/v1` | see ⚠️ below |

**⚠️ Local Ollama has a real setup gotcha the UI must surface.** An
HTTPS-deployed `activelog.ai` calling `http://localhost:11434` is subject to
Chromium's **Private Network Access** preflight requirements, and Ollama's own
CORS allowlist (`OLLAMA_ORIGINS`). The user must start Ollama with, e.g.,
`OLLAMA_ORIGINS=https://activelog.ai ollama serve` for the browser to reach it.
This is a one-line setup step, not a missing capability — but the settings
panel must say it plainly, and the "test connection" button must report the
real PNA/CORS error rather than a generic "failed." (This is the same class of
honest-error-handling discipline DeckBoss applied to `WhisperNetworkError` vs.
`WhisperApiError` — distinguish "couldn't reach it" from "it rejected us.")

### Gemini — a sixth OpenAI-compatible endpoint, not a sixth client shape

The design call on Gemini is to **route it through Google's own
OpenAI-compatible endpoint** (table row above), so it stays a sixth row in the
same single client rather than introducing a sixth client shape. The reason is
the same reason the other five collapse to one client: Google documents its
`/v1beta/openai/` shim to accept the OpenAI `chat/completions` request body and
`Bearer` key, so the existing BYOK `fetch()` path (`baseUrl` + `apiKey` + `model`
from `AppConfig`) covers Gemini with zero new code — only a new row in the
provider picker. This is strictly cheaper than a bespoke Gemini client and keeps
the "one shape, many endpoints" property true — which is what lets §8's
acceptance item 3 ("actually calls the key a user enters, against at least one
of {…}") extend to Gemini via the same machinery, rather than needing a second
test path. ⚠️ **Verify before wiring:** the exact base path
(`/v1beta/openai`), the current model id (e.g. `gemini-2.x-flash`-class), and
whether the OpenAI shim enforces the same `Bearer` auth as the native API all
evolve on Google's side and were not re-checkable against Google's live docs
from the environment this entry was written in — confirm them against
`ai.google.dev/gemini-api/docs/openai` at wire-up time, and have the "test
connection" button report the real `401`/`404` rather than a generic "failed"
(the same discipline as the Ollama note above).

**One real, checkable property that is architecturally different from every
other row in the table — verify, then decide.** Gemini's **native** API (the
`generateContent` shape, *not* the OpenAI shim) accepts **audio as a first-class
input modality** (`inlineData`/File-API audio parts): a single call can take raw
audio and do transcription *and* understanding together. Every other provider in
the table is text-in/text-out and therefore requires the §3 two-step
(STT → LLM) for voice→chat. If that property holds for the model a user selects,
Gemini is the one provider where §3's pipeline could take an **alternative
single-call path** — send the recorded `Blob` straight to Gemini with a prompt,
skip the separate Web Speech/Whisper transcription hop — **as an opt-in,
Gemini-only path, never as a replacement for the Web Speech default.** Two
honesty constraints on this, because the implication is large:

- It is reachable only through Gemini's **native multimodal API**, which is a
  *different* client shape from the OpenAI-compat shim the table row uses — so
  it is a second, provider-specific code path, not free with the row above.
  Whether the OpenAI shim passes audio input parts through identically should be
  confirmed, not assumed; if it does not, the single-call audio path needs the
  native client.
- It does **not** change §3's day-one default. Web Speech remains the
  zero-setup transcription engine; the Gemini single-call audio path is a
  quality/latency upgrade a BYOK-with-Gemini user can opt into, exactly as
  Whisper is an opt-in quality upgrade over Web Speech. The §3 honesty rule
  ("never claim offline transcription on the landing page until the user turns
  it on") applies unchanged: this is network-to-Google, not local.

🔮 **Phase 2**, not day one — both because it is provider-specific and because
the single-call path needs its own provenance handling in the log (the
`chat.exchange` event's `model` field would name a Gemini audio-capable model,
and the user should see that their audio went to Google, not to a local
transcriber).

### The free-allowance problem — the one real tension with the org's principles

This is the hardest call in the doc and it must be made explicitly, not
silently. **A "limited free tokens on a cheap model" feature requires an
org-operated endpoint that holds an API key and relays chat turns.** That is,
by definition, a backend this org operates — and the org has a documented
rejection of exactly that shape:

> **"Fleet learning: off the roadmap."** "It cannot be built without a
> DeckBoss-operated backend somewhere, which breaks the core local-first
> promise the shipped product already depends on for trust."
> (`deckboss-ref/ROADMAP.md`, Decided.)

The load-bearing word in that rejection is the same word that matters here:
*fleet learning* required an operated backend because it **aggregated users'
data.** A free-token proxy is categorically different in one axis and
identical in another:

- **Different:** a chat proxy is **stateless and log-blind.** It forwards the
  chat turns the user explicitly sends to the bot and an API response. It
  never sees the user's log entries, their storage credentials, or anything
  the dictation path captured. It touches no user data the product depends on
  for trust.
- **Identical:** it is still an org-operated endpoint, and it still carries a
  **cost liability** — an uncapped public proxy on a popular page is a
  wallet-drain vector.

So the rejection does not automatically forbid a free allowance, but it does
demand the allowance be built to constraints the original rejection would
recognize. The recommended architecture, in priority order:

1. **Day one: BYOK only, no org-operated allowance.** Every chat turn goes to
   a provider key the visitor themselves pasted. This keeps the no-operated-
   backend property literally true on launch and meets the "actually works, not
   staged" bar with zero cost liability. The panel shows "bring your own key"
   as the only path.
2. **Phase 2 (optional, gated): a free allowance only if all three hold —**
   (a) it runs as a **Cloudflare Worker the user can also deploy to their own
   account** (the BYO-Cloudflare pattern the vision explicitly names: "put the
   system in their cloudflare account easily," and the same pattern Step 3a
   adopts — `ROADMAP.md` §Step 3a); (b) the org-operated default Worker is
   **hard-capped per client** (per-IP and per-stored-device-id token quotas,
   enforced at the Worker, returning 429 past the cap), with billing alarms;
   (c) the Worker is **provably log-blind** — it has no binding to any store of
   user log data, and that's auditable in its (tiny) source. Under those three
   constraints the allowance is defensible; without any of them it recreates
   the operated-backend trust problem the org already rejected once.

**Quantifying the cost so the cap is real, not hand-wavy:** DeepSeek's pricing
page lists `deepseek-v4-flash` at **$0.14 / 1M input tokens (cache miss),
$0.28 / 1M output tokens** (cache-hit input $0.0028/1M). A casual visitor
doing 10 short chats a day at ~2k input + 1k output tokens each is roughly
$0.006/day, ~$0.18/month — trivial per user, but at 10,000 visitors that's
~$1,800/month if uncapped. The per-client cap is not paranoia; it is the
difference between a demo and a bill. (The model id `deepseek-chat` is being
deprecated 2026-07-24 in favor of `deepseek-v4-flash`, per the same page —
cite the live name, not the legacy one.)

### The on-device escape hatch — a zero-cost default, for Chrome users only

There is a third option that dissolves the free-allowance tension *for one
browser* without touching either of the two above, and it deserves to be named
so it isn't conflated with them. Chrome's on-device Prompt API (Gemini Nano —
the same surface §4 routes Tier C wake-classification through, ⚠️ Chromium-only
and conditional as detailed there) can, **when present**, also serve as a
zero-cost default chat backend: the model runs in the user's browser, so the
turn costs the org literally nothing, and there is no org-operated endpoint in
the path at all. This is categorically separate from both the BYOK-provider
question (no user key is involved — the model is browser-provided) and the
hosted-allowance question (no proxy, no cap, no bill, no `fleet-learning`-shaped
operated backend to defend). For a Chrome-desktop visitor who has the model
downloaded, "just talk to the log, no setup" can be true on day one without any
of the §5 constraints firing.

This is a ⚠️ **conditional accelerator, not a strategy**: it is absent on
Safari, Firefox, and mobile Chrome, so it cannot be the *only* free path, and
the free-allowance decision above still has to be made for everyone else. It is
also **provenance-honest by construction** (not a marker, a requirement): a
chat turn answered by the on-device model must still record a `chat.exchange`
event with `model` set to the on-device model id and `key_owner` set to
something like `"on-device"` (not `"byok"` and not `"org"`) — the log must not
imply an org-operated model served the turn when the user's own machine did. And the §4 caveat applies here unchanged and bears
repeating in this context: **the on-device Prompt API is local; Chrome's default
`SpeechRecognition` is not.** "Talk to the log with no setup" must not be
allowed to imply that the *dictation* path is private by default on Chrome —
the chat-backend can be on-device while the voice-capture still leaves the
device, and the UI has to say which is which (§3, §4). The two are independent
toggles, never one "Chrome AI" switch.

🔮 **Phase 2 for the chat-backend wiring** (Tier C's model-presence check is the
prerequisite, and §5.5's connector abstraction is the natural place for an
on-device model to register as a zero-cost connector); day one remains BYOK-only
per item 1 above, with this noted as the honest reason a Chrome user's
"free chat" need not wait for the hosted allowance to be built.

### Chat provenance in the log — reuse the spec's event

Every chat turn the user exchanges should be recorded as an `activelog-spec`
`chat.exchange` event — `body: { role, text, model, key_owner: "byok" }` — so
the log is honest about which model said what and that the user's own key paid
for it (`deckboss-ref/docs/cocapn-foundation-mirror/activelog-spec/README.md`,
event-type table). This is a one-field honesty invariant: a chat reply in the
log always carries its `model` and `key_owner`, never an anonymous string.

---

## 5.5 The connector-dispatcher paradigm — generalizing the BYOK provider into a named-connector primitive

The BYOK chat providers in §5 are, on reflection, the simplest case of a
broader pattern Casey wants the core to embody: a voice-first surface that can
address *anything reachable* by name and get a result back, with the same
interaction shape regardless of what is on the other end. His words, verbatim,
on the target experience:

> "I could say, 'cocapn, put together a series of prompts for claude code on
> my oracle instance, you have the ssh to the oracle instance in your
> connectors. I want to see an overview of your prompts before you send them'
> or 'cocapn, tell my laptop's hermes to begin research on tomorrow's project
> scope.' ... we become the ship's computer ... the audio frontend that can
> project a window with a dashboard of anything from markdowns of transcripts
> to controls created for controlling intuitively for whatever the application
> of the room they are in."

And his own scoping refinement, which is the load-bearing constraint for the
security model below:

> "the web-based version doesn't have to have the higher-risk connectors like
> an ssh to an oracle. but the paradigm that a log can reach out to anything
> that will connect to and treat it all the same from a voice-first UI/UX."

This subsection writes that up as an architectural pattern, not a feature list,
and marks every piece for which phase it is actually real in. **The entire
generalized paradigm is 🔮 later-phase for the reference core** — day one ships
only the one connector instantiation that is already ✅ real: the §5 BYOK chat
provider. The pattern is named here so the day-one chat panel is built *as* the
first connector rather than as a special case that later has to be untangled.

### A general "named connector" abstraction

Generalize §5's provider into a primitive:

> **A connector is anything wake-word-addressable that can receive a request
> and optionally return a result.** A chat model (§5), a remote agent over SSH,
> a hardware bridge, a research task queued on another machine — all are
> connectors. The voice-first interaction is identical regardless of what is on
> the other end: a wake/intent classifier (§4 Tier B exact-match today, Tier C
> embedder / on-device Prompt API later) decides *which* connector an utterance
> is addressed to, the dispatcher hands it a request, and any result is rendered
> back into the same surface that shows transcripts and chat.

The §5 BYOK chat provider is `connector("chat", invoke=openaiCompatFetch,
returns=text)` — the simplest shape (one request in, one text blob out, no side
effects outside the conversation). Adding "tell my laptop's hermes to begin
research" is `connector("hermes", invoke=sshOrLocalBrokerDispatch,
returns=taskHandle)` under the *same* dispatch seam, not a parallel feature
tree. This is why §4's `matchWakePhrase` was specified to route to "which
connector" rather than only "is this a chat turn": the intent classifier is the
front door to the dispatcher, and the chat panel is just the first room behind
it. Concretely, the day-one code path (`wakePhrase → chat provider fetch →
render reply`) is structured so that "chat provider" is one implementation of a
`Connector` interface (`invoke(request): Promise<result>`) rather than a
hardcoded special case — a one-interface refactor now, free, that keeps the
later generalization from being a rewrite.

### Confirm-before-execute is the default, not a feature

Any connector invocation that reaches **outside the browser tab** must show the
user what it is about to do before it happens. This is not a nicety; it is the
property that makes "a log that can reach out to anything" safe enough to ship,
and this org has already built the mechanism once. Two real precedents:

- **`pincher`'s veto/sandbox layer** is the exact allow/deny/confirm shape a
  remote connector dispatcher needs.
  `pincher-ref/pincher-core/src/security/veto.rs` defines a three-way
  `VetoDecision { Allow, Deny(reason), RequireConfirmation(reason) }`, a
  pluggable `VetoPolicy` trait behind a thin `VetoEngine::check()` dispatcher,
  and a default `RuleBasedVetoPolicy` whose `RequireCapability` rule turns any
  action needing an ungranted capability into a `RequireConfirmation`. That is
  precisely the "show me before you do it" gate: a connector call classified as
  safe-and-local returns `Allow`; a forbidden target returns `Deny`; everything
  that crosses the tab boundary returns `RequireConfirmation` until the user
  approves. The companion `pincher-ref/pincher-core/src/security/sandbox.rs`
  adds capability-scoped execution (`Capability { Network, FilesystemRead,
  FilesystemWrite, Subprocess, Gpu }`, `CapabilityManifest`, and a fail-closed
  landlock/bwrap path that refuses to run unsandboxed if sandboxing cannot be
  applied). This generalizes pincher's own known-vs-novel matcher distinction
  (`reflex/matcher.rs`: Exact ≥ 0.80 fires, Similar 0.55–0.80 flags, Novel <
  0.55 hands off — §4) from *local reflexes* to *remote connectors*: the
  equivalent of "Exact" is a connector call the user has approved before (or
  that stays in-tab); the equivalent of "Similar/Novel" is anything reaching a
  new target, which defaults to `RequireConfirmation`.
- **`git-native-agents`' named-agent message-passing** is the addressing
  precedent: a sender addresses a named recipient by writing a message into the
  recipient's workspace (`send(from, to, msg)` drops an `inbox/<id>` file and
  commits it in the recipient's repo — `docs/research/git-native-agents-deep-review.md`
  §1). The *shape* — "address a named thing, hand it a request, let it produce a
  result" — is exactly the connector-dispatch shape, and it is the model for how
  one connector (e.g. the log) addresses another named agent (e.g. "hermes" on a
  laptop). Honesty demands repeating what that same deep review found: the
  upstream implementation has real concurrency bugs (git index-lock contention
  under concurrent writes, reproduced at 92% failure on 12 concurrent sends) and
  zero tests, so it is a *pattern* precedent, not a drop-in dependency — fork it
  as a hardening task, not an import.

The dispatcher this core ships therefore reuses pincher's three-way decision
verbatim: **`Allow` for in-tab/no-side-effect calls (the §5 chat provider today);
`RequireConfirmation` as the default for anything that leaves the tab; `Deny`
for targets the deployment's connector allowlist does not contain.** Casey's "I
want to see an overview of your prompts before you send them" is not a feature
he asked for — it is the `RequireConfirmation` branch doing its job, and it is
on by default, not opt-in.

### Risk-tiering by deployment — the real security boundary, not a restriction

The SSH-credential problem ("the web version has the ssh to the oracle in its
connectors") is resolved by where the software runs, not by a policy the user
could talk their way past. State it as the actual boundary:

- **The hosted `activelog.ai` web build ships a safe connector set only** — the
  §5 BYOK chat providers (and, later, simple API-reachable services that take a
  user-supplied key over the same direct-`fetch`, no-org-proxy path). **No raw
  SSH, no direct hardware bus access, no shell.** These connectors cannot leave
  the tab in a dangerous sense because the tab has nothing dangerous to reach:
  a `fetch()` to an API endpoint is the most it does.
- **The high-stakes connector set — SSH sessions to remote boxes, hardware
  bridges, driving another machine's agent — is available only on the "download
  and run locally" deployment** that §3 already names as Casey's offline vision
  ("download the entire page with their data to run the whole thing locally").
  There, the software runs on hardware the user fully controls, and the
  high-stakes credentials (SSH keys, hardware-bus access) are held by a **local
  broker/agent process** on that machine — never entered into the browser tab,
  never transmitted through any server this org operates. The browser UI talks
  to the local broker over `localhost` (the same shape, and the same
  `OLLAMA_ORIGINS`-class setup step, as the §5 local-Ollama connector), and the
  broker is what actually holds the key and opens the SSH session after the
  dispatcher's `RequireConfirmation` fires.

The boundary, stated so it cannot be read as a soft policy: **SSH private keys
and equivalent high-stakes credentials must never be entered into the hosted web
version at all — by design, not by request.** The hosted build has no field for
them, no connector type that consumes them, and no code path that could exfiltrate
one, because the credential never reaches the tab. (`ROADMAP.md` §Step 5 frames
the deploy model as the user's own Cloudflare account / linked storage; the
local-broker path is the §3 offline property extended from "storage and
transcription run locally" to "connectors run locally," and it is the only
deployment shape in which the dangerous connector set is wired up.)

### Where the heaviest version of this actually lives

The "ship's computer" instantiation — SSH to remote dev boxes, addressing other
named agents, driving hardware, projecting a dashboard of controls for "the
application of the room they are in" — is the natural specialization of the
**`cocapn.ai` skin**, which `ROADMAP.md` §Step 5 already frames as the
hardware/control-oriented skin ("`fishinglog.ai`/`cocapn.ai`/`cocapn.com`
(hardware control, vessel voice backbone)"). The split is deliberate and load-
bearing:

- **The generic core (this doc, `activelog.ai`) ships the *pattern*** — the
  `Connector` interface, the §4 intent-classifier front door, the
  confirm-before-execute dispatcher, the risk-tiering rule that keeps the hosted
  build to the safe set. It ships exactly one real connector (BYOK chat) and the
  shape for more.
- **`cocapn.ai` ships the *dangerous instantiation*** — the SSH/hardware/named-
  agent connector set, the local broker that holds those credentials, the
  dashboard projection Casey describes. It is a later skin, gated identically to
  the other seven by the Step 5 "Skinning gate" (no skins until the core is real
  by §8's bar), and it is where §7's currently-parked hardware items (ESP32
  code-gen/upload, autopilot I/O panels) land when they land — this subsection
  does *not* un-park them, it names the skin and the primitive they would be
  built on top of when the gate opens.

This keeps the reference core honest (it claims only the chat connector, which
is ✅ real) while making the eventual `cocapn.ai` path a matter of instantiating
a primitive that already exists, rather than a fresh architecture that has to be
retrofitted onto a core that never expected it.

---

## 6. The additive log model — inherited, not redesigned

ActiveLog's on-disk record is DeckBoss's `LogEntry` with its additive
`corrections` array, for the same reason DeckBoss chose it — and the reason is
*not* the one a compliance pitch would give:

> **"No editing. Ever. Corrections are events, history is never rewritten."**
> (`activelog-spec/README.md`, "What's deliberately NOT in the spec"; quoted
> verbatim in `deckboss-ref/src/core/types/log-entry.ts`.)

The justification that earns its keep is **sync safety, not compliance**. The
sibling's own words (`log-entry.ts` doc comment, `deckboss-ref/ARCHITECTURE.md`
divergence #1): because edits are additive, two devices editing the same entry
never conflict, only merge — their two `Correction` objects union, always.
`last-write-wins` only applies to *which files exist at all* (a genuine
two-device create race on the same id, which UUIDs make vanishingly rare). The
merge rule is the one `activelog-spec` states plainly: *"a log set is merged by
set-union on `(dev, seq)`, sorted by `(ts, dev, seq)`. Append-only + unique
keys = conflict-free by construction. No CRDTs, no vector clocks, no server
authority."*

**ActiveLog inherits this whole**, including the invariant enforcer pattern
(`deckboss-ref/src/core/invariants.ts` — the write-path validator that proved
its value by being found incomplete once, missing `transcript`/`entities`/`tags`
in `IMMUTABLE_FIELDS`, and then fixed with a compile-time guard so the drift
can't recur — `deckboss-ref/ROADMAP.md`, "schema-pattern debate"). The generic
core is in some ways *easier* than DeckBoss here, because it has no GPS/depth/
species entities to protect; it starts with `transcript`, `tags`, and the new
`chat.exchange` payloads as the only mutable-via-correction fields.

One deliberate divergence worth naming: `activelog-spec` carries a per-device
**monotonic `seq` and an optional `prev` hash chain** for tamper-evidence
(`event.schema.json`: `prev`, pattern `^[a-f0-9]{64}`). DeckBoss banks the
cheap parts but doesn't use the hash chain, explicitly because it makes *no*
evidentiary claim (`deckboss-ref/ROADMAP.md`, "Regulatory/compliance stance:
explicitly disclaimed"). ActiveLog should make the same call: **store the
`(dev, seq)` keys and the `mono` monotonic clock because they're free and make
merge deterministic; do not enable the `prev` hash chain until a real
evidentiary use case appears with a real regulator attached.** Banking the
cheap parts and deferring the compliance-shaped parts is the discipline the
sibling already validated.

---

## 7. Explicitly out of scope for the reference core

One line each, with a reason, so the shipped page never implies these:

- **ESP32 code-generation and upload.** 🔮 This is the `cocapn-foundation`
  "Helm unit" tier — real design exists
  (`deckboss-ref/docs/cocapn-foundation-mirror/SAFETY.md`, the five-layer
  defense model), but it is voice-commanded *actuation* with a hardware
  safety surface, explicitly out of DeckBoss's scope and out of ActiveLog's
  day-one scope. A voice log that can *steer a boat* is a different product
  with a different liability profile; it earns its way in only after the log
  itself is real (`ROADMAP.md` §Step 5, "later phase").
- **Autopilot and video-feed I/O panels.** 🔮 Same root reason: an I/O panel
  for an ESP32 autopilot is the actuation tier, not the log tier. A passive
  *view* of an external video feed is a softer case, but even "show a camera
  feed in a panel" implies device-permission, WebRTC, and a hardware partner
  that don't exist. Defer until a skin (e.g. `cocapn.ai`) has a concrete
  hardware target.
- **Chart-drawing and the `capitaine.ai` vessel-vocabulary training.** 🔮
  "Draw a line on a chart from a timestamped, GPS-tagged voice note" needs a
  chart renderer, a geo-format, and — the genuinely hard part — *per-vessel
  vocabulary training* ("dropped the gear in" = "set the net," for a
  gillnetter). That last piece is a real ML/teaching feature, not a UI task.
  It belongs to a skin that has real fishermen producing the training data,
  which is gated on Step 1's field beta, not on this architecture.

These are parked, not killed — but parking them on the record is what keeps
the reference build from quietly becoming a nine-skin sprawl.

---

## 8. What "real" means for the reference core (acceptance, not announcement)

Mirroring Step 5's bar (`ROADMAP.md` §Step 5, "What 'real' means") and Step 1's
checkable-gate discipline, the core is "real" only when all of the following
are verifiably true, each of which falsifies a specific overclaim:

1. **A stranger with no setup can arrive, tap record, dictate, and a markdown
   file is written somewhere real** — OPFS at minimum, a linked Drive/R2/folder
   if they chose one. (Falsifies: "it's just a demo recorder that prints to
   the console.")
2. **The storage they picked actually persists across reload and across a
   fresh-device restore** — the same restore drill DeckBoss ran against
   `LocalZipAdapter` (`deckboss-ref/ROADMAP.md`, "The restore drill"): device A
   populates + syncs, device B with empty local state and the same storage
   recovers the entries. Text recovers byte-for-byte. (Falsifies: "sync works"
   when it silently never ran — the bug the sibling found and fixed in
   `registry.ts`.)
3. **The BYOK chat panel actually calls the key a user actually enters**,
   against at least one of {OpenAI, DeepSeek, OpenRouter, local Ollama}, and
   the response lands in the log as a `chat.exchange` event with `model` and
   `key_owner` set. (Falsifies: "chatbot panel" that's a static textarea.)
4. **Every tier-marker in this doc is represented on the page honestly** —
   features marked ✅ work; features marked ⚠️ surface their condition
   ("Chrome/Edge only," "configure OLLAMA_ORIGINS"); features marked 🔮 are
   absent, not labeled "coming soon." (Falsifies: the `fishinglog-agent`
   pattern — a feature claimed in copy that the code never implements.)
5. **No org-operated endpoint holds or aggregates user log data.** The
   no-operated-backend property is literally true on day one (BYOK only);
   if a free allowance ships later, it meets the §5 three-constraint test.

The eight skins do not start until 1–5 are true, verified the same way Step 1
verifies DeckBoss — exercised through the real use case, not "has a demo"
(`ROADMAP.md` §Step 5, "Skinning gate").

---

## 9. Open uncertainties (stated, with recommendations)

Where the evidence is genuinely incomplete, the call is made here rather than
silently in code, per `ROADMAP.md`'s "How this roadmap gets updated" rule that
a roadmap shouldn't quietly diverge from reality.

- **OPFS eviction durability vs. `navigator.storage.persist()`.** DeckBoss
  requests `navigator.storage.persist()` at boot because iOS Safari evicts
  non-installed web-app storage after ~a week idle (`deckboss-ref/ROADMAP.md`,
  "Pre-launch hardening pass"). ActiveLog must do the same and surface grant
  status in settings. **Uncertain:** whether OPFS as the default "save later"
  store is durable enough to be the *only* storage a casual user ever
  configures, or whether the page should nudge toward a linked cloud backend
  more aggressively. **Recommendation:** ship OPFS as default, show a clear
  "stored only in this browser" warning until a cloud backend is linked, and
  let field usage (the Step 1 discipline) decide whether the nudge needs to be
  louder.
- **The browser embedding store for Tier C wake-matching.** pincher uses
  `sqlite-vec` in Rust; there is no drop-in `sqlite-vec` in the browser.
  Candidates: `sqlite-wasm` with a hand-rolled cosine scan, or `localStorage`-
  backed vectors for small reflex sets. **Uncertain:** which is fast enough at
  the reflex counts a real user accumulates. **Recommendation:** defer the
  choice to Phase 2; Tier B (exact match) covers day-one needs with no vector
  store at all, exactly as pincher's hash-fallback covers the no-model case.
- **Manifest vs. directory-scan on the new `LocalFolderAdapter`.** Through
  `FileSystemDirectoryHandle` a recursive walk *is* possible, so an orphan-
  scan fallback is more natural here than on Drive (where the recursive walk
  was itself the bug). **Uncertain:** the cost of a full directory walk on a
  large folder on a slow phone. **Recommendation:** reuse the sibling's
  scoped fallback-scan pattern (scan only when local state is empty — the
  cold-start-recovery signature) rather than scanning on every sync.

---

## 10. Relationship to the parallel first-slice doc

A second, parallel task is drafting `docs/ACTIVELOG_FIRST_SLICE.md` — the
*first buildable slice* scope. This architecture is deliberately the broader
seam map; the slice doc should be able to point at §§1–3 for storage and §§3–4
for voice and pick the smallest slice that satisfies acceptance items 1–3
above (record → markdown somewhere real; one storage backend actually
persisting; BYOK chat against one provider). Everything marked 🔮 in this doc
is, by construction, outside the first slice. If the slice doc and this
architecture disagree on a seam, the architecture's contract (the
`StorageAdapter` shape, the `AppConfig`-never-syncs invariant, the additive
`LogEntry`) wins — because those are the inherited, already-hardened
boundaries, and the slice is built inside them, not against them.

---

*Architecture for the ActiveLog core. Evidence-cited throughout; where a claim
is uncertain it is marked and a recommendation given, never silently resolved.
Sibling citations resolve in `purplepincher/deckboss` (paths under
`src/...` and `docs/...`) and `purplepincher/pincher` (paths under
`pincher-core/src/...`), both already graduated repos in this org.*
