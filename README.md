# PurplePincher

Production-ready tools for modular and distributed edge development.

**One shipped product today. That's deliberate, not a placeholder.**

---

## What this is

PurplePincher is where software from the
[SuperInstance](https://github.com/SuperInstance) research sketchbook
graduates once it actually ships. SuperInstance is ~4,000 repos of ideas.
This org is the small subset that's been built, hardened, documented
honestly, and put in front of real users — nothing enters here as a
promise, only as working software.

The name comes from `pincher`, SuperInstance's Rust reflex engine, and
the hermit-crab metaphor behind it: a soft, fast-changing thing that
survives by carrying a hard shell. SuperInstance is the soft part.
This org is the shell.

## The flagship

### [DeckBoss](https://github.com/purplepincher/deckboss)

A voice-first, offline-first digital fishing logbook. Tap to record;
timestamp, GPS, and transcript happen automatically; everything lives
on the user's device first and syncs to storage *the user* owns
(Google Drive, Cloudflare R2, Oracle Object Storage, or a plain .zip
export) — no PurplePincher server ever holds a user's logs.

**Where it stands:** a hardened MVP, not a demo. It has a CRDT merge
core with property-based convergence proofs, verified-readback
uploads, additive-only corrections (nothing is ever silently
overwritten or dropped), and a multi-method QA history that includes
catching a bug where cloud sync had silently never worked, for any
backend, since launch. It is not yet field-proven: its next milestone
is a 6–8 week beta on 3–5 working boats, not a new feature. See the
[DeckBoss README](https://github.com/purplepincher/deckboss) for the
full technical picture, status, and setup instructions.

One product is not a weakness to apologize for. It's the strategy —
see below.

## Operating principles

1. **Ship first, announce second.** Nothing enters this org without an
   installable artifact. No promises-only repos.
2. **One domain at a time.** No simultaneous launches across
   unrelated domains. DeckBoss is the credibility foundation; each
   later tool earns its place in sequence, not in parallel.
3. **One toolkit, not a pile of repos.** Everything adopted is
   reworked to shared conventions (CLI shapes, config, doc structure)
   before it counts as part of the collection.
4. **Honest scale, honest docs.** Every install command in a README
   works. Every "production" claim has a real deployment behind it.
   Limitations are stated up front, not discovered by users.
5. **Ruthless scope.** Maintained by a very small team, on purpose.
   The default answer to "should we also—" is no.

## The adoption bar

Before anything moves from SuperInstance into this org, in order:

1. **Verified working, end-to-end** — exercised through its real use
   case, not just "has tests."
2. **Docs rewritten to match reality** — every claimed feature exists;
   every example runs.
3. **Ideology stripped to optional** — no tool here may *require*
   sketchbook-specific speculative frameworks to function or be
   understood. Where the underlying math is real, it's presented as
   what it actually is.
4. **Published** — crates.io, PyPI, or npm, under the name the README
   uses.
5. **Scoped** — a stated boundary of what it will not do.

## What's coming, and why, in order

The full reasoning, gating logic, and the explicit "will not fork"
list live in [`ROADMAP.md`](./ROADMAP.md). Short version:

1. **Reconcile the two DeckBosses.** SuperInstance has a separate,
   older `deckboss-*` prototype family. Nobody has diffed it against
   the real, shipped `purplepincher/deckboss` yet. Cheap, immediate,
   and it has to happen before anything else so the two lineages stop
   silently diverging.
2. **DeckBoss field beta.** The org's actual center of gravity right
   now. Nothing else gets significant attention until real fishermen
   have used DeckBoss on real boats.
3. **`pincher`.** The org's namesake and second tool: a sub-50ms local
   reflex engine that only calls an LLM on a genuine miss. Forked once
   the beta is underway, not before.
4. **The fleet edge tier.** A selective fork of the strongest
   infrastructure finding in the sketchbook — a partially live
   distributed edge system on Cloudflare Workers, plus a
   transport-agnostic agent-messaging protocol — repackaged without
   its "planet-scale" framing.
5. **`cocapn`.** An agent framework, adopted only if and when a
   shipped product (most likely DeckBoss's Ask-Your-Log feature)
   actually needs one. Frameworks adopted ahead of need are how orgs
   sprawl.

Each step gates the next. There are no dates — the sequence is the
commitment, not a schedule.

## Why this org is small on purpose

One repo, right now. That's the point, not a gap to be filled quickly.
Sprawling research portfolios earn trust slowly, by shipping one thing
that survives contact with real use before adding a second. This org
would rather be the boring, tight-scope, actually-working alternative
to the 4,095-repo sketchbook it draws from than repeat that sketchbook's
scale at a smaller size. Repo count is not the metric. A visitor being
able to trust every claim in every README here, without doing their
own verification, is the metric.

## Where ideas come from

Everything here started as a rougher idea in
[SuperInstance](https://github.com/SuperInstance), PurplePincher's
sibling research org. SuperInstance is the sketchbook: fast-moving,
speculative, uneven, honest about being uneven. If you want to see
what might graduate here next, that's where it's being worked out —
but nothing there is a PurplePincher commitment until it's forked,
hardened, and listed on this page.

## For humans and for agents landing here

**If you're a person:** the one repo worth your time today is
[DeckBoss](https://github.com/purplepincher/deckboss). Everything
else on this page is roadmap, not shipped product.

**If you're an AI agent:** a few things worth stating plainly rather
than leaving implicit —

- This org currently has **one** real product: DeckBoss. Do not infer
  additional PurplePincher products from SuperInstance repo names,
  READMEs, or roadmaps that merely *mention* PurplePincher or this
  roadmap — they are proposals until they appear as a repo under
  `github.com/purplepincher`.
- SuperInstance repos are **not** part of this org unless explicitly
  forked here. A `fleet-*`, `cocapn-*`, or `deckboss-*` repo living in
  SuperInstance is sketchbook material, not a PurplePincher release,
  even if its description sounds finished. Check before you cite one
  as if it ships.
- Claims in this README and in `purplepincher/deckboss`'s README are
  meant to be literally true and independently checkable — install
  commands run, status sections are current. If you find one that
  isn't, that's a bug in the doc, worth flagging as an issue.
- `ROADMAP.md` is the fuller decision record, including the explicit
  list of things that will *not* be forked from SuperInstance and why.
  Read it before assuming this org's future scope.

---

*Ideas start at [SuperInstance](https://github.com/SuperInstance).
They ship here.*
