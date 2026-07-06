<!--
  Disclaimer: This document is an illustrative sketch, not a design decision.
  It explores one plausible shape for the in-repo manifest that ROADMAP.md's
  "sharing chapter" gestures at but explicitly does NOT decide. The concrete
  fields and values below are meant to show what such a manifest *could* look
  like, grounded in real precedent from the systems surveyed in
  tagging-precedent-findings.md, adapted to this org's own vocabulary and
  the partnership concept. Nothing here is a specification, and no
  implementation work should be based on it without further design. The
  sketch is written to be read in tandem with that research document and
  with ROADMAP.md's "sharing chapter" and "watchlist" sections, which
  correctly treat this entire direction as open questions.
-->

# Illustrative manifest sketch

This file holds **one** concrete, illustrative example of a small in-repo
manifest file that an agent discovering a published captain-built ESP32
project could use to judge compatibility before reusing the work. The
example is deliberately tied to this org's vocabulary (cocapn, ESP32, the
partnership concept) and is modelled on the real fields found in
PlatformIO's `library.json`, HACS's `hacs.json`, and Arduino's
`library.properties` — all described in the research document
`docs/research/tagging-precedent-findings.md`.

The example below is not a schema; there is no canonical field set, no
required or optional rules, and no agreement on what the file should be
named. The only property preserved from every precedent is the pattern:
**a manifest file in the repository that an agent can fetch and parse
before deciding whether to invest more time in a candidate repo found by
recall via GitHub topics or description search**. The sketch exists to
make that pattern concrete enough to discuss.

---

## Example manifest: `compatibility.json`

This file would sit in the root of a published captain-built ESP32
project's repository. The example values are realistic (not placeholders)
and reflect a simple "cabin lights switch" that the captain and their
agent built together, as described in `docs/PARADIGM.md`.

```json
{
  "name": "helm-light-switch",
  "version": "1.0.0",
  "description": "Cabin lights controller for ESP32 — voice-created via captain-agent partnership, timing and safeties from the log.",
  "framework": "cocapn",
  "platforms": ["esp32"],
  "architectures": ["esp32"],
  "cocapn_min_agent_version": "0.3.0",
  "install": {
    "type": "esptool",
    "binary_path": "firmware/helm-light-switch.bin",
    "offset": "0x0"
  }
}
```

### Field by field explanation

| Field | Purpose | Real precedent |
|---|---|---|
| `name` | Human-readable name of the published component. HACS and Arduino both require a `name` field in their manifests (`hacs.json`'s `name`, `library.properties`'s `name`). | HACS `hacs.json` `name`, Arduino `library.properties` `name` |
| `version` | Semantic version of this published state. All three major precedents require a version tag: Arduino links it to a Git tag, HACS reads it from the manifest, PlatformIO's `library.json` has `version`. | Arduino (tag), HACS (`hacs.json` `version`), PlatformIO `library.json` `version` |
| `description` | Plain-language description of what the repo does. Matches the `.description` field used by the GitHub API itself and is required by HACS and Arduino. | HACS (`hacs.json` `description`), Arduino (`library.properties` `sentence` + `paragraph`) |
| `framework` | The agent-side framework this project is built for. This org's primary framework is `cocapn` (see `ROADMAP.md` Step 4). The field directly mirrors PlatformIO's `library.json` `"frameworks": ["espidf"]` — a single string for the most specific one. | PlatformIO `library.json` `frameworks` (array) |
| `platforms` | Real-world hardware platforms the firmware targets. Again modelled on PlatformIO's `library.json` `platforms` (e.g. `"espressif32"`). Here we use the simpler `"esp32"` to match this org's vocabulary. | PlatformIO `library.json` `platforms` |
| `architectures` | Chip architecture the binary was compiled for, matching the common `architectures` field in Arduino `library.properties` (e.g. `avr`, `esp32`). Listed separately to allow narrow targeting without repeating the platform list. | Arduino `library.properties` `architectures` |
| `cocapn_min_agent_version` | Minimum version of the cocapn agent framework required to interpret and reuse this project. Roughly analogous to HACS's `homeassistant` version constraint or PlatformIO's `dependencies` field. | HACS `hacs.json` `homeassistant`, PlatformIO `library.json` `dependencies` |
| `install` | Instructions for how to deploy the firmware to the target device. The `type` indicates the flashing tool; `binary_path` points to the compiled image inside the repo; `offset` (optional) is the flash address. No exact precedent in the surveyed systems (they all handle install via their own platforms), but an honest sketch of what an agent would need to actually reuse the artefact. | — (extrapolated, not copied) |

---

## Illustrative only — not a specification

The manifest shape above is **one** concrete illustration of the
topic+manifest pattern that `docs/research/tagging-precedent-findings.md`
recommends as the minimal viable convention for machine-discoverable
published work without an operated backend. The exact field names,
mandatory vs. optional status, file location, and naming convention are
all open questions that this document explicitly does **not** resolve.

**What this sketch does not mean (stated clearly, because the
disclaimer is not optional):**

- It does **not** mean that a `compatibility.json` file will appear in
  any PurplePincher repo or that any agent in this org will ever read it.
- It does **not** mean that the field set above is the right one or that
  it will survive a design review.
- It does **not** mean that this org has decided to build a manifest at
  all — the sharing chapter in `ROADMAP.md` remains a direction on the
  record, not a roadmap step, and this sketch inherits that status.
- It does **not** replace any work that would be needed to design,
  implement, and test the actual manifest convention. This is a research
  aid, not a design deliverable.

The purpose of this file is to give the abstract idea from the roadmap a
concrete shape that can be discussed, compared with real precedent, and
— if and when the direction becomes a step — evaluated against the
adoption-bar checklist that every fork in this org must pass.

---

*This sketch is not a decision. It is a sketch, labelled as one,
so that the org's honesty discipline applies to its own research as
rigorously as it applies to the sketchbook's over-claimed repos.*
