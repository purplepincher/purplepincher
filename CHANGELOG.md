# Changelog

All notable changes to this file are documented here. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), adapted for an
org-level meta-repo: entries below are org milestones (graduations, policy
decisions, corrections), not code changes. `ROADMAP.md` remains the living
decision record with full reasoning — this file is a dated index into it.

This repo has no package manifest or registry publication; there are no git
tags. Everything below is grouped under `[Unreleased]`.

## [Unreleased]

### Added

- Eleven repositories graduated from the `SuperInstance` sketchbook:
  DeckBoss (the field-facing flagship), pincher, plato-semantic-search,
  plato-engine-block-c, vessel-tuner, git-native-agents,
  conservation-guardian, sonar-vision, exocortex-mcp-ts,
  constraint-theory-core, intent-directed-compilation.
- The adoption bar itself, written into policy rather than left as a
  one-off research recommendation: verified working end-to-end, docs
  rewritten to match reality, speculative framing stripped, published to
  a real registry, explicitly scoped — checked in that order, mechanical
  checks before any adversarial review (`5636f2a`).
- `GRADUATION_CHECKLIST.md`: the adoption bar translated into a concrete,
  repeatable check procedure (`98d449c`).
- The two-layer intake standard for candidate repos (`5636f2a`).
- The org's front door rewritten as an essay (`docs/PARADIGM.md`) rather
  than a pitch deck (`804ea8a`).
- The sharing chapter ("the second boat"): a captain's agent can publish
  a new capability as a public repo under the captain's own account, so
  other captains' agents can discover and build from it — recorded as
  direction, not shipped code (`326ae31`).
- Prior-art research (via mmx/GLM) grounding two claims the org makes
  about its own direction: the ESP32/voice-to-firmware mechanism isn't
  novel, only the trigger is (`8b0c69e`, `bf8d20d`); voice-triggered
  GitHub publishing and compatibility-tagging precedent for the sharing
  chapter (`fcc18f2`, `442ee8f`).
- Cluster-survey research program across the ~4,095-repo sketchbook
  (plato-*, vessel-*, cocapn family, cuda/oxide, dev-tooling, lau-*,
  exocortex, edge-*, nexus-*, budget-guardian) — closed after the last
  two large unexamined clusters found nothing further worth graduating
  (`32a7296`, `3e74542`).

### Fixed

- A "nine vs. twelve" repo-count discrepancy an outside-perspective audit
  found between the README and the actual graduated-repo list, plus two
  repos that had graduated but weren't yet listed (`b26b0fd`).
- The pincher README/ROADMAP Step-2 status mismatch Fable's review
  flagged (`fdd267f`).
- Two unverifiable claims caught before merging a README rewrite —
  independent verification applies to this repo's own docs too, not
  just the code repos (`1d59f1f`).
- `docs/VISION.md` demoted to a short pointer at README/ROADMAP/PARADIGM
  after drifting into factual contradiction with the roadmap twice in
  three days; kept in sync with the current graduated-repo count
  (`2ef0e13`).

### Changed

- README restructured for production-readiness: leads with concrete
  content (the repo index, the adoption bar) ahead of any argument; the
  hermit-crab metaphor and inversion narrative moved to `docs/PARADIGM.md`
  where the front door had already said they belonged (`4e60cda`).
