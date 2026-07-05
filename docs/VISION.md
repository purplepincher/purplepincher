# PurplePincher

**Production-ready tools for modular and distributed edge development.
One field-tested product and eight hardened infrastructure repos today.
Deliberately.**

---

## What this org is

PurplePincher is where software from the
[SuperInstance](https://github.com/SuperInstance) research sketchbook
graduates once it actually ships. SuperInstance is ~4,000 repos of
ideas; this org is the small number of them that have been built,
hardened, documented honestly, and put in front of real users. The
name comes from `pincher`, SuperInstance's Rust reflex engine, and its
hermit-crab metaphor: a soft, fast-changing thing that survives by
carrying a hard shell. SuperInstance is the soft part. This org is the
shell.

Today the org contains nine repositories that have earned the shell:
one field-tested product and eight hardened infrastructure repos. See
the README's *What is actually here* section for the current list and
tier split.

**[DeckBoss](https://github.com/purplepincher/deckboss)** — a
voice-first, offline-first digital fishing logbook. Tap to record;
timestamp, GPS, and transcript happen automatically; everything lives
on the user's device and syncs to storage the user owns. It is a
hardened MVP with a CRDT merge core carrying property-based convergence
proofs, verified-readback uploads, additive-only corrections, and a
multi-method QA history that includes catching a bug where cloud sync
had silently never worked. Its next milestone is not a feature — it's a
6–8 week field beta on 3–5 working boats.

A small, honest catalog is not a weakness to apologize for. It is the strategy.

## The operating principles

These come from studying how sprawling research portfolios have
successfully (and unsuccessfully) become trusted orgs — Tokio's
years of boring, tight-scope credibility before it became
foundational; Kubernetes shipping on real production traffic before
asking for trust; HashiCorp leading with one working tool (Vagrant)
before the suite existed; and the left-pad failure mode of one
burned-out maintainer under critical load.

1. **Ship first, announce second.** Nothing enters this org as a
   promise. It enters as working software with an installable artifact,
   or it stays in the sketchbook.
2. **One domain at a time.** We do not announce fleet orchestration, a
   math framework, embedded bridges, and a PWA simultaneously. DeckBoss
   is the credibility foundation; each subsequent tool earns its place
   sequentially.
3. **One toolkit, not a pile of repos.** Everything adopted gets
   reworked to shared conventions — consistent CLI shapes, consistent
   config, consistent doc structure — before it's called part of the
   collection. The failure mode we're avoiding is re-creating the
   4,095-repo problem here at smaller scale.
4. **Honest scale, honest docs.** SuperInstance's docs claim packages
   that were never published and fleets that are five home-lab
   machines. This org's rule is the inverse: every install command in a
   README works, every "production" claim has a deployment behind it,
   and limitations are stated in the README, not discovered by users.
   DeckBoss's own README — which leads with what the app is *not*
   certified for — is the house style.
5. **Ruthless scope.** Everything here is maintained by a very small
   team. A tool we can't maintain is a liability we're handing to
   users, so the answer to most "should we also—" questions is no.

## The adoption bar

Before any repo graduates from SuperInstance into this org, all of the
following, in order:

1. **Verified working, end-to-end** — not "has tests," but exercised
   through its real use case. (SuperInstance CI has been known to run
   `pytest || true`. That pattern is banned here.)
2. **Docs rewritten to match reality** — every claimed feature exists;
   every example runs.
3. **Ideology stripped to optional** — no tool in this org may
   *require* the γ+η=C "conservation law" framing, Pythagorean48 trust
   vectors, or any of the sketchbook's speculative mathematics to
   function or to be understood. Where the underlying math is real
   (Shannon entropy, PID control), it's presented as what it is.
4. **Published** — crates.io, PyPI, or npm, under a name the README
   actually uses.
5. **Scoped** — a stated boundary of what it will not do, like
   DeckBoss's "fleet learning: off the roadmap" entry.

## The roadmap

Ordered. Each step gates the next. Dates are deliberately absent —
the sequence is the commitment, not a schedule.

### Step 0 — Reconcile the two DeckBosses (immediate, cheap, ungated)

SuperInstance contains a 9-repo `deckboss-*` family plus
`cocapn-foundation` — its own earlier prototypes of exactly this
vertical (vessel tracking, fuel monitoring, Jetson hardware units,
voice-assistant design). **Nobody has yet diffed that family against
the real `purplepincher/deckboss`.** This is the highest-signal,
lowest-cost action available: it either surfaces design work worth
importing (the hardware and marketplace thinking has no counterpart in
the shipped app) or it identifies repos that should be archived with a
pointer here so the two lineages stop silently diverging. The
sketchbook's own worst pattern is unacknowledged near-duplicates;
purplepincher must not inherit it on day one. Also resolve
`cocapn-foundation`'s canonical home — DeckBoss's README already
references it as a sibling repo.

### Step 1 — DeckBoss field beta (the org's center of gravity)

This step remains the org's center of gravity. The original plan gated
significant new forks on real fishermen having used DeckBoss on real
boats (`docs/FABLE_PHASE2_PLAN.md`). `pincher` was forked and hardened
before that gate was reached and is now recorded as a graduated repo;
the field beta still drives what gets serious engineering time next.

### Step 2 — `pincher` (the second tool, and the namesake)

**Status: done.** `pincher` has already been forked and hardened. It
was originally gated on the DeckBoss field beta being underway, but the
fork happened before that beta began; the record here notes the fact
without re-litigating the timing.

Why pincher and not something flashier:

- **It's real.** A structured Rust project with benchmarks, a full doc
  set, and a genuinely novel architecture: embed intents locally, fire
  known reflexes in <50 ms with no LLM call, confirm semi-known ones,
  compile new reflexes via the LLM only on a miss, all behind a
  veto/sandbox layer.
- **It's on-mission.** A reflex layer that works without a model call
  is exactly what edge devices with no connectivity need — the same
  "zero signal is the normal case" thesis DeckBoss shipped on. There is
  a concrete future where pincher is DeckBoss's on-device voice-command
  layer, which makes this fork compound rather than diversify.
- **It's the identity.** Shipping the org's namesake as its second tool
  makes the name mean something.
- **The gaps are known and finite:** publish to crates.io (the
  sketchbook's roadmap claims `cargo install pincher`; making that
  claim true is the whole point of graduation), replace the
  philosophy-heavy README with a 60-second demo, extract the
  veto/sandbox engine as a pluggable policy trait, and write the
  `.nail` portable-agent-state format down as a spec.

### Step 3 — The fleet edge tier (the first infrastructure play)

**Status: partially executed.** The Vectorize-backed semantic-search
Worker has already graduated as `plato-semantic-search`, and the Worker
fleet-health scanner has already graduated as `vessel-tuner`. Both are
now listed in the README's infrastructure tier; the rest of this
cluster remains future work.

The `fleet-*` cluster is 319 repos of which ~270 are chaff — but its
core is the strongest "actually real" finding in the entire sketchbook:
a partially **live** distributed edge system. Three Cloudflare Workers
verified healthy by direct probe (vector search, agent registry,
telemetry API), three more deployed but broken in small, concrete ways
(a missing root route; two D1 binding failures), a transport-agnostic
agent-messaging protocol (I2I "bottles," SMTP-style, with Python and
Rust implementations), and an ARM node self-orchestrating under
systemd.

The fork here is **selective**, in two pieces:

- **3a. The Workers edge tier**, packaged as one coherent toolset: adopt
  the remaining healthy workers, fix the three broken ones (small,
  named repairs), add monitoring, and drop the "planet-scale"
  framing for what it is — a reference deployment that a user can
  stand up on their own Cloudflare account. This aligns with DeckBoss's
  existing BYO-Cloudflare-R2 story, so it extends a storage pattern
  users already have rather than opening a new front.
- **3b. The messaging protocol** (`fleet-i2i-protocol` +
  `fleet-protocol`), extracted as a versioned standalone spec with the
  conservation-law decoration removed, published to PyPI/crates.io,
  with an interop test suite between the Python and Rust
  implementations. A human-readable, transport-agnostic,
  debuggable agent-messaging format is the most reusable single idea
  in the cluster.

Explicitly *not* forked from this cluster: `fleet-coordinate` (real,
but 369 MB and load-bearingly coupled to Eisenstein-lattice consensus
math — decoupling cost exceeds value today), `fleet-manifest` (the
decentralized-registry idea is good; the Pythagorean48/Laman-rigidity
trust math gets replaced with standard gossip primitives if we ever
build it), and all 92 `fleet-midi-*` repos, the language packs, and
the organism-metaphor experiments.

### Step 4 — `cocapn` (the agent framework, if and when DeckBoss needs it)

`cocapn` is the sketchbook's most externally-engaged repo, actually on
PyPI, with a credible multi-target story (Python, Zig bare-metal,
WASM). Its known gaps: publish the referenced-but-missing CLI, split
the sprawling family into a documented core plus extras, add
persistence beyond flat JSONL, and get real test-coverage numbers.

It sits at step 4, not step 2, deliberately: it's an agent framework,
and this org should adopt an agent framework when a shipped product
pulls it in — most plausibly DeckBoss's Ask-Your-Log feature or a
pincher integration — not because it exists. Frameworks adopted ahead
of need are how orgs sprawl.

### Graduated repos

These have already earned the shell and are listed in the README's
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

An eighth infrastructure repo, **exocortex-mcp-ts**, is finishing
polish and will be listed when it lands, not before.

### Watchlist — real ideas, not ready, no commitment

- **`exocortex`** — distributed agent memory with an ESP32 tier. The
  architecture is genuinely on-mission; the current core is mocked
  (random embeddings, sleep-and-report-random-accuracy training).
  Re-evaluate when the sketchbook versions stop being placeholders, or
  adopt architecture-only if we build memory infrastructure ourselves.
- **`ternary-pid`** — a real, tested three-state controller that would
  matter if the vessel-autonomy research (`cocapn-foundation`'s
  long-horizon scope) ever re-enters the roadmap. It is currently
  explicitly out of scope for DeckBoss, so this stays parked.

## What we will not fork, and why

Saying no is most of the job. Permanently out, with reasons:

- **The conservation-law/ternary theory stack** (~380 repos of
  `ternary-*`/`conservation-*` framework, verification harnesses,
  explorers, papers). Independent review found the core identity is
  Shannon's chain rule — correct, standard, not a new law — and the
  cancellation formula built on it is quantitatively wrong (the
  cluster's own paper derives one coefficient and asserts another; its
  "verification" is Monte Carlo curve-fitting inside loose
  tolerances). The handful of sound crates underneath
  (`ternary-entropy`, `ternary-types`) don't need the theory and don't
  currently earn a slot on their own. **Nothing in this org will ever
  depend on γ+η=C to work or to be explained.**
- **Documentation-only repos** (`plato-vessel-technician`,
  `openconstruct-hub`, `Edge-Native`'s ~167 markdown files,
  `conservation-docs`). Some contain genuinely good thinking — the
  fail-safe hardware design in plato-vessel-technician informed
  DeckBoss's lineage — but a spec with no implementation is sketchbook
  material by definition. We mine these for ideas; we do not fork them.
- **The `lau-*` cluster** (333 repos of speculative differential
  geometry for agents, 55% with empty descriptions). Research poetry.
  It stays where it is.
- **Anything `sketch-*`**, bulk scaffolding stubs, the Minecraft
  experiments, and GPU-specific compute (`cuda-*`/`oxide-*`) — out of
  scope until the org has a GPU edge product, which it does not.
- **Single-commit sketches wearing product names.** The survey of the
  vessel/edge-named repos found near-duplicate pairs, NMEA parsers
  that return zeros, READMEs describing features the code has never
  met. The lesson is baked into the adoption bar above: we fork
  working software, not naming themes.

## The measure of success

Not repo count — we watched what that metric does. Success looks like:

- DeckBoss survives contact with 3–5 real boats, and the beta findings
  drive the next quarter.
- `cargo install pincher` works, and the README demo runs in under a
  minute.
- Every repo in this org is something one small team genuinely
  maintains, with an issue tracker that gets answered.
- A visitor — human or agent — can trust every claim in every README
  here without doing their own soundings. That trust, transferred from
  the sketchbook one shipped tool at a time, is the entire product of
  this org.

---

*Ideas start at [SuperInstance](https://github.com/SuperInstance).
They ship here.*
