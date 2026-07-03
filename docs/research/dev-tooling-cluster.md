# The dev-tooling cluster (`si`, `onboard`, `superinstance-mcp`) — deep dive

Scope note up front: this is the **meta-infrastructure** of the SuperInstance
fleet — tools *for managing/installing/coordinating* the fleet itself, not edge
products. Three repos were flagged early in the research and never deep-dived:

- `si` — advertised as "SuperInstance Developer Tool Ecosystem CLI — one
  command to install/compose/manage all developer tools."
- `onboard` — "Automatic repo integration tool — add any SuperInstance tool to
  any GitHub repo with one command."
- `superinstance-mcp` — "MCP server exposing the SuperInstance fleet as tools
  to Claude Code."

The premise worth testing, stated in the original task: *these are tools FOR
building/managing tools, not edge products themselves — potentially directly
relevant to purplepincher's own multi-agent build process.* Verdict after
reading the code (not just READMEs), running what's runnable, and checking
npm/crates.io/CI: **two of three are broken or forbidden by purplepincher's
own adoption bar, and the third is a borrow-the-pattern-not-the-code outcome.
None is forkable as-is.** Receipts below.

All three were cloned (`--depth 1`) into `clones/` and read in full. Where the
toolchain allowed it I built and ran them; for the one TypeScript repo I drove
the actual MCP handshake. None of the three has any GitHub Actions workflows
(`has_actions: null` on all three via `gh api`), so there are no CI logs to
check — that itself is a finding.

## The complete list, what each actually is

| Repo | Language | What it actually is |
|---|---|---|
| `si` | Bash (one file, 1,199 lines) | A single `si.sh` script advertised as the ecosystem CLI (`si doctor` / `si list` / `si add <tool>` / `si init`). **The script does not parse.** The `TOOLS` bash associative array is malformed (plaintext entries sitting inside the array parens), and lines 239–1089 contain the same `install_fleet_daemon_local()` and `install_fleet_dashboard_local()` function bodies copy-pasted **24 times each**, interleaved with random fragments of an unrelated notebook-installer JSON heredoc. Even `si version` errors at array initialization. |
| `onboard` | Bash (one file + 2 sourced installers, 757 + 35 + 52 lines) | A bash script that clones a target repo via `gh repo clone`, runs stack/framework detection, dispatches per-tool installers, commits to a branch `onboard-<unix-epoch>`, pushes, and opens a `gh pr create` with a generated markdown body. This is the only one of the three whose script parses and runs end-to-end. The workflow skeleton is sound; the per-tool installers are mostly fiction (see below). |
| `superinstance-mcp` | TypeScript (one file, 683 lines) | A real, runnable MCP server using `@modelcontextprotocol/sdk`. Eight tools (`fleet_status`, `fleet_search`, `fleet_budget`, `conservation_check`, `ternary_validate`, `crate_info`, `fleet_agents`, `ecosystem_stats`). Builds clean, completes the MCP handshake, dispatches all eight tools correctly. Published on npm. The substantive content of most tools is the γ+η=C conservation-law mythology the purplepincher vision explicitly bans. |

All three repos: **0 stars, 0 forks, 0 open issues, no Actions workflows**, last
pushed 2026-06-14. True commit counts (not just what `--depth 1` showed): **si =
3, onboard = 5, superinstance-mcp = 4**. Tiny, recent, single-burst commits —
`si` and `onboard`'s latest commit on both is the same message
`housekeeping: sync from oracle2 construct`, the tell-tale sign of agent-driven
content synced from another fleet repo rather than authored.

## Maturity assessment, with receipts

I did not trust READMEs. Where I could, I built and ran the code; for claimed
external artifacts (npm/crates.io packages, CI) I checked the registries and
the GitHub APIs directly.

### `si` — Tier 4, broken at the parser

**The script does not run. At all.** Concrete evidence:

```
$ bash -n si.sh
si.sh: line 112: syntax error near unexpected token `—'
si.sh: line 112: `  fleet-dashboard  — Multi-Agent C2 Dashboard'

$ bash si.sh version
si.sh: line 33: TOOLS: fleet-dashboard: must use subscript when assigning associative array
```

Even `si version` — the simplest possible subcommand — fails during array
initialization, before the dispatch `case` is ever reached. The file's
`install_notebook_local()` function (opened at line 239) is never closed
properly; instead its body contains, in order, the same
`install_fleet_daemon_local()` definition (~17 lines) repeated **24 times** and
the same `install_fleet_dashboard_local()` definition repeated **24 times**,
interleaved with stray fragments of a `devcontainer.json` heredoc (`"name":
"A2A Notebook Agent"`, `"image": "python:3.12"`, etc.) that belongs in
`onboard/installers/notebook.sh`. The pattern is unmistakable: `si.sh` is a
botched merge of `onboard.sh`'s installer logic, executed by some agent (the
`oracle2 construct` reference in the commit message lines up with the
`fleet-oracle2` repo elsewhere in the fleet). Other defects visible in the
parts that *do* parse:

- Line 114, 116, 182, 184, 209, 211: `[ -f "$pwd)/fleet-dashboard/index.html" ]`
  — `$pwd` is an undefined variable (should be `$(pwd)`), so the test evaluates
  to `[ -f ")/fleet-dashboard/index.html" ]`, which is always false.
- The TOOLS array's claimed "7 tools" headline (line 124, line 1132) is wrong
  even by its own contents — only 6 entries are valid key/value pairs.
- There is no `status` function; the `status)` case (line 1180) just calls
  `doctor` again.

**Not published anywhere.** No npm wrapping, no brew formula, no curl|bash
installer in the repo. The only path to "use" it is to copy the broken file.

**Fork/polish recommendation: do not fork. Archive-grade.** This is a textbook
instance of the exact failure mode the purplepincher vision calls out by name:
"single-commit sketches wearing product names" and "READMEs describing features
the code has never met." The concept (a thin dispatcher CLI for an ecosystem)
is fine but trivially reimplementable — if purplepincher ever wants a
`pincher-ecosystem` dispatcher, it is 100 lines of clean bash from scratch, not
a rescue of 1,199 lines of corrupted merge output. **Negative value as a
starting point.**

### `onboard` — Tier 2, sound skeleton, fiction installers

This is the only one of the three that actually runs. Concrete evidence I
gathered myself:

```
$ bash -n onboard.sh                # parses clean, no errors
$ bash onboard.sh --help            # prints help correctly
$ source ./onboard.sh
$ detect_stack ../superinstance-mcp # → "node"  ✓
$ detect_frameworks ../superinstance-mcp  # → ""  ✓
$ join_tools_comma a b c            # → "a,b,c"  ✓
$ install_i2i_vessel /tmp/empty     # → "⏳ i2i-vessel coming soon" + empty marker file
```

The **workflow skeleton** (onboard.sh lines 609–687) is the genuinely useful
artifact in this cluster: clone → detect stack → dispatch installers → check
`git status --porcelain` for changes → branch → commit → push →
`gh pr create` with a generated markdown body. Idempotent (skips if already
cloned, prints "Nothing changed — skipping commit and PR" if no diff). It's
about 80 lines and is a clean reference for "bot that opens tool-integration
PRs against repos."

The **per-tool installers, however, are mostly fiction**:

- **`install_i2i_vessel` (onboard.sh:326)** and **`install_ring_buffer`
  (onboard.sh:336)** are pure no-ops. Each checks for a `.i2i-vessel-installed`
  / `.ring-buffer-installed` marker file, prints "⏳ coming soon", `touch`es
  the marker, and returns. But they are in the dispatch table
  (onboard.sh:649–650) as if they install something, and `onboard foo --add
  i2i-vessel` will exit 0, commit the empty marker file, push, and open a PR
  titled "🤖 Add i2i-vessel via onboard". **The PR is a lie.** The help text
  (onboard.sh:39–40) at least says "coming soon" for these; the dispatch
  behavior does not match the help.
- **`install_git_storage` (onboard.sh:293)** runs `npm add git-storage` (or
  falls back to `jq '.dependencies["git-storage"] = "latest"'`). I checked
  npm: `git-storage` *does* exist on the registry, but it is an **unrelated
  abandoned 1.0.0 package** with an empty description, last published once and
  never updated. **The onboard script would silently install the wrong thing
  into the victim repo.** The actual SuperInstance `git-storage` is not on npm.
- **`install_stunt_double` (onboard.sh:148)** and **`install_mmx_toolkit`
  (onboard.sh:226)** generate a `.stunt.yml` / append `mmx-toolkit>=0.1` to a
  Python config. Neither verifies the target package exists. `stunt-double`'s
  curl-based config fetch assumes `SuperInstance/stunt-double/main/stunt-double.yml`
  is a real file — not verified here.
- **`install_notebook` (installers/notebook.sh)** is the only non-trivial
  real installer — writes a `.devcontainer/devcontainer.json` (Python 3.12 +
  Node 22 features) and an `agent-entrypoint.sh`. Self-contained, no external
  fetches. This is the canonical, un-corrupted version of the same installer
  that `si.sh` mangled. Confirms the `si.sh` provenance theory.
- **`fleet-installers.sh`** defines `install_fleet_daemon` and
  `install_fleet_dashboard` (52 lines total). It is sourced into `onboard.sh`
  by the `for f in installers/*.sh` loop on line 14, **but the dispatch table
  on lines 645–653 never calls either function.** Pure dead code. (It's also
  the same two functions that were copy-pasted 24 times into the corrupted
  `si.sh` — same heredoc bodies, including the public-test broker
  `wss://broker.hivemq.com:8000/mqtt`.)

Other real defects:

- **GNU-only `sed -i`** on lines 255, 258, 267, 276 (mmx-toolkit installer).
  BSD `sed` (macOS default) requires `sed -i ''`. The mmx-toolkit installer
  will error on the Macs the team uses. Workaround exists (`brew install
  gnu-sed`) but the script doesn't check or warn.
- **`gh pr create --base main`** (onboard.sh:680) hardcodes the default branch
  as `main`. Will fail on repos with `master` or any other default.
- **`detect_and_suggest`** (onboard.sh:689) advertises "auto-detect + suggest
  tools" but the suggestion logic is hardcoded: always suggests
  `stunt-double`, optionally adds `mmx-toolkit` if Python is detected. The
  detected stack/framework drives nothing else. The "git-storage for web apps"
  suggestion is gated on the presence of `.github/workflows/`, which has
  nothing to do with whether the repo is a web app.

**Not published anywhere** (no brew, no npm wrapping). The only path to use is
`git clone` and `./onboard.sh`.

**Fork/polish recommendation: do not fork as-is. Borrow the pattern.** The
skeleton (clone → detect → install → PR) is worth keeping as a reference for
*what* such a tool looks like, but a purplepincher version should be ~150 lines
of fresh bash (or Go) that:

1. Targets **real purplepincher packages** (`pincher` once it's on crates.io,
   the `fleet-i2i-protocol` Rust crate once that fork lands) — not a fictional
   registry of mostly-nonexistent SuperInstance tools.
2. Makes the installer-actually-installs-something invariant non-negotiable:
   every entry in the tool registry must verify the target package exists
   before committing, and the "coming soon" no-op pattern from
   `install_i2i_vessel` is banned (those tools should simply not be in the
   dispatch table).
3. Uses portable `sed` (or sidesteps it — `awk`/`python`/`jq` are all safer
   for JSON/TOML/YAML mutation than `sed`).
4. Drops the `--base main` assumption (use `gh repo view --json
   defaultBranchRef`).

The conceptual value of having a `pp-onboard` (or similar) tool in
purplepincher's own multi-agent build process is real but only at Step 3+ of
the roadmap — once there *is* a purplepincher toolkit to onboard *to*. Today
the org has one product (DeckBoss) and zero shared infrastructure tools, so an
onboard-style tool would have nothing to integrate. Park the pattern, not the
code.

### `superinstance-mcp` — Tier 1 mechanically, Tier 4 by adoption bar

This is the most interesting split in the cluster: the code is genuinely
functional, and it's forbidden by purplepincher's own rules. Both can be true.

**Mechanical verification, all run myself:**

```
$ npm install                       # 98 packages, 0 vulnerabilities
$ npm run build                     # tsc, zero errors
$ printf '{"jsonrpc":"2.0","id":1,"method":"initialize",...}
          {"jsonrpc":"2.0","method":"notifications/initialized"}
          {"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}' \
    | node dist/index.js

→ {serverInfo: {name: "superinstance-mcp", version: "0.1.0"}, capabilities: {tools:{}}}
→ {tools: [fleet_status, fleet_search, fleet_budget, conservation_check,
           ternary_validate, crate_info, fleet_agents, ecosystem_stats]}
→ [stderr] [SuperInstance MCP] Server started — 8 tools available
→ [stderr] [SuperInstance MCP] C = log₂(3) = 1.584962500721156
```

The MCP SDK integration (`@modelcontextprotocol/sdk` `Server` +
`StdioServerTransport` + `CallToolRequestSchema`/`ListToolsRequestSchema`) is
correct and idiomatic. Tool schemas are well-formed. Error handling wraps the
whole dispatch in try/catch and returns proper `isError` responses. The
`fetchWithTimeout` helper (5s AbortController) is clean. The
`handleFleetStatus` and `handleFleetSearch` tools gracefully fall through
(live API → static/local fallback) on network failure. **As a piece of
TypeScript, this is the most competent code in the cluster by a wide margin.**

**Published:** `superinstance-mcp@0.1.0` is real on npm (verified against the
registry), with **10 downloads in the last week** (2026-06-26 to 2026-07-02) —
real but tiny. So `npx superinstance-mcp` would actually work.

**Then the adoption bar hits.**

**Adoption-bar item 3 (ideology stripped to optional) — hard fail.** Five of
the eight tools are pure γ+η=C mythology:

- `fleet_status` computes "fleet-wide totals" from a hardcoded array of 5
  fake agents (`panther: gamma=0.42, eta=0.31`, etc. — the names match the
  author's hostnames, see `/home/phoenix` below) and reports `delta = C -
  totalC` against `C = Math.log2(3)`. This is the exact framing the
  purplepincher vision bans: *"Nothing in this org will ever depend on γ+η=C
  to function or to be explained."*
- `fleet_budget` is `(C - gamma_used - eta_produced)` with a status string.
- `conservation_check` is `gamma + eta <= C` with status-string theater.
- `ternary_validate` validates values are in {-1, 0, +1}. Real (5 lines of
  actual logic) but mythology-flavored and trivially replaceable.
- `ecosystem_stats` returns a hardcoded object: `crates: 25, workers: 12,
  repos: 18, tests: 847, theorems_proven: 1`. Every one of those numbers is
  invented; the comment on `theorems_proven` is `// Conservation law: γ + η ≤
  log₂(3)`, which is not a theorem, it's a constant.

**Adoption-bar item 2 (docs match reality) — hard fail.** The README (35 KB)
and the two docs files (COOKBOOK.md 40 KB, DEVELOPER.md 32 KB — **107 KB of
docs for 24 KB of source**) substantially oversell what's there:

- README line 55: *"Semantic fleet knowledge — Search 1,200+ indexed crates,
  patterns, and solutions."* The entire SuperInstance org has ~4,200 *repos*
  total (most not crates); the local CRATES registry in this server has 10
  entries. The 1,200 figure is invented and `fleet_search` can't reach it
  anyway — it falls through to the 10-entry local registry whenever the SHOAL
  server (default `http://localhost:8787`) and the vector API are unreachable.
- README line 59: *"Ecosystem registry — 25+ published crates with metadata
  and conservation roles."* Verified against crates.io:

  | Crate (as claimed by MCP) | Claimed version | crates.io reality |
  |---|---|---|
  | `shoal` | 0.3.2 | **0.1.0 only** (fabricated version) |
  | `openagent` | 0.4.1 | **0.1.10 latest** (fabricated version) |
  | `wavefront` | 0.2.0 | 0.2.3 (stale but real) |
  | `ternary` | 0.1.0 | 0.1.3 (stale) |
  | `fleet-auth` | 0.2.1 | **does not exist on crates.io** |
  | `fleet-metrics-cron` | 0.1.5 | **does not exist** |
  | `fleet-vector-api` | 1.0.0 | **does not exist** (it's a Cloudflare Worker, not a crate) |
  | `fleet-dashboard-api` | 0.3.0 | **does not exist** (also a Worker) |

  **4 of the 10 advertised crates don't exist on crates.io at all. 2 more
  have inflated version numbers.** "25+ published" is not even close.

- The COOKBOOK.md describes integrations that are pure pseudocode: a GitHub
  Actions "conservation gate," a Slack/Discord bot, GitLab CI, Jenkins
  pipeline, Docker entrypoint, Kubernetes admission controller, Rust fleet
  client, Grafana monitoring stack. None of these are working recipes — they
  are template-shaped prose around `conservation_check` calls. This is the
  docs-overselling-code pattern the purplepincher vision explicitly calls out
  ("READMEs describing features the code has never met"), at scale.

**Two real bugs that would bite any user trying the published install path:**

- `.mcp.json` (root of repo) contains:
  `"args": ["tsx", "/home/phoenix/repos/superinstance-mcp/src/index.ts"]` —
  the author's hardcoded home directory. Anyone who follows `install.sh`'s
  line 33 (`cp "$SCRIPT_DIR/.mcp.json" "$CLAUDE_CONFIG_FILE"`) gets a config
  pointing at a path that doesn't exist on their machine. The DEVELOPER.md
  troubleshooter (line 1006) repeats the same pattern with `/home/user/...`.
  Neither file uses `$SCRIPT_DIR` or `$(pwd)` or `npx superinstance-mcp`
  (which would actually work given the npm publish).
- `install.sh` line 39: `timeout 3 npx tsx "$SCRIPT_DIR/src/index.ts" 2>&1 ||
  true` followed unconditionally by `echo "✅ Server test complete"`. The
  `|| true` swallows any failure, including the server crashing on startup.
  The installer will report success regardless of whether the server runs.
  (This is the `pytest || true` pattern the purplepincher adoption bar was
  written to ban, in installer form.)

**Fork/polish recommendation: forbidden by adoption-bar item 3; do not fork.**
Even setting aside the γ+η=C problem, what's left after stripping the
mythology is:

- ~5 tools that return hardcoded fake data (5 agents, ecosystem stats).
- `fleet_search`, which depends on `SHOAL_URL` (default localhost:8787) and
  a `VECTOR_API_URL` hardcoded to `casey-digennaro.workers.dev` — neither of
  which exists in purplepincher's world.
- A correct MCP-over-stdio scaffolding pattern.

That last item has some reference value: if purplepincher ever wants to ship
an MCP server (e.g. exposing pincher's reflex layer, or DeckBoss's
log-query tools, to Claude Code), the *shape* of `superinstance-mcp`'s
`Server` + `StdioServerTransport` + `CallToolRequestSchema` wiring is a
correct 80-line example of how to do it with the official SDK. Use it as a
reference for the SDK pattern; **do not adopt the server, the tools, the
crate registry, the agent roster, or any of the docs.** A purplepincher MCP
server, if one is ever warranted, starts from a blank `npm init` and the
`@modelcontextprotocol/sdk` README — not from this codebase.

## Cross-cutting observations

**The cluster's premise is the fiction it rests on.** All three tools assume
there is a real, installable SuperInstance developer ecosystem to manage,
onboard, and expose. There isn't. The `si`/`onboard` tool registries are full
of names (`stunt-double`, `mmx-toolkit`, `git-storage`, `i2i-vessel`,
`ring-buffer`, `notebook`) that are either not published, published under
different/unrelated names, or no-op stubs. The MCP server advertises a fleet
of 5 agents and 25+ crates that are largely invented. **The meta-infrastructure
is more elaborate than the infrastructure it claims to manage.** This matches
the pattern flagged across the rest of the research (the budget-guardian,
fleet, and ternary clusters all show the same docs-ahead-of-code shape), but
it's especially concentrated here because these tools' entire job is to be the
*interface* to the ecosystem — and the interface is more polished than the
thing behind it.

**The two clean installers in `onboard/installers/notebook.sh` are the most
honest code in the cluster.** Self-contained, no external fetches, no fake
package references, no ideology. If there's a single file in these three repos
worth keeping as reference material, it's that 35-line `.devcontainer/`
generator — and even that is more useful as "what a clean installer looks
like" than as something to import, because purplepincher doesn't ship
devcontainer-based agents.

**No CI on any of the three.** `has_actions: null` across all three via
`gh api /repos/SuperInstance/{si,onboard,superinstance-mcp}`. So a `si.sh`
that doesn't parse and an `install.sh` that reports success on crash both
shipped without a single automated check catching them. The purplepincher
adoption-bar item 1 ("not 'has tests' — SuperInstance CI has been known to run
`pytest || true`") was written about a different cluster but applies almost
verbatim here: `install.sh`'s `timeout 3 npx tsx ... || true` is the same
pattern, in installer form.

## Bottom line

- **`si`**: do not fork. Code is corrupted past recovery; even `si version`
  errors. Concept is trivially reimplementable from scratch when needed.
  Negative-value starting point.
- **`onboard`**: do not fork as-is. **Borrow the workflow skeleton** (clone →
  detect → install → commit → PR, ~80 lines) as a reference pattern if/when
  purplepincher has a real toolkit to onboard into (Step 3+ of the roadmap).
  Fresh ~150-line implementation, not a rescue. Banned in any fork: the
  no-op "coming soon" installers, the wrong-package `npm add git-storage`,
  the GNU-only `sed -i`.
- **`superinstance-mcp`**: **forbidden by adoption-bar item 3.** Real
  TypeScript, real MCP handshake, real npm publish — but five of eight tools
  are the γ+η=C mythology the org has sworn never to depend on, the docs are
  107 KB of oversell against 24 KB of source, the `.mcp.json` ships the
  author's home directory hardcoded, and the installer claims success on
  crash. Use only as a reference for the `@modelcontextprotocol/sdk` wiring
  pattern if purplepincher ever ships its own MCP server.

Net: **zero of three forkable. One pattern worth borrowing (onboard's PR-bot
skeleton), one SDK wiring worth referencing (the MCP server's setup), one
 outright write-off (si).** None of this is "genuinely useful developer
 tooling purplepincher could adopt for its own multi-agent build process"
 today — the only candidate with that shape (onboard) targets a fictional
 ecosystem, and the actual purplepincher multi-agent build process (kimi,
 aider, GLM, mmx, Claude per the roadmap) has no current need for an
 onboard-style integration bot. Park the patterns; do not port the code.

---

*Methodology: all three cloned `--depth 1` into `clones/`. Source read in
full. `bash -n` and direct execution against local test dirs used to verify
the bash scripts. `npm install && npm run build && node dist/index.js` driven
through the actual MCP handshake (`initialize` → `notifications/initialized`
→ `tools/list`) for the TypeScript server. crate existence checked against
`crates.io/api/v1/crates/<name>` with a User-Agent; npm package and download
counts checked against `registry.npmjs.org` and `api.npmjs.org/downloads`.
Repo metadata and CI presence checked via `gh api /repos/SuperInstance/<r>`.
No edits or pushes made; this research session left no footprint on the
SuperInstance org.*
