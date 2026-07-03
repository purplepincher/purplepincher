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

Full detail, evidence, and citations: [`docs/research/deckboss-family-diff.md`](./docs/research/deckboss-family-diff.md).

### Step 1 — DeckBoss field beta (the org's center of gravity)

Nothing else gets significant attention until real fishermen have used
DeckBoss on real boats. This is already planned in detail
(`docs/FABLE_PHASE2_PLAN.md`) — the beta is not a future idea, it's the next
scheduled milestone for the org's one shipped product.

**Why this gates literally everything else.** The industry pattern this org
is explicitly modeling itself on (see the README's Tokio/Kubernetes/
HashiCorp references) is unambiguous: credibility comes from one thing
working in production before a second thing is announced. A field beta on
working vessels — not a demo, not a closed alpha, actual fishermen logging
actual trips on actual boats for 6–8 weeks — is this org's equivalent of
production traffic. Forking new tools before that data exists would be
expanding the storefront before the first product has a single paying
customer walk through the door. It would also be a direct violation of
principle 2 ("one domain at a time") — announcing infrastructure work while
the flagship's core value proposition is still unproven in the field.

**What "gates" means concretely:** Step 2 does not start in earnest until
the beta is *underway* (not necessarily complete — see Step 2's framing
below), and Steps 3 and 4 do not get serious engineering time until Step 1's
findings have been digested and, if necessary, folded back into DeckBoss
itself.

### Step 2 — `pincher` (the second tool, and the namesake)

First fork after the beta is underway — not before, and deliberately not
DeckBoss's flashiest or best-known cousin.

**Why pincher, specifically, earns second place:**

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
  its hermit-crab metaphor. Shipping the namesake as the second tool, once
  the flagship has field data behind it, makes the name mean something
  earned rather than aspirational.
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

This is the org's first step outside a single product and into shared
infrastructure, which is why it comes after two products have already
established the org's credibility and conventions.

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

- **3a. The Workers edge tier**, packaged as one coherent toolset. Adopt the
  three healthy workers as-is. Fix the three broken ones — these are small,
  named repairs (a missing root route; two D1 binding failures), not
  redesigns. Add monitoring so "broken" gets caught before a user finds it,
  not after. And drop the "planet-scale" framing the sketchbook uses for
  what is, honestly, a five-vessel home-lab fleet (one of the five offline)
  — present it as what it actually is: a reference deployment a user can
  stand up on their own Cloudflare account. This choice also aligns with
  DeckBoss's existing BYO-Cloudflare-R2 sync story, so it extends a storage
  pattern this org's users already have rather than opening an unrelated new
  front.
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

**Known gaps, all closeable:** publish the CLI the docs already reference but
that doesn't yet exist; split the sprawling repo family into a documented
core plus optional extras (consistent with this org's "ideology stripped to
optional" bar); add real persistence beyond flat JSONL; and get honest
test-coverage numbers rather than claimed ones.

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
- **`git-native-agents`** — multi-agent coordination built entirely on git
  primitives: each agent is a repo, messages are files committed to a
  recipient's `inbox/`, memory is `memory/*.txt` plus git tags,
  "thought branches" for speculative exploration, merge-based consensus. No
  broker, no database, no scheduler required — git is the infrastructure.
  Honestly scoped by its own docs to 5–50 agents before it would need to move
  to a broker (O(N²) routing beyond that). Charming, small, auditable — and
  currently just a POSIX shell script (`orchestrator.sh`) with likely race
  conditions on concurrent `tick` operations and no test suite. A real fork
  means a Rust or Go re-implementation, not a repackaging. Worth it only if
  the org develops an actual multi-agent operations need — and notably, this
  org already runs a multi-model build process for DeckBoss itself (kimi,
  aider, GLM, mmx alongside Claude), which is the most plausible internal
  pull that could justify this fork later.
- **The budget-guardian family** — token/time/build budget enforcement for AI
  coding workflows. Described elsewhere in the research as among the most
  production-minded *unshipped* things anywhere in the sketchbook, and it's
  directly adjacent to how this org actually builds software day to day. A
  candidate for a small, quick win slotted between major roadmap steps,
  precisely because it's low-risk and immediately useful rather than
  speculative.
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
