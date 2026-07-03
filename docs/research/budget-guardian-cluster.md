# The budget-guardian family — deep dive

Scope note up front: `grep -iE 'budget|guard|watchdog'` over all 4,200 SuperInstance
repo names returns 31 hits (full list in
`notes/all_repo_names.txt` / `notes/budget_guard_repos_full.jsonl`). Of those, about
half belong to a **different** cluster entirely — the physics/conservation-law
mythology and formal-verification "GUARD DSL" ecosystem already flagged skeptically
elsewhere in this research (`noether-guard`, `guard-constraints`, `guard-dsl`,
`guardc`, `guardc-v3`, `guard2mask`, `guard2mask-gpu`, `lau-conservation-guard`,
`plato-lab-guard`, `zeroclaw-guard`, `energy-budget`, `ternary-budget`,
`thermal-budget`, `guard`). Those are not about AI-coding-workflow budgets and are
excluded below.

The actual "practical, production-minded tools for enforcing token/time/build
budgets on AI coding workflows" family is **13 repos**. All 13 were cloned
(`--depth 1`) into `clones/` and read in full — source, tests, CI configs and
actual CI run logs (not just badges). Where the local toolchain allowed it
(Node/npm and gcc/make were available; `cargo`/`pip` were not), I built and ran
the test suites myself rather than trusting READMEs or green badges.

## The complete list, what each actually does

| Repo | Language | What it actually is |
|---|---|---|
| `build-guardian` | TypeScript | JS/TS **build-bundle-size** budget tracker: per-entry size/time/memory budgets, bloat-trend regression, Webpack/Vite/esbuild adapters, Slack/Prometheus/GitHub-PR-comment exporters. Not LLM/token related — it's a frontend build-size guardian. |
| `storage-guardian` | TypeScript | Generic **file storage** dedup/budget tool: content-hash duplicate detection, dedup, storage budget + alerting, trend analysis, S3/filesystem/in-memory adapters. Not LLM/token related. |
| `conservation-guardian` | Python | **Workflow cost/waste analyzer** for LLM pipelines: `WorkflowBudget` (token/cost/node-count limits), `Profiler`/`NodeSample` (per-node stats), `WasteDetector` (overprompted / low-utilization / expensive-model findings), `WorkflowDAG` (redundant-call + dead-branch detection), adapters for generic JSON, OpenAI usage records, and LangChain callback data. This is the one genuinely built for "is my AI workflow wasting money." |
| `dify-budget-watchdog` | Rust | In-memory token/cost budget tracker with a 4-phase model (Normal → PreTransition → Transitioning → PostTransition) and a hardcoded model-downgrade chain (GPT-4o→mini, Opus→Sonnet→Haiku, etc.) plus per-team-member quotas. Framed as "for Dify workflows" but contains **zero Dify-specific code** — no HTTP client, no wiring to Dify's API or workflow runner. It's a generic budget/phase library with Dify-flavored naming. |
| `uv-cache-guardian` | Rust | CLI + library that measures the `uv` (Python package manager) cache directory and reports disk/bandwidth/CI-time "conservation law" violations, with a KL-divergence eviction *recommendation* engine and CI-download-overlap analysis. |
| `cache-guardian-c` | C | A straight C11 port of `uv-cache-guardian`'s algorithms (budget checks, KL divergence, eviction ranking, phase detection, CI overlap) as a standalone library, no CLI. |
| `rig-budget-guard` | Rust | Token-budget **middleware that actually wraps [Rig](https://github.com/0xPlaygrounds/rig)'s real `Prompt`/`Chat`/`Completion` traits** — genuine interception of LLM calls for a specific Rust agent framework, with per-model budgets, phase detection, warning-header injection, and Serde audit snapshots. |
| `fastloop-guard` | Rust | Real async (Tokio) Unix-socket **semantic query cache daemon**: BLAKE2b exact-match gate + MinHash/Jaccard fuzzy-match gate to dedupe near-identical LLM queries before they hit the API. About *reducing* spend via caching, not budget *enforcement* per se. |
| `fleet-budget` | TypeScript (Cloudflare Worker + D1) | The only repo in the family with **hard enforcement**: a `γ + η ≤ C` capacity ledger backed by a SQLite `CHECK` constraint on Cloudflare D1, so an over-budget commit is rejected by the database itself, not just by application logic. Reserve → commit → release lifecycle, event-sourced audit log, race-condition handling via D1 SERIALIZABLE isolation. Generic compute/network capacity envelope for SuperInstance's own internal "fleet" of agents, not LLM-token-specific. |
| `codex-budget-guard` | Rust | Budget enforcement (daily/weekly/monthly token limits, phase detection, auto-throttle) framed for OpenAI's Codex CLI specifically. |
| `ToolGuardian` | TypeScript | The largest codebase in the family (6,671 lines): reliable function-calling for AI agents — schema validation, retry/fallback, execution sandboxing with timeouts, monitoring/metrics, lifecycle hooks. Adjacent to the budget family (resource/time governance for tool calls) rather than literally "budget." |
| `token-budget-energy` | TypeScript (Cloudflare Worker + KV) | Tiny stub: a KV-backed "energy credit" ledger for a different mythology cluster ("Cocapn fleet" / Lucineer org — nautical "vessel" framing, not the SuperInstance AI-coding-budget line). |
| `cuda-budget` | Rust | Tiny stub with the same Cocapn-fleet framing; auto-generated-looking README ("Key Types: `Allocation`, `AgentBudget`, `BudgetManager`" with no real prose). |

## Maturity assessment, with receipts

I did not trust READMEs or CI badges. Where I could, I cloned, built, and ran the
actual test suite myself; otherwise I pulled the real GitHub Actions job logs
(not just pass/fail) and cross-checked install claims against the real package
registries (npm, PyPI, crates.io).

**Tier 1 — real, tested, verifiably working (I ran the tests myself):**

- **`build-guardian`** — published on npm as `@superinstance/build-guardian`
  (currently `0.2.0`, 111 downloads/month). I ran `npm ci && npm test`: **55/55
  tests pass**, matching the README's claimed test count exactly. CI is green
  except the `publish` job (fails for the mundane reason that a version is
  already published / no fresh `NPM_TOKEN` in that run — not a code problem).
  Zero runtime dependencies, clean TypeScript.
- **`storage-guardian`** — published as `@superinstance/storage-guardian`
  (`0.2.0`, 73 downloads/month). I ran the suite: **39/39 tests pass**. CI green.
- **`cache-guardian-c`** — no package registry (C library), but I built it
  myself with `gcc -Wall -Wextra -std=c11` and ran `make test`: **27/27 tests
  pass**, zero warnings.
- **`conservation-guardian`** — published on PyPI (`0.2.0`; `pip install
  conservation-guardian` genuinely works, confirmed against the PyPI JSON API).
  878 lines of real pytest coverage (`test_guardian.py` + `test_edge_cases.py`),
  and the actual "CI" GitHub Actions workflow passes cleanly across Python
  3.10/3.11/3.12 (I read the job logs, not just the badge). There *is* a second
  "Test" workflow that fails, but the failure is `ruff` catching 10 cosmetic
  lint issues (unused imports, an f-string with no placeholders) — not a
  functional bug.

**Tier 2 — real code, genuinely partial:**

- **`dify-budget-watchdog`** — 44 inline `#[test]`s (matches the README's "44+
  unit tests" claim exactly), and the real CI logs show `cargo build`, `cargo
  test`, and `cargo clippy` all passing — only `cargo fmt --check` fails
  (cosmetic). But: it advertises a crates.io badge and `cargo add
  dify-budget-watchdog` install instructions, and **the crate does not exist on
  crates.io** (confirmed via the crates.io API — `does not exist`). And despite
  the "Dify Integration Pattern" architecture diagram in the README, grepping
  the source turns up no `reqwest`/`hyper`/HTTP client and no Dify-specific
  glue at all — you'd wire every call yourself.
- **`uv-cache-guardian`** — genuinely published on crates.io (`0.1.0`, 17
  downloads — real, not zero). CI is green except cosmetic `cargo fmt`. But its
  "intelligent eviction" is **advisory only** — I grepped for
  `remove_file`/`remove_dir` across the source and found none; it recommends
  what to evict but never deletes anything, and never shells out to the real
  `uv` binary (no `Command::new` calls anywhere). It measures a directory and
  prints a report.
- **`rig-budget-guard`** — the most architecturally serious "real interception"
  implementation in the family: it genuinely implements Rig's `Prompt`,
  `Chat`, and `Completion` traits against `rig-core = "0.38"` (a real,
  external LLM-framework dependency, not vendored). CI logs show Check/Test/
  Clippy all green, only `fmt` fails. But it too claims a crates.io badge that
  is false — `rig-budget-guard` does not exist on crates.io — and it only
  works if your application is built on the Rig Rust framework.
- **`fastloop-guard`** — real Tokio Unix-socket server with real tests
  (`test_cache.rs`, `test_similarity.rs`, `test_integration.rs`); CI's `test`
  job passes, but `fmt` **and** `clippy` both fail (clippy failing is a real
  lint/quality signal, not cosmetic like the others).
- **`fleet-budget`** — the most rigorous *design*: budget enforcement as a
  literal SQLite `CHECK` constraint, so violations are structurally
  impossible rather than merely checked in application code. 713 lines,
  a real D1 schema, a real Cloudflare Worker. But **zero test files, no CI at
  all**. The README's race-condition-safety claims (two agents committing
  simultaneously) are asserted in prose, never exercised by a test.

**Tier 3 — broken or stub:**

- **`codex-budget-guard`** — **cannot build.** `Cargo.toml` has
  `conservation-checker = { path = "/tmp/codex-conservation-checker", ... }` —
  an absolute path that only exists on the original author's machine. The real
  CI log shows `cargo check` failing immediately with `No such file or
  directory`. Interestingly, the crate genuinely was published to crates.io at
  some point (0.1.0, 16 downloads) with the correct versioned dependency
  (`conservation-checker = "^0.2"`, confirmed via crates.io's dependency API) —
  so it worked once, and a later commit (plausibly one of the "AGENT.md
  ensign duty log" housekeeping commits visible in the run history) broke the
  git HEAD by swapping in a local dev path that was never reverted.
- **`ToolGuardian`** — the biggest, most ambitious codebase, and the one I
  spent the most effort actually running. `npx vitest run` (after fixing the
  README's `npm test` invocation, which hangs in watch mode) gives **27 of 102
  tests failing, across 5 of its 6 test files**, including core promised
  behavior: a test asserting a timed-out execution reports `status: 'failed'`
  instead gets `'success'`; a test asserting an `execution:failed` event fires
  never sees the handler called at all; the fallback/retry integration test
  gets the inverse of the expected result. Its CI is separately broken too —
  the workflow is configured for npm's lockfile cache but the repo only ships
  a `pnpm-lock.yaml`. Real, substantial, currently incorrect on core claims.
- **`token-budget-energy`** / **`cuda-budget`** — thin stubs (a handful of
  files, no tests, no CI, one has an auto-generated-looking README with
  placeholder section headers and no real prose). These belong to a sibling
  mythology cluster ("Cocapn fleet" / Lucineer, nautical "vessel" framing)
  rather than the SuperInstance AI-coding-budget line, and read as
  early scaffolding rather than finished tools.

## Would any of this genuinely help this project's own multi-agent build process?

Being concrete about what "this project's own build process" actually needs:
dispatching kimi/aider/GLM/mmx/Claude subagents (sometimes concurrently, into
worktrees), wanting to know if a dispatch is burning tokens/time
disproportionately, and wanting independent verification before merge. None of
these 13 repos is a drop-in fit — nothing here has a kimi, aider, GLM, or Claude
Agent SDK adapter, unsurprisingly, since none of them were built with this
specific tool stack in mind. But two are close enough to be worth using as a
starting point rather than inspiration only, and it's worth being honest about
which ideas are real fits versus which just sound like fits:

- **Best actual fit: `conservation-guardian`.** It's published, tested, and its
  `GenericAdapter(records=[...], field_map={...})` is specifically designed for
  "I have my own JSON usage records, map the fields." If each subagent dispatch
  in this session already produces (or could easily be made to produce) a
  small record — model/tool, tokens, latency, cost — `WorkflowBudget` +
  `WasteDetector` would genuinely tell you things worth knowing: which subagent
  dispatches are consistently overprompted, whether spend is trending up
  session over session, whether a daily cap is being approached. This is the
  one piece of real, working software in the family that maps onto an actual
  gap in how this session currently operates (there is currently no
  cross-tool token/cost accounting at all).
- **Best architectural idea, not best code: `fleet-budget`.** The
  DB-CHECK-constraint enforcement pattern is exactly right for a scenario where
  multiple agent processes could concurrently blow past a shared budget — which
  is a real risk once dispatches are genuinely parallel rather than
  orchestrated sequentially by one Claude session. But it's untested code
  proposing to be the safety backstop for budget enforcement, which is the
  one place you want tests before you trust it. Today's actual usage pattern
  (a single Claude session sequencing dispatches) doesn't need hard concurrent
  DB enforcement yet — soft accounting (conservation-guardian) covers the
  current need; fleet-budget's pattern is the right thing to reach for if
  the workflow ever moves to genuinely independent concurrent agent processes.
- **Real but not directly on-target:** `fastloop-guard`'s semantic-cache idea
  (dedupe near-identical queries before spending on them again) is a
  legitimately useful concept for a session that re-asks similar research
  questions across kimi/GLM/mmx, but kimi/aider/GLM/Claude/mmx don't share a
  common request interface, so wiring one cache in front of all of them is
  real integration work with no existing adapter to lean on.
- **Not useful for this purpose:** `build-guardian` and `storage-guardian` are
  genuinely the best-engineered, best-tested code in the whole family — but
  they budget JS bundle size and file storage, not AI agent spend. (Build-
  guardian could matter later for DeckBoss's own frontend bundle-size CI gate,
  which is a real but separate use case from what was asked here.)
  `uv-cache-guardian`/`cache-guardian-c` budget disk/CI-cache, not agent
  tokens. `rig-budget-guard` requires the Rig Rust framework, which this
  project doesn't use. `codex-budget-guard` is Codex-CLI-specific and broken
  besides. `dify-budget-watchdog` is Dify-specific in name only and unpublished.
  `ToolGuardian`'s tool-calling-reliability idea is the right shape for
  "verify subagent work" but its timeout/failure-event handling is
  demonstrably broken right now.

## Fork/polish recommendation, prioritized

1. **Fork `conservation-guardian` first.** It is the one tool in this family
   that is simultaneously (a) real and independently verified working, (b)
   actually about LLM workflow cost/waste rather than an adjacent resource, and
   (c) close enough to plug-and-play that the main missing piece is a small
   adapter, not new architecture. Concrete gaps to close before/while forking:
   - No adapter exists for any CLI-based coding agent (kimi, aider, GLM CLI,
     Claude Code subagents, mmx) — only Generic/OpenAI/LangChain. Writing a
     thin JSONL-record-per-dispatch convention plus a `GenericAdapter` field
     map is the actual work, not a rewrite.
   - Fix the 10 `ruff`-flagged issues (`ruff check src/ tests/ --fix`) before
     forking so the "Test" workflow is green from day one instead of carrying
     over a known-failing CI job.
   - It has no persistence-over-multiple-sessions story beyond
     `profiler.save()/load()` to a single JSON file — fine for one repo's
     history, would need a small decision (one file per project? per day?) to
     scale to tracking this account's multiple concurrent projects.

2. **Treat `fleet-budget` as a design reference, not a fork target yet.** The
   `γ+η≤C` CHECK-constraint pattern is worth reusing *if and when* this
   workflow needs concurrent-safe enforcement, but the concrete, named gap is
   that it currently has **zero tests** for the one thing that matters most
   (the reserve/commit/release race-condition claims). Don't forward-port
   untested budget-enforcement code into anything that would actually block
   or reject work. If this becomes worth building, write the test suite for
   the lifecycle and race conditions first, using this repo's schema and API
   shape as the starting design, not as trusted working code.

3. **Do not fork `codex-budget-guard`, `dify-budget-watchdog`, `rig-budget-guard`,
   `ToolGuardian`, `token-budget-energy`, or `cuda-budget` for this purpose.**
   Named reasons: `codex-budget-guard` doesn't build as committed (broken path
   dependency); `dify-budget-watchdog` has no real integration with anything
   and isn't published despite claiming to be; `rig-budget-guard` requires a
   framework this project doesn't use; `ToolGuardian`'s core promised behavior
   (timeout enforcement, failure events) fails its own tests right now;
   `token-budget-energy`/`cuda-budget` are stubs from an unrelated mythology
   cluster.

4. **Honest bottom line:** this family is *not* "quietly production-ready
   infrastructure waiting to be picked up," which is closer to what the
   earlier passing survey mention implied. It's a mix of one genuinely solid,
   tested, published tool that happens to fit a real gap in this session's own
   process (`conservation-guardian`), two well-engineered tools solving a
   different problem than advertised by the family name (`build-guardian`,
   `storage-guardian`), a cluster of Rust crates with real inline tests but
   unpublished/aspirational integration claims (`dify-budget-watchdog`,
   `rig-budget-guard`, `uv-cache-guardian`'s eviction), one good idea let down
   by zero test coverage (`fleet-budget`), one broken build
   (`codex-budget-guard`), one large ambitious tool whose core claims don't
   hold up under its own test suite (`ToolGuardian`), and two stubs. "Small,
   quick win between major steps" undersells the actual work needed — writing
   the kimi/aider/GLM/Claude adapter for conservation-guardian is genuinely
   small; anything beyond that in this family currently is not.
