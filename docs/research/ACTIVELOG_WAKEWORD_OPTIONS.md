# ActiveLog wake-word options — real browser KWS research

**Scope:** Real, verified options for wake-word / keyword-spotting (KWS) that could run
in a browser tab, scoped to the boat-deck use case named in
[`ACTIVELOG_ARCHITECTURE.md`](../ACTIVELOG_ARCHITECTURE.md) §4 and the open
uncertainty in §9: noisy environment, distinguish a fixed wake-word like "cocapn"
from ambient engine noise and conversation, low false-positive rate, and — for the
always-listening part — no server round-trip (privacy + latency).

**Why this exists:** The architecture doc parked "real semantic wake/intent matching"
as Tier C (🔮 later-phase), and its §9 open-uncertainty note names two candidates for
the Tier C vector store — "`sqlite-wasm` with a hand-rolled cosine scan, or
`localStorage`-backed vectors" — on the stated premise that "there is no drop-in
`sqlite-vec` in the browser." [`ACTIVELOG_FIRST_SLICE.md`](../ACTIVELOG_FIRST_SLICE.md)
§2 Phase 2b then leaves the *whole* wake-word mechanism as decision D4 (Open),
recommending pause-segmentation + explicit activation and rejecting online wake-word
for v1. This doc closes that fork with verified 2026-07-08 evidence rather than
recalled-from-training prose, and gives a Phase 2 recommendation a reviewer can
confirm or reject on the record.

**Method:** Every package/page claim below was checked against the live source — npm
package pages, GitHub repos (star counts, last-release dates, license badges, FAQ
verbatim), and the sqlite-vec docs site — on 2026-07-08, not from memory. Where a
number is stated, the source is named in line. Where I could not verify a number
from public docs (e.g. Porcupine's exact free-tier device cap), that is said plainly
rather than guessed.

---

## Honest verdict, up front

1. **For Phase 2, Tier B (exact/fuzzy phrase match against the live Web Speech
   transcript) is genuinely sufficient, and nothing fancier should be built.** It
   reuses the Phase 1 transcriber, runs in microseconds, has zero new download, and
   for a *small, known* wake vocabulary it is the right tool — a dedicated KWS model
   is redundant when a recognizer is already running. This confirms the first-slice
   doc's own D4 recommendation (ship 2a pause-segmentation + explicit activation),
   and the org's principle 5 ("ruthless scope") argues against building Tier C until
   a real field need appears.
2. **Neither Porcupine nor openWakeWord is a clean fit for an offline-first, open,
   no-operated-backend org.** Porcupine is real, accurate, and has a genuine WASM
   web SDK — but its *engine* is proprietary and AccessKey-gated with opaque
   account limits, and it forces a COOP/COEP header requirement. openWakeWord is
   fully open and offline — but it has **no browser port** (the maintainer says so
   on the record), its last release was Feb 2024 (~2.5 years stale), and its
   pre-trained models are **CC BY-NC-SA (non-commercial)**.
3. **The `sqlite-vec` question has a real answer now: the architecture doc's §9
   premise is stale in letter but correct in spirit.** `sqlite-vec` does ship WASM
   builds today, so "there is no drop-in `sqlite-vec` in the browser" is no longer
   literally true. **But there is still no production-grade drop-in**: it cannot be
   loaded as an extension into the canonical `@sqlite.org/sqlite-wasm` (WASM SQLite
   can't load extensions), so it must be *statically compiled into a custom WASM
   build*, and the only published browser artifact is an explicitly-unversioned
   demo package whose npm readme is the word `TODO`. Worse for the actual use case:
   at the reflex count a real user accumulates (dozens to low-hundreds of vectors),
   sqlite-vec's KNN/ANN machinery is overkill — a brute-force cosine scan over a
   `Float32Array` is sub-millisecond and needs no vector DB at all.

Each is detailed below with the evidence.

---

## 1. Porcupine (Picovoice) — real and accurate, but the engine is proprietary and gated

### What's real and verified

- **A genuine, maintained Web/WASM SDK exists.** `@picovoice/porcupine-web` on npm
  is at **v4.0.1, published ~13 days before this check** (late June 2026), with
  **~22,925 weekly downloads**, **Apache-2.0** license, and TypeScript types
  (npm package page, 2026-07-08). The GitHub repo `Picovoice/porcupine` has
  **4.9k stars / 577 forks / 1,510 commits** and an Apache-2.0 license badge. It
  runs in Chrome, Edge, Firefox, and Safari via WebAssembly; microphone audio is
  handled by the Web Audio API and a `WebVoiceProcessor` helper, and the engine is
  packaged as a Web Worker.
- **It is genuinely offline after init.** Audio is processed on-device; nothing is
  sent to a cloud. The `.pv` parameter model and `.ppn` keyword files are cached in
  IndexedDB after first load. This is the one option here that meets the brief's
  "no server round-trip for the always-listening part" literally.
- **Custom wake-words are self-service.** You type the phrase ("cocapn") into the
  Picovoice Console and it trains a model "within seconds" via *transfer learning*
  (no bespoke audio collection), then download the `.ppn` for platform `Web (WASM)`.
  There is also an API call, `Porcupine.trainWakeWordFromPhrase(...)`, to train
  programmatically (npm readme). So a "cocapn" wake-word is achievable.
- **Two real infra requirements the UI must surface:** (a) multithreaded WASM needs
  `SharedArrayBuffer`, which requires serving the page with
  `Cross-Origin-Opener-Policy: same-origin` and
  `Cross-Origin-Embedder-Policy: require-corp` — without them it falls back to
  single-threaded `ArrayBuffer`; (b) worker usage needs IndexedDB (so Firefox
  Incognito must run on the main thread). These are stated in the npm readme.

### Licensing model — the load-bearing nuance

The SDK **bindings** are Apache-2.0. The **engine is not.** Porcupine requires a
Picovoice `AccessKey` at initialization, and the Porcupine intro doc states
verbatim: *"AccessKey is your authentication and authorization token for using
Picovoice. It also verifies that your usage is within the limits of your account."*
In other words the precompiled engine/model artifacts are proprietary and gated by
account-level quotas enforced at runtime via the AccessKey — the open-source license
covers only the thin JS/C glue.

What the public docs *do* say about tiers: there is a **"Free Plan"** (console
signup is "free, no credit card required"); a **"Free Trial"** described as letting
"enterprise developers and teams **evaluate** Picovoice before committing to a paid
plan"; and beyond that, commercial use is **"Talk to Sales … We'll put together a
plan tailored to your business."**

What the public docs **do not** say (and I could not verify from any page I could
fetch): the concrete free-tier device cap (e.g. "N monthly active devices"). That
number lives inside your Picovoice Console account, not on a public pricing page. So
the honest statement is: **the limits are real and enforced, but opaque until you
hold an account, and production use is explicitly a sales conversation, not a
published tier.** For an org whose entire posture is "no operated backend, no
licensing trap, honest docs" (`PARADIGM.md`), a proprietary, AccessKey-gated engine
whose terms can change with the account is a structural mismatch — even though the
SDK code is permissively licensed.

### Accuracy / false-positive characteristics — real, but vendor-measured

Picovoice maintains an open-source benchmark, `Picovoice/wake-word-benchmark`
(**157 stars**, Apache-2.0, 31 commits). Its methodology is genuinely relevant to
the boat-deck case: it reports **miss-rate at 1 false-alarm per 10 hours** across
six keywords, with the audio **mixed with noise at 10 dB SNR** using the DEMAND
dataset (18 real environments — kitchen, office, traffic, etc.) over a LibriSpeech
background. The Porcupine README's headline claim, drawn from that benchmark, is
that Porcupine is **"11.0 times more accurate and 6.5 times faster"** than the best
of PocketSphinx and Snowboy.

Two honesty caveats, both worth bank: (1) it is a **vendor-run** benchmark —
Picovoice measuring itself against two competitors that are, respectively,
**abandoned** (Snowboy) and **legacy** (PocketSphinx); (2) **openWakeWord is not in
it**, so this benchmark cannot compare Porcupine to the open-source option this org
would most likely consider. The sensitivity knob (a float in `[0,1]` per keyword,
documented in the SDK) is the real FP/Miss tradeoff lever in deployment, and tuning
it against real deck noise would be a required field step, not something a benchmark
settles.

### Bottom line on Porcupine

Technically the strongest browser KWS option that exists today: real WASM, genuinely
offline, trainable on "cocapn," and noise-robust in a way the architecture doc's
Tier B cannot match. **It is disqualified for this org on licensing grounds, not
technical ones** — proprietary gated engine, opaque quotas, COOP/COEP header
constraints, and "Talk to Sales" for anything real. It is the right answer for a
different org; it is the wrong answer for an offline-first, open, no-operated-
backend reference build. Keep it on file as the commercial reference point.

---

## 2. openWakeWord (and the OSS field) — open and offline, but not in a browser

### What's real and verified

- **`dscripka/openWakeWord`** is the leading open-source KWS: **2.5k stars / 298
  forks / 227 commits**, latest release **v0.6.0 on 2024-02-11** — and the README's
  "Updates" section's *last entry is 2024-02-11*, i.e. the project's last meaningful
  motion was roughly **2.5 years before this check**. There are 111 open issues. It
  is not abandoned (the repo is live), but it is clearly low-activity.
- **Architecture** (from the README): ONNX melspectrogram → a Google
  `speech_embedding` backbone (Apache-2.0, from TFHub) → a small per-word
  classifier. Models process **80 ms** audio frames and emit a `[0,1]` score. It
  runs on `onnxruntime` and `tflite-runtime`. It optionally chains **Speex noise
  suppression** (Linux x86/Arm64 only) and a **Silero VAD** to cut false positives —
  both directly relevant to noisy-deck robustness. Self-reported target performance
  is **<5% false-reject, <0.5 false-accepts/hour** with threshold tuning, evaluated
  against the ~5.5h **Dinner Party Corpus** (far-field speech + music + noise).
- **Pre-trained models** ship for only `alexa`, `hey mycroft`, `hey jarvis`, `hey
  rhasspy`, `weather`, `timers` — **English only, and no "cocapn."** Custom models
  are trainable from 100%-synthetic TTS data via a Google Colab notebook in "<1
  hour." The README's own comparisons claim parity-with-or-better-than Porcupine on
  `alexa`/`hey mycroft`, but explicitly hedge: "sample sizes are small … these
  results should be interpreted cautiously … the only claim being made is that
  openWakeWord models are broadly competitive."

### The licensing trap

The **code** is Apache-2.0. The **pre-trained models are CC BY-NC-SA 4.0**
(Creative Commons **NonCommercial**, ShareAlike). The README states why: "due to the
inclusion of datasets with unknown or restrictive licensing as part of the training
data." That NonCommercial clause means the shipped models **cannot be used in any
commercial deployment** without separate licensing — a hard blocker for a product
that ships to the public, and a trap for anyone who reads "Apache-2.0" on the repo
and stops there. Custom models you train yourself from clean data could avoid this,
but that is a research/legal project, not a drop-in.

### WASM/browser portability — the decisive finding, on the record

This is the question the brief asks most directly, and openWakeWord's own FAQ
answers it unambiguously:

> **"Can openWakeWord be run in a browser with javascript?"** — *"While the ONNX
> runtime does support javascript, much of the other functionality required for
> openWakeWord models would need to be ported. **This is not currently on the
> roadmap**, but please open an issue/start a discussion if this feature is of
> particular interest."*

The offered workarounds are (a) stream audio from the browser to a **Python backend
via websockets** (the `examples/web` scripts from 2023) — which **defeats the
offline-first thesis entirely** — or (b) "potential" use of `pyodide`. So:
**openWakeWord does not run in a browser, and its maintainer is not building that.**

The honest nuance: the model *files* (`.tflite`/`.onnx`) are portable formats, and
`onnxruntime-web` (ONNX Runtime's real, maintained JS/WASM binding) exists. So in
principle the *model* could run under `onnxruntime-web` — but that requires porting
the melspectrogram preprocessor and the streaming/glue logic the Python library
provides. That is a real porting project (weeks, with audio DSP), not a drop-in, and
the project that would normally do it has been quiet since Feb 2024. There is no
maintained `openwakeword-js`/`-web` package today.

### The rest of the OSS field (verified negative, briefly)

- **Mycroft Precise / Snowboy / PocketSphinx** — the usual "open KWS" names — are
  respectively Mycroft-discontinued, abandoned (Kitt-AI/Snowboy archived), and
  legacy C. openWakeWord's README declines to even benchmark them because they are
  unmaintained or far behind Porcupine. Not candidates for a 2026 browser build.
- **TFLite Micro / "micro-speech"** (Google's embedded KWS) and **microWakeWord**
  (`kahrendt/microWakeWord`, which openWakeWord's README recommends for ESP32) are
  real and open, but target microcontrollers, not the browser. microWakeWord is the
  right answer for the **ESP32/Helm-unit tier** (the org's later `cocapn` hardware
  track, `ROADMAP.md` Step 5/6), where always-on KWS physically belongs — not for a
  browser tab.

### Bottom line on openWakeWord

The right *conceptual* fit (open, offline, noise-robust, trainable on synthetic
data) and the wrong *actual* fit for the browser (no JS port, not on the roadmap,
stale, NonCommercial models). If this org ever needs true offline browser KWS, the
honest path is a scoped spike that ports an openWakeWord-style model to
`onnxruntime-web` — but that is a research deliverable, not a Phase 2 dependency,
and it should not be assumed to be quick.

---

## 3. Tier B (Web Speech transcript match) — sufficient, and here is the real opinion

The architecture doc §4 defines Tier B as exact/regex match of a wake-phrase against
the live Web Speech transcript — no embedding model, no KWS model, the browser port
of pincher's **Exact** tier. The brief asks for a real opinion on whether anything
fancier is justified for "a small, known wake-word vocabulary." The opinion, with
the reasoning:

**Tier B is genuinely sufficient for Phase 2's actual job, which is "organize an
active dictation session via a handful of known wake-phrases," not "always-listening
hands-free KWS on a noisy deck."** Three reasons it is the right tool:

1. **It is free in every dimension.** It reuses the Phase 1 transcriber that
   `ACTIVELOG_FIRST_SLICE.md` §1 already ships (Web Speech API default, inherited
   from deckboss). Zero new model download, zero new WASM, zero new dependency,
   match cost in microseconds. A dedicated KWS model is *redundant* when a
   recognizer is already transcribing — you would be running two speech models to
   find a word the first one already output.
2. **For a made-up wake-word like "cocapn," a cloud STT + fuzzy match is at least as
   accurate as any small generic KWS, and far more accurate than a *pre-trained*
   KWS.** No off-the-shelf KWS model has ever heard "cocapn" — Porcupine would need
   a custom-trained `.ppn`, openWakeWord a custom-trained model from synthetic TTS.
   Web Speech's large cloud model, by contrast, will attempt to transcribe anything,
   and a small fuzzy matcher (Levenshtein distance ≤1–2, or a few alias strings like
   "co captain"/"co cap'n"/"cocapn") catches the predictable mishearings. False
   positives are *tighter* than a KWS model's, because you only fire on a specific
   string, not on a confidence score over continuous audio.
3. **The remaining weaknesses are addressable engineering, not blockers.** The two
   real ones: (a) Web Speech is **network-backed on Chrome** — so the wake layer is
   *not* offline, which the architecture doc §3 already states honestly and which
   means Tier B does **not** satisfy the brief's "no server round-trip for
   always-listening" clause; (b) `SpeechRecognition` is "Limited availability"
   (MDN), absent on Firefox, and the on-device variant (`processLocally` /
   `available()` / `install()` + the `on-device-speech-recognition`
   Permissions-Policy) is arriving but not yet a reliable cross-browser default. The
   first-slice doc's Phase 2b recommendation resolves both by **not** shipping
   always-listening at all — it ships pause-segmentation (2a) + **explicit
   activation** (tap-to-wake) — so Tier B's online-ness never becomes a wake-word
   privacy regression.

**The one case where Tier B is genuinely *not* sufficient**, stated plainly so it is
not discovered late: if a field user proves they need **true always-on, hands-free,
offline** wake on the boat deck — mic open for hours, "cocapn" fires an action with
no tap and no network — then Tier B fails the offline clause and you are back in the
KWS market this doc surveys. **That requirement is not in Phase 2** (the first-slice
doc defers it explicitly under D4), and the org's principle 5 says do not build for
it until it is proven. So: ship Tier B + explicit activation for Phase 2, and treat
"always-on offline deck wake" as a Step-6+ / `cocapn` hardware-tier question (where
microWakeWord on an ESP32 is the honest answer), not a Phase 2 browser question.

---

## 4. The `sqlite-vec` question, answered directly

The brief asks: is there now (or was there ever) a real, working `sqlite-wasm` +
vector-search combination usable in a browser, or does that combination not really
exist in a maintained form? Verified answer:

### `sqlite-vec` is real, maintained, and does ship WASM — but it is not a drop-in

`asg017/sqlite-vec` (successor to `sqlite-vss`) is **7.8k stars / 330 forks / 464
commits / 88 releases**, **MIT OR Apache-2.0**, latest release **v0.1.9 on
2026-03-31** (actively maintained), a Mozilla Builders project sponsored by Fly.io,
Turso, SQLite Cloud. Its README explicitly claims it "runs anywhere SQLite runs
(Linux/MacOS/Windows, **in the browser with WASM**, Raspberry Pis, etc.)." So the
literal claim "there is no `sqlite-vec` in the browser" — which the architecture
doc's §9 used as its premise for deferring — is **now stale**. sqlite-vec does have
WASM.

**But "has a WASM build" is not "is a drop-in for `@sqlite.org/sqlite-wasm`," and
that distinction is the whole ballgame.** The sqlite-vec docs page "Browser (WASM)"
(`alexgarcia.xyz/sqlite-vec/wasm.html`) states the constraint verbatim:

> *"It's not possible to dynamically load a SQLite extension into a WASM build of
> SQLite. So `sqlite-vec` must be **statically compiled into custom WASM builds**."*

In other words you cannot `npm install @sqlite.org/sqlite-wasm`, then load sqlite-vec
into it the way you load a `.so`/`.dylib` into desktop SQLite. You must **compile
your own** WASM SQLite with sqlite-vec baked in and ship *that* binary — a real
maintenance burden that re-triggers every time either SQLite or sqlite-vec moves
(which both do, frequently). The sqlite-vec Node/Deno/Bun bindings (`sqlite-vec` on
npm) do not help here — they load a *native* extension into a *native* SQLite
(`better-sqlite3`, `node:sqlite`, `bun:sqlite`), none of which run in a browser.

### The only published browser artifact is an unversioned demo

The docs point to one npm package for browser use: **`sqlite-vec-wasm-demo`**.
Verified on npm 2026-07-08: its **readme is the literal string `TODO`**, it has
**301 weekly downloads and 2 dependents**, and the sqlite-vec docs warn it is "a
**demonstration** … may change at any time. **It doesn't follow the Semantic
versioning of `sqlite-vec`**." This is not something to build a product feature on.
So the honest, direct answer to the brief's question #4:

> **As of 2026-07-08 there is no production-grade, versioned, drop-in `sqlite-vec`
> for the browser.** The combination "canonical `sqlite-wasm` + loadable `sqlite-vec`
> extension" does not exist (WASM SQLite cannot load extensions); the only paths are
> maintaining a custom-compiled WASM SQLite+sqlite-vec binary yourself, or using an
> explicitly-unversioned demo package with a `TODO` readme. The architecture doc's
> §9 deferral was therefore **right in conclusion** (don't pick the Tier C store
> yet) even though one of its **premises is now stale** (sqlite-vec *does* have
> WASM). This doc records that correction so the architecture doc can be fixed.

### The deeper point the §9 uncertainty didn't need sqlite-vec to settle

§9 frames the uncertainty as "which is fast enough at the reflex counts a real user
accumulates": `sqlite-wasm`+cosine vs `localStorage`-backed vectors. The boring,
verified answer is **neither sqlite-vec nor sqlite-wasm is needed at all**. Tier C's
real data is a user's set of wake-phrase reflexes — pincher's Exact/Similar/Novel
tiers against an `all-MiniLM-L6-v2` (384-dim) embedding. A real user accumulates
**dozens to low-hundreds** of these, not millions. At that scale, a brute-force
cosine similarity over a flat `Float32Array` (persisted in IndexedDB) is
**sub-millisecond** and uses no index, no virtual table, no extension, and no
binary-compile step. sqlite-vec's KNN/ANN machinery (matryoshka, IVF, DiskANN — all
present in its repo as `sqlite-vec-diskann.c`, `sqlite-vec-ivf*.c`) exists to make
*million*-vector search fast; it is pure overhead at reflex scale. So when/if Tier C
is ever built, the store question answers itself: **plain cosine over a typed array,
in a Web Worker, persisted to IndexedDB — no vector database of any kind.** That
removes the last technical reason Tier C ever needed sqlite-vec, and it removes the
§9 "which is fast enough" uncertainty outright.

---

## Recommendation for Phase 2

**Phase 2 should ship Tier A (pause/silence segmentation) + Tier B (exact/fuzzy
transcript match) + explicit activation (tap-to-wake), and build no KWS model and no
vector store.** This is exactly what `ACTIVELOG_FIRST_SLICE.md` §2 already
recommends under decision D4; this research confirms it on the evidence rather than
on instinct. Concretely:

- **Phase 2a (pause segmentation):** unchanged from the first-slice doc. Ships now.
- **Phase 2b (wake mechanism):** ship **explicit activation + Tier B transcript
  match** as the wake layer. Tap-to-wake is the honest primary; Tier B
  exact/fuzzy-phrase match against the Web Speech transcript is the "magic" layer
  that costs nothing because the recognizer is already running. Label it honestly in
  the UI as "online wake-word" wherever Web Speech is network-backed (per
  architecture doc §3), which is consistent with deckboss's `hadNetworkError`
  discipline.
- **Defer Tier C (semantic wake/intent matching) indefinitely**, gated on a real
  field need that Tier B provably cannot meet. Principle 5 ("ruthless scope") is the
  deciding rule: building a fancier tier nobody has proven necessary is exactly the
  scope-creep the org hardens against.

**Two corrections this research implies for `ACTIVELOG_ARCHITECTURE.md`, for the
reviewer to apply or reject:**

1. **§9's premise "there is no drop-in `sqlite-vec` in the browser" is now stale in
   letter.** sqlite-vec ships WASM. The accurate 2026-07-08 statement is: "*sqlite-vec
   has WASM, but it cannot be loaded into the canonical `sqlite-wasm` (WASM SQLite
   can't load extensions), so it requires a custom-compiled binary, and the only
   published browser artifact is an unversioned demo.*" The §9 *recommendation*
   (defer the choice; Tier B covers day-one) stands unchanged.
2. **When/if Tier C is ever scoped, the store decision is already made by the
   data shape:** plain cosine over a `Float32Array` in IndexedDB at reflex-scale.
   Neither `sqlite-wasm` nor `sqlite-vec` is warranted, which dissolves the §9
   "which is fast enough" open question. This should be recorded so a future Phase
   doesn't re-litigate a vector-DB choice that the scale rules out.

**One forward-looking note for the boat use case specifically, parked not
actioned:** *if* true always-on, offline, low-FP "cocapn" wake on a noisy deck ever
becomes a *proven* requirement (it is not, in Phase 2), none of the browser options
here is clean — Tier B is online on Chrome, Porcupine is proprietary/gated,
openWakeWord has no browser port. The honest architecture at that point is to move
always-on wake **off the browser tab and onto the ESP32/Helm-unit tier** using
`microWakeWord` (open, synthetic-data-trained, microcontroller-grade), and treat the
browser as the *consumer* of an already-fired wake signal over the device link. That
is a `cocapn` / Step-6+ decision owned by the hardware track
(`docs/ACTIVELOG_ARCHITECTURE.md` §7), not a Phase 2 browser deliverable — and it is
the one configuration in this whole survey where always-on KWS actually earns its
keep.

---

## Sources (all fetched/verified 2026-07-08)

- npm — `@picovoice/porcupine-web` (v4.0.1, Apache-2.0, ~22,925 weekly downloads,
  README: SharedArrayBuffer/COOP-COEP, AccessKey, IndexedDB caching,
  `trainWakeWordFromPhrase`).
- GitHub — `Picovoice/porcupine` (4.9k★ / 577 forks / 1,510 commits, Apache-2.0;
  README "Performance": 11.0× more accurate / 6.5× faster than PocketSphinx/Snowboy).
- GitHub — `Picovoice/wake-word-benchmark` (157★, Apache-2.0; methodology: miss-rate
  at 1 false-alarm/10h, 10 dB SNR DEMAND noise, LibriSpeech; 3 engines only).
- Picovoice docs — `/docs/porcupine/` (AccessKey "verifies that your usage is within
  the limits of your account"; Free Plan / Free Trial / "Talk to Sales" commercial).
- GitHub — `dscripka/openWakeWord` (2.5k★ / 298 forks / 227 commits, latest release
  v0.6.0 2024-02-11; code Apache-2.0, **models CC BY-NC-SA 4.0**; FAQ: browser port
  "not currently on the roadmap"; Speex NS + Silero VAD; English only).
- GitHub — `asg017/sqlite-vec` (7.8k★ / 330 forks / 464 commits / 88 releases,
  MIT/Apache-2.0, latest v0.1.9 2026-03-31; README: "in the browser with WASM").
- sqlite-vec docs — `alexgarcia.xyz/sqlite-vec/wasm.html` ("not possible to
  dynamically load a SQLite extension into a WASM build of SQLite … must be
  statically compiled into custom WASM builds") and `/js.html` (Node/Deno/Bun only).
- npm — `sqlite-vec-wasm-demo` (v0.1.9, readme = "TODO", 301 weekly downloads, 2
  dependents; docs: "demonstration … doesn't follow semantic versioning").
- Internal — `docs/ACTIVELOG_ARCHITECTURE.md` §§3–4, 9; `docs/ACTIVELOG_FIRST_SLICE.md`
  §2 + decision D4; `CONTRIBUTING.md` principle 5.

---

*Research for the ActiveLog core, branch `rd-wakeword-2026-07-08`, for independent
review before any merge. Every external claim is dated and sourced; where the honest
answer was "no good option exists," it is said plainly rather than dressed up as a
theoretical possibility — the same honesty discipline the architecture doc's tier
markers (✅ / ⚠️ / 🔮) exist to enforce.*
