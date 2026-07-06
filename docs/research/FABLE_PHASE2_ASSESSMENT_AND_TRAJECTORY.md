# Phase 2 assessment and the trajectory from here

*Fable 5 pass, 2026-07-05. Every factual claim below was verified live today
— fresh clones of `purplepincher/purplepincher` and `purplepincher/deckboss`,
the GitHub org API, crates.io's API, PyPI's API, and raw README fetches of
the graduated repos. Nothing is recalled from prior sessions or trusted from
summaries. Where I cite a prior Fable document, I re-read it in the clone
first.*

---

## Verdict up front

**Phase 2 is done, minus a punch list you can finish in about a day. It
should be closed deliberately, with a record — and the phase structure
itself should be retired with it.** The mission's center of gravity has
already moved, whether or not anyone announces it: the highest-value open
work is no longer research or synthesis, it's (a) four specific engineering
gaps that the org's own restore drill says gate DeckBoss's boat 1, and (b) a
15-minute human publish action. Everything else — more surveys, more essay
polish, any sharing-chapter code — is now displacement activity wearing a
mission's clothes.

The rest of this document is the evidence, then the ordered roadmap.

---

## Part 1 — Is Phase 2 essentially complete? Yes. Here is what "done" means here.

There is no external customer to sign off, so the honest internal signal for
a synthesis phase being done is this: **the phase's outputs stop pointing at
more of the phase and start pointing outside it.** By that test, Phase 2
closed itself sometime in the last day or two:

- The fork pipeline has converged. Eleven repos graduated; the last several
  cluster surveys of ~600+ repos each yielded 2–3 candidates apiece, and
  `FABLE_SYNTHESIS.md` — a verdict I stand behind on re-read — already
  concluded the mission's differentiating output is the audit *method*
  (now literally codified as adoption-bar item 6 and
  `GRADUATION_CHECKLIST.md`), not additional forks.
- The docs have converged. README, ROADMAP, PARADIGM, and the four
  prior-art honesty checks now form a coherent, self-correcting record —
  including one case (`tagging-precedent-findings.md`) where the research
  falsified the roadmap's own illustrative gesture and the roadmap was
  corrected in place. That's the system working as designed.
- The org's own plan of record (`docs/FABLE_NEXT_MISSIONS_PLAN.md`) already
  said the quiet part: *"If every item above completes and the beta still
  hasn't started, the correct next action is nothing."*
- The perception audit's one-change recommendation shipped. Verified live:
  `purplepincher/.github` exists, the profile README renders, and the org
  bio is now the exact grammatical sentence the audit proposed, verbatim.

**But "essentially complete" is not "complete," and the gap is checkable.**
Five items remain that are squarely inside Phase 2's original scope —
synthesis, fork decisions, polish, keeping the org's own claims literally
true. All five verified live today:

1. **The front door has a new literal-truth bug, of the exact class the
   outside audit warned about.** `README.md` line 84 says "Eleven
   repositories have earned the shell" (correct: 1 product + 10
   infrastructure). Line 196 — inside the "If you are an agent reading
   this" section, the org's explicit trust contract — says "**eight**
   supporting infrastructure repos." Ten are listed above it. This exact
   sentence renders on the org landing page via the synced profile README
   (verified in the raw fetch). Counting repos is the first thing an agent
   does; the org wrote that fact into its own audit
   (`FABLE_OUTSIDE_LOOKING_IN.md` §3) and is in breach of it again.

2. **`docs/VISION.md` has drifted into contradiction with ROADMAP.md —
   again.** VISION Step 0 still says "**Nobody has yet diffed that
   family**" (line 101) while ROADMAP Step 0 says "Status: done" with the
   full findings. VISION's adoption bar has five items; the real bar has
   six (two-layer intake is missing entirely). This is the *second* time
   this specific document has gone stale — `FABLE_NEXT_MISSIONS_PLAN.md`
   §4 ordered the same class of fix once already. ROADMAP.md's own update
   policy names this the cardinal failure: "A roadmap with two disagreeing
   answers to the same question is worse than no roadmap." My
   recommendation is structural, not another patch — see roadmap item 1.

3. **ROADMAP Step 0's decided actions 2–4 were never executed.** Verified
   against the GitHub API today: `SuperInstance/deckboss-net`,
   `deckboss-agent`, `deckboss-1`, `deckboss-ai-pages` are all
   `archived: false` with unchanged descriptions; no disambiguation lines,
   no hardware-roadmap relabels. These were decided in the roadmap, then
   re-ordered as dispatch item D in the next-missions plan, and still
   haven't happened. They are `gh` one-liners.

4. **The cocapn tarball audit was planned as "happens now" and never ran.**
   `docs/research/` contains `cocapn-namespace-decision.md` (the decision
   doc, correctly parked for the owner) but no
   `cocapn-namespace-audit.md` — the evidence-gathering dispatch (item B:
   download the published 0.3.0, diff against the three claimant repos'
   histories, check PyPI maintainer state) that the plan of record said
   should not wait. PyPI still shows 0.3.0, four releases, no author
   metadata. More on the nudge question in Part 2d.

5. **`cargo install pincher` still does not work.** Verified against
   crates.io's API: neither `pincher` nor `pincher-core` exists. The
   pincher README is honest about this (line 169 says so plainly — good),
   but ROADMAP Step 2 says "Status: done" while adoption-bar item 4
   (published) is unmet, and "cargo install pincher works" is literally one
   of the org's four stated measures of success. The remaining action is
   human-only (crates.io credentials) and the checklist for it already
   exists. This is the single cheapest high-value action available to the
   owner anywhere in the org.

Finish those five and write one closing paragraph into the mission record,
and Phase 2 is done — not abandoned, *completed*, with the same
deliberateness the roadmap applies to everything else.

---

## Part 2 — The load-bearing questions, answered with positions

### 2a. Stop the cluster surveys. This is the natural stopping point, and it's now.

**Position: let the two in-flight surveys (`superinstance-*`, `grand-*`)
land, fold their findings in, write a short closing note in ROADMAP.md, and
run no further cluster surveys. The survey program is over.**

The argument is the mission's own accumulated evidence, not fatigue:

- **The hit rate has collapsed.** Recent surveys of ~600+ repos found 2–3
  fork candidates each — and the org then declined to fork most of those,
  correctly, because nothing shipped pulls them in. A pipeline whose
  output is "candidates the gate rejects" is producing inventory, not
  value.
- **My own synthesis already made this call and it has held.** The
  differentiating asset is the method — three mechanical filters plus
  published adversarial review — and it's now written into policy. The
  method's value does not grow with more applications to the same account;
  it grows when the org's *products* earn attention and the method is
  visible behind them.
- **The escape hatch already covers the tail.** FABLE_SYNTHESIS Q5
  established the triage rule empirically: filter-failing repos get a
  second look only if a verified Tier-A repo vouches for them. Once
  `superinstance-*` and `grand-*` land, every major naming family in the
  account will have had a real examination. The remainder is chaff by a
  filter with a measured track record, and the escape hatch bounds the
  hidden-gem risk at effectively zero ongoing cost.
- **The roster is already at its outer limit.** The outside audit was
  right that twelve visible repos is past what "one domain at a time" can
  comfortably bear. Even a *great* new candidate now costs more in
  coherence than it adds in capability. The marginal survey isn't just
  low-yield — its best-case outcome is a repo the org shouldn't take.

One honest tension to put on the record while closing the program, because
the record is the product: **the org's graduations outran its own plan of
record.** `FABLE_NEXT_MISSIONS_PLAN.md` §3 explicitly declined
`plato-semantic-search`, `plato-engine-block-c`, `exocortex-mcp-ts`,
`vessel-tuner`, and `git-native-agents` ("adoption waits for a pull") — and
all of them, plus `sonar-vision`, graduated anyway, with no recorded
decision superseding the plan. The forks are real and done; I'm not
proposing to relitigate them. But the *rule* is now ambiguous: is the
adoption gate "a shipped product pulls it in" (the plan, and Step 4's
stated logic) or "it passes the quality bar" (what actually happened)? Step
3/4 gating depends on which one is true. Write one paragraph into ROADMAP.md
saying which rule governs going forward and why the plan was overridden.
Silence here is exactly the roadmap-vs-reality divergence the update policy
bans.

### 2b. DeckBoss: the beta is *not* purely human-gated, and that changes everything about what the team does next.

I checked the actual status rather than assuming, and the assumption
embedded in this mission's recent framing — "the beta is a human-world
action; the team waits" — is **stale in both directions**:

- **The human side is further along than the framing assumes.** The
  deckboss ROADMAP records: "**Update: fishermen are lined up.**"
  Recruiting collateral exists (`docs/beta-recruiting/
  RECRUITING_COLLATERAL.md`). The `cocapn-foundation` safety-doc mirror
  landed (`docs/cocapn-foundation-mirror/`). The long-lead-time human item
  is substantially de-risked.
- **The code side is *less* done than the framing assumes.** The restore
  drill — the org's own adversarial test of its single most load-bearing
  promise — ends with: "**Gates boat 1 … status: not yet clear to
  proceed.**" Specifically: the audio-rehydration gap is real and unfixed
  (no code path anywhere in `src/` ever calls `readBlob()` to restore a
  local copy — the retention policy's central promise is a decision that
  was made but never implemented); the stale-manifest single point of
  failure is reported and unmitigated (a file present in storage but
  absent from the manifest is invisible to every device's pull, forever);
  and the real-backend restore drill (Drive/R2, not `LocalZipAdapter`) has
  never run.

So the answer to "what should the team do while the human-gated step is
pending" is: **the human-gated step is not the gate right now — the team
is.** With fishermen lined up, every day the boat-1 code gaps stay open,
the bottleneck is the agents, not the humans. This is the most valuable,
fully-ungated, on-center-of-gravity engineering work available anywhere in
the org, and it is not speculative — the org's own drill named it:

1. Audio rehydration: a real design pass (background rehydrate vs.
   explicit download vs. stream-on-play — the drill says this needs a
   decision, not a guess) and then the implementation.
2. The manifest fallback-scan decision (`listFiles()`-based recovery
   sweep, with the cost tradeoff stated) and implementation.
3. A real-backend restore drill against R2/S3-compatible storage with
   test credentials — agent-executable end-to-end, and the drill doc
   already specifies the exact adversarial cases it couldn't reach from
   the in-memory adapter (OAuth expiry mid-recovery, eventual-consistency
   windows, the GoogleDriveAdapter prefix quirk).
4. The Whisper offline-retry queue, already named in the deckboss roadmap
   as the "more complete fix" and a week-2/3 candidate.

What stays human: device census, the hands-on setup session per tester,
the physical phone drill. Those are the last gates, and they're short once
1–3 land.

### 2c. The sharing chapter: hold the line. Zero code — including "clearly-labeled proofs of concept."

**Position: no prototype, and I'll argue it rather than hedge.**

The case *for* a small PoC is real and worth stating fairly: it's cheap,
the GitHub-API plumbing is proven feasible (the voice-publish prior-art doc
found a working tutorial doing exactly voice→GitHub-API), and a PoC
labeled "illustrative" seems consistent with an org that values showing
over telling. I reject it anyway, on three grounds:

1. **A PoC now would prototype the easy half and prove nothing.** The hard
   parts of the sharing chapter are the parts that don't exist upstream of
   it: there is no ESP32 chapter producing work to publish, no captain, no
   log for "publish this" to resolve against. What *can* be prototyped
   today — create a repo via API, put a manifest in it, search topics — is
   precisely what four rounds of prior-art research already established
   other people have built. The org would be writing code to re-verify its
   own citations. The in-flight manifest-file *sketch* (a document, not
   code) is the correct maximum: accept it, label it illustrative, stop.
2. **The gate is the org's most-repeated public commitment, and "clearly
   labeled" doesn't exempt code from it.** ROADMAP.md says the direction
   "gets no engineering time ahead of Step 1's gate, same as everything
   else in this file." The org's entire differentiation is that its stated
   rules bind its actual behavior — the outside audit showed exactly how
   expensive checkable rule-breaking is here. A repo of sharing-chapter
   code, however honestly captioned, is engineering time, and every future
   reader of the roadmap would be entitled to ask what else "no" means.
3. **The beta can reshape the direction.** The sharing chapter's whole
   premise sits on captains actually living with the log. Six weeks of
   real field usage is about to generate the first non-AI evidence this
   product has ever had. Prototyping the second boat before the first boat
   has left the dock isn't ambition; it's the sketchbook's failure mode
   with better production values — and the essay already has the correct,
   fully-honest form of this idea on the record, four-times
   prior-art-checked, at zero code. That's not an unbuilt liability. That
   was the deliverable, and it's finished.

### 2d. cocapn: nudge once — about the dropped audit, not the decision.

The decision (which repo owns the PyPI name) was correctly scoped as the
owner's call, and two days of silence on a gated-behind-Step-4 decision is
not a problem — Step 4 has no start date, so nothing is blocked. Silence
on the *decision* remains right.

But the **audit** is a different object, and it fell through the cracks:
the plan of record explicitly scheduled it as agent work, now
("Deferring cheap evidence-gathering on an already-open question isn't
discipline, it's just letting the trail go cold"), and it never ran —
verified by the absence of `cocapn-namespace-audit.md` in
`docs/research/`. The right move is: run the audit (one dispatch, no
mutations, pure evidence), attach it to the decision doc via the standing
point-don't-overwrite pattern, and then surface *one* line to the owner —
"the evidence you'd want for the cocapn call is now in the repo;
recommendation unchanged/changed; no urgency, it gates nothing until
Step 4." That's not nagging a parked decision; it's completing a dropped
work item and telling the decision-maker their inputs are ready.

### 2e. Two more things the evidence surfaced

**Stop maintaining three parallel decision records.** The org now has the
roadmap stated in full (ROADMAP.md), compressed (README), and at
two-thirds scale (docs/VISION.md) — and VISION.md has now drifted into
factual contradiction twice in three days. This is structural, not
carelessness: any document that restates rather than points will always
lag. Demote VISION.md to a one-screen pointer ("the decision record is
ROADMAP.md; the front door is README.md") or delete it. One fewer surface
where the org's literal-truth promise can silently break, forever.

**pincher's README is honest but still half-sketchbook.** The install
section now truthfully says crates.io publishing is pending (good — this
was a straight lie by omission in the outside audit's snapshot). But the
top of the file is still the philosophy-first essay, including the exact
"That's not cache. That's *understanding*" line the audit flagged, and
Step 2's own definition of done includes "replace the philosophy-heavy
README with a 60-second demo." Either do the rewrite or amend Step 2's
text to say the philosophy stays deliberately. Both are fine; the current
state — "done" step, undone item — is the one option the update policy
doesn't allow.

---

## Part 3 — The trajectory: retire the phases, adopt the org's own clock

One reframe before the list. "Phase 3" is the wrong name for what comes
next, because the phase structure belonged to the *mission* (survey →
synthesize), and the mission's output was an org whose ROADMAP.md already
contains the only sequencing that matters now: Step 1 gates Steps 3 and 4.
The team's identity shifts with it: from surveyor/synthesizer to
builder/maintainer, with smaller, pull-driven dispatches. The measure of a
good week stops being "documents produced" and becomes "is boat 1 closer."

Ordered. Each item says why it's above the next.

1. **Truth pass on the org's own surfaces** (hours, one dispatch).
   Fix "eight" → "ten" in README.md line 196 (and confirm the profile
   README sync propagates it); demote `docs/VISION.md` to a pointer;
   restate ROADMAP Step 2's status honestly ("fork live and hardened;
   crates.io publish pending owner sign-off — adoption-bar item 4 not yet
   met"); resolve the pincher-README/Step-2 mismatch either direction.
   First because it's the cheapest item on the list and the org's entire
   product is that these claims check out — a known-false claim on the
   front door outranks everything except data loss.

2. **DeckBoss boat-1 gate** (the real work, this week+). Audio
   rehydration design + implementation; manifest fallback-scan decision +
   implementation; real-backend restore drill on R2; then the Whisper
   retry queue. Exit criterion is written already: the restore drill's
   "not yet clear to proceed" flips, on evidence. Everything below this
   line is a side quest; with fishermen lined up, this is the org's
   critical path and the team owns all of it.

3. **The 15-minute human item: publish pincher** (owner). `pincher-core`
   then `pincher`, per the existing checklist. Verified today: both names
   still read as nonexistent on crates.io (re-check availability first,
   per the re-verify rule). This makes a measure-of-success line true and
   costs nothing. It sits third only because agents can't do it —
   surface it to the owner alongside this document.

4. **Close the loose Phase-2 threads** (two small dispatches, parallel to
   2). (a) Execute ROADMAP Step 0 actions 2–4 in SuperInstance,
   verify-then-act — confirmed unexecuted today. (b) Run the cocapn
   tarball audit, attach it to the decision doc, one-line nudge to the
   owner per 2d.

5. **Land and close the survey program** (when the in-flight dispatches
   return). Fold `superinstance-*` and `grand-*` findings into the
   decision record; hold any candidates to the full bar with no special
   momentum; then add the closing note: the account has been surveyed,
   the triage rule and its escape hatch govern the remainder, no further
   cluster surveys. Also write the one-paragraph reconciliation of the
   pull-based adoption rule vs. the actual graduation history (§2a) —
   the record should not contain a rule the org demonstrably doesn't
   follow.

6. **Declare Phase 2 closed, in writing** (one paragraph in the mission
   record, pointing at items 1–5 as the completed punch list). Cheap, and
   it prevents the failure mode this assessment exists to catch: a phase
   that never ends because nobody said it ended.

7. **Steady state during the beta: small, pull-driven, mostly quiet.**
   Answer issues. Keep READMEs literally true. Monitor the two live
   Workers so "broken" is caught before a user finds it. Support the
   daily tester check-ins with materials when asked. Explicitly *not*:
   new forks, Step 3/4 work, sharing-chapter code, new essays, more
   perception audits. The next-missions plan already wrote the standing
   order and it stands: if nothing needs doing, the correct next action
   is nothing.

8. **After the beta: the findings drive everything.** Fold field results
   into DeckBoss first. Only then does Step 3 (fleet edge tier) come off
   the shelf — with the pull question finally answerable from real usage
   instead of taste, and with the adoption rule from item 5 written down
   so the decision is governed, not vibes.

### What this roadmap says no to, in one place

No further cluster surveys after the two in flight. No sharing-chapter
code, labeled or otherwise, until Step 1's gate clears. No new
graduations during the beta window. No third copy of the roadmap. No
"Phase 3" — the org's roadmap steps are the clock now. And no manufactured
work while the beta runs: this mission has correctly declined to invent
momentum three times already; the steady state in item 7 is the fourth
time, made standing policy.

The mission's ethos has been that a well-evidenced "stop" is a real
deliverable. Phase 2's deliverable turned out to be an org that can now
tell its own team to stop researching and go finish the boat. That is
what completion looks like here.
