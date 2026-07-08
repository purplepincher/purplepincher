# Chrome built-in AI — real, current status (Prompt API / Gemini Nano)

**Researched:** 2026-07-08, on branch `rd-chrome-ai-2026-07-08`.
**Scope:** what Chrome's on-device AI APIs *actually* are right now —
namespace, shipping status, model-download reality, the real code shape, and
whether the Summarizer/Writer/Rewriter APIs are real. Written to verify and
deepen (not duplicate) the deliberately-vague Chrome-AI subsections added on
branch `activelog-architecture-addendum-2026-07-08` to
[`../ACTIVELOG_ARCHITECTURE.md`](../ACTIVELOG_ARCHITECTURE.md) (§4 "Chrome's
on-device Prompt API — an optional accelerator for Tier C", and §5's "on-device
escape hatch"). Those subsections said, repeatedly and on purpose, *"verify the
current enablement surface"* and *"the exact JS entry point"* against live docs;
this document is that verification.

> **⚠️ This space moves fast. Findings here go stale fast.** Everything below was
> read directly from Chrome's own developer docs on 2026-07-08; the "Last
> updated" date of each cited page is given so the staleness is auditable, not
> asserted. The single most volatile facts are (a) which Chrome version is
> current stable, (b) whether the base Prompt API on the web needs an origin
> trial, and (c) the exact model variant sizes — re-check all three before
> shipping anything that depends on them. If you are reading this more than
> ~3 months past the researched date, treat it as a lead, not a source of truth,
> and re-fetch the cited pages.

The honesty markers match the architecture doc: **✅ real today**, **⚠️ real but
conditional**, **🔮 later phase / not day-one**.

---

## TL;DR — the verdict, up front

- **The Prompt API is real and shipping in stable Chrome** (on the web since
  Chrome 148; in extensions since Chrome 138), reached through a global
  **`LanguageModel`** class — *not* `window.ai`, which is a stale name from
  2024 dev-channel iterations that no longer matches the shipping surface. ✅
- It is **Chromium-only and desktop-only**, absent on all mobile Chrome, iOS,
  Safari, and Firefox, and requires a real **hardware floor** (≥16 GB RAM or
  >4 GB VRAM, ≥22 GB free disk, desktop OS) plus a one-time multi-GB model
  download. ⚠️
- Therefore: **safe to build against as a progressive-enhancement / fallback
  path** (feature-detect with `LanguageModel.availability()` and degrade
  gracefully), **not safe as the only path** on any page real users will visit
  this year. That is exactly the shape the architecture doc already specifies
  for Tier C ("prompt-capable-when-present, exact-match-when-not").
- **The Summarizer API is the most mature** on-device win (stable since Chrome
  138) — if ActiveLog wants one low-risk local capability, transcript
  summarization via `Summarizer` is it. The **Writer/Rewriter/Proofreader APIs
  are real and Chrome-specific, but still in origin trial (Developer Trial),
  not stable.** The "captain could write a book via the Writer API" idea maps
  onto a real API but overstates what the Writer API is shaped for — see §4.

---

## 1. Current real availability — namespace, stable-vs-trial, the flag process

### The namespace is a global `LanguageModel`, not `window.ai`

The early-2024 developer-channel surface (`window.ai.languageModel`) is gone.
The shipping API is a set of **top-level global classes**, one per task:

| API | Global entry point | Status (web) | Status (extensions) |
|---|---|---|---|
| Prompt API | `LanguageModel` | **Chrome 138** → see nuance | Chrome 138 (stable) |
| Summarizer API | `Summarizer` | Chrome 138 (stable) | Chrome 138 (stable) |
| Translator API | `Translator` | Chrome 138 (stable) | Chrome 138 (stable) |
| Language Detector API | `LanguageDetector` | Chrome 138 (stable) | Chrome 138 (stable) |
| Writer API | `Writer` | Developer trial (origin trial) | Developer trial |
| Rewriter API | `Rewriter` | Developer trial (origin trial) | Developer trial |
| Proofreader API | `Proofreader` | Developer trial (origin trial) | Developer trial |

(Source: Chrome's "API status and overview" table, page last updated
2025-09-12, read 2026-07-08.) Feature-detection is `"LanguageModel" in self`,
not a `window.ai` existence check.

### The base Prompt API on the web is no longer origin-trial-gated — a correction to the addendum

This is the single most important verification, and it **updates** the
addendum's framing. The architecture addendum (§4) described the Prompt API as
"origin-trial / flag-gated, requires the browser to have downloaded the on-device
model once." Read charitably that was correct when drafted conservatively, but
**the gating has since moved**, exactly as the addendum warned it might ("the
gating mechanism has shifted across Chrome versions and must not be assumed from
memory"). The real picture today:

- **Base Prompt API on the web: shipped stable.** Chrome's Prompt API page
  (header "Chrome 148", linked to an **Intent to Ship** — not Intent to
  Experiment — page last updated 2026-05-19, read 2026-07-08) shows the base API
  at Chrome 148 in stable. `LanguageModel.create()` with default parameters
  works on a production HTTPS origin with **no origin-trial token** and **no
  flag**.
- **Only *sampling-parameter tuning* (`topK` / `temperature`) is still
  origin-trial-gated** — a separate status-table row, "Origin trial for sampling
  parameters", Chrome 148, Intent to Experiment. The Prompt API doc's own
  caution confirms this: *"Language model parameters exclusively apply to the
  Prompt API for Chrome Extensions or if you use the Prompt API with the Origin
  Trial enabled."* So default-temperature prompting is open; turning the
  temperature dial still needs a token.
- **In Chrome Extensions: stable since Chrome 138**, and the previously-required
  `"permissions": ["aiLanguageModelOriginTrial"]` manifest entry is now an
  **expired** origin-trial permission that extension authors are told to remove.

**Net for a web app today:** no flag, no token, no enrollment needed to call
`LanguageModel` with defaults — *provided* the visitor is on Chrome ≥148 desktop,
meets the hardware floor, and has the model downloaded (§2). The flags below are
**only for localhost development**, not production:

1. `chrome://flags/#optimization-guide-on-device-model` → Enabled
2. `chrome://flags/#prompt-api-for-gemini-nano` → Enabled (or "Enabled
   multilingual")

(Source: "Get started with built-in AI", last updated 2025-05-20, read
2026-07-08.)

> **Version-timeline honesty.** Chrome 138 (the wave that shipped Summarizer /
> Translator / Language Detector stable, and the Prompt API in extensions) was
~mid-2025. Chrome 138 → 148 is ten 4-week milestones ≈ early 2026. So at this
> document's 2026-07-08 research date the web Prompt API has been in stable for
> roughly a few months — real, but recently landed. Chrome's release calendar
> (not re-verified here) should be checked to confirm 148 is the current-or-past
> stable rather than upcoming before any "it's in stable Chrome *today*" claim
> goes on a shipped page.

### Chromium-only, desktop-only — confirmed, with the exact exclusions

The hardware/OS section is identical across all the foundation-model API pages
and is explicit about what is **not** supported:

- **Supported OS:** Windows 10/11, macOS 13+ (Ventura), Linux, ChromeOS from
  Platform 16389.0.0 onward — but **only on Chromebook Plus** devices.
- **Explicitly NOT supported:** "Chrome for Android, iOS, and ChromeOS on
  non-Chromebook Plus devices are not yet supported by the APIs which use
  foundation models."
- **Safari / Firefox:** the APIs are W3C **Web Incubator Community Group
  (WICG)** proposals ("We are requesting feedback from the W3C, Mozilla, and
  WebKit for each API"). They are not shipping in either. This matches the
  addendum's "not present in Safari, Firefox, or mobile Chrome" exactly.

The addendum also says "Chrome/Edge." Edge is Chromium-based, but Chrome's docs
name only Chrome and do **not** confirm Edge enables the on-device model. Treat
Edge as *probably-capable* (Chromium) but **unconfirmed by the source docs**;
state "Chrome" on the page unless Edge is separately verified.

---

## 2. Model download / readiness — the real API surface and the real minimum spec

### `availability()` returns one of four exact strings

This is the "is the model ready / needs-download / unavailable" check, and its
contract is pinned (source: "Get started with built-in AI", the `availability()`
section, read 2026-07-08):

| Return value | Meaning | What your code does |
|---|---|---|
| `"available"` | Model ready; create a session now. | Call `create()`. |
| `"downloadable"` | Device qualifies but the model isn't downloaded yet. | **Require user activation**, then call `create()` with a `monitor` to show download progress. |
| `"downloading"` | A download is already in progress. | Wait; surface progress; do not call `create()` again concurrently. |
| `"unavailable"` | Device/options not supported (insufficient power, disk, or the requested modality/language isn't supported). | Fall back to the non-on-device path. Never call `create()`. |

Two non-obvious rules that the architecture must respect:

- **Pass the same options to `availability()` as to `prompt()`/`promptStreaming()`.**
  The doc is explicit: *"Always pass the same options to the `availability()`
  function that you use in `prompt()` or `promptStreaming()`. This is critical,
  as some models may not support certain modalities or languages."* A
  text-only `availability()` check does **not** authorize an audio-input prompt
  later — re-check with `{ expectedInputs: [{type:'audio'}] }` before relying on
  audio.
- **`availability()` itself can trigger the download** in some cases (e.g. shortly
  after a fresh profile starts, when the Gemini-Nano-powered scam-detection
  feature is active). It is not a pure query.

### `create()` is gated on real user activation when a download is needed

If `availability()` is `"downloadable"`, the browser requires **sticky user
activation** (a click/tap/keydown) before `create()` will start the download.
Check `navigator.userActivation.isActive`. Progress arrives via a `monitor`
callback:

```js
const session = await LanguageModel.create({
  monitor(m) {
    m.addEventListener('downloadprogress', (e) => {
      // e.loaded is 0..1
      console.log(`Downloaded ${(e.loaded * 100).toFixed(0)}%`);
    });
  },
});
```

### The real minimum spec (and it excludes a lot of real laptops)

Pinned from the hardware-requirements section (identical across all
foundation-model API pages, read 2026-07-08):

- **Storage:** ≥ **22 GB free** on the volume holding the Chrome profile. If free
  space later drops below **10 GB**, the model is **auto-deleted** and re-downloads
  only when space recovers.
- **Compute — one of:**
  - **GPU:** strictly **> 4 GB VRAM**, **or**
  - **CPU:** ≥ **16 GB RAM** **and** ≥ **4 CPU cores**.
- **Audio input on the Prompt API additionally requires a GPU** (see §3 — this
  is load-bearing for ActiveLog's voice pipeline).
- **Network:** unmetered connection, **for the initial download only**.
  *"Subsequent use of the model does not require a network connection. No data is
  sent to Google or any third party when using the model."* (This is the
  privacy claim the addendum's §4/§5 caveat leans on, now quoted from the
  source.)
- **Languages (from Chrome 149):** English, Spanish, Japanese, German, French
  for input and output.

**The honest implication:** "≥16 GB RAM **or** >4 GB VRAM, ≥22 GB free disk,
desktop only" excludes a meaningful fraction of real-world hardware — most
phones (all, in fact), older laptops, and low-RAM machines. This is why
`availability()` must be the gate, not an assumption.

### Model size — Chrome deliberately does not publish an exact byte count

The docs say only that built-in models *"should be significantly smaller"* than
the 22 GB eligibility floor and that *"the exact size may vary slightly with
updates,"* directing developers to `chrome://on-device-internals` for the live
number. What the model-management doc *does* pin is the **variant selection**
mechanism (source: "Understand built-in model management in Chrome", last
updated 2025-10-21, read 2026-07-08):

> Chrome estimates the device's GPU performance by running a representative
> shader, then decides to either download a **larger, more capable Gemini Nano
> variant (such as 4B parameters)**, a **smaller, more efficient variant (such as
> 2B parameters)**, or fall back to **CPU-based inference**.

So the model is one of a 2B- or 4B-parameter variant, chosen per-device — i.e.
hundreds of megabytes to low single-digit gigabytes. **Do not put a specific
byte size on a shipped page**; the docs intentionally don't, and it varies by
variant and version. Cite the variant class, not a guessed number.

### The model can disappear mid-session — strengthen the "conditional" framing

This is a fact the addendum didn't have and that materially sharpens its
"⚠️ conditional accelerator, not a strategy" marker (source: same
model-management doc):

- *"The model can be deleted at any time, even mid-session, without regard for
  running prompts. This means an API that was available at the start of a session
  could suddenly become unavailable."*
- Triggers: free-disk-space drop below threshold, enterprise policy disabling the
  feature, or **30 days** without meeting eligibility (which can include API
  usage and device capability).
- After purge it is **not** auto-redownloaded — a new `create()` call is required.
- Updates are **full re-downloads** (not deltas), **hot-swapped** with no
  downtime, though *"it's possible for a prompt running at the exact moment of
  the swap to fail."*

**Consequence for the architecture:** even a successful `availability() ===
'available'` at session start is not a lifetime guarantee for that session. The
on-device path must handle a prompt suddenly throwing because the model was
purged under it — degrade to the Tier B / cloud path on that error, don't crash.
This is the same class of "the local thing you checked for can vanish" honesty
that DeckBoss applied to `WhisperNetworkError` vs `WhisperApiError`.

---

## 3. Real code shape — current, correct method names

Assembled verbatim from the Prompt API doc (last updated 2026-05-19, read
2026-07-08). This is what wiring Tier C / the on-device chat backend should
actually call; do not infer a plausible-looking API from memory.

```js
// 1. Feature-detect the global, not window.ai.
if (!('LanguageModel' in self)) { /* not Chrome-desktop or not new enough */ }

// 2. Check readiness. Pass the SAME options you'll prompt with.
const status = await LanguageModel.availability({
  expectedInputs:  [{ type: 'text', languages: ['en'] }],
  expectedOutputs: [{ type: 'text', languages: ['en'] }],
});
if (status === 'unavailable') { /* fall back to Tier B / BYOK cloud */ }

// 3. Create a session. `monitor` + `downloadprogress` only fire if a
//    download is needed; `signal` destroys the session on abort.
const session = await LanguageModel.create({
  initialPrompts: [
    { role: 'system', content: 'You classify the intent of a log utterance.' },
  ],
  // expectedInputs/expectedOutputs as above.
  monitor(m) { m.addEventListener('downloadprogress', e => setPct(e.loaded)); },
  // signal: controller.signal,   // optional AbortSignal
});

// 4a. Whole-result prompt. Returns a string.
const reply = await session.prompt('Is this a chat turn or a new log entry? ...');

// 4b. Streaming prompt. Returns a ReadableStream<string>.
const stream = session.promptStreaming('Summarize my last three entries...');
for await (const chunk of stream) { render(chunk); }

// 4c. Structured output (JSON Schema) — constrains the response.
const decision = await session.prompt(text, { responseConstraint: { type: 'boolean' } });

// 5. Context management.
await session.append([{ role: 'user', content: 'Here is prior context…' }]); // seed, no generate
const fork = await session.clone();           // branch the conversation
session.contextWindow;   session.contextUsage;            // token budget (ints)
session.addEventListener('contextoverflow', …);           // history got trimmed
// prompt/promptStreaming/clone each also accept { signal } to stop mid-run.

// 6. Tear down.
session.destroy();   // after this, any further call rejects.
```

Other load-bearing details from the same doc:

- **`responseConstraint`** takes a JSON Schema (or regex) and forces structured
  output — directly useful for Tier C intent classification ("reply `true`/`false`
  whether this utterance is addressed to the chat connector"). It consumes
  context window; measure with `session.measureContextUsage({ responseConstraint })`.
- **`contextoverflow`** event + `QuotaExceededError` (with `requested`/`contextWindow`
  fields): the session auto-trims oldest non-system turns on overflow, and throws
  only if it still can't fit.
- **Permission Policy:** cross-origin iframes need `allow="language-model"`.
  **Not available in Web Workers** (Chrome cites the complexity of establishing a
  responsible document per worker). Plan to call it from the main thread or a
  same-origin iframe.
- **TypeScript:** the `@types/dom-chromium-ai` npm package carries current
  typings for all these APIs — use it rather than hand-rolling declarations.
- **Multimodal inputs (text / image / audio).** This is a *new* capability the
  architecture addendum's §5 did not fully account for: the **on-device** Prompt
  API itself accepts audio (`AudioBuffer`, `Blob`, `ArrayBuffer`, `ArrayBufferView`)
  and image (`HTMLImageElement`, `ImageData`, `Blob`, …) parts in a message —
  not just text. **Audio input requires a GPU** (per the hardware section). This
  means the "single-call audio→answer" path the addendum attributed only to
  *cloud* Gemini native multimodal could *also* run fully on-device, on a
  GPU-capable Chrome-desktop machine, via the Prompt API — a genuinely local
  alternative to both cloud Web Speech (transcription) and cloud BYOK Whisper.
  Mark this 🔮 Phase 2 and verify audio support per-device with
  `availability({ expectedInputs: [{type:'audio'}] })` before relying on it.

---

## 4. The Summarizer / Writer / Rewriter / Proofreader APIs — are they real?

**Yes — all four are real, separate, Chrome-shipped APIs**, not generic labels
for "some LLM writing help." They are part of Chrome's "built-in AI" family, each
with its own global class, its own explainer, and its own status. The honest
question is *which are stable* and *what each is shaped for*. (Source: "Built-in
AI APIs" status table, last updated 2025-09-12; "Writer API" page last updated
2025-10-22; both read 2026-07-08.)

| API | Global | Shape | Status today |
|---|---|---|---|
| **Summarizer** | `Summarizer` | `create({type,format,length,sharedContext})` → `summarize()` / `summarizeStreaming()`; **text→text** | ✅ **Stable, Chrome 138** (web + ext) |
| **Writer** | `Writer` | `create({tone,format,length,sharedContext,…})` → `write()` / `writeStreaming()`; **text→text** | ⚠️ **Developer Trial (origin trial)**, Chrome 137–148 |
| **Rewriter** | `Rewriter` | revise/resize/retone existing text | ⚠️ **Developer Trial (origin trial)**, joint OT with Writer |
| **Proofreader** | `Proofreader` | interactive grammar/style correction; uses **LoRA weights** on top of the base model | ⚠️ **Developer Trial (origin trial)** |

They share the same model-download lifecycle and hardware floor as the Prompt
API (§2). `Writer.create()` even takes the same `monitor` / `downloadprogress`
shape. The `Writer` options are literally `tone: 'formal'|'neutral'|'casual'`,
`format: 'markdown'|'plain-text'`, `length: 'short'|'medium'|'long'`, plus an
optional `sharedContext` and language hints.

### The "Writer API" question — honest read for the "captain writes a book/podcast" idea

The org's planning elsewhere floats "the captain could write a book or podcast
via conversation with the write api." Mapping that onto Chrome's actual Writer
API:

- **It is a real, Chrome-specific API** (`Writer` global, part of the
  "Writing Assistance APIs" explainer from `explainers-by-googlers`). So it is
  *not* just a generic "any LLM writing assistant" reference — there is a
  concrete surface to map to.
- **But it is origin-trial-gated today**, not stable. A production page depends
  on registering for the Writer/Rewriter joint origin trial and shipping a token;
  without it the API is absent. Do not build a load-bearing feature on it for a
  page real users visit this year without an OT token **and** a fallback.
- **And it is task-shaped for short-form drafting**, not long-form generation.
  The option set (`tone`, `format`, `length: short|medium|long`, `sharedContext`)
  and the doc's own use cases ("write a review / blog post / email / support
  request / draft intro") describe an *assistant that drafts a discrete piece of
  writing from a prompt plus optional context*, reusing a `sharedContext` across
  multiple outputs. It is not engineered to produce a book-length coherent
  narrative in one call, and its `length` tops out at "long" (a relative, not
  book-scale, setting).
- **For long-form / flexible creative generation, the on-device Prompt API is the
  better map** — same Gemini Nano model, no origin trial (stable 148), no
  task-shape constraints, with streaming and structured output. If the idea is
  "captain converses with an on-device model to draft a podcast script," route it
  at the Prompt API, not the Writer API. The Writer API is the right tool for
  "rewrite this paragraph more formally" or "draft a short post from this idea" —
  short, discrete, tone-shaped artifacts.

**Bottom line on the idea:** there *is* a real Chrome Writer API, so the
reference isn't vapor — but "write a book via the Writer API" conflates a
short-form drafting assistant (origin-trial-gated) with long-form generation, and
should be reframed as either "Prompt-API-driven long-form drafting" (on-device,
stable, flexible) or "Writer-API-assisted short-form polishing" (origin trial,
constrained), depending on which job is actually meant.

---

## 5. Verifying and deepening the architecture addendum

Point-by-point check of the claims in
[`../ACTIVELOG_ARCHITECTURE.md`](../ACTIVELOG_ARCHITECTURE.md) §4 ("Chrome's
on-device Prompt API") and §5 ("on-device escape hatch"), now against live docs:

| Addendum claim | Verified? | Note / correction |
|---|---|---|
| "Chromium-only (Chrome/Edge desktop)" | ✅ correct | Chrome confirmed; Edge is Chromium but **not separately documented** — say "Chrome" unless Edge is verified. Mobile excluded. |
| "origin-trial / flag-gated" | ⚠️ **now outdated for the base API** | Base Prompt API on the web **shipped stable in Chrome 148** (Intent to Ship). Flags are **localhost-dev only**. Only `topK`/`temperature` tuning remains origin-trial-gated. See §1 — this is the headline correction. |
| "requires the browser to have downloaded the on-device model once" | ✅ correct | Plus the §2 hardening: the model can be **purged mid-session**, so "downloaded once" is not permanent. |
| "not present in Safari, Firefox, or mobile Chrome in the same form" | ✅ correct | Confirmed: Android/iOS/non-Chromebook-Plus ChromeOS unsupported; Safari/Firefox are WICG incubations only. |
| "Reached through Chrome's built-in AI runtime" / "Gemini Nano" | ✅ correct | Global `LanguageModel` class; Gemini Nano, 2B- or 4B-param variant auto-selected. |
| "verify the exact JS entry point" | ✅ now pinned | Global `LanguageModel`, **not `window.ai`**. Full surface in §3. |
| "verify the current enablement surface (origin-trial token vs flag vs GA)" | ✅ now pinned | GA stable 148 web / 138 ext; flag = localhost only; OT = sampling params + Writer/Rewriter/Proofreader. |
| "on-device Prompt API is local; Chrome's default `SpeechRecognition` is not" | ✅ correct & worth repeating | Prompt API: *"No data is sent to Google or any third party when using the model."* Default `SpeechRecognition` ships audio to a Google recognizer (MDN). Opposite privacy footprints — never one "Chrome AI" toggle. |

**New facts the addendum doesn't yet carry** (candidates to fold in when it's
next revised):

1. **Mid-session model purge** (§2) — even an `"available"` session can lose its
   model under disk pressure. The on-device path must degrade gracefully mid-prompt.
2. **On-device multimodal audio input** (§3) — the Prompt API itself takes audio
   (on GPU machines), so the "single-call audio→answer" path the addendum framed
   as *cloud-Gemini-only* also has a **fully-local** variant. 🔮 Phase 2, gated on
   `availability({expectedInputs:[{type:'audio'}]})`.
3. **Summarizer is stable since Chrome 138** (§4) — the lowest-risk on-device
   capability for ActiveLog (transcript summarization) is more mature than the
   Prompt API on the web.

None of these contradict the addendum's architecture decisions; they make its
"⚠️ conditional" and "🔮 Phase 2" markers more precise, which is exactly the
discipline that doc is written in.

---

## 6. What's safe to build against today vs. not

Stated so a page real users will visit this year can be planned against it:

**✅ Safe to build against as a progressive enhancement / fallback:**
- The **Prompt API** (`LanguageModel`) with **default parameters**, behind a
  `LanguageModel.availability()` feature-detect that falls back to Tier B
  (exact-match) / BYOK cloud on any non-`"available"` state. Chrome ≥148 desktop
  only. Plan for mid-session purge (§2).
- The **Summarizer API** (`Summarizer`) for transcript/entry summarization —
  stable since Chrome 138, the most mature of the set.

**⚠️ Real but conditional — needs an origin-trial token before production
reliance, and a fallback:**
- **Writer / Rewriter / Proofreader** APIs (Developer Trial, Chrome 137–148).
  Fine to experiment; not safe as a load-bearing feature without an OT token and
  a non-on-device fallback for everyone else.
- Prompt API **sampling-parameter tuning** (`topK`/`temperature`) — same, OT-gated.

**❌ Not safe to assume on a public page at all:**
- That the on-device model is *present* on a given visitor's machine. The
  hardware floor (§2) excludes most phones and many laptops; `availability()`
  is the only honest gate, and even a passing check can be invalidated by purge.
- That any of this works on **mobile, Safari, or Firefox.** It does not.
- Any specific **model byte size** or **model version** — Chrome pins neither and
  says to read `chrome://on-device-internals`.

**The shape this implies for ActiveLog is already the shape the architecture doc
specifies:** prompt-capable-when-present, exact-match-when-not, behind one
`matchWakePhrase` seam — and for the on-device *chat backend* (§5 escape hatch),
the same: wire `LanguageModel` as a zero-cost connector that registers only when
`availability()` returns `"available"`, degrade to BYOK otherwise, and always
record `key_owner: "on-device"` in the `chat.exchange` event. The research here
turns the addendum's "verify before wiring" TODOs into concrete method names and
one corrected gating fact; it does not change the architecture.

---

## Sources (all read 2026-07-08; "last updated" as published by Chrome)

- **The Prompt API** — `https://developer.chrome.com/docs/ai/prompt-api` (last
  updated 2026-05-19). Namespace `LanguageModel`, `create`/`prompt`/`promptStreaming`,
  multimodal inputs, `responseConstraint`, session management, Permission Policy.
- **Get started with built-in AI** — `https://developer.chrome.com/docs/ai/get-started`
  (last updated 2025-05-20). The four `availability()` return values, user
  activation, localhost flags, hardware requirements.
- **Built-in AI APIs (status overview)** —
  `https://developer.chrome.com/docs/ai/built-in-apis` (last updated 2025-09-12).
  The per-API status table (stable vs Developer Trial), Chrome versions, Intent
  links.
- **Understand built-in model management in Chrome** —
  `https://developer.chrome.com/docs/ai/understand-built-in-model-management`
  (last updated 2025-10-21). 2B/4B variant selection, mid-session purge, hot-swap
  updates, no JS model-version query.
- **Writer API** — `https://developer.chrome.com/docs/ai/writer-api` (last
  updated 2025-10-22). `Writer.create({tone,format,length,…})`, `write`/
  `writeStreaming`, origin-trial Chrome 137–148, joint with Rewriter.
- **Built-in AI (landing)** — `https://developer.chrome.com/docs/ai/built-in`.
  API inventory and Early Preview Program context.
- Standards track: W3C Web Incubator Community Group (WICG) proposals —
  `https://github.com/webmachinelearning/prompt-api` (Prompt API explainer) and
  `https://github.com/explainers-by-googlers/writing-assistance-apis/`
  (Writer/Rewriter). TypeScript typings: `@types/dom-chromium-ai` on npm.

*Research only. No code changed. Re-verify the cited pages before any of this
lands on a shipped page — see the staleness warning at the top.*
