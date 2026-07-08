<!--
  Status note (Fable, 2026-07-08): this is a direction paper, not a step,
  not a spec, and not a decision record. It develops the sharing chapter's
  own open questions ("how a repo declares compatibility, what the publish
  flow looks like, and what the agent's search actually queries" —
  ROADMAP.md, "The sharing chapter") one level further down, in the same
  register docs/research/ILLUSTRATIVE_MANIFEST_SKETCH.md established:
  concrete enough to discuss, labelled clearly enough that nobody builds
  from it without a real design pass. It gets no engineering time ahead of
  the gates already on the record — Step 1's field beta and Step 5's
  ActiveLog core come first, and this document does not relitigate that
  sequencing anywhere. Where this content should eventually live is argued
  explicitly at the end, rather than assumed by the fact of this file
  existing.
-->

# Rooms — a repo an agent can step into

## The sentence that already exists

The idea this paper develops was not invented for it. It is sitting, today,
in a graduated repo in this org. `pincher`'s `AGENT.md` — an agent-persona
file inherited from the sketchbook's "Ensign" convention — opens:

> I reside in this repository. This is my room.

That file is half-real in exactly the way this org's whole method exists to
handle: the sentence is load-bearing and true (an agent that lives with a
repo, keeps a duty log in `memory/`, and knows its own scope is a genuinely
useful shape), while the "Fleet Neighbors" table below it points at
sketchbook repos that were never hardened. The org's standing move — keep
the real kernel, strip the overclaim — applies to its own inherited files
too. This paper is that move, applied to one sentence.

The word "room" has a second real ancestor. `plato-engine-block-c`, also
graduated, is described in this org's own README as "a ~2 kLOC C99 runtime
for a sensor/actuator/alarm 'room'" — a bounded physical space, served by
one small runtime, with a declared set of things that can be sensed and
actuated in it. The surrounding `plato-room-*` cluster in the sketchbook
(~23 repos) is mostly 100–400-LOC stubs (`docs/research/plato-cluster-survey.md`),
with one more real artifact worth remembering: `plato-room-configs`, a
tested Rust crate that loads and schema-validates JSON room configurations.
So the sketchbook's "room system" is, honestly: two real artifacts, one
good word, and a lot of stubs. The word survives because the two artifacts
did.

## The idea, stated as the next beat of an existing argument

`docs/PARADIGM.md`'s "The second boat" ends with a shell on a beach: a
captain publishes a build to their own GitHub account, and the next
captain's agent finds it by public search and starts from it instead of
from scratch. The sharing chapter (`ROADMAP.md`) draws the decision-record
line around that — no operated backend, the captain's own account, public
search — and then states plainly that the mechanism doesn't exist: how a
repo declares compatibility, what the publish flow looks like, what the
agent's search queries. All open.

"A room can be a repo" is the vacancy chain run one more beat. The second
boat's agent *salvaged* the shell — took the published work as a starting
point for its own build. A room-ready repo is a shell the next agent
doesn't strip for parts. It **moves in**: arrives cold, reads one
conventional surface, and knows within a minute what this place can do,
how to ask for it, what it must never do unattended, and where the state
lives. The repo stops being source code that happens to have a README and
becomes a place organized around a specific first-class reader — not the
maintainer, not the human browser, but the zero-shot agent that will
operate it on someone's behalf tomorrow, on a boat, over voice.

That reframing is the entire content of the idea. Everything below is
working out what it would concretely take, using only mechanisms this org
has already built or already verified as precedent — because the honest
finding of this paper is that almost nothing new has to be invented. The
pieces exist. What doesn't exist is the convention that assembles them,
and this paper sketches that convention without deciding it.

One scope note before the mechanism. Casey's framing of this direction
includes examples far from the ESP32 chapter — a book or a podcast written
by conversation, marketing managers, research on demand. The room
abstraction is deliberately indifferent to that range: a room is anything
with declared levers and manuals, whether the levers flash firmware or
draft chapters. Nothing in this paper depends on which kind of room comes
first, and no mechanism is proposed here for any specific one of those
examples.

---

## 1. What makes a repo room-ready

### Start from what was already verified, not from imagination

`docs/research/tagging-precedent-findings.md` verified eight real
discovery systems (HACS, Arduino Library Manager, PlatformIO, the MCP
Registry, Obsidian, VS Code, npm/Cargo/PyPI, awesome-lists) and found
every one of them converges on the same two-layer shape: **a topic for
recall, a small in-repo manifest for precision** — with the trust work
done centrally (curation or authenticated registry), which the no-operated-
backend constraint rules out here, deferring trust to the reading agent.
`docs/research/ILLUSTRATIVE_MANIFEST_SKETCH.md` then sketched what the
manifest could look like for the reuse case: name, version, framework,
platforms, install instructions. That sketch answers the second boat's
question — *"can I reuse this artifact?"*

A room asks a bigger question: *"what can I **do** here, and how do I
ask?"* The reuse manifest describes a thing to take away; a room manifest
describes a place to operate. Concretely, a cold agent walking into a
room needs three layers, and this org has a real, already-shipped ancestor
for each one.

### Layer one — the door (recall and identity)

Unchanged from the sharing chapter's grounded refinement: a GitHub topic
for recall, a manifest at a conventional path for precision. The room
manifest *contains* the compatibility sketch's fields (name, version,
platforms, framework, install) rather than replacing them — a room-ready
repo is a strict superset of a reuse-ready one, so nothing about this
layer reopens what the tagging research already settled. Recall stays
plain public GitHub search, the identical machinery this org's own surveys
ran on. No new mechanism at this layer, and none needed.

### Layer two — the levers (the capability declaration)

This is the layer the compatibility sketch doesn't have, and it is the one
place where a real, shipped artifact in this org transfers almost
unchanged: **pincher's skill files.**

A pincher skill (`pincher/skills/*.toml`) is already a structured answer
to "how does a new capability declare itself to a voice-first matcher?"
Its actual shape, from the shipped `docker-ops.toml`:

- `[skill]` — name, version, description, tags (the identity layer,
  overlapping layer one);
- `[skill.intent]` — a `canonical` phrasing plus `variations`: the
  utterances a voice operator might actually say. This is the field that
  makes the declaration *voice-first* rather than API-first — the manifest
  ships the intent surface, not just the function signature;
- `[skill.capabilities]` — `required` and `optional` capability names
  (`subprocess`, `network`, …): what the action needs permission to touch;
- `[skill.reflex]` — an `action_template` (the executable shape) and a
  `confidence_seed` (where this reflex *starts* on the trust ladder).

A room's lever declaration is, to first approximation, **a pack of these**
— one entry per thing the room can do, each carrying its phrasings, its
action in a declared shape, its capability requirements, and a stated
safety class. When a discovering agent walks in, what it imports first is
not code; it is the intent surface, which its own matcher can embed and
serve locally from then on. The room teaches the visiting agent's spinal
cord its verbs.

Two parts of pincher's shape transfer with their *meaning* intact, which
is why this isn't a loose analogy:

- **`capabilities` is what makes safety checkable.** pincher's veto engine
  and its signed `Capability`/`CapabilityManifest`/`CapabilityToken`
  machinery (`pincher-core/src/security/`) can only gate what is declared.
  A lever that says `required = ["subprocess", "network"]` is a lever a
  veto layer can reason about before anything runs; a lever that declares
  nothing is unauditable by construction. The capability field is not
  metadata — it is the hook the entire safety story hangs on.
- **`confidence_seed` is what makes trust gradual.** In pincher, a reflex
  earns its way to fire-without-confirmation through use (multiplicative
  confidence updates, success ×1.005 / failure ×0.95). A seed value in a
  manifest says where a reflex starts, never where it ends. Section 2
  leans on this hard.

And one part of pincher explicitly does **not** transfer, stated so nobody
imports the wrong half: a pincher reflex is a *private, learned* row —
per-pairing, mutable, resident in one agent's SQLite store, with a
confidence that reflects one operator's history. A room manifest is
*public, declarative, versioned in git*. The declaration travels;
**the confidence does not.** One captain's earned trust in a lever is not
another captain's, and any convention that let a published manifest carry
a high effective confidence into a stranger's agent would be smuggling
trust across the exact boundary Section 2 exists to guard.

### Layer three — the manuals (agent-perspective documentation)

The precedent here is also already in the org, in that same half-real
`AGENT.md`: one conventional, agent-addressed file at a conventional
place. Stripped of the persona theater and kept to what a cold operator
actually needs, the manual states in prose what the manifest states in
structure, plus the things structure can't carry:

- what this room is for, and — adoption-bar item 5 applied to rooms —
  the stated boundary of what it will **not** do;
- where state lives and what owns it (the DeckBoss discipline: whose
  storage, what never leaves the device, what syncs where);
- what a visiting agent must **verify before acting**, not just before
  reading — the room's own re-verification checklist, the same
  "re-verify before executing" rule `ROADMAP.md` already applies to
  fork decisions, written down by the publisher for the visitor;
- the physical half, where there is one: which wire, which pin, what the
  human should see when it works — the PARADIGM essay's fourth step
  ("tells you how to build the other half"), which no manifest field can
  express and which is the half only the human can execute.

Whether the levers live inside the manual file, beside it, or in the
manifest proper is exactly the kind of open question a real design pass
settles and this paper doesn't. What this paper does claim: **three
layers, each with a shipped ancestor** — the tagging research's
topic+manifest for the door, pincher's skill shape for the levers, the
AGENT.md kernel for the manuals — is the honest minimum, and nothing on
that list requires inventing a mechanism this org hasn't already built
once in some adjacent form.

---

## 2. Discovery and trust for a community of rooms

### The position, stated first

**Caveat emptor, with the visiting operator's own agent doing the vetting
at point of use — and the graduation bar deliberately not extended to
third-party rooms.** Not both hands raised: the second option (an
equivalent adoption bar a third-party room must pass before another
captain's agent trusts it) is rejected, for three reasons that are each
already on this org's record.

**First, the constraint already decided it.** The tagging research's own
closing finding was that every real discovery system does its trust work
centrally — a curated list, an authenticated registry — and that the
no-operated-backend constraint rules that layer out, leaving the
quality-filtering work "to be done by every discovering agent, every
time." A mandatory bar for third-party rooms would need someone to
administer it; the administrator is an operated trust backend with extra
steps; a "verified room" badge or list run by this org fails on the same
axis the fleet-learning rejection named — who operates the infrastructure.
The door that rejection left open ("tools that read exports users chose to
push to their own storage — never anything DeckBoss itself operates") is
the door this stays inside.

**Second, the bar is a publisher's discipline, and this org is not the
publisher.** The graduation checklist is something this org applies to
*itself*, to code it ships under its own name. It has no standing to
impose that on a stranger's account, and — the sharper point — no ability
to verify it stays true after the moment of checking. The org's whole
research program exists because self-description drifts from reality;
a bar checked once at "admission" and never again would reproduce the
exact staleness problem `ROADMAP.md`'s "re-verify before executing"
rule was written against.

**Third — and this is what makes caveat emptor viable rather than
reckless — the org has already measured that point-of-use vetting is
cheap.** The two-layer intake's own record: the layer-one mechanical
checks (does CI actually pass unmasked, does the benchmark measure
anything, does the claimed artifact exist) caught ~95% of the
docs-oversell cases across the entire mission for free, and the one time
layer two was measured, four cheap models found two real bugs for about
$0.50. Trust-at-read-time is not an aspiration; it is a costed, practiced
procedure. The community-of-rooms trust model is that procedure, run by
the visitor, scaled to the visit.

### What that means mechanically

The three trust artifacts line up with the three things a room ships:

- **The manifest is a claim.** Nothing more. An agent that treats a
  manifest as a credential has recreated the "trust the README" failure
  mode this org exists to catch. Anyone can put any topic on any repo —
  the tagging research's own words — and anyone can write any manifest.
- **The intake is the check.** Before a visiting agent operates a foreign
  room, it runs the same two-layer intake this org runs before a fork,
  scaled down and at read-time: mechanically, do the declared levers
  correspond to code that exists and tests that actually run unmasked;
  adversarially (when the stakes warrant the $0.50), what does a cheap
  panel find wrong with it. The room's manuals can *assist* this — a
  publisher may include their own intake report in-repo, the way this
  org's graduated forks publish their adversarial-review findings in
  their READMEs as a condition of graduation — but a visiting agent
  treats a self-reported intake as one more claim to spot-check, never
  as a substitute for its own.
- **Confidence is earned, locally, afterward.** A foreign room's levers
  enter the visiting agent's reflex store at the confirm tier — low seed,
  never Exact — regardless of what the manifest asked for, and climb the
  same multiplicative ladder every local reflex climbs. Unattended
  operation of someone else's room is not granted at discovery; it is
  the *result* of a history between this operator and that room. pincher
  already ships this exact ladder; no new trust mechanism is required,
  only the rule that manifests can seed but never vault it.

Claim, check, earn. The one thing this org can contribute without
operating anything is the **convention itself**: publish its own intake
checklist in a form a stranger's agent can run, the way it already
publishes the adoption bar. That is a document, not a service — the
difference between handing every captain the org's soundings and running
a harbor authority.

---

## 3. "Voice proposes an automation, the system assembles it"

### The answer is yes, it is the same shape — said plainly, then qualified honestly

pincher's core loop is the mechanism, already built, already graduated:
an utterance is embedded locally; ≥0.80 similarity fires a known reflex
immediately with no LLM call (~50ms, $0); 0.55–0.80 fires with
confirmation; **below 0.55 — a genuine miss — the intent routes to the
LLM-as-compiler, which produces a new action; the result passes the veto
engine, runs in the sandbox, and is stored as a new reflex with a
confidence that then lives or dies by results**
(`pincher/ARCHITECTURE.md`, "Data Flow"). The tagline on that repo is the
mechanism's whole theory: *vector DB as runtime, LLM as compiler.* And
`docs/ACTIVELOG_ARCHITECTURE.md` §4 has already ported the seam's shape
to the browser tier — the `matchWakePhrase` seam that starts as exact
match and can later host an embedder behind the same interface.

A captain saying *"every morning, pull the overnight track, compare it
against the log's gear-set entries, and tell me which sets drifted"* is a
miss. No local reflex matches it. It routes to the compiler. The only
thing that changes in the room world is the **compiler's target
vocabulary**: instead of emitting one shell command, it emits a plan over
the *declared levers of N rooms* — each lever already a typed, phrased,
capability-annotated action, because Section 1 put it in the manifest.
The compiled plan is stored as a new reflex, exactly as today; the next
morning the same sentence is an Exact match and fires locally, no LLM,
no cloud. Compilation happens once; the automation becomes a reflex. The
"cortex teaches the spinal cord" line pincher's README kept on purpose
(`ROADMAP.md`, Step 2) generalizes from single commands to compositions
without changing what it means.

So: a room-composition request **is** reflex-compilation-on-a-miss,
generalized from "one new local command" to "a plan over declared
capabilities." Same trigger (Novel), same compiler role (LLM), same
storage (a new reflex), same afterlife (confidence earned by results).
Citing the existing mechanism is the point — inventing a second one for
this would be exactly the failure this org's discipline forbids.

### The three ways the shape genuinely differs

Saying "same shape" and stopping would be the confident-sounding version.
Three things are really different, and each changes a design constraint:

1. **The veto layer must move up an abstraction level.** pincher's veto
   engine pattern-blocks shell commands (`rm -rf /`, fork bombs). A
   composition's dangerous failure isn't a string pattern — it's a
   *capability combination* (this plan touches the network *and* an
   actuator; this plan writes to storage the operator didn't name). The
   veto for compositions therefore has to check the plan's aggregated
   capability requirements against what the operator granted — which is
   only possible because the manifests declare capabilities per lever,
   and which pincher's existing signed-capability machinery
   (`Capability`/`CapabilityManifest`/`CapabilityToken`) is the natural
   ancestor of. Existing mechanism, new altitude.

2. **First fire cannot be silent.** A compiled single command is cheap to
   show the human before it runs. A cross-room automation is *designed*
   to run unattended, later, repeatedly — which means its first execution
   is the only guaranteed moment a human is present, and it should be
   walked in the partnership's own debugging register (PARADIGM: the
   human reads the world, the agent reads the code; each is the other's
   only sensor into a domain it cannot enter). Confirmation on first fire
   is not a UX nicety here; it is the structural difference between an
   automation and an incident.

3. **The compiler's inputs now include strangers' claims.** Compiling a
   local command, the only untrusted input is the LLM's own output, and
   the sandbox bounds it. Compiling across rooms, the plan inherits the
   truthfulness of N third-party manifests. This is the real coupling
   between this section and the last one: **Section 2's intake is a
   precondition of Section 3's assembly**, not a parallel concern. An
   assembler that composes unvetted rooms has automated the act of
   believing READMEs.

### The conference of managers, located honestly

The other half of the vision — persistent agents a captain can address
one at a time or in conference, directing their attention across a
transcript collection or dispatching research — has its coordination
prior art already graduated: `git-native-agents`, where each agent is a
repo, each message a committed file in an inbox, each memory a tag —
`spawn`, `send`, `tick`, no broker, no database, hardened past a real
`.git/index.lock` concurrency race with a dedicated test suite. The
resonance with this paper is worth stating because it isn't decorative:
in `git-native-agents` an *agent* is already a repo; "a room can be a
repo" arrives at the same primitive from the other side. A room is a
repo an agent steps into; a git-native agent is a repo an agent *is*.
That the org's coordination substrate and its (sketched) capability
substrate land on the identical primitive — a git repository as the unit
of place, memory, and audit — is the strongest structural argument that
the room direction is an extension of things already real here rather
than a new invention.

Honesty requires the other half of that repo's record too: the deep
review (`docs/research/git-native-agents-deep-review.md`) found the
maintainer's own demo fleet was silently lost to the nested-repo design
(gitlinks with no `.gitmodules` — five empty directories where the
example agents used to be), among other real, since-hardened bugs. The
primitive is real and graduated; the "conference of managers" is a **use
pattern on top of it that nobody has built**, and this paper adds no
mechanism for it beyond noting that the substrate it would stand on
exists and has been independently reviewed.

---

## 4. "Openclaw managers" and "a scout" — resolved the honest way

Both names appear in the source vision, both are evocative, and neither
is defined anywhere in this org's hardened record. The honest move is not
to invent confident definitions here. It is to report what the record
actually contains — which turns out to be more interesting than an
invented definition would have been — and to flag the naming decision as
its own future pass.

**"OpenClaw" is a real, pre-existing name with real baggage.** It is a
SuperInstance-branded agent persona (`docs/research/openconstruct-cluster.md`
notes it alongside `cudaclaw` and an `openclaw.md`), and its on-record
work product is the 15-repo, single-day, identical-subject "world-class
README rewrite (zero-shot agent audit)" pass across the openconstruct
cluster — with a leaked template instruction (`.openclaw/workspace/...`)
found verbatim in a generated file, next to fabricated star counts
(`docs/research/papers-inventory.md`). In other words: the name fits the
org's claw lineage perfectly (`purplepincher`, `pincher`, the `.nail`
format, `gastrolith`, the hermit-crab argument itself), and the only
thing ever shipped under it is the exact bulk-generated overclaim pattern
this org exists to catch. The *role* Casey is pointing at — a persistent,
direction-owning manager agent, addressable in conference or alone — is
coherent and has a real substrate (a coordinator-role agent in
`git-native-agents`). The *name* should not be adopted by this paper's
fiat: taking a sketchbook name means the same thing here that taking a
sketchbook repo means — the thing behind it gets rebuilt to the bar
first, and whether the name is worth carrying despite its record is a
deliberate branding decision for a dedicated naming pass, not a side
effect of a direction paper.

**"Scout" already existed in this org once — and the record of what it
was is literally empty.** `agents/scout` was one of the five agents in
`git-native-agents`' own maintainer demo fleet (alongside architect,
builder, analyst, memory-keeper). The deep review found the entire demo
fleet unrecoverable — dangling gitlinks, no content, five empty
directories. The org's only on-record scout is a name with nothing behind
it; the definition did not survive. The role Casey describes — dispatch
an agent to analyze and research and report back — is exactly the
worker-role shape `git-native-agents` ships (`spawn` a worker, `send` it
a charter, `tick` it, read its outbox), and it is also, at the level of
method, the move this org's entire research program is made of: go
search, verify against primary sources, report what's actually there.
A grounded definition is available and nearly writes itself. It is still
not adopted here, for the same reason as above: this paper's job is
mechanism-direction, and names that will appear in captains' mouths
deserve their own pass, made on purpose.

One sentence of guidance for that future pass, offered rather than
decided: the org's naming has been strongest where the crab biology was
doing real argumentative work (the shell, the vacancy chain, the molt)
and weakest where it was persona theater (the Ensign files' fleet
tables). "Scout" needs no metaphor and already describes the job;
"OpenClaw" is all metaphor and carries a record. That asymmetry is
probably the answer, but it should be chosen, not defaulted.

---

## 5. Where this belongs in the org's document taxonomy

The org already runs a deliberate two-register split for exactly this
idea, documented in both files' own editorial notes: the *argument* lives
in `docs/PARADIGM.md` ("the next step of an argument belongs in the essay
making it"), and the *decision-record distinction* lives in `ROADMAP.md`
("distinctions between decisions belong where decisions are recorded").
The sharing chapter exists in both, on purpose, in different registers.
So the placement question for this content is really: which register is
it in, and the answer is — mostly neither, and that's informative.

**It is not a ROADMAP entry**, because nothing in it is a decision. The
sharing chapter's honesty paragraph ("Nothing in this section exists…
all open questions") remains true word for word after this paper, and
this paper must not be allowed to soften it. The only ROADMAP change this
direction earns, *when that file is next updated for its own reasons*, is
one folded-in pointer from the sharing chapter to this document — folded
in, not appended, per "How this roadmap gets updated" ("new findings get
folded in, not appended as a separate pile"), and phrased as "the open
questions now have a candidate shape on the record," not as "designed."

**It is not, wholesale, a PARADIGM section** — though one beat of it will
eventually be. PARADIGM's own editorial rule is that every paragraph must
move the reader toward one realization, and manifest field tables, trust
mechanics, and veto-layer altitude arguments would swamp the essay
exactly the way the first editorial note says a slow build would swamp
the README. But the *narrative kernel* here — the vacancy chain's next
beat, the crab that doesn't strip the shell for parts but moves in, the
repo as a place built around the agent as first-class reader — is the
"Back to the crab" argument continued, and by PARADIGM's own stated rule
that beat belongs in the essay, as "The second boat" did, when the
direction has matured enough to deserve the essay's permanence. That is
a recommended future edit, deliberately not performed by this paper: it
would extend PARADIGM's bright-line honesty section ("The second boat
does not ship either") with a third does-not-ship clause, and edits to
that load-bearing section deserve their own focused pass.

**What it is**: the third register both existing documents point at as
missing — the mechanism sketch. The org already established this
register's rules in `docs/research/ILLUSTRATIVE_MANIFEST_SKETCH.md`:
concrete enough to be discussed and compared against precedent,
disclaimed clearly enough that no one builds from it without a design
pass. This paper differs from that file in one way that argues for a
different shelf: the manifest sketch is a research aid (one illustrative
artifact, downstream of one research question), while this is synthesis
across the org's own shipped mechanisms — it cites `docs/research/` but
is not itself a verification of anything external. So: **`docs/ROOMS.md`,
beside `PARADIGM.md` rather than under `research/`** — direction-paper
register, essay-adjacent, mechanism-facing — with the two pointers above
(one folded into ROADMAP's sharing chapter, one eventual beat in the
essay) as the recommended integrations, each made in its own register by
its own pass, the way the org made them for the sharing chapter itself.

---

## What is real, stated plainly

Nothing proposed in this document exists. There is no room manifest, no
convention for one, no agent that walks into a foreign repo and imports
its intent surface, no cross-room composer, no capability-level veto for
compiled plans, no conference of managers. The chapters this all sits on
top of don't ship either: the ESP32 chapter is a direction, the sharing
chapter is a direction, and the ActiveLog core (Step 5) — the nearest
thing to a first surface any of this could ever land on — was sequenced
*today* and has not started. This paper adds no step, changes no gate,
and requests no engineering time; Step 1's field beta and Step 5's
core-first sequencing stand exactly as written.

What is real, and is the entire reason this direction is worth putting on
the record now:

- **pincher** — graduated. The reflex loop (embed locally, fire Exact,
  confirm Similar, compile Novel via LLM, veto, sandbox, store, earn
  confidence) and the skill-file declaration shape (`intent` phrasings,
  `capabilities`, `action_template`, `confidence_seed`) both ship today.
- **git-native-agents** — graduated. Spawn/send/tick coordination on git
  primitives, concurrency-hardened, independently deep-reviewed with its
  real bugs on the record.
- **plato-engine-block-c** — graduated. A tested C99 runtime for a
  sensor/actuator/alarm "room"; the word's real ancestor at the embedded
  tier, with `plato-room-configs` as a second real artifact in the
  otherwise-thin sketchbook cluster.
- **DeckBoss** — shipped. The storage-ownership discipline (user-owned
  storage, additive log, credentials never crossing the sync boundary)
  that any room's "where state lives" manual would inherit.
- **The tagging research and the manifest sketch** — verified precedent
  that topic+manifest is the smallest workable discovery convention with
  no operated backend, and one illustrative manifest already on record.
- **The two-layer intake** — a practiced, costed procedure (mechanical
  checks free, adversarial panel ~$0.50) that makes read-time trust a
  budget line instead of a hope.

Every mechanism this paper reaches for is on that list. That is the
paper's actual claim, and its only one: the room direction requires
assembling things this org has already built and verified — not
inventing new ones — and the assembly itself is the open design work,
honestly gated, honestly not started.

---

*Direction paper. The essay register is [PARADIGM.md](./PARADIGM.md); the
decision record is [ROADMAP.md](../ROADMAP.md) — this paper decides
nothing and is subordinate to both. Sibling citations resolve in
`purplepincher/pincher`, `purplepincher/git-native-agents`, and this
repo's `docs/research/`.*
