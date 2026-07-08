# Contributing to PurplePincher

Welcome. This org moves deliberately around a single product (DeckBoss) and a small set of supporting infrastructure repos. If you’re a new human contributor or an AI agent picking up work cold, this document tells you how the org actually operates — no process invented for this document, everything drawn from [ROADMAP.md](./ROADMAP.md), [GRADUATION_CHECKLIST.md](./GRADUATION_CHECKLIST.md), and the [README](./README.md).

## Operating principles and the adoption bar

Five principles govern every decision here:

1. **Ship first, announce second.**
2. **One domain at a time.**
3. **One toolkit, not a pile of repos.**
4. **Honest scale, honest docs.**
5. **Ruthless scope.**

A repo earns its shell — becomes an official PurplePincher repo — by passing the **six‑item adoption bar** in order. The checks are defined fully in the [Graduation Checklist](./GRADUATION_CHECKLIST.md). The intellectual reasoning behind each bar lives in [ROADMAP.md](./ROADMAP.md). Read both before proposing any new fork.

## How to pick up work

1. **Read the “How this roadmap gets updated” section of [ROADMAP.md](./ROADMAP.md).** The roadmap warns that its own factual claims go stale. Always re‑verify a claim against the *current* state of a repo before acting on it.
2. **Run the adoption‑bar checklist fresh** against the current state of any candidate repo. Do not trust a past pass — a five‑minute re‑check is cheap; discovering a fork decision was based on stale evidence later is not.
3. **Do not propose a fork without completing items 1‑6 of the checklist** (mechanical layer‑1 checks first, then layer‑2 adversarial review for survivors). The org has declined forks that looked promising on paper but failed the concrete re‑verification steps.
4. Individual repos may contain their own `CONTRIBUTING.md`; follow those for repo‑specific setup, testing, and style.

## What “done” means for a PurplePincher repo

A repo is **done** (graduated) when:

- It appears in the **“Graduated repos”** section of the README (currently 11 repos, listed there).
- It has met every adoption‑bar item, verified by a current run of the [Graduation Checklist](./GRADUATION_CHECKLIST.md).
- The repo’s README contains no claim that cannot be demonstrated in under a minute by a stranger following the install instructions.

## Center of gravity

The org’s current **center of gravity is Step 1 — the DeckBoss field beta** (see [ROADMAP.md](./ROADMAP.md)).  

Consequences:

- **Steps 3 and 4 do not receive serious engineering time** until the beta’s findings are digested and the product is improved accordingly (per the roadmap’s own words).
- New work that diversifies away from DeckBoss before that field feedback exists is not a priority.

## Practical steps for a new contributor

- **Pick up an existing graduated repo.** Each has an issue tracker. Look for “good first issue” labels or open design discussions.
- **If you’re an AI agent:** treat the roadmap as truthful but not authoritative — re‑check any statement before acting. Follow the checklist rigorously, publish adversarial‑review results in the README of any fork, and never invent new process beyond what is already documented here.
- **Before submitting a PR that changes behaviour:** verify the CI is green on that repo (use `gh api repos/purplepincher/<repo>/actions/runs`).
- **Licensing:** all PurplePincher repos use the MIT license.

## Further reading

- [PURPLEPINCHER README](./README.md) — the what and why.
- [ROADMAP.md](./ROADMAP.md) — the full decision record.
- [GRADUATION_CHECKLIST.md](./GRADUATION_CHECKLIST.md) — the concrete adoption‑bar procedure.
