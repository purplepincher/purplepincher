# Next missions — the beta-wait window

Status: plan of record, ready to dispatch. Positions below are stated, not
offered as menus — reasoning is included so a later session can overturn them
deliberately. This plan sits downstream of `ROADMAP.md` and does not
re-litigate anything decided there; it answers the narrower question of what
to actually do *now*, given where all three workstreams verifiably landed.

Facts this plan is built on (all verified directly against the repos this
session, not recalled): DeckBoss is a hardened MVP with a live landing page
at `https://purplepincher.github.io/deckboss/`; its only unchecked roadmap
item is human ("start recruiting field testers"). `purplepincher/pincher`
exists as a real fork with all four named polish gaps closed (CI test fixed,
veto trait extracted, `.nail` spec written, crates.io readiness audited, two
hardcoded-secret copies removed, mocks feature-gated, `silo-core` removed);
what remains is three decisions plus a publish, per
`pincher/docs/CRATES_IO_READINESS.md`. The `pincher-cli` package builds a
binary named `pincher` (`pincher-cli/Cargo.toml:1-2,14-15`) while the README
advertises `cargo install pincher`. `ternary-types` is unpublished on
crates.io, used shallowly in `pincher-core/src/route/mod.rs` (enum
comparisons, `i8::from`, negation/addition) and deeply in
`hybrid-bridge/src/ternary_bridge.rs` (`TritVector`, `TernaryMatrix`) —
per `pincher/docs/prep-notes/git-dependency-resolution-options.md`. Whether
`hybrid-bridge` should publish at all is itself an open question
(readiness audit, open question 5). The broad SuperInstance cluster-survey
phase is complete — every major naming family has had a real deep-dive.
The unpushed `VISION_superinstance.md` still exists in session scratch
(`scratchpad/superinstance-research/notes/`) and will not survive the
session. No Rust toolchain exists in this environment; pincher verification
must go through the repo's GitHub Actions CI.

---

## 1. The sequencing position: finish, don't start

**Position: the beta gate governs *starting new domains*, not finishing
already-started work — and by that reading, almost everything on the
candidate list should not happen.**

The tempting middle ground — "the beta is human-gated and might take weeks,
so do parallel groundwork: pre-evaluate `plato-semantic-search`,
`plato-engine-block-c`, `exocortex-mcp-ts` for future forks" — is wrong,
and it's worth saying exactly why rather than waving at the gate:

- **The research needed to decide is already done.** Every one of those
  repos has a completed, evidence-grade deep-dive with test suites actually
  run. "Fork evaluation" beyond that isn't evaluation; it's pre-adoption
  polish of tools no shipped product currently pulls in — Step 3+ by
  another name.
- **Pre-work on unadopted candidates has negative expected value under this
  org's own rules.** `ROADMAP.md`'s update policy requires re-verifying any
  candidate's *current* state before forking, because the sketchbook drifts.
  Groundwork done now would need full re-verification at adoption time
  anyway — so it buys nothing, and meanwhile it produces exactly the
  inventory-of-half-adopted-things this org exists to not be.
- **The failure mode isn't idleness, it's manufactured momentum.** This
  mission has already twice correctly declined to invent work (the
  CUSTOMIZING.md catch, the Phase-4 "no more ungated DeckBoss work" call).
  This is the third instance of the same discipline.

What *does* clear the bar during the wait — three categories, nothing else:

1. **Finishing pincher.** Step 2 was explicitly user-cleared and is 90%
   done. Leaving the org's namesake permanently one-decision-from-published
   violates "ship first, announce second" more than finishing it does — the
   org's own measure of success literally includes "`cargo install pincher`
   works." This is completion of an open thread, not a new domain.
2. **Keeping the org's own documents literally true.** Both meta-repo
   READMEs now contain factually false or stale claims (details in §4).
   Principle 4 makes fixing these mandatory, not optional polish.
3. **Cheap, gated-on-nothing investigation and human-bottleneck leverage.**
   The cocapn tarball audit (decision support the owner already has a doc
   waiting on), and materials that accelerate the actual gate — recruiting
   collateral for the field beta.

Everything else — fleet edge tier, plato forks, exocortex, vessel-tuner,
new DeckBoss features — **waits, genuinely.** Not because waiting is safe,
but because none of it has a pull yet, and adopted-without-pull is the
documented root cause of the 4,095-repo problem this org is a reaction to.

## 2. The pincher decisions (recommendations for the owner)

These are stated as recommendations with reasoning, the same register as
the landing-page plan's architecture call. Two of the three are decisions
the owner delegates or confirms; the third is irreducibly human.

### 2a. `ternary-types`: Option C, scoped — inline in `pincher-core`, don't publish `hybrid-bridge` at 0.1

**Recommendation: replace `ternary-types` in `pincher-core` with a minimal
local `Ternary` enum, and mark `hybrid-bridge` `publish = false` for the
0.1 release — which makes its deep `TritVector`/`TernaryMatrix` usage (and
its git dependency) irrelevant to publishing.**

Reasoning:

- The options doc treats the two usage sites as one problem. They aren't.
  `pincher-core`'s usage is shallow — an `i8`-convertible three-state enum
  with comparison, negation, and addition; roughly 100 lines to own
  outright. `hybrid-bridge`'s usage is deep — but `hybrid-bridge` is the
  crate the root README doesn't even list as user-facing, and the readiness
  audit already flags its publish intent as an open question. Answer that
  question "no, not at 0.1" and the deep usage stops blocking anything:
  cargo only checks the published crate's own dependency tree, and
  `pincher-cli` depends only on `pincher-core`. `hybrid-bridge` keeps its
  git dependency as an unpublished workspace member, losing nothing.
- Option A (publish `ternary-types` upstream) is *feasible* — the same
  person owns both accounts — but it's the wrong move: it would put a
  permanent, unretractable crates.io artifact under the SuperInstance
  account, whose entire honest framing is "unmaintained sketchbook," and
  make purplepincher's namesake depend on it forever. `ROADMAP.md` already
  ruled the ternary crates "don't currently carry enough independent value
  to earn a fork slot"; publishing one as a side effect of unblocking
  pincher is that fork slot through the back door.
- Option B (vendor under a new name, e.g. `pincher-ternary`) is the
  fallback, not the first choice: it works, but it puts ~2,000 lines of
  third-party code plus a *second new published crate* under purplepincher's
  name — more public surface for strictly less ownership than C, and a
  standing violation of "one toolkit, not a pile of repos."
- The one real risk in C is the seam: `pincher-core`'s `TernaryGraph`/
  `RoomGraph` APIs are public and `hybrid-bridge` consumes them, so
  swapping the enum type means conversion glue at that boundary
  (`ternary_bridge.rs` already has exactly this pattern for its local
  `TernaryGate` type, so it's an established shape, not an invention).
  The dispatch brief below (A1) requires mapping every cross-crate seam
  *first* and aborting to Option B if the refactor turns out deeper than
  the options doc's read — a cheap, explicit escape hatch.

### 2b. Naming: rename the package `pincher-cli` → `pincher`

**Recommendation: rename. The package should be `pincher`, matching the
binary, the product, and the README.**

- `cargo install` takes a *package* name. Every doc this org has written —
  pincher's own README, `ROADMAP.md`'s measure of success, the
  purplepincher README — says `cargo install pincher`. Keeping `pincher-cli`
  means either every one of those docs changes to the clunkier command
  forever, or the org's flagship claim is false on day one. Renaming one
  `Cargo.toml` field now, pre-publish, is the only moment this is free.
- The `-cli` suffix convention exists for projects where a same-named
  *library* crate owns the bare name (`wasm-bindgen`/`wasm-bindgen-cli`).
  Here the library is already separately named `pincher-core`, so the bare
  name is unclaimed by any sibling and the ecosystem-survey verified it
  unpublished on crates.io as of 2026-07-03. Take it.
- One caveat, deliberately owned by the human step: crates.io name
  availability can't be re-verified from this environment (their API blocks
  this environment's direct access). The publish checklist (A3) makes the
  ten-second re-check the first line of the publish procedure, per the
  roadmap's re-verify-before-executing rule.

### 2c. Publishing: human-only, but reduce it to a 15-minute checklist

No agent runs `cargo publish` — real credentials, hard-to-reverse public
action, exactly as the earlier dispatch briefs already established. What
agents *can* do is compress the human's job to a verified checklist plus a
CI dry-run that's already green (work items A1/A2/A3 below). Publish order,
for the record: verify names available → `cargo publish -p pincher-core` →
wait for index → `cargo publish -p pincher` (the renamed CLI). The README's
`cargo install pincher` line becomes true at that moment and not before —
until then, pincher stays off every "shipped" list (see §4).

## 3. The next fork candidate: `conservation-guardian` — and staged as use-first, not fork-first

**Position: the next SuperInstance repo worth real effort is
`conservation-guardian`, and the honest form of that effort is a two-week
internal usage trial, not a fork.** And explicitly: none of the others —
not `plato-semantic-search` (67/67 tests), not `plato-engine-block-c`
(35/35), not `exocortex-mcp-ts` (87/87), not `vessel-tuner`, not
`git-native-agents`.

The argument is the roadmap's own Step 4 logic applied consistently: the
gate is never "is the code good" — several of those repos are verifiably
good — it's "does something this org ships or runs actually pull it in
today." Walk the shortlist against that bar and exactly one survives:

- `plato-semantic-search`, `plato-engine-block-c`, `plato-core`: real,
  tested, and pulled in by nothing. They're Step-3-adjacent infrastructure
  for a fleet tier the org has deliberately not started. Watchlist, where
  they already are.
- `vessel-tuner`: scans a Cloudflare Workers fleet. The org operates zero
  Workers. Its pull arrives with Step 3, not before.
- `exocortex-mcp-ts` / `exocortex-kernel-c`: the two solid artifacts in an
  otherwise-hollow family — but agent memory infrastructure serves no
  shipped product. The watchlist entry already says exactly this.
- `git-native-agents`: the closest second place, because an internal pull
  arguably exists (this org runs a multi-agent build process). But the
  deep review showed it fails its own core claim at 12 concurrent writers,
  and the org's existing worktree discipline already does the job it would
  do. Harden-in-place stays available if the current process ever breaks;
  it hasn't.
- **`conservation-guardian`: the only one with a live, felt, *current*
  need.** This org's build process runs four non-Claude toolchains
  specifically because usage budgets are a real constraint (it's a standing
  instruction in this project's own memory), and budget enforcement for
  LLM/agent workflows is precisely what this tool does. It's also the
  cheapest possible adoption: already published on PyPI, clean CI across
  three Python versions, test suite verified by actually running it — so it
  can be *used* today with zero fork cost. If the trial (work item E) shows
  the org wants behavior it doesn't have, that's the moment a fork earns a
  slot, with real internal usage data behind it instead of a hunch. If the
  trial shows it's shelf-ware, the org spent one small dispatch learning
  that, not a fork-and-polish cycle.

## 4. Non-fork missions: the doc refreshes are overdue, and the cocapn audit happens now

**The meta-repo refresh is not premature — it's mandatory, because both
documents now assert things that are false.** This isn't "add pincher
marketing"; it's principle 4 (every claim literally true) applied to the
org's own front door:

- `purplepincher/purplepincher/README.md` says "One repo, right now"
  ("Why this org is small on purpose") — false; the org has three
  (`deckboss`, `purplepincher`, `pincher`). Its "What's coming" list, item
  1, still says nobody has diffed the deckboss families — done, per
  `ROADMAP.md` Step 0's own "Status: done." Its agent-guardrails section
  says SuperInstance names "are proposals until they appear as a repo under
  `github.com/purplepincher`" — `pincher` now *does* appear there while
  being deliberately unshipped, so the guardrail as written now actively
  misleads the agents it exists to protect. The refresh must present
  pincher as exactly what it is: forked, hardened, unpublished pending
  owner decisions — on no "shipped" list until `cargo install pincher`
  works.
- `docs/VISION.md` Step 0 says "Nobody has yet diffed that family" and
  Step 2 lists four pincher gaps as open — all four are closed. Same
  document-integrity rule from `ROADMAP.md`'s own update policy: fold
  findings in, don't let the doc and reality diverge.
- `ROADMAP.md` Step 2 itself needs a status block (fork live, gaps closed,
  three decisions pending → pointer to this plan), same pattern as Step 0's.
- One salvage item with a hard deadline: `VISION_superinstance.md` exists
  only in this session's scratch directory and dies with the session. It
  was always meant to mirror into `SuperInstance/SuperInstance/docs/VISION.md`
  (the same README↔VISION pattern purplepincher already uses). Push it now
  or re-derive it later from nothing.

**The cocapn namespace audit also happens now.** It's the decision doc's
own recommended first step, it's cheap (download a tarball, diff against
three repos' histories, check PyPI maintainer state), it's investigation
rather than adoption (Step 4 stays fully gated), and its output only
appreciates — whichever future session executes Step 4 needs this answer,
and the evidence (PyPI state, repo histories) can only drift further apart
in the meantime. Deferring cheap evidence-gathering on an already-open
question isn't discipline, it's just letting the trail go cold.

Also in this category: `ROADMAP.md` Step 0's concrete actions 2–4 (archive
the four toy `deckboss-*` repos, add disambiguation lines to `deckboss-ai`/
`DeckBoss`, relabel the three hardware repos) were decided and — as far as
any record shows — never executed. They're `gh` commands and one-line README
edits in an account the org has admin on. Verify-then-act (work item D).

## 5. Ordered work items (dispatchable briefs)

Priority order. A1→A2→A3 are sequential; B, C, D, E, F are independent of
A and of each other. Every brief is written for a dispatchee who has *not*
read this plan. Per standing practice: aider gets only single-new-file
documentation tasks; no agent touches credentials or runs a publish; all
merges get independently re-verified in the canonical branch by the
orchestrator, never trusted from self-report.

**P0 — restated, not dispatchable: recruit field testers.** Human item,
longest lead time, the actual gate on everything strategic. F below is the
only agent leverage on it.

### F. Field-beta recruiting collateral — **mmx/MiniMax** (text only)

The highest-leverage item in this plan, because it's the only one on the
critical path. Brief: *Draft recruiting and onboarding collateral for a
field beta of DeckBoss, a free voice-first fishing logbook PWA (source
material attached: the landing-page copy at
https://purplepincher.github.io/deckboss/ and the "Human action items"
section of the repo's ROADMAP.md — every claim must trace to one of those
two; invent no features, statistics, or testimonials). Audience: commercial
fishermen in PNW ports, skeptical of tech and of marketing. Deliver in
markdown: (1) a ~150-word recruiting post for fishing forums/Facebook
groups — plain declarative sentences, leads with "free, no account, works
with zero bars, your data stays yours," states plainly it's a beta and
we're asking for 3–5 boats for 6–8 weeks; (2) a dock-noticeboard one-pager
version of the same (larger claims-to-words ratio, tear-off-tab friendly);
(3) first-contact SMS templates for the device census ("what phone, what
browser?"), the week-one parallel-logging ask, and the daily check-in —
each under 300 characters, conversational, no forms; (4) a one-page
tester-facing "what we'll ask of you" sheet covering: keep your current
logging method running in week one, record at least once daily in genuinely
loud conditions, expect a 5-minute daily call/text. Do not promise support
SLAs, hardware, or compensation — none exist.* Output lands in
`purplepincher/deckboss/docs/beta-recruiting/` after orchestrator review
(mmx has fabricated URLs before — verify every link and claim before
committing).

### A1. `ternary-types` removal from the publishable set — **kimi** (worktree `pincher-kimi5`, branch `kimi/ternary-inline`)

Brief: *In the `purplepincher/pincher` Rust workspace: (1) Add
`publish = false` to `hybrid-bridge/Cargo.toml`'s `[package]` — this crate
is workspace-internal for the 0.1 release; do not change its code or its
`ternary-types` git dependency. (2) In `pincher-core`, remove the
`ternary-types` dependency by defining a minimal local ternary type (a
three-variant enum with the `i8` conversion, equality, `Neg`, and `Add`
semantics the existing code actually uses — read
`pincher-core/src/route/mod.rs` first and enumerate every operation
performed on `ternary_types::Ternary` before writing anything) and
updating `route/mod.rs` and its tests to use it. Preserve the public API
shape of `TernaryGraph`/`RoomGraph` (method names and semantics unchanged;
only the edge-weight type's crate of origin changes). (3) `hybrid-bridge`
depends on `pincher-core` — before coding, map every place `hybrid-bridge`
passes ternary values across that boundary; where it does, add explicit
conversions following the existing `TernaryGate` conversion pattern in
`hybrid-bridge/src/ternary_bridge.rs`. **Abort condition: if the seam map
shows `pincher-core`'s public API re-exports `ternary-types` types in ways
that force nontrivial `hybrid-bridge` rework (more than mechanical
conversions at call sites), stop, write the seam map to
`docs/prep-notes/ternary-seam-map.md`, and report back instead of forcing
it** — there is a fallback plan (vendoring) that a human will choose
deliberately. (4) Remove `ternary-types` from `[workspace.dependencies]`
only if no member still uses it (hybrid-bridge still will — leave it
declared for that crate alone). No Rust toolchain exists in your
environment: your deliverable is a pushed branch where GitHub Actions CI
runs the full test suite green. Do not touch CI configuration, the veto
trait, or anything outside the files named above.*

### A2. Package rename + publish dry-run in CI — **GLM/opencode** (worktree `pincher-glm2`, branch `glm/rename-and-dryrun`, after A1 merges)

Brief: *In the `purplepincher/pincher` Rust workspace: (1) Rename the
`pincher-cli` package to `pincher`: change `name` in
`pincher-cli/Cargo.toml` (the directory may stay `pincher-cli/`; update the
root `Cargo.toml` members path comment if any, and every doc reference to
the package name — grep for `pincher-cli` across `*.md` and fix each hit;
the binary is already named `pincher` and must stay so). (2) Verify the
README's install section says `cargo install pincher` and now matches
reality-to-be; fix any residual `cargo install pincher-cli` references.
(3) Add a CI job (extend the existing GitHub Actions workflow, don't
replace it) that runs `cargo package -p pincher-core` and uploads nothing —
packaging is the verification. Note and accept: a full
`cargo publish --dry-run` of the `pincher` package will fail until
`pincher-core` actually exists on crates.io, because its path dependency
carries a registry version; do NOT try to work around that (no dependency
rewriting, no `--offline` hacks) — instead have the CI job for the CLI
package run `cargo check -p pincher` and leave a one-line comment in the
workflow explaining why the CLI's dry-run is deferred to the human publish
step. (4) No Rust toolchain locally — deliverable is a pushed branch with
the new CI job green. Do not run or attempt `cargo publish` in any form,
with or without `--dry-run`, outside CI.*

### A3. Publish checklist doc — **aider (deepseek)** (single new file, after A2 merges)

Brief: *In the `purplepincher/pincher` repo, create exactly one new file,
`docs/PUBLISH_CHECKLIST.md`, and change nothing else — no source files, no
other docs, no Cargo.toml. Content: a step-by-step checklist for a human
with crates.io credentials and a local Rust toolchain to publish this
workspace, in this exact order: (1) confirm `pincher` and `pincher-core`
are both still unclaimed on crates.io (check the website, not just the API);
(2) `git pull` latest `main`, confirm CI is green on the exact commit being
published; (3) `cargo test --workspace` locally; (4)
`cargo publish -p pincher-core --dry-run`, review the file list it prints
for anything that shouldn't ship (secrets, scratch files, `docs/prep-notes/`
is fine to include or exclude — note it either way); (5)
`cargo publish -p pincher-core`; (6) wait for the crates.io index to pick it
up (~1–2 min, `cargo search pincher-core`); (7)
`cargo publish -p pincher --dry-run`, same review; (8)
`cargo publish -p pincher`; (9) verify from a clean directory that
`cargo install pincher` succeeds and `pincher --version` runs; (10) only
after step 9: update the repo README's status line and notify whoever
maintains `purplepincher/purplepincher` that the org has a second shipped
tool. Include a short "if something fails" note: crates.io publishes are
permanent — on a bad publish, yank (`cargo yank`) and publish a patch
version; never attempt deletion. State at the top that `hybrid-bridge` is
`publish = false` and intentionally excluded.*

### B. cocapn namespace audit — **GLM/opencode** (investigation only, no writes to any SuperInstance repo)

Brief: *Investigation task, evidence-gathering only — you must not rename,
archive, publish, or modify any repository or package. Background: the PyPI
package `cocapn` (currently 0.3.0) is claimed as the package name by three
different GitHub repos — `SuperInstance/cocapn`, `SuperInstance/cocapn-py`,
`SuperInstance/cocapn-python` — and a prior research pass found the
published artifact matches none of their current source. Your job: (1)
`pip download cocapn==0.3.0 --no-deps` (and list what other versions exist
on PyPI — download the oldest too if cheap), unpack the sdist/wheel, and
inventory its actual module structure and metadata (name, author, version,
description). (2) Clone all three repos with full history and search each
repo's history (not just HEAD — `git log --all --diff-filter=A`, per-file
content search) for the published artifact's distinctive files/functions.
State plainly which repo, at which commit, best matches the published
0.3.0 — or that none does. (3) Record what PyPI publicly shows about the
package's maintainer account(s) and upload dates. (4) Deliver a findings
document with evidence (file-level diffs or hash matches, not vibes) and a
one-paragraph recommendation for which repo should own the name going
forward, explicitly framed as decision support for the project owner.
A prior hypothesis doc exists claiming `cocapn-py` is the likely owner
(in-repo version 1.0.0, most passing tests) — test that hypothesis against
your evidence; do not assume it.* Deliverable →
`purplepincher/purplepincher/docs/research/cocapn-namespace-audit.md`,
committed by the orchestrator after verification, with the existing
`cocapn-namespace-decision.md` updated to point at it (the standing
point-don't-overwrite correction pattern).

### C. Meta-repo truthfulness refresh — **kimi** (worktree on `purplepincher/purplepincher`)

Brief: *In the `purplepincher/purplepincher` repo, three files need updating
because reality moved and the org's rule is that its docs never lag reality.
Hard constraint: `pincher` is forked and hardened but NOT published — it
must not appear as "shipped," a "product," or in any list a visitor would
read as an available tool. The org still has exactly one shipped product
(DeckBoss). Changes: (1) `README.md` — fix "One repo, right now" (the org
now has three repos: `deckboss`, this one, and the in-preparation `pincher`
fork); update "What's coming" item 1 (the deckboss-family diff is done —
point to ROADMAP Step 0's findings instead of saying nobody has done it);
update item 3's pincher line to reflect "forked and hardened, publishing
pending final owner decisions"; and amend the agent-guardrails bullet that
says SuperInstance ideas "are proposals until they appear as a repo under
github.com/purplepincher" — a repo now appears there without being shipped,
so the guardrail should say the authoritative signal is being listed as
shipped in this README, not mere repo existence. (2) `docs/VISION.md` —
Step 0 says "Nobody has yet diffed that family": mark it done with a
pointer to `ROADMAP.md` Step 0. Step 2's four named pincher gaps are all
closed: say so, with the remaining state being "publish decisions pending."
(3) `ROADMAP.md` — add a Status block to Step 2 in the same format as Step
0's ("Status: fork live and hardened; three publish decisions pending —
see `docs/FABLE_NEXT_MISSIONS_PLAN.md`"), folding in: veto trait extracted,
`.nail` spec written, secrets removed, mocks gated, silo-core removed,
crates.io readiness audited. Change nothing else — no restructuring, no
tone changes, no new sections beyond the status block.*

### D. Step 0 leftovers in the SuperInstance account — **kimi** (needs `gh` with SuperInstance access)

Brief: *Housekeeping in the SuperInstance GitHub account, executing
already-decided actions from `purplepincher/purplepincher`'s ROADMAP.md
Step 0 (read its "Concrete actions" list, items 2–4 — item 1 is done).
Verify-then-act on each: check current state first (a repo may already be
archived or relabeled), act only where the decided action hasn't happened,
and record what you found vs. what you changed. (1) Archive
`SuperInstance/deckboss-net`, `deckboss-agent`, `deckboss-ai-pages`,
`deckboss-net-pages`, and `deckboss-1` (`gh repo archive`) — marketing
pages, toy demos, and a stale fork per the diff research. (2) Add a
one-line disambiguation to the top of the READMEs of `deckboss-ai` and
`DeckBoss` (capitalized): they are unrelated to
`purplepincher/deckboss` (the fishing logbook) despite the name — one
sentence, link to purplepincher/deckboss, change nothing else in those
files. (3) Edit the repo descriptions (not READMEs) of
`deckboss-hardware`, `deckboss-marketplace`, `deckboss-fab` to prefix
"[hardware roadmap, not active]". Do not archive anything not named here;
do not touch `cocapn-foundation` or any `cocapn-*` repo (a separate
investigation is running); archiving is reversible but treat it as
significant — if any of the five repos shows commits newer than 2026-06-15,
skip it and flag instead.*

### E. `conservation-guardian` usage trial — **GLM/opencode**

Brief: *Evaluate whether `SuperInstance/conservation-guardian` (published
on PyPI) is worth adopting into a multi-agent software build process as a
budget/cost-enforcement layer. This is a usage trial, not a fork and not a
code review — a prior pass already verified its tests pass and CI is clean;
do not re-audit the code. Do: (1) `pip install` it exactly as its README
instructs — record whether the README's install and quickstart are
literally true (this ecosystem's docs have repeatedly overclaimed; that
alone is a finding). (2) Stand up its minimal configuration for a realistic
scenario: enforcing per-task budgets across scripted CLI invocations of
LLM tooling (simulate the workload if you lack live API budgets — the
question is the enforcement mechanics: how budgets are declared, what
happens at the limit, whether violations are blocked or merely reported,
what state it keeps and where). (3) Answer concretely: could this wrap
dispatches to CLI coding agents (kimi/opencode/aider-style tools) with real
enforcement, or does it only work inside Python code that imports it? (4)
Deliver a 1–2 page trial report: what worked verbatim, what didn't, the
integration shape it would need here, and a recommendation — adopt as-is /
fork-and-adapt / drop — with the single strongest reason for it.*
Deliverable → `purplepincher/purplepincher/docs/research/
conservation-guardian-trial.md` after orchestrator verification.

### G. Salvage `VISION_superinstance.md` — **orchestrator, this session, not delegated**

Not a dispatch — a deadline. The file exists only at
`scratchpad/superinstance-research/notes/VISION_superinstance.md` and dies
with this session. Push it to `SuperInstance/SuperInstance/docs/VISION.md`
(mirroring the README↔VISION pattern already established in
purplepincher/purplepincher, and consistent with the prior user
authorization that updated that repo's README), with a one-line link from
the README's tail. Verify the pushed content renders and the link resolves.

### H. Independent final verification — **orchestrator** (the gate on all of the above)

Per standing convention, no tool's self-report is trusted: A1/A2 get their
CI runs pulled via `gh api` (not screenshots of claims) and a same-class-bug
sweep (the double-secret lesson: when an agent fixes an instance, check for
siblings — for A1, grep the whole workspace for residual `ternary_types::`
outside `hybrid-bridge`); C and the B/E research docs get read in full
against the "must not present pincher as shipped" and "evidence, not vibes"
constraints before any push; D's `gh` actions get re-listed from the API
afterward to confirm archive/description state. Nothing merges on
self-report. A3's checklist then goes to the project owner along with this
plan's §2 recommendations — the publish itself, the naming sign-off, and
the ternary decision confirmation are the three human actions this plan
ends on, alongside P0 recruiting.

---

## What this plan deliberately does not contain

Restated so a future session doesn't mistake absence for oversight: no
Step 3 fleet work; no plato-*, exocortex, or vessel-tuner forks or
"pre-evaluations" (research is complete; adoption waits for a pull); no new
DeckBoss features (its own roadmap sequences Spot Memory and the rest
*after* field-tester contact, and its remaining pre-beta item is human);
no cocapn adoption (Step 4 stays gated — item B is evidence-gathering for
a decision doc that already exists); no `cargo publish` or PyPI/repo
mutations by any agent anywhere. If every item above completes and the
beta still hasn't started, the correct next action is nothing — check in
on recruiting, and wait.
