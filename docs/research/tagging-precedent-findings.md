# Tagging precedent: what real tooling actually does for "repo compatible with framework X"

## The question, stated tightly

`purplepincher/purplepincher`'s "sharing chapter" (`ROADMAP.md`) and
"The second boat" (`docs/PARADIGM.md`) describe a direction: a captain's
agent builds something for an ESP32 (or another device), publishes it as a
real public GitHub repo under the captain's own account, and *other*
captains' agents discover it via **public GitHub search over native
metadata (topics/tags, description)** and reuse it instead of building from
scratch. The load-bearing constraint is that **no operated backend exists** —
discovery must be plain GitHub search that anyone can run, the same
machinery this org used to survey 4,095 repos (`gh search repos`, topics,
descriptions). The roadmap's own illustrative gesture is that "a GitHub
topic on the repo could be the entire convention," explicitly labelled "a
gesture at feasibility, not a design."

This note asks: is there real, working precedent for marking a repo as
"compatible with / built for / discoverable by [framework/platform X]" using
GitHub's own native metadata, where **real tooling actually searches/indexes
by it** (not just a convention nobody built tooling for)? For every example,
the discipline is the same one this org already applies: verify the claim,
don't repeat a project's own README about how discoverable it is.

## The headline finding (up front, because it's load-bearing)

**No major working system uses "a GitHub topic as the *entire* convention."**
Every system with genuine machine-discovery tooling layers **a manifest file**
on top of any topic/badge, and almost all of them back it with either a
**central registry you publish to** or a **PR-submitted curated list** that
the tooling actually reads. In every verified case, the GitHub topic (where
one exists) is at most a **quality gate or a human-browsing aid** — never the
mechanism the tooling uses to discover repos. Topics give *recall* (find
candidates); manifests give *precision* (structured compatibility/version/
install data). Real tooling wants both, and distrusts the first without the
second. This is directly relevant to the roadmap's gesture, and the
recommendation at the end returns to it.

## Findings, per system (each verified against primary sources)

### 1. HACS (Home Assistant Community Store) — the strongest candidate, and it does NOT discover by topic

This was the most promising precedent (an explicit `hacs` topic is widely
cited), so it got the deepest verification. The claim "HACS discovers repos
by the `hacs` topic" is **false as stated**.

**The actual convention** (from hacs.xyz/docs/publish/start, verified):
- A repo must have **GitHub topics defined** (plural — any topics, not a
  specific one).
- A root **`hacs.json` manifest** is required, with at minimum a `name`; it
  also carries `content_in_root`, `filename`, `country`, `homeassistant` (min
  HA version), `hacs` (min HACS version), `persistent_directory`.
- A README, a description, and at least one GitHub **release** (not just a
  tag) are required.

**What HACS actually reads at runtime** (from hacs.xyz/docs/faq/data_sources,
verified): HACS has exactly two data sources:
1. **HACS Data** — pre-generated JSON at `data-v2.hacs.xyz/<section>/<type>.json`
   served from Cloudflare R2 buckets (e.g. `integration/data.json`).
2. The **GitHub REST API** — used only for *custom repositories the user
   manually added by URL*, and to refresh already-known repos.

**How that JSON index is built** (the decisive evidence). The generator is
`scripts/data/generate_category_data.py` in `hacs/integration`, run by the
`generate-hacs-data.yml` GitHub Action every 2 hours. Reading the script
directly: `get_category_repositories()` calls
`async_github_get_hacs_default_file(category)`, which fetches the contents of
a file (`integration`, `plugin`, `appdaemon`, …) from the **`hacs/default`
repo** — a hand-maintained, alphabetically-sorted text list of repo full
names. There is **no GitHub topic search anywhere** in the generator. It
iterates that curated list, hits the GitHub API per-repo for metadata +
releases + the `hacs.json` manifest, and writes the index to R2.

**How a repo enters that list** (hacs.xyz/docs/publish/include, verified):
the owner opens a PR against `hacs/default` adding their repo to the right
file. Automated checks run (brands, manifest, hacs-validation, "the
repository has topics defined," releases, owner, images…). A human reviews.
"New additions still take months to be reviewed and included." Only after
merge does the next scheduled scan pick the repo up.

So the `hacs` topic is a **required quality gate checked during inclusion
review** — not the discovery mechanism. A user *could* run
`gh search repos --topic hacs` themselves (the topic is genuinely searchable,
and many repos carry it), but **HACS itself does not**; its store is a
curated registry + a generated index + a manifest. This is half the picture
the roadmap's "a topic could be the whole convention" gesture is missing, and
it's the half that matters most because HACS is the example most people would
cite for exactly that gesture.

### 2. Arduino Library Manager — the closest real-world analog to "ESP32 project, tagged, discoverable." Curated list + manifest, no topics.

This is the closest precedent to the actual ESP32 use case in the roadmap, so
it's the most directly informative.

**The actual convention** (from `arduino/library-registry` README, verified
verbatim): "If you would like to make a library available for installation via
Library Manager, just submit a pull request that adds the repository URL to
[the list] (`repositories.txt`)." The library must ship a
**`library.properties`** manifest (`name`, `version`, `author`, `maintainer`,
`sentence`/`paragraph` description, `category`, `url`, `architectures` — note
the explicit compatibility field). A release or tag is required; the index
"always uses tagged versions."

**Discovery mechanism:** the curated `repositories.txt` + an automated
**ArduinoBot** that re-checks each library for compliance on every update
(PR-driven). The registry repo itself carries topics `arduino` and
`arduino-library` — but those are for finding *the registry*, not for
discovering individual libraries. **No topic-based discovery of libraries.**

This is essentially the same architecture as HACS (curated list + manifest +
automated compliance bot), arrived at independently. That two of the most
used "community extensions for an open platform" systems converged on the same
shape is itself the finding: pure topic discovery was not enough for either.

### 3. PlatformIO Registry — explicit compatibility fields in a manifest, central registry

**The actual convention** (docs.platformio.org, verified): a `library.json`
manifest with, notably, explicit machine-readable compatibility fields:
- `"frameworks": ["espidf", "freertos"]` (or `"*"` for all) — "A list with
  compatible frameworks."
- `"platforms": "*"` — the same idea for hardware platforms.

This is the clearest example of **structured compatibility metadata** of the
exact kind an ESP32-discovering agent would want (e.g. "is this built for
`espidf`?"). PlatformIO also reads Arduino's `library.properties` and mbed's
`module.json`, so it's a real cross-format consumer.

**Discovery mechanism:** the **PlatformIO Registry**
(registry.platformio.org), published to explicitly with `pio pkg publish` — a
central package registry in the npm mold, **not** GitHub topics. (It can also
consume VCS URLs directly, but registry search is the discovery path.)

### 4. MCP Registry (and GitHub's own `github.com/mcp`) — manifest + central registry, again

Directly relevant since this org's own `exocortex-mcp-ts` / cocapn tooling is
MCP-adjacent.

**The actual convention** (modelcontextprotocol.io/registry + the
`modelcontextprotocol/registry` repo, verified): a standardized **`server.json`**
manifest with the server's unique name in **reverse-DNS** form
(`io.github.user/server-name`), package location (npm name / remote URL /
Docker image), execution instructions (args, env vars), and discovery data
(description, capabilities). Namespace **authentication** (GitHub or DNS
challenge) ties a name to a verified owner.

**Discovery mechanism:** a **centralized REST API registry** you publish
`server.json` to. The registry explicitly "will not provide a commercial grade
search engine" itself and expects downstream aggregators. GitHub's new **MCP
Registry at `github.com/mcp`** (127 servers as of this writing, with install
buttons) is one such aggregator / primary view. **No GitHub-topic-based
discovery** — listing is an explicit publish step to a registry with verified
namespacing.

(Separate and not to be confused with discovery: a repo-local `.mcp.json` is
an emerging *client* convention for telling an MCP-capable editor how to run
servers for *that repo's own development*. That is configuration, not a
discovery/indexing mechanism.)

### 5. Obsidian community plugins — curated JSON list, no topics

**Verified** by reading `community-plugins.json` in `obsidianmd/obsidian-releases`:
it's a single curated JSON array, each entry `{id, name, author, description,
repo}`. Plugins are added by PR. The plugin repo ships a `manifest.json` and a
`versions.json`. **Discovery is the curated list, not GitHub topics.** Same
shape as HACS and Arduino again.

### 6. VS Code extensions — own marketplace, not topics

VS Code extensions are identified by `publisher.name` in `package.json` and
published to the **Visual Studio Marketplace** (or the Open VSX Registry) via
`vsce`/`ovsx`. Discovery is the marketplace, **not GitHub topics**. No
evidence of a topic-based discovery path distinct from the marketplace.

### 7. npm keywords / Cargo categories / PyPI classifiers — own registries; not cross-referenced by GitHub topics in practice

These live in their own registries (npmjs.com, crates.io, pypi.org) and are
searched there. npm `keywords`, Cargo `categories`/`keywords`, and PyPI
classifiers are all package-registry metadata. **No verified tooling joins
them to GitHub topic search** — maintainers sometimes mirror a keyword as a
GitHub topic by hand, but nothing automates the join or relies on it. So even
in the package-registry world, the cross-reference to GitHub topics is
aspirational, not a working mechanism.

### 8. Awesome-lists — manual curation with a *linter*, not topic scraping

Worth checking because the task asked specifically whether awesome-list
maintainer tooling scrapes GitHub topics. It does not. The canonical tool,
**`awesome-lint`** (sindresorhus/awesome-lint, README verified), is a Markdown
**linter**: it checks badge presence, list-item format
(`- [name](url) - description`), trailing slashes, repo age, etc. It validates
a **hand-curated** list; it does not discover repos by topic. So awesome-lists
remain what the task hypothesized — curated lists, not machine-discoverable
metadata — and provide no closer precedent for topic-based discovery.

## The pattern, abstracted

Every system above that has **genuinely working machine-discovery tooling**
combines the same three layers (one or more per system):

| System | Manifest (precision) | Registry / curated list (the actual index) | Topic's real role |
|---|---|---|---|
| HACS | `hacs.json` | curated `hacs/default` → JSON on R2 | required quality gate, not discovery |
| Arduino Lib Mgr | `library.properties` | curated `repositories.txt` + ArduinoBot | none (registry repo itself is topic'd) |
| PlatformIO | `library.json` (+`frameworks`/`platforms`) | central PlatformIO Registry | none |
| MCP Registry | `server.json` (reverse-DNS) | central REST API registry | none |
| Obsidian | `manifest.json`/`versions.json` | curated `community-plugins.json` | none |
| VS Code | `package.json` (`publisher.name`) | Marketplace / Open VSX | none |
| npm/Cargo/PyPI | `package.json`/`Cargo.toml`/classifiers | own central registries | none, not cross-referenced |
| Awesome-lists | (Markdown item format) | manual curation | none (linting only) |

Two things are conspicuously **absent** from this table: (a) any row where a
GitHub topic *is* the discovery mechanism, and (b) any row with no manifest.
The manifest is the universal ingredient; the topic is never sufficient on its
own.

**Why did they all converge here?** Because topic-only discovery has a
precision problem — anyone can add any topic to any repo, so a topic search
returns signal plus a lot of noise. HACS, Arduino, and the MCP Registry all
added either curation or authenticated namespacing precisely to solve the
"can't trust self-description" problem. That is the identical failure mode
this org already documented in its own 4,095-repo survey ("published packages
that don't exist, near-duplicate pairs, NMEA parsers that return zeros,
READMEs describing features the code never implemented") — and the reason the
roadmap's "adoption-bar checklist" exists. Pure topic discovery inherits
exactly that failure mode at the discovery layer.

## Recommendation

Given what's actually real out there, **"a GitHub topic as the entire
convention" is not a reasonable starting point for machine-discoverability.**
Every verified precedent with working tooling adds a manifest on top, and the
manifest — not the topic — is what the tooling actually reads. Stated plainly
against the evidence: there is no successful system of the shape the roadmap
illustratively gestures at. The gesture is feasible as *recall* (topics are
genuinely searchable via `gh search repos --topic X` and the GitHub search
API), but insufficient as *precision* (nothing about a topic tells an agent
the repo's compatibility, version, install path, or whether it even runs).

The encouraging half of this finding is that the missing piece — **a small
manifest in the repo** — is **fully compatible with the roadmap's hard
constraint of "no operated backend."** A manifest lives in the captain's own
repo (where the published code already lives), so an agent's discovery flow can
be a pure two-step over public GitHub, with no PurplePincher-operated endpoint
anywhere:

1. **Recall:** `gh search repos --topic <convention>` (or the GitHub search
   API) to find candidate repos — plain public search, identical machinery to
   this org's own surveys. This is the part a topic can do.
2. **Precision:** fetch each candidate's manifest (e.g. a `library.json`-style
   file with `frameworks`/`platforms`/version fields) to get the structured
   compatibility and install data needed to actually reuse the work — and to
   filter out the noise that topic-only recall will inevitably return.

So the grounded refinement of the roadmap's gesture is: **a topic *plus* a
manifest**, where the topic is the recall signal and the manifest is the
precision signal. This is the minimum that every real precedent converges on,
and it preserves the property the roadmap cares most about — nobody operates
infrastructure between a captain and the captain's own data.

What real precedent also warns about, honestly: the layer every registry-style
system *additionally* builds — curation (HACS/Arduino/Obsidian) or
authenticated namespacing (MCP) — exists precisely to solve the
trust/quality problem that topic search creates. The roadmap's "no operated
backend" constraint rules out a curated/operated registry by definition, so
this org would inherit that trust gap and have to solve it a different way:
**at the agent's read-time, by re-verification**, which is exactly what the
roadmap's existing adoption-bar checklist already gestures at ("re-verify
before executing, not before reading"). In other words, real precedent says
the quality-filtering work that a registry does centrally would, in the
roadmap's no-backend shape, have to be done by every discovering agent, every
time, against the manifest plus a fresh check of the repo's current state.
That is a real cost, and it's the cost of the no-backend property — not a
reason to abandon the property, but a reason to go in with eyes open about
what a topic+manifest convention does and does not buy.

**Bottom line:** the roadmap's own honesty label ("a gesture at feasibility,
not a design") is correct to hedge. Real precedent says a topic alone is too
thin to be the whole convention; the smallest convention that actually
enables machine-discovery — without any operated backend — is **a topic (for
recall) plus a small manifest in the repo (for precision and structured
compatibility)**, with quality/trust deferred to per-discovery agent-level
re-verification rather than a central registry.

## Sources (all primary, all verified during this task)

- `purplepincher/purplepincher` `ROADMAP.md` §"The sharing chapter" and
  `docs/PARADIGM.md` §"The second boat" (cloned and read in full).
- hacs.xyz/docs/publish/start (general requirements, `hacs.json` schema),
  hacs.xyz/docs/publish/include (default-repo inclusion + automated checks),
  hacs.xyz/docs/faq/data_sources (HACS Data vs GitHub REST API).
- `hacs/integration` `scripts/data/generate_category_data.py` and
  `.github/workflows/generate-hacs-data.yml` (the generator reads
  `hacs/default` curated files; no topic search).
- `arduino/library-registry` README (PR-to-`repositories.txt` flow,
  `library.properties` manifest, ArduinoBot re-checks).
- docs.platformio.org `manifests/library-json/fields/frameworks.html` and
  `librarymanager/creating.html` (`frameworks`/`platforms` compat fields;
  `pio pkg publish` to the central registry).
- modelcontextprotocol.io/registry (centralized `server.json` registry,
  reverse-DNS namespacing, REST API) and the `modelcontextprotocol/registry`
  repo (`docs/reference/api/official-registry-api.md`,
  `docs/design/roadmap.md`).
- `github.com/mcp` (GitHub's MCP registry UI; ~127 servers; install flow).
- `obsidianmd/obsidian-releases` `community-plugins.json` (curated JSON list).
- `sindresorhus/awesome-lint` README (Markdown linter, not a topic scraper).
