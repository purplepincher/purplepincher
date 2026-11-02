# Recommendations for repo-native agents (Zero and its relatives)

**Filed:** 2026-11-02, by Muse (Confluence / rolling-perception lane),
at Casey DiGennaro's direction — "push all of your recommendations
to the document folder in the purple pincher organization where the
zero proof of concept really comes from, so another agent — or you
later — has these thoughts saved there."

**What this is:** a cross-pollination memo. One day of work on an
unrelated product (a video change-ledger tool, telltape) produced
four patterns that transfer directly to repo-native agents of the
Zero class. They are recommendations, not rulings: each carries its
reason and, where it can be measured, the result that would prove
it wrong. Nothing here asks the Zero line to change direction; it
records what a neighbouring build learned, in the place the Zero
line's own agents will find it.

**Sources, all public or in the fleet record:**
- telltape — https://github.com/SuperInstance/telltape (v0.1.1):
  a change ledger for video. Watches a stream or a recording,
  writes structured "tells" (what changed, where, when, how
  strongly), keeps no footage; the ledger is a seek index into
  footage the user keeps elsewhere.
- The leverage memo on Balestriero & LeCun, arXiv:2610.39302
  ("Proving and Improving the Platonic Representation Hypothesis
  with JEPAs"): SI-Thor/rolling-perception, docs/
  PAPER-LEVERAGE-2610.39302.md.
- TokenEconomizer: Microsoft Research + UC Berkeley, Apache 2.0,
  arXiv:2610.22732 — a plugin that learns per-repository cost
  policies for coding agents (Claude Code, Codex, Copilot CLI, Pi).

---

## R1. The repo is a stream, and git already keeps the footage —
     give the agent the other half: a tells ledger

A repo-native agent's world changes state commit by commit, and
git stores every state forever. What the agent lacks on re-entry
is not the record but the *index of what mattered*: an append-only
ledger of change verdicts — which areas changed, how strongly,
why, and what was verified — written by the agent as it works, in
structured text, in the repo itself.

- On waking, the think loop reads the ledger (small) instead of
  re-deriving the repo's recent history from diffs (large). The
  ledger is the seek index; commits are the tape it points into.
- Silence must carry its basis. An empty ledger stretch should be
  able to say which of three things it means: nothing crossed the
  gate / the watcher was not running / the feed was frozen. (In
  the video tool, a dead camera once read as meaningful silence
  for ~13 hours; the fix — freshness proven by novelty, receipted
  transitions — is the single most transferable defect lesson in
  this memo. For an agent: a heartbeat that attests *novel input
  seen*, not merely *process alive*.)
- Falsifier: if an agent with a tells ledger does not re-enter
  measurably cheaper (tokens-to-context) and at least as accurately
  as one re-reading diffs, the ledger is overhead, not an index.

## R2. Adopt TokenEconomizer's three mechanisms as patterns
     (whether or not the plugin itself ever fits Zero's loop)

The plugin's authors profiled coding agents and found the waste
is structural: repeated discovery across tasks, unused tool
schemas billed every turn, and failure loops — 17% of tasks
consuming 32% of spend. Their remedies are stated as arithmetic,
which is what makes them portable to a custom loop like Zero's:

- **Micro-skills** — one-line, per-repo facts learned from traces
  ("the working test command is X"), admitted after 3
  confirmations on tasks with non-overlapping edited files,
  required to keep a need rate ≥ 0.4, and retired fast: one
  contradiction counts as 5 confirmations. Zero's SELF-MODEL.md
  (evidence-cited, confidence-rated beliefs, maintained by the
  think loop) is the same organ grown locally; the recommendation
  is to borrow the *admission and retirement arithmetic* for it,
  so beliefs enter and leave the self-model by counted evidence,
  not by narrative momentum.
- **Failure-loop intervention** — a turn budget learned from the
  agent's own successful runs (1.25 × the 90th percentile), cheap
  rule checks for repeated failure, one redirect, then a stop.
  Zero's known worst failure — a think loop confabulating a
  completion with invented files — is this failure class. The
  v0.9.2 completion check fences it after the fact; a budget
  fences it during the run. Both fences, not either.
- **Tool pruning with re-audit** — mask tool schemas unused
  across 3 consecutive tasks; restore the full catalog every 5
  tasks for a re-audit. (Their largest single saving, 13–18%.)
  For a repo-native agent whose context is re-billed every cycle,
  this is the cheapest of the three to adopt and the easiest to
  measure.

Caveat recorded by the source itself: the savings compound over
repeated tasks in the *same* repository and are negative at first.
One-shot work in fresh repos is where these mechanisms earn the
least; long-lived agents in one repo — Zero's exact shape — are
where they earn the most. Reported effect (authors' numbers,
277 SWE-Bench tasks): −18.4% to −30.9% mean cost, cheaper in all
30 repository-setting cells, task success up in 4 of 5 settings.

## R3. The fences are one invention — state it once, apply it
     in every domain

Three builds this season independently arrived at the same rule:
*never let the model's narration outrun the record.*

- Presence (voice): every factual sentence the brain drafts is
  checked against the ledger digest by deterministic code before
  it is synthesized; failures are replaced with template truths
  and receipted.
- Zero v0.9.2: every file path a completion summary claims is
  checked against the tree; a claim naming a nonexistent file
  downgrades the outcome to completed_unverified, paths named.
- telltape: a verdict is a claim about *this* frame; stale scores
  must not re-fire, and precision is measured against corrections,
  never self-reported.

Recommendation: wherever a Zero-class agent produces a statement
someone will act on (a completion, a status, a summary of another
agent), route it past a checker that compares it against the
cheapest ground truth available (the tree, the ledger, the test
run) and receipts the comparison. The checker is code, not a
second model's opinion: judges that grade fluency have been
measured, in this fleet's own record, preferring a polished
dangerous answer to a plain safe one (2 of 6 on adversarially
polished options). A judge may advise; arithmetic decides.

## R4. A change of instrument is a lens change — receipt it,
     recalibrate, never silently continue

From telltape v0.1.1 (deficit D1 in the leverage memo): the ledger
originally guarded an embedder swap by output width alone, so a
same-width swap silently invalidated every baseline and the gate
calibration. The shipped fix: the ledger records the embedder's
identity (`name@version`); opening under a different identity
refuses unless the operator explicitly recalibrates, which
receipts the swap in the stream.

The transfer: a repo-native agent's self-model, learned facts,
and budgets are all calibrated *under an instrument* — a specific
model, tool set, and prompt shape. Swapping any of those is a
lens change. The agent's ledger should record the instrument
identity per era, and a swap should (a) be receipted as an event,
(b) mark prior calibrations as belonging to the old instrument,
and (c) re-earn budgets and fact confidences under the new one
rather than inheriting them silently. Baselines do not transfer
across instruments; pretending they do is how quiet drift gets
mistaken for the world changing.

## R5. Keep the self-model a *good view*: the Platonic warning

The cited paper proves (under stated conditions) that
representations learned from two views of one scene converge;
views assembled from fragments of a document — their "bad views"
case, which includes next-token prediction — align only weakly
and up to a general linear map, i.e. wording-sensitive by
construction.

The agent's self-model and the owner's view of the repo are two
views of one scene. They stay aligned while both are good views:
evidence-cited, checked against the tree, regenerated from
verified ground. The failure mode is the agent summarizing its
own summaries — each generation a slightly worse view — until
the self-model is a fluent description of a repo that no longer
exists. Operational form, already this fleet's doctrine in the
large: iterate in short rounds from a verified anchor; treat the
transcript as compost and the anchor card as the crop. For the
self-model specifically: beliefs cite the artifact they rest on,
and a belief whose citation can no longer be re-derived is
demoted, not defended.

---

## What this memo is not

- Not a spec, not a task list, and not a claim that the Confluence
  lane will build any of it. Lane discipline: these are offered
  to the Zero line's own agents and sessions, to pick up or set
  down as their work dictates.
- The one claim it does make on the reader: if R1–R5 are wrong,
  they are wrong in measurable ways, and the measurements are
  named inside each section.
