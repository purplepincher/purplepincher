# The `cocapn` PyPI namespace collision — a decision recommendation

Synthesized via mmx/MiniMax, 2026-07-04, building on the finding in
[`cocapn-family-deep-dive.md`](./cocapn-family-deep-dive.md): three
repos (`SuperInstance/cocapn`, `cocapn-py`, `cocapn-python`) all claim
the `cocapn` package name, and the artifact actually published on PyPI
(0.3.0) matches none of their current source.

**This is a recommendation for the project owner to weigh, not a
decision already acted on** — no repos have been renamed/archived and
no PyPI package has been touched.

## Best hypothesis (moderate confidence, not verified)

One of these three repos was likely the "async fleet engine" described
by the published 0.3.0 artifact at some point, and has since been
substantially refactored or repurposed, making current source
unrecognizable against what's live on PyPI. `cocapn-py`'s in-repo
version being `1.0.0` while PyPI shows `0.3.0` is a specific red flag —
either a separate, disconnected publishing pipeline existed, or version
numbers were bumped without ever re-publishing.

**This hypothesis is a starting point for investigation, not a
conclusion** — the actual published 0.3.0 tarball has not been
downloaded and diffed against any of the three repos' history in this
research pass.

## Recommendation, if a decision is wanted

Candidate: **`cocapn-py`** as the name-owning repo going forward —
reasoning: it's the only one of the three at a `1.0.0` version (a
stability signal), it has the most passing tests (36 vs. 27 vs. `cocapn`
core's own test count), and "thin Python SDK" is the more generally
publishable shape of the three. This is a candidate worth testing
against real evidence, not a final call — the audit step below should
inform or override it.

## Recommended action sequence, in order

1. **Audit first, before anything else.** Download the actual published
   `cocapn` 0.3.0 tarball from PyPI and diff its real source against
   all three repos' current history. This resolves the mystery with
   evidence rather than guessing, and should happen before any repo
   gets renamed or any package gets touched.
2. Check who currently holds PyPI maintainer/upload rights for the
   `cocapn` name.
3. If the 0.3.0 source traces to one of the three repos (even if it has
   since diverged), that repo is the natural owner — reconcile forward
   from there rather than starting over.
4. If 0.3.0 matches none of the three, that's a separate decision point
   (an abandoned/orphaned publish) with its own options, not covered
   further here.
5. Only after the above: consider archiving or clearly relabeling the
   non-owning repos so future visitors don't inherit the same
   confusion this research pass had to untangle.

**This document does not authorize any of the above — it's decision
support, pending the project owner's sign-off, same as the earlier
`ternary-types` git-dependency options in `pincher`'s own research.**
