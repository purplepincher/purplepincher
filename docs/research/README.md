# Research index

The evidence base behind [`../../ROADMAP.md`](../../ROADMAP.md)'s claims and
[`../VISION.md`](../VISION.md)'s positioning. Produced by a multi-agent
research pass (kimi, GLM/opencode, Claude subagents, mmx/MiniMax) into the
sibling [SuperInstance](https://github.com/SuperInstance) account, 2026-07.

Every recommendation in the roadmap traces back to one of these documents.
If a roadmap claim looks surprising, the reasoning and citations are here,
not asserted from nothing.

| Document | What it covers |
|---|---|
| [`ecosystem-survey.md`](./ecosystem-survey.md) | The broad landscape: SuperInstance's real scale, the bulk-repo-creation pattern, verified package-publication status, the full fork-candidate shortlist. Start here. |
| [`11-named-repos.md`](./11-named-repos.md) | Deep dive on the vessel/edge-named `plato-*` repo family. |
| [`cocapn-technical-fit.md`](./cocapn-technical-fit.md) | Whether SuperInstance's embedded repos technically connect to `cocapn-foundation`'s vessel-autonomy design. |
| [`fleet-cluster.md`](./fleet-cluster.md) | Deep dive on the 319-repo `fleet-*` cluster — the strongest "genuinely live infrastructure" finding in the whole research set, verified with direct HTTP probes. |
| [`ternary-conservation-cluster.md`](./ternary-conservation-cluster.md) | Whether the "conservation law γ+η=C" is rigorous (it's Shannon's chain rule, correctly implemented, with an internally inconsistent cancellation formula on top). |
| [`plato-ng-investigation.md`](./plato-ng-investigation.md) | Whether `plato-ng` is the canonical framework `plato-vessel-core` claims it is (it isn't — a sibling sketch with its own internal contradiction). |
| [`deckboss-family-diff.md`](./deckboss-family-diff.md) | The most consequential finding: SuperInstance's `deckboss-*` family is mostly unrelated to fishing; the real design ancestor is `cocapn-foundation`. |
| [`git-native-agents-deep-review.md`](./git-native-agents-deep-review.md) | Empirically tested (not just read) — reproduced real concurrency bugs via live testing rather than repeating an unverified claim. |
| [`openconstruct-cluster.md`](./openconstruct-cluster.md) | Why the 22-repo `openconstruct` SDK family isn't a fork candidate. |
| [`edge-vessel-bridge-deep-dive.md`](./edge-vessel-bridge-deep-dive.md) | Corrects `ecosystem-survey.md`'s `edge-*` picks — neither held up under direct code inspection — and finds a better, previously unflagged candidate instead. |
| [`nexus-cluster-deep-dive.md`](./nexus-cluster-deep-dive.md) | Corrects `ecosystem-survey.md`'s `nexus-*` pick and finds a real, verified 190K-LOC outlier (`nexus-runtime`) the original survey missed entirely. |
| [`budget-guardian-cluster.md`](./budget-guardian-cluster.md) | Actually built and ran the 13-repo budget-guardian family's test suites — corrects an overly-even "these are real, useful tools" read with per-repo verdicts (one repo doesn't even build as committed). |
| [`lau-cluster-deep-dive.md`](./lau-cluster-deep-dive.md) | Confirms the 333-repo `lau-*` cluster is exactly what it looked like — thematically generated math/physics crates, no cross-repo integration, no hidden outlier. Not a fork target. |
| [`exocortex-deep-dive.md`](./exocortex-deep-dive.md) | Demotes `exocortex` from its Tier-1 ranking — the "S3-compatible" claim is fabricated and its tests have never once actually run — and finds the real fork candidates hiding in the family are two previously unlisted repos, not the Python core everyone assumed was the centerpiece. |
| [`mmx-industry-patterns.md`](./mmx-industry-patterns.md) | External research on real-world precedents for distilling a sprawling research account into a focused org. |

**A pattern worth naming**: independently, across at least five of these
documents, the same defect shows up — SuperInstance's docs describe
features, functions, or formulas that the actual code doesn't have or
contradicts. Treat any *unverified* corner of that ecosystem with the same
skepticism these documents apply throughout: check the code, not the README.
