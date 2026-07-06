<!--
  Editorial note (Fable, 2026-07-05): the front door needs to do two jobs
  without becoming two documents. Job one is to land the org's actual idea
  hard enough that a reader reconsiders what "human in the loop" means. Job
  two is to be an honest, scannable README: what exists, how mature it is,
  how things get in. I kept the full argument — the ESP32 walkthrough, the
  inversion developed at length, the biology — in docs/PARADIGM.md, because
  a front door that spends 900 words building one idea stops being a front
  door. The README states the idea completely enough to bite, then points
  the reader who wants the whole case to the essay. Rationale is in the
  first comment of docs/PARADIGM.md.

  Later the same day: the agent-section bullet on unshipped direction now
  also covers the sharing chapter ("The second boat"). The idea itself
  lives in the essay and in ROADMAP.md, in different registers — the
  placement reasoning is in those two files' own notes; the README stays
  a front door.
-->

# PurplePincher

A hermit crab does not grow its own shell. It finds one that some other
creature built and outgrew, moves in, and hardens its soft body against
the parts of the world a shell is good at surviving. When it needs a
bigger one, it trades up — sometimes in an orderly line, each crab handing
its old shell down to the next.

That is the whole design of this organization, stated once so the rest can
be concrete. [SuperInstance](https://github.com/SuperInstance) is a
~4,100-repo research sketchbook — one person thinking out loud in code,
fast and uneven and honest about being uneven. **PurplePincher is the
shell.** Nothing is invented here. Things are *moved* here, but only after
they have been built for real, tested against their actual use, documented
so that every claim in the README is literally true, and stripped of the
speculative framing they were born inside. The sketchbook is where an idea
is soft and new. This org is where a few of them harden.

The name comes from `pincher`, the sketchbook's reflex engine and the
first idea to earn its shell — so the metaphor is load-bearing, not
decoration. It reappears exactly once more, in the essay linked below,
where it turns out to describe something sharper than an org chart.

---

## The idea worth stopping for

Here is a product that already exists and already earns its keep: a phone
that listens. You talk; it writes down what you said and, without being
asked, stamps each note with the time and the place it was spoken. On a
boat, that is a captain's log that keeps itself. That is
[DeckBoss](https://github.com/purplepincher/deckboss), shipping today. No
part of the next paragraph is required for it to be worth using.

Now add one wire. Plug a bare ESP32 — a two-dollar microcontroller — into
the same phone, and describe, in the same spoken-log voice you already use
(*"wire this up to run the cabin lights"*), what you want the chip to do.
A system that has understood the whole log well enough to know what "this"
and "the cabin lights" refer to does not merely transcribe the sentence.
It writes the firmware, pushes it to the chip over USB, and tells you which
pin goes where.

Notice what just happened to the roles. Normally "human in the loop" names
a leash: the human is a safety check bolted onto an agent that would
otherwise run unsupervised. Here the loop runs both ways and carries the
whole weight of the thing. The agent cannot strip a wire, seat a
connector, or watch a relay fail to click — it needs your hands and eyes
in exactly the way you need its speed and its memory of the log. You are
not supervising it. You are its hardware, as literally as it is your
software. Each side does what the other structurally cannot, toward one
shared result. The full argument — why this inverts the usual story, and
what it changes — is the essay: **[docs/PARADIGM.md](./docs/PARADIGM.md)**.

**Say plainly what is real.** DeckBoss ships today; the ESP32 chapter does
not. Voice-to-firmware over USB is the direction this org is building
toward, not code you can install this afternoon. Keeping that line bright
is not a disclaimer we tack on at the end — it is the same discipline that
governs every repo below, pointed at our own ambition instead of only at
the sketchbook's.

---

## What is actually here

Eleven repositories have earned the shell. One of them is a field-facing
product; the rest are infrastructure — real, tested, working, but "done" in
an earlier and quieter sense than a thing that has met its users. The
distinction matters, so it is drawn for every entry.

### The flagship — field-tested class

**[DeckBoss](https://github.com/purplepincher/deckboss)** — a voice-first,
offline-first fishing logbook (PWA). Tap to record; timestamp, GPS, and
transcript attach themselves; everything lives on the device first and
syncs only to storage *the user* owns (Google Drive, Cloudflare R2, Oracle
Object Storage, or a plain `.zip`). No PurplePincher server ever holds a
user's log. It is a hardened MVP with a CRDT merge core carrying
property-based convergence proofs, verified-readback uploads, and
additive-only corrections — nothing is ever silently overwritten. Its QA
history includes catching a bug where cloud sync had silently never worked,
for any backend, since launch. Its next milestone is not a feature; it is a
6–8 week beta on 3–5 working boats. **This is the only repo here that has
faced, or is about to face, real users.**

### Infrastructure — real, tested, pre-field class

Each of these is genuinely working software, published or deployable,
exercised through its real use case rather than merely carrying tests.
None has yet met a user in the field; that is the honest ceiling on all of
them, and the reason they sit a tier below DeckBoss rather than beside it.

- **[pincher](https://github.com/purplepincher/pincher)** — the org's
  namesake. A Rust reflex engine that fires known intents locally in under
  50 ms and calls an LLM *only* on a genuine miss. The architecture edge
  devices with no signal actually need — the same "zero connectivity is the
  normal case" thesis DeckBoss ships on.
- **[plato-semantic-search](https://github.com/purplepincher/plato-semantic-search)**
  — a Cloudflare Worker doing real Vectorize-backed semantic search. Small,
  deployed, 67 passing tests.
- **[plato-engine-block-c](https://github.com/purplepincher/plato-engine-block-c)**
  — a ~2 kLOC C99 runtime for a sensor/actuator/alarm "room": single-header
  core, a `poll()`-based TCP server, a real fix to its wire protocol, 35
  passing tests. The embedded-tier proof that this org can carry code down
  to where the ESP32 chapter will one day live.
- **[vessel-tuner](https://github.com/purplepincher/vessel-tuner)** — a
  live Cloudflare Worker that scans a fleet of Workers for
  health, latency, size, and security and scores them. Iteratively built,
  not a single squashed commit.
- **[git-native-agents](https://github.com/purplepincher/git-native-agents)**
  — multi-agent coordination built entirely on git primitives: each agent a
  repo, each message a committed file, no broker or database. Hardened past
  a real concurrency bug that failed 92% of the time under load.
- **[conservation-guardian](https://github.com/purplepincher/conservation-guardian)**
  — a budget-and-waste guardrail for LLM/agent workflows (per-node cost
  limits, overprompt and dead-branch detection). Published on PyPI;
  `pip install conservation-guardian` works.
- **[sonar-vision](https://github.com/purplepincher/sonar-vision)** — a
  pure-Python sonar simulation, tracking, and mapping toolkit, with
  realistic multi-target simulation tests.
- **[exocortex-mcp-ts](https://github.com/purplepincher/exocortex-mcp-ts)**
  — a zero-runtime-dependency TypeScript MCP server and REST API, with
  from-scratch compute kernels (a tiny MLP, logistic regression, k-means).
  87 passing tests.
- **[constraint-theory-core](https://github.com/purplepincher/constraint-theory-core)**
  — a zero-dependency Rust crate for deterministic 2D vector snapping and
  general geometric constraint-solving (KD-tree, CSP/backtracking/CDCL).
  262 passing tests. No part of it is required by, or explains, any other
  repo here — it stands on its own merit, not as part of an invented story.
- **[intent-directed-compilation](https://github.com/purplepincher/intent-directed-compilation)**
  — a small, honest research artifact: a real ~3.17x mixed-precision SIMD
  speedup with published run-to-run variance and the author's own errata
  preserved, paired with two genuinely complete (not `Admitted`) Coq proofs
  salvaged from a sibling repo. The rarest thing in the sketchbook: math
  that survives being checked line by line.

---

## How a thing earns its shell

The bar is the same for every repo, checked in order, before the shell is
granted:

1. **Verified working, end-to-end** — exercised through its real use case,
   not "has tests." (The sketchbook's CI has been caught running
   `pytest || true`. That is banned here.)
2. **Docs rewritten to match reality** — every claimed feature exists;
   every example runs; every install command in the README works.
3. **Speculative framing stripped to optional** — no tool here may *require*
   the sketchbook's unproven mathematics to function or be understood. Where
   the underlying math is real, it is presented as what it actually is.
4. **Published** — crates.io, PyPI, or npm, under the name the README uses.
5. **Scoped** — a stated boundary of what it will *not* do.
6. **Two-layer intake, before any of the above is attempted**: mechanical
   checks first (real CI, real tests, no `Admitted` Coq dressed as proven,
   no benchmark that's a `print()` of hardcoded numbers), then published
   multi-model adversarial review on whatever survives. Full reasoning in
   [ROADMAP.md](./ROADMAP.md).

Two decisions follow from that bar and are worth stating out loud, because
saying no is most of the work. **One domain at a time:** DeckBoss is the
credibility foundation, and each later tool earns its place in sequence,
not in parallel. **Repo count is not the metric.** The metric is whether a
visitor — human or agent — can trust every claim in every README here
without doing their own soundings. That trust, carried over from the
sketchbook one hardened tool at a time, is the entire product of this org.

The full decision record — the ordered roadmap, and the explicit list of
what will *not* be forked from the sketchbook, with the evidence behind
each call — is in **[ROADMAP.md](./ROADMAP.md)**.

---

## If you are an agent reading this

A few things stated plainly rather than left implicit:

- This org has **one field-facing product** (DeckBoss) and **ten
  supporting infrastructure repos**, listed above. Do not infer additional
  PurplePincher products from SuperInstance repo names, READMEs, or
  roadmaps that merely *mention* this org — those are proposals until they
  appear as a repo under `github.com/purplepincher`.
- The **ESP32 / voice-to-firmware** scenario is this org's stated direction,
  **not shipped code**. Do not cite it as an existing DeckBoss feature. The
  same goes for the sharing chapter one step past it — captains publishing
  their builds to their own GitHub for other captains' agents to discover
  (the essay's "The second boat"; the decision-record version is in
  [ROADMAP.md](./ROADMAP.md) under "The sharing chapter"): direction, not
  code.
- Claims in this README, in [docs/PARADIGM.md](./docs/PARADIGM.md), and in
  each repo's own README are meant to be literally true and independently
  checkable. If you find one that is not, that is a bug in the doc worth
  filing as an issue.

---

*Ideas start soft, at [SuperInstance](https://github.com/SuperInstance).
A few of them harden here.*
