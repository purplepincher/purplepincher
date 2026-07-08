<!--
  Editorial note (Fable, 2026-07-05): "The sharing chapter" section below
  (after the watchlist) is a direction placed on record, not a roadmap
  step. It lives in this file, and not only in docs/PARADIGM.md, because
  its load-bearing content is a decision-record distinction — why it is
  not the fleet learning deckboss's own ROADMAP.md rejected — and
  distinctions between decisions belong where decisions are recorded. The
  essay carries the same idea in its own register.
-->

# PurplePincher Roadmap

This is the fuller decision record the [README](./README.md) points to. The
README states the org's operating principles and the adoption bar once, and
gives a compressed, five-line version of what's coming next. This document
does not repeat that reasoning — it expands on it: the full argument behind
each step, the things that were seriously considered and rejected, and the
specific evidence each call rests on.

If you're an AI agent picking up work in this org: read this document before
proposing a new fork, before assuming a SuperInstance repo is "basically
ready," or before treating anything in the *watchlist* or *will not fork*
sections below as settled forever rather than settled *for now, given what we
know today*. See [How this roadmap gets updated](#how-this-roadmap-gets-updated)
at the bottom — it is not optional reading.

## Recap: operating principles and the adoption bar

(Full versions live in the README; restated briefly here because every step
below is an application of them.)

**Principles:** ship first, announce second; one domain at a time; one
toolkit, not a pile of repos; honest scale, honest docs; ruthless scope.

**Adoption bar** — every step below, before it counts as "done":

1. Verified working, end-to-end (not "has tests" — SuperInstance CI has been
   caught running `pytest || true`; that pattern is banned here).
2. Docs rewritten to match reality — every claimed feature exists, every
   example runs.
3. Ideology stripped to optional — no tool in this org may *require* the
   γ+η=C "conservation law" framing, Pythagorean48 trust vectors, or any of
   the sketchbook's speculative mathematics to function or to be understood.
   Where the underlying math is real (Shannon entropy, PID control), it's
   presented as what it is.
4. Published — crates.io, PyPI, or npm, under a name the README actually
   uses.
5. Scoped — a stated boundary of what it will not do.
6. **Two-layer intake, before any of the above is even attempted.**
   Adopted from a strategic review of this mission's own research
   (`docs/research/FABLE_SYNTHESIS.md`), after noticing that the one
   trait separating the sketchbook's few genuinely real artifacts from
   its many oversold ones was whether the author subjected the work to
   adversarial review:
   - **Layer one — mechanical, free, applied to everything**: does CI
     actually compile and pass unmasked tests (not `pytest || true`);
     count `Qed` vs `Admitted` on any raw `.v` Coq files (reject if
     markdown-wrapped so it can't even compile); does the benchmark
     code measure something real, or is it a `print()` of hardcoded
     numbers? These three checks alone found ~95% of the "docs oversell
     code" cases across this entire mission's research — apply them
     first, before spending anything else.
   - **Layer two — published multi-model adversarial review, only on
     survivors**: run the shortlist that passes layer one past a small
     panel of cheap models with an honest "what's wrong with this
     code" prompt, and publish whatever they find in the fork's own
     README as a condition of graduation — not buried, not softened.
     This is cheap (the one time this mission measured it, four models
     found two real bugs for about $0.50) and it's the practice that
     makes `intent-directed-compilation` the strongest research
     artifact this mission found in ~15 examined "papers" repos.

---

## The roadmap steps

Ordered. Each step gates the next. Dates are deliberately absent — the
sequence is the commitment, not a schedule.

### Step 0 — Reconcile the two DeckBosses

**Status: done.** The diff (`deckboss-family-diff.md`) landed, and it
overturned the assumption this step was originally written on — the finding
below supersedes the "9-repo prototype family" framing.

**What the diff actually found.** SuperInstance's `deckboss-*` name is a
**false-friend cluster, not a prototype lineage.** Of the 11 repos matching
`deckboss`/`cocapn-foundation`, most have nothing to do with fishing:
`deckboss-1`/`deckboss-ai`/`deckboss-hardware`/`deckboss-fab`/
`deckboss-marketplace` are literal forks of `Lucineer/deckboss*`, a generic
Jetson/RPi onboarding CLI — the "deckboss" name originates there, not from a
fishing product. `deckboss-ai` turned out to be an unrelated AI spreadsheet
engine; `DeckBoss` (capitalized) is an unrelated agent-orchestration
platform. `deckboss-net`/`deckboss-agent`/the `-pages` repos are marketing
pages and toy demos with zero real logic (`deckboss-net`'s "reliable
messaging" README's only code is `fn main() { println!(...) }`).

**The real design ancestor is `cocapn-foundation`** — confirmed by direct
evidence, not inference. `purplepincher/deckboss`'s own README cites it by
name as the sibling repo holding `SAFETY.md`. `log-entry.ts`'s doc comment
directly quotes `activelog-spec`'s "No editing. Ever." principle, verbatim.
`purplepincher/deckboss` shares no git history with anything in
SuperInstance — it's an independent, from-scratch codebase that knowingly
and deliberately implements a scoped-down version of `cocapn-foundation`'s
design, not a coincidental namesake and not a fork of the `deckboss-*`
family.

**What's actually worth absorbing** (real, specific, not currently in
DeckBoss): the Helm ESP32 hardware/safety layer (`cocapn-foundation/
SAFETY.md`'s five-layer defense model — electrical, firmware watchdog +
500ms TTL, protocol replay-protection, voice command-class escalation, sea
trial checklist); the device-profiles-as-data pattern (new device support
ships as a reviewable JSON file, never a firmware fork); timestamp-based
media anchoring (camera frames linked to speech by overlapping timestamps,
no foreign key needed); structured `catch.assertion` parsing with a
`parse_conf` field; per-device hash chains for tamper-evidence (currently
irrelevant given DeckBoss's explicit non-evidentiary stance, but cheap to
bank); BYOK chat with log-query tools as a v2 path for Ask-Your-Log; the
T0-T3 connectivity-tier framework as a testing/prioritization checklist.

**What's already superseded** — DeckBoss solved these better than anything
on the SuperInstance/Lucineer side: a *working* sync implementation (nothing
over there has one at all); tested additive-corrections semantics with a
CODEOWNERS-locked invariant enforcer; real storage-BYOK (a different,
harder, more important problem than the LLM-key `byok.ts` boilerplate
shared across the Lucineer siblings); and, named explicitly as the single
biggest quality gap between the two sides, **honesty about maturity** —
every SuperInstance/Lucineer sibling oversells its README relative to its
code, while DeckBoss's README is unusually calibrated the other direction.

**Concrete actions, in priority order:**

1. Mirror `cocapn-foundation`'s `activelog-spec` schema and `SAFETY.md`'s
   five-layer model into `purplepincher/deckboss/docs/` directly. DeckBoss's
   safety story currently cites a "Claude Design handoff bundle" repo whose
   own README says "these are prototypes, not production code" — a fragile
   foundation for a citation this load-bearing.
2. Archive `deckboss-net`, `deckboss-agent`, `deckboss-ai-pages`,
   `deckboss-net-pages` — marketing pages and toy demos, nothing to carry
   forward. Archive `deckboss-1` too (stale fork, its "character sheet"
   hardware-profiling idea is speculative and low-value for a software-only
   PWA).
3. Add a one-line disambiguation to `deckboss-ai` and `DeckBoss`
   (capitalized) — real, different products wearing the same name by
   coincidence; anyone browsing SuperInstance by name would otherwise
   reasonably assume they're related to `purplepincher/deckboss`, and they
   are not.
4. Relabel (don't archive) `deckboss-hardware`/`marketplace`/`fab` as
   "hardware roadmap, not active product" — they're real, working code
   (a vendor-tier marketplace calculator, a parametric OpenSCAD case
   generator) for a future Helm/Jetson hardware line, just not applicable to
   today's software-only PWA.

**All four executed, 2026-07-08.** Item 1 was already done in an earlier
pass (`purplepincher/deckboss/docs/cocapn-foundation-mirror/` contains
`SAFETY.md`, `activelog-spec/`, and its own `README.md`). Items 2–4 were
decided here but left unexecuted until now: all five repos in item 2 are
archived (verified via the GitHub API, `archived: true` on each); the
disambiguation line from item 3 is live on both `deckboss-ai` and
`DeckBoss`'s repo descriptions; the three hardware repos carry the
"[Hardware roadmap, not active product]" prefix from item 4. Step 0 has
no remaining open items.

Full detail, evidence, and citations: [`docs/research/deckboss-family-diff.md`](./docs/research/deckboss-family-diff.md).

### Step 1 — DeckBoss field beta (the org's center of gravity)

This step remains the org's center of gravity. The original plan gated
significant new forks on real fishermen having used DeckBoss on real
boats; `pincher` was forked and hardened before that gate was reached
and is now recorded as a graduated repo. The beta is still the next
scheduled milestone for the org's field-tested product, DeckBoss.

**Why this still matters.** The industry pattern this org is explicitly
modeling itself on (see the README's Tokio/Kubernetes/HashiCorp
references) is unambiguous: credibility comes from one thing working in
production before a second thing is announced. A field beta on working
vessels — not a demo, not a closed alpha, actual fishermen logging
actual trips on actual boats for 6–8 weeks — is this org's equivalent of
production traffic. Expanding the storefront before the flagship's core
value proposition is proven in the field would violate principle 2
("one domain at a time").

**What "gates" means now:** Step 1 still drives what gets serious
engineering time next. Steps 3 and 4 do not get serious engineering
time until Step 1's findings have been digested and, if necessary,
folded back into DeckBoss itself.

**What "the gate opened" actually means (added 2026-07-08).** Three
independent reviewers (aider, mmx/MiniMax, and a Fable synthesis pass)
converged on the same gap in this step: the roadmap treats gate
digestion as something that happens *to* the org while infrastructure
waits, but never defines what passing looks like. "Findings have been
digested" is not checkable as written. This section makes it checkable.
The numeric thresholds below are stated judgment calls, not derived
facts — no evidence exists yet about how Sitka captains respond to
outreach, because zero have been contacted. They are set here so the
gate has a definition that reality can correct, rather than no
definition at all; if the beta's actual numbers argue for different
thresholds, update this section with the evidence, per
[How this roadmap gets updated](#how-this-roadmap-gets-updated).

The gate has three stages, each independently checkable:

1. **Recruited.** At least 3 working boats committed (the floor of this
   file's own "3–5 real boats" measure of success), of which at least 2
   are not the project owner's own vessel. F/V Eileen counts as a boat;
   it cannot count as the signal. A founder using his own product
   proves willingness, not adoption — the beta exists to test whether
   the voice-first bet lands with captains who have no stake in it
   being true. Each ask gets logged (name, date, channel) the day it's
   sent; the fallback clock below runs off that log.
2. **The beta produced signal.** After the 6–8 week window (or earlier,
   if it fails early — a failure is also signal), measured only through
   channels that keep the no-backend promise intact (daily check-in
   notes, the on-device diagnostics export, and the captain's own
   storage as witness — no telemetry exists and none gets added for
   this): retention (at least 2 non-owner captains still logging
   unprompted in the final two weeks), capture-in-conditions (each
   active captain has at least 5 entries recorded in genuinely loud or
   wet conditions, since that's the product's core hypothesis), a
   completed week-one parallel-log ground-truth comparison per captain,
   and verified sync to storage each captain owns.
3. **Digested.** One findings document, committed to
   `purplepincher/deckboss/docs/field-beta-findings.md`, answering
   three named questions with evidence: does voice capture survive real
   deck conditions; does sync-to-own-storage work unattended for people
   who didn't build it; do captains come back unprompted after the
   novelty wears off. The gate is open on the day that document lands
   **and** a dated decision note re-prioritizing Steps 3 and 4 is added
   here in light of it. Not before.

**A negative result opens the gate too — differently.** If captains are
recruited, onboarded, and then stop using it for reasons intrinsic to
voice capture (hands wet, won't talk to a phone mid-haul, transcript
not trusted) rather than fixable bugs, that is a real answer to the
question the beta asked. It gets written up in the same findings
document, and what it unblocks is a reconsideration of the voice-first
bet — not Steps 3 and 4, which exist to serve a product whose thesis
would then be in question. Distinguish this carefully from recruiting
failure, which is a channel-and-message problem and says nothing about
the product.

**The fallback clock**, so outreach going quiet is a scheduled decision
instead of an indefinite stall: at **T+14 days** from the first send,
fewer than 2 replies from roughly 10 direct asks means switch channel,
not just message — the written drafts become the leave-behind, and the
primary channel becomes the in-person dock conversation. At **T+45
days**, fewer than 3 committed boats means run the beta at reduced
scope with whoever committed (F/V Eileen included) rather than letting
the gate block the org indefinitely — record the downgrade here, dated,
and the findings document must say plainly that the signal is weaker
for it.

### Step 2 — `pincher` (the second tool, and the namesake)

**Status: forked and hardened; not fully done.** The original plan made
this the first fork after the DeckBoss field beta was underway; that
sequencing gate was passed before the beta actually began, and the
record here notes that without re-litigating why. What's real: a
structured, hardened Rust project with a genuinely novel architecture
(below). What isn't yet: adoption-bar item 4 (published) is unmet —
verified against crates.io's own API, neither `pincher` nor
`pincher-core` exists there, so `cargo install pincher` still does not
work despite being one of this org's own stated measures of success.
The README is honest about this. Publishing is a ~15-minute action
gated only on the project owner's crates.io credentials — the readiness
checklist (`docs/CRATES_IO_READINESS.md` in the `pincher` repo) is
already done. This is currently the single cheapest high-value action
available to the owner anywhere in this org.

**Confirmed for real, 2026-07-08**, not just claimed: the readiness
checklist's earlier gaps (a `cargo fmt` failure, an unresolved
`pincher-cli`-vs-`pincher` package-naming question, and a dry-run
publish that had never actually been runnable) are now closed — a
toolchain became available, `cargo fmt` was fixed, the CLI package was
renamed to `pincher` to match the README's own `cargo install pincher`
claim, and `cargo publish --dry-run` passes on both publishable crates
(commit `e84b13a`, CI green). See `docs/FINISH_THIS_PHASE.md` for the
exact remaining commands — there is nothing else blocking this but
Casey running them.

One more item this step's own "done" status previously glossed over:
the original plan called for "replace the philosophy-heavy README with
a 60-second demo." That did not fully happen — the crates.io claim was
corrected, the SuperInstance-sketchbook links and the design-system
section were removed (real work, genuinely done), but the README's
opening still carries the hermit-crab/"cortex teaches the spinal cord"
framing, including the exact "that's not cache, that's understanding"
line. Decision, on the record rather than left as a silent gap: **the
philosophy stays, deliberately.** It's original writing specific to
this crate (not sketchbook material), it doesn't make any unverifiable
claim, and a rewrite would spend real effort for a stylistic
preference rather than an honesty problem — unlike the crates.io gap
above, this isn't blocking anything. Revisit only if a future reader
finds it actually gets in the way of understanding what pincher does.

**Why pincher, specifically, earned second place:**

- **It's real.** A structured Rust project with benchmarks, a full doc set,
  and a genuinely novel architecture: embed intents locally, fire known
  reflexes in under 50ms with no LLM call, confirm semi-known ones, compile
  new reflexes via the LLM only on a miss, all behind a veto/sandbox layer.
  This is not a sketch — it has the shape of production software already.
- **It's on-mission, not just adjacent.** A reflex layer that works without a
  model call is exactly what edge devices with no connectivity need — the
  same "zero signal is the normal case" thesis DeckBoss already shipped on
  (offline-first, local-first, sync-when-available). There is a concrete,
  plausible future where pincher literally becomes DeckBoss's on-device
  voice-command layer. That makes this fork *compound* — it strengthens the
  existing product — rather than *diversify* into an unrelated second
  product line. Diversifying this early would violate principle 2.
- **It's the identity.** The org is named after this tool's reflex engine and
  its hermit-crab metaphor. Shipping the namesake as the second tool makes
  the name mean something earned rather than aspirational.
- **The gaps to close are known, finite, and small** — this is not a
  ground-up rebuild:
  1. Publish to crates.io. The sketchbook's own roadmap already claims
     `cargo install pincher` works. Making that claim literally true is the
     entire point of graduation into this org (adoption-bar item 4).
  2. Replace the philosophy-heavy README with a 60-second demo (adoption-bar
     item 2 — docs matching reality, item 3 — ideology stripped to optional).
  3. Extract the veto/sandbox engine as a pluggable policy trait, so it's
     usable independent of pincher's specific reflex set.
  4. Write the `.nail` portable-agent-state format down as an actual spec,
     not just an implicit convention in the code.

### Step 3 — The fleet edge tier (the first infrastructure play)

**Status: partially executed.** The Vectorize-backed semantic-search
Worker has already graduated as `plato-semantic-search`, and the Worker
fleet-health scanner has already graduated as `vessel-tuner`. Both are
now listed in the README's infrastructure tier; the rest of this
cluster remains future work.

This is the org's first step outside the flagship product and into
shared infrastructure. Some of its pieces graduated ahead of the
remaining `fleet-*` work, which is why the status above is split.

**Source cluster:** `fleet-*` is 319 repos, of which roughly 270 are chaff —
92 of them alone are a separate `fleet-midi-*` music-generation sub-project,
64 of which are five-kilobyte stubs. But the surviving core (~40–50 repos)
is, per the dedicated deep-dive, **the single strongest "actually real"
finding in the entire sketchbook**: a partially *live* distributed edge
system, not just a shared naming prefix. Concretely, as verified by direct
HTTP probe:

- **Three Cloudflare Workers healthy and returning 200**: `fleet-vector-api`
  (Vectorize-backed semantic search), `fleet-registry-worker` (KV-backed
  agent registry), `fleet-dashboard-api` (D1-backed telemetry).
- **Three more deployed but broken in small, concrete, named ways**:
  `fleet-edge-worker` (404 on its root route only — the rest of the design is
  real, a capacity-gated WorkItem/WorkResult pattern), `fleet-budget` and
  `fleet-metrics-cron` (both 500, most likely D1 binding/provisioning gaps,
  not design failures).
- **A transport-agnostic agent-messaging protocol** — I2I "bottles," modeled
  on SMTP/email headers, speech-act aware (inform/request/promise), with
  working Python (`fleet-protocol`) and Rust (`fleet-i2i-protocol`)
  implementations that reference each other and are actually used by two
  named agents (Forgemaster and Oracle2) to coordinate builds.
- **An ARM node self-orchestrating under systemd**: `fleet-oracle2`, an
  Oracle Cloud ARM64 free-tier box running 15 real systemd units (services
  and timers) on a 5-minute cron pulse — described in the deep-dive as "the
  most alive repo in the cluster."

**The fork here is selective, in two pieces, not a wholesale adoption:**

- **3a. The Workers edge tier**, packaged as one coherent toolset. The
  Vectorize-backed semantic-search piece and the fleet-health scanner
  have already graduated as `plato-semantic-search` and `vessel-tuner`;
  this item now covers the remaining `fleet-*` Workers. Adopt the
  remaining healthy workers as-is. Fix the three broken ones — these are
  small, named repairs (a missing root route; two D1 binding failures),
  not redesigns. Add monitoring so "broken" gets caught before a user
  finds it, not after. And drop the "planet-scale" framing the
  sketchbook uses for what is, honestly, a five-vessel home-lab fleet
  (one of the five offline) — present it as what it actually is: a
  reference deployment a user can stand up on their own Cloudflare
  account. This choice also aligns with DeckBoss's existing
  BYO-Cloudflare-R2 sync story, so it extends a storage pattern this
  org's users already have rather than opening an unrelated new front.
- **3b. The messaging protocol** (`fleet-i2i-protocol` + `fleet-protocol`),
  extracted as a versioned, standalone spec with the conservation-law
  decoration removed, published to PyPI (Python side is not currently
  published) and crates.io, with an interop test suite proving the Python
  and Rust implementations actually speak the same wire format to each
  other. A human-readable, transport-agnostic, debuggable agent-messaging
  format is arguably the single most reusable idea anywhere in the 4,095-repo
  sketchbook — most of what's there is domain-specific; this is
  domain-agnostic plumbing.

**Explicitly *not* forked from this cluster, and why each is excluded:**

- **`fleet-coordinate`** — real (a genuine "Zero Holonomy Consensus" design,
  agents agree via independent geometric projection rather than voting), but
  369 MB and load-bearingly coupled to Eisenstein-lattice consensus math and
  a `holonomy-consensus` dependency. The decoupling cost exceeds the value it
  would deliver today. Revisit if a future product actually needs
  leaderless consensus.
- **`fleet-manifest`** — the decentralized-registry idea itself is good ("no
  central registry, every agent has a copy"). But trust is currently encoded
  as Pythagorean48 vectors, and subgroup self-coordination is decided by
  Laman rigidity (E=2V−3) — eccentric math with no operational advantage
  proven over standard gossip-protocol trust primitives. If this org ever
  builds a registry like this, the plan is to replace that math with
  standard gossip primitives, not adopt it as designed.
- **All 92 `fleet-midi-*` repos**, the seven language packs (arabic,
  chinese, finnish, japanese, latin, navajo, sanskrit — "reason natively in
  X" experiments), and the roughly fifteen organism-metaphor repos
  (`consciousness`, `immune`, `nervous-system`, `biosphere`, etc.) — separate
  concerns entirely, out of scope for an edge-development toolkit regardless
  of maturity.

### Step 4 — `cocapn` (the agent framework, if and when DeckBoss needs it)

`cocapn` is the sketchbook's most externally-engaged repo — actually
published on PyPI already, unlike almost everything else in scope — with a
credible multi-target story spanning Python, Zig (bare-metal), and WASM.

**Known gaps, all closeable — but one is now known to be more serious than
it looked.** A full 77-repo family deep-dive
([`docs/research/cocapn-family-deep-dive.md`](./docs/research/cocapn-family-deep-dive.md))
found the PyPI-published `cocapn` 0.3.0 **does not actually match**
`SuperInstance/cocapn`'s source, and **three separate repos**
(`cocapn`, `cocapn-py`, `cocapn-python`) all claim the same package name.
This has to be resolved before anything else in this step, not treated
as one gap among several — but the resolution isn't "pick which of the
three repos owns the name." A follow-up tarball audit
([`docs/research/cocapn-pypi-tarball-audit.md`](./docs/research/cocapn-pypi-tarball-audit.md),
2026-07-06) downloaded the actual published 0.3.0 sdist and compared it
directly against all three repos: it matches **none of them** — no
file-hash, class-name, or dependency overlap, and the published
tarball's own changelog cites commits that don't exist in any of the
three. The published package is orphaned, not owned by a knowable
source repo — the audit's own §7 notes the live artifact may come from
a source outside all three named repos entirely. The decision this step
is actually gated on starts with a different first question than "which
repo owns the name": who currently holds the PyPI upload credentials
for the `cocapn` project (checkable via PyPI's own collaborator
management), since that fact controls what happens next independent of
which repo is "morally" the source. The audit's §7 lays out the
standard resolution paths once that's known — consolidate under one
repo, transfer ownership via PyPI, or, if no maintainer can be
identified, ask PyPI support to mediate a dormant-project name dispute. This correction folds the audit's finding
forward per this document's own rule that new investigation results get
folded in rather than left stale next to a newer, disagreeing note. The
other gaps still stand: publish the CLI the docs already
reference but that doesn't yet exist (and note `cocapn-cli` on crates.io
is a theming library, not the referenced CLI, per the same deep-dive);
split the sprawling repo family into a documented core plus optional
extras; add real persistence beyond flat JSONL; get honest test-coverage
numbers rather than claimed ones.

**Also newly found**: a coherent, genuinely real Rust/C/Zig/WASM
bare-metal tier underneath the Python core (deadband/PID/NMEA/device
primitives — `cocapn-marine`, `cocapn-core`, `cocapn-c`, `cocapn-zig`,
`cocapn-wasm`), several with real test coverage (45+ passing tests on
the C port). This strengthens the case for eventually adopting this
family — there's more real, working substance here than the original
survey credited — but the namespace mess above has to be untangled
first, or purplepincher would inherit a naming collision on day one of
this step.

**Why step 4 and not step 2, deliberately.** `cocapn` is an agent framework,
and this org's rule is that it adopts an agent framework when a shipped
product actually pulls it in — most plausibly DeckBoss's Ask-Your-Log
feature, or a pincher integration once pincher exists — not because the
framework exists and looks credible in isolation. Frameworks adopted ahead
of a concrete need are exactly how orgs sprawl; this is principle 5 (ruthless
scope) applied to the one candidate in this roadmap that would be easiest to
justify adopting early on pure technical merit. The gate isn't "is cocapn
good enough" (it may already be) — the gate is "does a shipped PurplePincher
product currently need it," and today none does.

Related note: the broader technical-fit investigation into cocapn-foundation
found that the closest concrete embedded/hardware assets in the sketchbook
(`plato-vessel-core`, a real ESP32/RP2040 C client with an explicit cocapn
tie) implement a completely different data model (PLATO tiles) from what
cocapn-foundation actually needs (ActiveLog events), and that most of the
other candidate repos (`Edge-Native`, `openmind-esp32-bridge`,
`ESP32-Plane-Radar`, `plato-serialize`, `grand-pattern-embedded`) are naming
coincidences or parallel-universe designs with no real cocapn tie at all.
This reinforces rather than changes Step 4's placement: the framework side
(`cocapn` proper) is closer to adoptable than the hardware side
(`cocapn-foundation`'s vessel/helm vision), and the two should not be
conflated when this step is eventually executed.

---

## A rule this org actually followed, stated so the record matches reality

An earlier planning pass (`docs/FABLE_NEXT_MISSIONS_PLAN.md`) explicitly
declined to fork `plato-semantic-search`, `plato-engine-block-c`,
`exocortex-mcp-ts`, `vessel-tuner`, and `git-native-agents` on the stated
principle that "adoption waits for a pull" — a shipped product needing
the capability, not the capability merely clearing the quality bar. All
five graduated anyway, alongside `sonar-vision`, with no decision on
record superseding that plan. The forks are real, hardened, and not in
question — this isn't a relitigation. But the *rule* the org actually
runs on needs to be stated honestly, because Step 3 and Step 4's gating
logic depends on which one is true: **the adoption bar (verified working,
honest docs, stripped ideology, published, scoped, two-layer intake) is
the real gate.** A pull from a shipped product is a *reason* to prioritize
a fork sooner, not a *precondition* for forking it at all. Say this
plainly rather than let a written rule and an actual practice quietly
diverge — a roadmap that says one thing while the org does another is
exactly the failure mode [How this roadmap gets updated](#how-this-roadmap-gets-updated)
exists to prevent.

---

## Step 5 — ActiveLog: one shared core, then skins (added 2026-07-08)

**Status: not started; deliberately sequenced.** Casey laid out a real,
large vision: a shared voice-capture-and-storage core — pick a storage
location (local folder, the user's own Cloudflare account, Google
Drive, a database, an Oracle instance), activate the mic, dictate into
a structured log, organize it via pause/wake-word detection, add a
wake-word chatbot panel (BYOK across Ollama/DeepSeek/OpenAI/OpenRouter/
DeepInfra, plus a small free allowance on a cheap model), optionally
add an I/O panel for hardware (ESP32 autopilot, camera feeds) — white-
labeled across nine domains: `activelog.ai` (the embeddable core
itself), `personallog.ai` (personal assistant framing), `businesslog.ai`
(teams, spreadsheets, financial paper trails), `activeledger.ai` (pure
finance/trading), `fishinglog.ai`/`cocapn.ai`/`cocapn.com` (hardware
control, vessel voice backbone), `deckboss.ai`/`deckboss.net`
(the existing flagship), plus a `capitaine.ai` skin pairing the log with
OpenCPN-style chart visualization — draw a line on a chart from a
timestamped, GPS-tagged voice note, using vessel-specific vocabulary the
captain taught the assistant (e.g., "dropped the gear in" meaning "set
the net," for a gillnetter).

**Why this doesn't start as a nine-domain build.** This is exactly the
shape of expansion principle 2 ("one domain at a time") and principle 3
("one toolkit, not a pile of repos") exist to prevent, and it's exactly
the kind of work Step 1's own gate (above) says shouldn't get serious
engineering time until the DeckBoss field beta has real findings — zero
captains have used the flagship product yet. Given the choice, on
2026-07-08, between overriding that gate, holding this entirely until
the gate opens, or something in between, the call made was: **build one
real reference implementation of the shared core on `activelog.ai`
itself first — not nine simultaneous builds.** This isn't a workaround
for the gate; it's principle 3 taken seriously. A "pile of repos" failure
here would look like nine shallow, half-working skins instead of one
real toolkit — the identical failure mode the roadmap's own "what we
will not fork" section catalogs at length elsewhere in the sketchbook
this org hardens against.

**What "real" means for the reference implementation**, so this doesn't
quietly become another overclaiming page: every capability shown must
actually work when a stranger visits `activelog.ai` — the storage-picker
flow must actually persist somewhere real (a local folder via the File
System Access API, or a real linked Cloudflare/Drive/DB account), the
mic-to-markdown path must actually transcribe and actually write, and
the BYOK chat panel must actually call the key a user actually enters.
Anything described in the original vision that isn't buildable to that
bar yet — ESP32 code-generation-and-upload, the free-tier metering on a
cheap model, ESP32/autopilot I/O panels, the `capitaine.ai` chart-drawing
skin's vessel-vocabulary training — is a later phase, not a "coming
soon" label on day one. A phased build plan and the specific first-slice
scope is being drafted (Fable synthesis, in progress as of this entry)
and will supersede this paragraph once it lands.

**Skinning gate.** The eight domain-specific skins do not start until
the `activelog.ai` core is real by the bar above, verified the same way
Step 1 verifies DeckBoss — not "has a demo," works, exercised through
its real use case. This mirrors Step 1's own gate structure deliberately:
a step doesn't count as passed because it was announced, it counts
because someone outside the org who built it could actually use it.

---

## Graduated repos

Repos that have already earned the shell and are listed in the README's
*What is actually here* section:

- **[DeckBoss](https://github.com/purplepincher/deckboss)** — the
  field-tested product.
- **[pincher](https://github.com/purplepincher/pincher)** — the org's
  namesake reflex engine.
- **[plato-semantic-search](https://github.com/purplepincher/plato-semantic-search)** —
  Vectorize-backed semantic search.
- **[plato-engine-block-c](https://github.com/purplepincher/plato-engine-block-c)** —
  embedded C runtime.
- **[vessel-tuner](https://github.com/purplepincher/vessel-tuner)** —
  Worker fleet-health scanner.
- **[git-native-agents](https://github.com/purplepincher/git-native-agents)** —
  git-primitive multi-agent coordination.
- **[conservation-guardian](https://github.com/purplepincher/conservation-guardian)** —
  budget-and-waste guardrail for LLM/agent workflows, on PyPI.
- **[sonar-vision](https://github.com/purplepincher/sonar-vision)** —
  sonar simulation, tracking, and mapping toolkit.
- **[exocortex-mcp-ts](https://github.com/purplepincher/exocortex-mcp-ts)** —
  zero-dependency TypeScript MCP server, 87 passing tests.
- **[constraint-theory-core](https://github.com/purplepincher/constraint-theory-core)** —
  zero-dependency geometric constraint-solving crate, 262 passing tests.
  Forked on its own merit, not as part of any larger story — see
  `docs/research/FABLE_SYNTHESIS.md` for why no unifying narrative was
  invented for this and the next entry.
- **[intent-directed-compilation](https://github.com/purplepincher/intent-directed-compilation)** —
  a real, modest AVX-512 benchmark with honest errata, paired with two
  Coq proofs salvaged from `constraint-theory-math` — one genuinely
  complete (`XOR-ISOMORPHISM.v`, 8 Qed, 0 Admitted), one whose headline
  theorem is not proven (`INTENT-HOLONOMY-DUALITY.v`, 5 Qed but 7
  `admit.`, the source author's own 30% confidence preserved verbatim).
  Corrected here after the org's own README repeated the "two genuinely
  complete" claim inaccurately — caught during a fact-checking pass.

---

## Watchlist — real ideas, not ready, no commitment

Things worth tracking because the underlying idea is sound, but that don't
belong on the numbered roadmap because either the code isn't there yet or the
org doesn't yet have the need that would justify the fork cost.

- **`exocortex`** — distributed agent memory with an ESP32 tier. The
  architecture is genuinely on-mission for an edge-development org. The
  current core, however, is mocked: random embeddings, a training loop that
  sleeps and then reports a random accuracy number rather than training
  anything — and a follow-up deep-dive
  ([`docs/research/exocortex-deep-dive.md`](./docs/research/exocortex-deep-dive.md))
  confirmed this is worse than it first looked: the "S3-compatible storage"
  claim is fabricated (zero S3 code anywhere), and the test suite has never
  once actually collected or run, masked green by `pytest || true`. Re-evaluate
  the Python core when the sketchbook's own versions stop being placeholders
  — or, alternatively, adopt the architecture only (not the code) if this org
  ever builds memory infrastructure itself from scratch. **If this cluster is
  ever revisited, start with `exocortex-mcp-ts` (a real, tested, non-mocked
  MCP server, 87/87 tests passing) or `exocortex-kernel-c` (real embedded C)
  instead of the Python core** — the deep-dive found these are the only
  genuinely solid artifacts in the whole 15-repo family.
- **`ternary-pid`** — a real, tested, CI-covered three-state (bang-bang)
  position-form PID controller with anti-windup, derivative filter,
  cascade, and feedforward support. It would matter if the vessel-autonomy
  research (the long-horizon scope implied by `cocapn-foundation`) ever
  re-enters the roadmap — a three-state actuator controller is a plausible
  building block for autopilot-adjacent work. It is currently explicitly out
  of scope for DeckBoss (which does not do autonomy), so this stays parked
  until that changes. Worth forking for the PID implementation specifically,
  never for the conservation-law framing it's currently bundled under (see
  below) — it does not need that framing to work, and independent review
  found real engineering gaps to fix first regardless (no `dt` parameter, no
  output clamping, non-standard cascade wiring, and a `PLUG_AND_PLAY.md`
  that documents a 3-argument API the code doesn't actually have).

---

## The sharing chapter — a direction on the record, not a step

The essay ([docs/PARADIGM.md](./docs/PARADIGM.md), "The second boat")
describes what happens one step past the ESP32 chapter: a captain and
their agent build something new — a voice bridge into an autopilot nobody
has bridged, a new interface on a working ESP32 program, a new
application layered on an existing one — and the captain, by voice, with
their own GitHub account connected, has the agent publish that work as a
real public repository **under the captain's own account**. From then on,
any other captain's agent asked for a similar ability has two routes:
build it from scratch, or first search public GitHub for already-published
work carrying tags/metadata that mark it as compatible with the
captain's-agent stack (the `cocapn` line — the same family Step 4 tracks,
and the design ancestor DeckBoss's own docs cite), and start from that
instead. A second effect rides along for free: the published repo is an
inbound path. Someone who has never heard of any of this finds a
captain's project through ordinary GitHub browsing because it solves
their problem, and meets the product through the work it produced.

This entry exists mostly to draw one line precisely.

**This is not the fleet learning DeckBoss rejected.** DeckBoss's own
roadmap struck fleet learning deliberately — removed, not deferred —
because "it cannot be built without a DeckBoss-operated backend
somewhere, which breaks the core local-first promise the shipped product
already depends on for trust" (deckboss `ROADMAP.md`, "Fleet learning:
off the roadmap"). The load-bearing word there is *operated*. Fleet
learning required this org to run a server that aggregates users' data;
the moment such a server exists, "your log never touches our
infrastructure" stops being true. The sharing chapter has no operated
server in it anywhere. Each published project lives in the individual
captain's own GitHub repo, under an account the captain already has,
published by the captain's explicit choice; discovery is a public GitHub
search that anyone can run without asking anyone's permission — the
identical mechanism this mission's own research used to find every repo
in this org (`gh search repos`, topics, descriptions). Nobody aggregates
anything, and there is no PurplePincher endpoint to trust or to breach.
The rejected idea and this one differ on exactly the axis the rejection
named — who operates the infrastructure. There, DeckBoss would have had
to. Here, nobody does: GitHub already exists, and the account is already
the captain's.

Worth noting, because it shows this is consistency rather than reversal:
the fleet-learning rejection itself left the door open in exactly this
shape — "the only acceptable shape is a separate, clearly-labeled tool
that reads exports users chose to push to their own storage — never
anything DeckBoss itself operates." The sharing chapter is that shape,
applied to builds instead of log data: work a captain chose to push to
storage the captain owns (a GitHub repo), read by tools — other captains'
agents — that never route through anything this org runs.

**What is real, stated plainly.** Nothing in this section exists. There
is no voice-triggered repo creation, no compatibility-tagging convention,
no agent that searches GitHub before building — and the ESP32 chapter it
all sits on top of doesn't ship either (see the essay's own honesty
section and the two prior-art checks under `docs/research/`). No
mechanism beyond what is written here has been designed: how a repo
declares compatibility, what the publish flow looks like, and what the
agent's search actually queries are all open questions.

A prior draft of this entry gestured at "a GitHub topic could be the
entire convention." Real precedent says that gesture was too thin, and
it's worth correcting rather than leaving on the record inaccurate:
`docs/research/tagging-precedent-findings.md` verified eight real
systems with genuinely working discovery tooling (HACS, Arduino Library
Manager, PlatformIO, the MCP Registry, Obsidian, VS Code, npm/Cargo/PyPI,
awesome-lists) and found every one of them layers a small manifest file
on top of any topic/badge — a bare topic is never the actual discovery
mechanism, even where a specific topic is a required quality gate (HACS's
own `hacs` topic is checked during a curated-list PR review, not searched
at runtime — its generator reads a hand-maintained list, not GitHub
search). Still illustrative, not decided, but now grounded: the smallest
convention with real precedent behind it is a **topic for recall plus a
small in-repo manifest for precision** (compatibility, version, install
path) — closest to PlatformIO's `library.json` `frameworks`/`platforms`
fields — with the trust/quality work every one of those eight systems
does centrally (a curated list, an authenticated registry) instead
deferred to per-discovery agent-level re-verification, since a
centrally-curated list is exactly the kind of operated infrastructure
this section's core constraint rules out. This direction still gets no
engineering time ahead of Step 1's gate, same as everything else in this
file. It goes on the record now so this call and the fleet-learning
rejection can be read side by side, and so a future reader can see the
org said no to one and yes to the other for the same reason:
neither answer ever puts this org between a captain and the captain's own
data.

---

## The cluster-survey program is closed

Every major naming family in the SuperInstance account has now had a
real, code-level examination: `fleet-*`, `ternary-*`/conservation,
`lau-*`, `plato-*`, `cuda-*`/`oxide-*`, the full `cocapn-*` family,
`nexus-*`, `edge-*`, `vessel-*`, `deckboss-*`, `git-native-agents`,
`budget-guardian`, `openconstruct`, dev-tooling, `flux-*`, `agent-*`,
`constraint-*`, `si-*`, `crdt-*`, and — the last two, closing the
program — `superinstance-*` (the account's own namesake family, 29
repos, no fork candidate found) and `grand-*` (42 repos, no fork
candidate found). The hit rate collapsed well before these last two:
recent surveys of 600+ repos each found 2–3 candidates apiece, most of
which the org correctly declined to fork because nothing shipped pulled
them in. **No further cluster surveys are planned.** The account's
naming-family survey is complete; anything not already found is chaff
by a filter with a measured track record, bounded by one escape hatch —
a filter-failing repo gets reconsidered only if a verified Tier-A repo
already in this org references or depends on it. The differentiating
asset this whole program produced was never any single fork; it's the
audit method itself, now standing policy (adoption-bar item 6). Future
energy goes to the org's own products, not more of the sketchbook.

---

## What we will not fork, and why

Saying no is most of the job. This section exists because it's the part a
compressed README can't afford the space for, and because "we looked and
decided not to" is a materially different and more valuable claim than
silence. Permanently out, with the specific evidence behind each call:

- **The conservation-law/ternary theory stack** — roughly 380 repos across
  `ternary-*` and `conservation-*` naming, plus verification harnesses,
  visual explorers, and paper drafts. A dedicated independent review found
  the core identity, `γ + η = C`, is literally Shannon's chain rule
  (`H(X) = I(X;G) + H(X|G)`) — correct, standard information theory, but not
  a new law of any kind, physical or otherwise. The cluster's headline
  applied claim — a cancellation formula `δ(n) = (1/√n)(1 − 3/(2n))` for
  ternary-vote aggregation — is quantitatively wrong: the correct
  closed-form coefficient (derivable by CLT, and confirmed by independent
  Monte Carlo) is `≈0.6515/√n`, meaning the cluster's own formula
  overpredicts cancellation by roughly 50%. Most tellingly, **the cluster's
  own paper (`PAPER.md:28–31`) derives the correct 0.6515 coefficient in its
  proof sketch, then states a theorem asserting a coefficient of 1.0 a few
  lines later** — the wrong number survives inside the same document as the
  right one. Its "verification" script (`delta_clt.py`) calls results
  "VERIFIED within bounds" using an acceptance threshold of 0.10 absolute
  error, which is a loose tolerance wide enough to pass the wrong formula,
  not a proof. The handful of genuinely sound crates underneath the theory
  (`ternary-entropy`, `ternary-types` — small, clean, correctly-implemented
  information-theory primitives) don't depend on the conservation-law
  framing to work, and don't currently carry enough independent value to
  earn a fork slot on their own merits. **Nothing in this org will ever
  depend on γ+η=C to function or to be explained** — this is adoption-bar
  item 3, applied at full strength to the cluster it was written about.
- **Documentation-only repos** — `plato-vessel-technician` (a well-written
  Deckboss hardware/failsafe/voice spec with genuinely good ideas — mechanical
  override cables, breakaway shear pins, dual bilge floats — but zero code),
  `openconstruct-hub`, `Edge-Native`'s roughly 167 markdown files describing a
  parallel-universe NEXUS edge-AI platform with different primitives than
  anything this org uses, and `conservation-docs` (100+ research essays and
  two paper drafts with no code, no build, no tests — and whose own internal
  documents in places contradict the conservation hypothesis they're
  arguing for). Some of these contain genuinely good thinking — the
  fail-safe hardware design in `plato-vessel-technician` already informed
  DeckBoss's own lineage of design decisions — but a spec with no
  implementation is sketchbook material by definition, no matter how good
  the writing is. **We mine these for ideas. We do not fork them**, because
  adoption-bar item 1 (verified working end-to-end) cannot be satisfied by a
  document.
- **The `lau-*` cluster** — 333 repos of speculative differential geometry
  applied to agent coordination, 55% of them with empty descriptions.
  Research poetry, in the literal sense that it reads better as an idea than
  it functions as software. It stays where it is.
- **Anything `sketch-*`, bulk scaffolding stubs, the Minecraft experiments,
  and GPU-specific compute (`cuda-*`/`oxide-*`)** — out of scope until this
  org actually has a GPU-dependent edge product, which it does not and has
  no near-term plan to build. A follow-up deep-dive
  ([`docs/research/cuda-oxide-cluster-deep-dive.md`](./docs/research/cuda-oxide-cluster-deep-dive.md))
  confirmed this call with hard evidence: of 173 repos, only 3 contain
  actual GPU/CUDA code; the rest describe GPU workloads in READMEs without
  ever calling a CUDA API. The conceptually-relevant ideas mixed in (A2A
  messaging, trust engines, reflex/edge runtimes) already have better,
  already-identified implementations in `fleet-*`, `pincher`, `exocortex`,
  and `nexus-runtime` — nothing here is a new asset worth the GPU baggage.
- **Single-commit sketches wearing product names.** The broader ecosystem
  survey of vessel/edge-named repos specifically found near-duplicate repo
  pairs, NMEA parsers that parse nothing and return zeros (see
  `cocapn-forth`'s `parse-gga`, which drops its inputs entirely), and READMEs
  describing features the code has never implemented. This is not a
  one-off risk, it's a pattern across the sketchbook, which is exactly why
  adoption-bar items 1 and 2 exist and are checked *in order*, before
  anything else: we fork working software, not naming themes, and we verify
  that before we read a single word of the README.
- **`si`, `onboard`, `superinstance-mcp`** — the ecosystem's own meta-tooling
  for installing/composing/onboarding its developer tools. Appealing on
  paper (this org runs its own multi-agent build process and could plausibly
  use exactly this kind of tooling), but a deep-dive
  ([`docs/research/dev-tooling-cluster.md`](./docs/research/dev-tooling-cluster.md))
  found zero of three forkable: `si` fails adoption-bar item 3 outright (4 of
  10 claimed "published crates" don't exist on crates.io, 2 more have
  fabricated version numbers, a hardcoded `/home/phoenix/...` path ships in
  `.mcp.json`, and `install.sh` reports success unconditionally via `|| true`
  regardless of what actually happened). `onboard` and `superinstance-mcp`
  aren't outright broken but aren't ready either. Two things worth
  *referencing*, not forking: `onboard`'s PR-bot skeleton as a pattern, and
  `superinstance-mcp`'s MCP-server wiring as a worked example.

---

## How this roadmap gets updated

This document is a snapshot, and the research it's built on has already
demonstrated — repeatedly, across multiple independent passes — that the
sketchbook's own claims about itself go stale or were never true (published
packages that don't exist, "planet-scale" fleets that are five home-lab
machines with one offline, `cargo install` commands that don't work yet,
verification scripts that pass on loose tolerances). A roadmap built entirely
on top of that research inherits the same risk if it isn't treated as
provisional and re-checked before it drives action.

Concretely:

- **This document records reasoning, not a currently-true state.** Every
  factual claim above (a worker returning 200, a repo being unpublished, a
  formula being wrong) was true as of the cited research pass. Software
  moves; a repo cited here as broken may already be fixed, and one cited as
  healthy may have regressed.
- **Re-verify before executing, not before reading.** It's fine to read this
  roadmap and plan around it. Before actually forking a repo, run the
  adoption-bar checklist fresh against that repo's *current* state — don't
  trust this document's snapshot of it. A five-minute re-check (does the
  install command still work, is the worker still returning 200, has the
  README changed) is cheap; discovering a fork decision was based on
  stale evidence after the fact is not.
- **When a step's premise turns out to be wrong, fix this document, don't
  work around it silently.** If Step 0's eventual diff finds the
  `deckboss-*` family is irrelevant, or Step 3's broken workers turn out to
  need a redesign rather than a small fix, update the relevant section here
  with what was actually found, rather than letting the roadmap and reality
  quietly diverge — that divergence is precisely the failure mode this whole
  roadmap exists to avoid repeating.
- **New investigation findings get folded in, not appended as a separate
  pile.** If a targeted investigation (like the Step 0 DeckBoss-family diff)
  produces concrete findings, rewrite the relevant section around them
  instead of leaving this document's reasoning stale next to a newer note
  that supersedes it. A roadmap with two disagreeing answers to the same
  question is worse than no roadmap.
- **Anyone — human or agent — extending this roadmap with a new candidate
  repo should hold it to the same adoption-bar checklist above**, in the
  same order, before proposing it for a numbered step or even the watchlist.

---

## The measure of success

Not repo count — the org this roadmap is reacting against optimized for
exactly that metric, and the result was 4,095 repos of uneven, frequently
non-functional, often over-claimed software. Success here looks like:

- DeckBoss survives contact with 3–5 real boats, and the field-beta findings
  — not a release calendar — drive what the org works on next quarter.
- `cargo install pincher` works, and the README's demo runs in under a
  minute for someone who has never seen this org before.
- Every repo in this org is something one small team genuinely maintains,
  with an issue tracker that actually gets answered.
- A visitor — human or agent — can trust every claim in every README here
  without doing their own independent soundings to check it. That trust,
  transferred from the sketchbook one shipped tool at a time, is the entire
  product of this org.

---

*Ideas start at [SuperInstance](https://github.com/SuperInstance). They ship
here. This roadmap is how they get evaluated on the way.*
