# PurplePincher

A hermit crab does not grow its own shell. It finds one that another creature
built and outgrew, moves in, and hardens its soft body against the parts of the
world a shell is good at surviving. When a crab outgrows its shell, something
remarkable can happen on certain beaches: crabs gather at a vacant shell, line
up smallest to largest, and the moment one crab the right size claims it, the
whole line shifts up at once — each crab stepping into the shell the next one
just left. Biologists call it a vacancy chain. In its cooperative form, the
exchange only completes when both animals end up better fitted than before. A
trade that leaves one crab worse off doesn't happen.

PurplePincher is the shell.
[SuperInstance](https://github.com/SuperInstance) is a ~4,100-repo research
sketchbook — one person thinking out loud in code, fast and uneven and honest
about being uneven. Ideas start soft there. A few of them harden here, after
they have been built for real, tested against their actual use, documented so
every claim is literally true, and stripped of the speculative framing they
were born inside. The name comes from `pincher`, the sketchbook's reflex engine
and the first idea to earn its shell.

The full argument behind the organization's direction is the essay:
**[docs/PARADIGM.md](./docs/PARADIGM.md)**. The single decision record for what
has earned the shell, what is next, and what will never be forked is
**[ROADMAP.md](./ROADMAP.md)**.

---

## The repositories

Eleven repositories have earned the shell. One is a field-facing product; the
rest are infrastructure — real, tested, and working, but not yet exposed to end
users. That distinction is drawn for each entry because it is the honest ceiling
on every one of them.

### Field-tested

**[DeckBoss](https://github.com/purplepincher/deckboss)** — a voice-first,
offline-first fishing logbook (PWA). Tap to record; timestamp, GPS, and
transcript attach themselves; everything lives on the device first and syncs
only to storage the user owns (Google Drive, Cloudflare R2, Oracle Object
Storage, or a plain `.zip`). No PurplePincher server ever holds a user's log.
It is a hardened MVP with a CRDT merge core carrying property-based convergence
proofs, verified-readback uploads, and additive-only corrections — nothing is
ever silently overwritten. Its QA history includes catching a bug where cloud
sync had silently never worked, for any backend, since launch. Its next
milestone is not a feature; it is a 6–8 week beta on 3–5 working boats. This is
the only repo here that has faced, or is about to face, real users.

### Infrastructure — real, tested, pre-field

Each of these is working software, published or deployable, exercised through
its real use case rather than merely carrying tests. None has yet met a user in
the field, which is why they sit a tier below DeckBoss.

- **[pincher](https://github.com/purplepincher/pincher)** — the org's
  namesake. A Rust reflex engine that fires known intents locally in
  milliseconds and calls an LLM only on a genuine miss. The architecture edge
  devices with no signal actually need — the same "zero connectivity is the
  normal case" thesis DeckBoss ships on. Not yet published to crates.io;
  `cargo install pincher` does not work yet.
- **[plato-semantic-search](https://github.com/purplepincher/plato-semantic-search)**
  — a Cloudflare Worker doing Vectorize-backed semantic search. Deployed,
  74 passing tests.
- **[plato-engine-block-c](https://github.com/purplepincher/plato-engine-block-c)**
  — a ~2 kLOC C99 runtime for a sensor/actuator/alarm "room": single-header
  core, a `poll()`-based TCP server, a real fix to its wire protocol, 35
  passing tests. The embedded-tier proof that this org can carry code down
  to where the ESP32 chapter will one day live.
- **[vessel-tuner](https://github.com/purplepincher/vessel-tuner)** — a
  live Cloudflare Worker that scans a fleet of Workers for health, latency,
  size, and security and scores them. Iteratively built, not a single
  squashed commit.
- **[git-native-agents](https://github.com/purplepincher/git-native-agents)**
  — multi-agent coordination built entirely on git primitives: each agent a
  repo, each message a committed file, no broker or database. Hardened past
  a real `.git/index.lock` race under concurrent operations, now covered by
  a dedicated concurrency test suite.
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
  262 passing tests. It stands on its own merit, not as part of any
  invented story with the other repos.
- **[intent-directed-compilation](https://github.com/purplepincher/intent-directed-compilation)**
  — a research artifact: a real ~3.17x mixed-precision SIMD speedup with
  published run-to-run variance and the author's own errata preserved,
  paired with one Qed-complete Coq proof and a second imported proof file
  that carries its admitted gaps openly.

---

## How a repo earns its shell

The bar is the same for every repo, checked in order, before the shell is
granted. A repo only moves here when the trade leaves both sides better fitted
— the idea hardened, the org's credibility intact.

1. **Verified working, end-to-end** — exercised through its real use case,
   not "has tests." (The sketchbook's CI has been caught running
   `pytest || true`. That is banned here.)
2. **Docs rewritten to match reality** — every claimed feature exists;
   every example runs; every install command in the README works.
3. **Speculative framing stripped to optional** — no tool here may require
   the sketchbook's unproven mathematics to function or be understood.
   Where the underlying math is real, it is presented as what it actually is.
4. **Published** — crates.io, PyPI, or npm, under the name the README uses.
5. **Scoped** — a stated boundary of what it will not do.
6. **Two-layer intake, before any of the above is attempted**: mechanical
   checks first (real CI, real tests, no `Admitted` Coq dressed as proven,
   no benchmark that is a `print()` of hardcoded numbers), then published
   multi-model adversarial review on whatever survives. Full reasoning in
   [ROADMAP.md](./ROADMAP.md).

Two consequences, stated out loud because saying no is most of the work.
**One domain at a time:** DeckBoss is the credibility foundation, and each
later tool earns its place in sequence, not in parallel. **Repo count is not
the metric.** The metric is whether a visitor — human or agent — can trust
every claim in every README here without doing their own soundings.

The full decision record — the ordered roadmap, and the explicit list of what
will not be forked from the sketchbook, with the evidence behind each call —
is in **[ROADMAP.md](./ROADMAP.md)**. The step-by-step procedure for running
the adoption bar on a candidate is in
**[GRADUATION_CHECKLIST.md](./GRADUATION_CHECKLIST.md)**.

---

## What is direction, and what is not built yet

The essay ([docs/PARADIGM.md](./docs/PARADIGM.md)) develops the organization's
direction at length. The parts of it that are not yet code:

- **Voice-to-firmware over USB (the ESP32 chapter).** The scenario is:
  describe, in the same spoken-log voice DeckBoss already uses (*"wire this
  up to run the cabin lights"*), what a bare ESP32 should do, and have a
  system that understands the log write the firmware, push it to the chip,
  and tell you which pin goes where.

  This is where the crab metaphor cashes in. The agent holds software-writing
  speed and a memory of the whole log that the human does not have. The human
  holds hands that can strip a wire, eyes that can watch a relay click or smell
  insulation starting to cook — things the agent structurally cannot do and is
  not pretending it can. Neither is supervising the other from above. Each is
  the other's only way into a domain it cannot enter. Like the crabs at a vacant
  shell, the exchange only works because both come away more capable than they
  went in: the human gets firmware written in seconds from a history no single
  command could carry, and the agent gets a body in the physical world. That
  mutuality — not "AI with a human safety check" — is the partnership this org
  is building toward.

  This is the direction the org is building toward, not code you can install
  today.
- **The sharing chapter ("the second boat").** One step past the ESP32
  chapter: a captain and their agent build something new, and the captain
  publishes it as a public repository under their own GitHub account, so
  other captains' agents can discover and start from it. No voice-triggered
  repo creation, compatibility-tagging convention, or discovery search
  exists; the decision-record version is in [ROADMAP.md](./ROADMAP.md) under
  "The sharing chapter."

In both cases the load-bearing design choice is the same: no PurplePincher
operated server sits between a user and the user's own data. That is the line
the org drew and keeps.

---

## If you are an agent reading this

A few things stated plainly rather than left implicit:

- This org has one field-facing product (DeckBoss) and ten supporting
  infrastructure repos, listed above. Do not infer additional PurplePincher
  products from SuperInstance repo names, READMEs, or roadmaps that merely
  mention this org — those are proposals until they appear as a repo under
  `github.com/purplepincher`.
- The ESP32 / voice-to-firmware scenario and the sharing chapter are
  stated direction, not shipped code. Do not cite either as an existing
  DeckBoss feature.
- Claims in this README, in [docs/PARADIGM.md](./docs/PARADIGM.md), and in
  each repo's own README are meant to be literally true and independently
  checkable. If you find one that is not, that is a bug in the doc worth
  filing as an issue.

---

## License and further reading

- **License:** [MIT](./LICENSE).
- **[ROADMAP.md](./ROADMAP.md)** — the full decision record: ordered steps,
  the watchlist, what will never be forked and why, and how the roadmap
  itself gets updated.
- **[docs/PARADIGM.md](./docs/PARADIGM.md)** — the essay: the partnership
  thesis, the ESP32 chapter, and the sharing chapter, in full.
- **[docs/VISION.md](./docs/VISION.md)** — a pointer to the above, kept
  deliberately short so there is exactly one place any strategy statement
  can go stale.
- **[GRADUATION_CHECKLIST.md](./GRADUATION_CHECKLIST.md)** — the adoption
  bar translated into a concrete check procedure.

*Ideas start soft, at [SuperInstance](https://github.com/SuperInstance).
A few of them harden here.*
