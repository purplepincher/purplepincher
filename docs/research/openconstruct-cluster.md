# The `openconstruct-*` Cluster — Deep Dive

**Author:** opencode research pass · **Date:** 2026-07-03
**Scope:** All 23 repos matching `openconstruct` (case-insensitive) in `SuperInstance/*`,
shallow-cloned and read in full (README, source, build manifests, git log). The
ecosystem-survey (`notes/ecosystem-survey.md` §2, §3 #11) flagged this cluster as
22 repos / "medium-high" purplepincher fit but explicitly did not deep-dive it; one
of its members (`openconstruct-hub`) is one of the 11 originally-named repos.

---

## TL;DR (read this first)

1. **The headline `SuperInstance/OpenConstruct` repo is a near-verbatim fork of
   `NVIDIA/OpenShell`** (a real, popular upstream — **7,372 stars, 913 forks,
   Apache-2.0, "the safe, private runtime for autonomous AI agents," actively pushed
   today). It is a genuine GitHub fork (`parent`/`source` = `NVIDIA/OpenShell`),
   a single commit ("ci: add CI workflow"), and contains **fewer** files than
   upstream (813 vs 874). SuperInstance's *original* additions on top are: one
   real CLI crate (685 LOC), one real crate (`openshell-signal-chain`, 4,914 LOC),
   two small crates (`openshell-construct` 273 LOC, `openshell-registry` 309 LOC),
   and **six 4-LOC stub crates** with research-flavored names
   (`openshell-constraint-theory`, `openshell-fleet-homology`,
   `openshell-flux-fracture`, `openshell-flux-vm`, `openshell-holonomy-consensus`,
   `openshell-pythagorean48`) — plus a rebranded README.

2. **The "multi-language agent-onboarding SDK" story does not hold.** Of the 20
   repos that claim to be language SDKs / bindings, **exactly ONE (`openconstruct-c`)
   actually links to the C ABI.** The other "SDKs" (python, go, ruby, swift, zig,
   ts, java, cs, rust, …) are pure in-language **reimplementations** with **zero
   FFI** — no ctypes/cffi/cgo/JNI/P-Invoke/dlopen, no `#include "openconstruct.h"`.
   Several falsely advertise themselves as bindings in their package manifests
   (`openconstruct-rust` has `categories = ["api-bindings"]`; `openconstruct-java`
   pom says "Java binding"; `openconstruct-ruby` gemspec says "Thin Ruby binding").

3. **The C ABI itself is a toy, and its README documents an API that does not
   exist.** `openconstruct-abi/src/lib.rs` is 320 real LOC with 7 unit tests, but
   the "module registry" is **10 hardcoded modules** of SuperInstance math-research
   flavor ("spectral-graph-core", "sheaf-cohomology", "tropical-algebra"), and the
   "config generation" emits **malformed JSON** (uses Rust's `{:?}` Debug formatter
   for the modules array). Its README advertises `oc_session_choose_interface`,
   `oc_session_connect`, `oc_session_declare`, etc. — **none of which exist in the
   code.** The SDKs implement the README's fictional 5-phase protocol; the examples
   repo invents a *third* fictional API (a `ws://` coordinator + heartbeat); the
   docs and landing page describe a *fourth*. Five different systems, none of which
   is the real one.

4. **Verdict: fragmented and incomplete, like everything else found so far.** This
   is **not** a "genuinely useful, coherent multi-language SDK story." The one
   genuinely useful artifact in the cluster is the upstream `NVIDIA/OpenShell`
   itself — which is not SuperInstance's to fork-and-polish (it's already a
   well-maintained Apache-2.0 project; you'd just *use* it, or contribute upstream).
   SuperInstance's own original code here is a local-only CLI toy + a toy C ABI +
   20 cosmetic look-alikes.

---

## 1. The real inventory (23 repos, not 22)

The survey said 22; `gh repo list SuperInstance … | grep -i openconstruct` returns
**23** — the discrepancy is the capitalized `OpenConstruct` (the headline fork),
which a case-sensitive `-i` grep catches but the survey's first-token grouping
(`openconstruct-*`) missed.

| Repo | Lang | GH size | Checkout | Created | Pushed | ★ | What it claims to be |
|---|---|---:|---:|---|---|---:|---|
| **`OpenConstruct`** | Rust | 9.5 MB | 17 MB | 2026-05-20 | 2026-06-08 | 1 | **Fork of NVIDIA/OpenShell** — "plug-and-play shell commands … agent workspaces" |
| `openconstruct-hub` | Makefile | 175 KB | 1.1 MB | 2026-05-29 | 2026-05-29 | **2** | "Integration Hub — meta-repo and entry point" |
| `openconstruct-abi` | Rust | 11 KB | 268 KB | 2026-05-29 | 2026-06-08 | 1 | "C ABI — any language that can call C can onboard" |
| `openconstruct-c` | C | 9 KB | 232 KB | 2026-05-29 | 2026-05-29 | 1 | "C bindings — header, examples, test harness" |
| `openconstruct-python` | Python | 8 KB | 260 KB | 2026-05-29 | 2026-05-29 | 1 | "Python thin client" |
| `openconstruct-go` | Go | 5 KB | 228 KB | 2026-05-29 | 2026-05-29 | 1 | "Go SDK" |
| `openconstruct-ruby` | Ruby | 6 KB | 252 KB | 2026-05-29 | 2026-05-29 | 1 | "Ruby SDK" |
| `openconstruct-swift` | Swift | 5 KB | 240 KB | 2026-05-29 | 2026-05-29 | 1 | "Swift binding — iOS/macOS agent apps" |
| `openconstruct-zig` | Zig | 4.2 MB | 236 KB | 2026-05-29 | 2026-05-29 | 1 | "Zig binding — thin client" |
| `openconstruct-ts` | TypeScript | 38 KB | 420 KB | 2026-05-29 | 2026-05-29 | 1 | "TypeScript SDK" |
| `openconstruct-java` | Java | 22 KB | 288 KB | 2026-05-29 | 2026-05-29 | 1 | "Java binding — enterprise, Android, JVM" |
| `openconstruct-cs` | C# | 9 KB | 268 KB | 2026-05-29 | 2026-05-29 | 1 | "C# SDK" |
| `openconstruct-jupyter` | Python | 14 KB | 300 KB | 2026-05-29 | 2026-05-29 | 1 | "Jupyter notebook integration" |
| `openconstruct-esp32` | C++ | 14 KB | 272 KB | 2026-05-29 | 2026-05-29 | 1 | "Embedded client for ESP32" |
| `openconstruct-jetson` | C++ | 18 KB | 288 KB | 2026-05-29 | 2026-05-29 | 1 | "GPU edge node — Jetson" |
| `openconstruct-rust` | Rust | **36 MB** | 324 KB | 2026-05-29 | 2026-06-08 | 1 | "Rust SDK" (bloat is in git history, not the tree — see §7) |
| `openconstruct-catalog` | Rust | **48 MB** | 336 KB | 2026-06-01 | 2026-06-08 | 0 | "Tech catalog and module discovery" (same bloat caveat) |
| `openconstruct-kernel` | Rust | 12 KB | 284 KB | 2026-05-30 | 2026-06-08 | 1 | (no description) |
| `openconstruct-modular` | Rust | 16 KB | 356 KB | 2026-06-01 | 2026-06-08 | 0 | (no description) |
| `openconstruct-mercury` | Mercury | 13 KB | 272 KB | 2026-05-29 | 2026-05-29 | 1 | "Formal verification in Mercury" |
| `openconstruct-examples` | — | 15 KB | 288 KB | 2026-05-29 | 2026-05-29 | 1 | "Examples cookbook — runnable for every part" |
| `openconstruct-docs` | — | 382 KB | 1.1 MB | 2026-05-29 | 2026-06-08 | 1 | "Agent-centric documentation" |
| `openconstruct-landing` | HTML | 54 KB | 312 KB | 2026-05-29 | 2026-06-08 | 1 | "Landing page — any agent, any hardware, any language" |

**Engagement:** 0 forks across all 23. Stars: median 1, max 2 (`openconstruct-hub`).
Everything was created in a 12-day window (2026-05-20 to 2026-06-01) and **every
repo is exactly 1 commit** (see §6).

---

## 2. The reframe: `OpenConstruct` is a fork of `NVIDIA/OpenShell`

This is the single most important fact and it changes the whole assessment.

**Upstream `NVIDIA/OpenShell`** (verified via `gh api repos/NVIDIA/OpenShell`):
- 7,372 stars, 913 forks, Apache-2.0, **not** archived, pushed 2026-07-03 (today).
- Tagline: *"OpenShell is the safe, private runtime for autonomous AI agents."*
- Sandboxed execution environments for agents, declarative YAML policies, prevents
  unauthorized file access / data exfiltration / uncontrolled network. Ships with
  agent skills, gateway, Helm chart, Docker/Podman microVM sandboxes.
- Real install paths that **do** work: `curl … install.sh | sh`, `uv tool install
  openshell` (PyPI), `helm install openshell oci://ghcr.io/nvidia/openshell/helm-chart`.
- A large, structured Rust workspace: `openshell-cli`, `openshell-server`,
  `openshell-sandbox`, `openshell-policy`, `openshell-router`, `openshell-bootstrap`,
  `openshell-ocsf`, `openshell-core`, `openshell-providers`, `openshell-tui`, three
  compute drivers (kubernetes / docker / vm), Python SDK, gRPC proto, Helm/K8s deploy.

**`SuperInstance/OpenConstruct`** (verified via `gh api … /parent`):
- `fork: true`, `parent.full_name = NVIDIA/OpenShell`, `source.full_name = NVIDIA/OpenShell`.
- **1 commit**, subject "ci: add CI workflow", authored by "Casey Digennaro" 2026-06-08.
- **813 files vs upstream's 874** — i.e. it has *deleted* ~60 upstream files
  (e.g. `snap-package.yml`, `stale.yml`, `sync-docs.yml`, `rust-cache-seed.yml`,
  `publish-docs-website.yml`, the `create-rfc`/`test-release-canary` agent skills).
- 347 files "differ" — almost entirely because the fork is an **older snapshot**
  of a fast-moving upstream, not because of edits.

**What SuperInstance actually added** (the `Only in OpenConstruct` files, excluding
`.git/`):

| Addition | Real? | LOC |
|---|---|---:|
| `crates/openconstruct-cli/` (the onboarding CLI) | **Yes — the main original piece** | 685 |
| `crates/openshell-signal-chain/` | Yes (24 files) | 4,914 |
| `crates/openshell-construct/` | Yes (1 file) | 273 |
| `crates/openshell-registry/` | Yes (1 file) | 309 |
| `crates/openshell-constraint-theory/` | **No — stub** | 4 |
| `crates/openshell-fleet-homology/` | **No — stub** | 4 |
| `crates/openshell-flux-fracture/` | **No — stub** | 4 |
| `crates/openshell-flux-vm/` | **No — stub** | 4 |
| `crates/openshell-holonomy-consensus/` | **No — stub** | 4 |
| `crates/openshell-pythagorean48/` | **No — stub** | 4 |
| `AGENT.md`, `DEPENDENCIES.md`, `Makefile`, rebranded README, ~7 CI yamls | trivial | — |

The six 4-LOC stubs are pure SuperInstance math-research flavor (cf. the
ecosystem-survey's finding that ~21% of the whole org is `ternary`/`lau`/
`conservation`/`constraint` research). They are `pub mod foo;`-style placeholders
grafted onto NVIDIA's tree.

**So the headline "agent-onboarding platform" repo is ~95% NVIDIA's code, ~5%
SuperInstance's.** The README is rebranded to pitch onboarding, but the actual
NVIDIA/OpenShell product is a *sandbox runtime*, not an onboarding system.

---

## 3. The C ABI: real code, toy scope, README lies about the API

`openconstruct-abi/src/lib.rs` — 320 LOC, `edition = "2024"`, **no dependencies**
(Cargo.toml has empty `[dependencies]`), 7 unit tests that pass on the symbols.
It is genuinely `extern "C"`, uses `Box::into_raw`/`Box::from_raw` correctly,
has null-safety, and frees `CString`s properly. As Rust FFI mechanics go, it's
competent. **But:**

- **The "module registry" is 10 hardcoded modules** — all SuperInstance
  math/plato research concepts: `spectral-graph-core`, `conservation-regime`,
  `sheaf-cohomology`, `symplectic-geometry`, `plato-room`, `plato-puppeteer`,
  `plato-manus`, `conservation-protocol`, `spectral-deadband`, `tropical-algebra`.
  None of these correspond to installable SuperInstance packages (the survey
  confirmed almost nothing in the org is on a registry).

- **Only phases 1, 2, and 5 are implemented.** Real symbols: `oc_registry_*`,
  `oc_session_start`, `oc_declare_agent`, `oc_select_modules`, `oc_session_phase`,
  `oc_generate_config`, `oc_string_free`, `oc_result_free`. Phase 3 (interface
  selection) and phase 4 (connection setup) exist *only as constants*
  (`OC_PHASE_INTERFACE_SELECTION`, `OC_PHASE_CONNECTION_SETUP`) — no function
  advances the session into or out of them.

- **`oc_generate_config` emits malformed JSON.** It hand-rolls a `format!`
  string (no `serde_json`) and prints the modules vector with `{:?}` — Rust's
  Debug formatter — producing e.g. `["spectral-graph-core", "plato-room"]`
  *only* by luck of how `Vec<String>` debug-prints; it is not a real JSON
  serializer and would break on any string containing a quote/backslash.

- **The README documents a fictional API.** `openconstruct-abi/README.md` lines
  21–44, 50–60 advertise `oc_session_create(reg)`, `oc_session_declare(sess,
  name, model)`, `oc_session_select_modules(sess, csv)`,
  `oc_session_choose_interface(sess, type)`, `oc_session_connect(sess, addr)`,
  `oc_session_generate_config(sess)`. **None of these exist in `lib.rs`.** The
  real functions are `oc_session_start()` (no registry arg), `oc_declare_agent`,
  `oc_select_modules`, `oc_generate_config`. The README's C quick-start would
  not link.

- **License inconsistency:** `Cargo.toml` has no `license` field; `README.md`
  says "MIT"; the parent `OpenConstruct` is Apache-2.0 (inherited from NVIDIA).

It has **zero connection to NVIDIA/OpenShell.** It does not call into the
sandbox, the gateway, the policy engine, or anything. It is a standalone,
in-memory phase machine.

---

## 4. The "multi-language SDK" claim, debunked

This is where the cluster's coherence collapses. Systematic check of all 20
non-headline repos for FFI evidence (`extern "C"`, ctypes/cffi/dlopen, cgo, JNI,
P-Invoke, `@cDecl`, `@cImport`, `#include "openconstruct.h"`, linking
`-lopenconstruct_abi`):

| Repo | Bind to C ABI? | What it actually does | Verdict |
|---|---|---|---|
| **`openconstruct-c`** | **YES** (`#include "openconstruct.h"`, `Makefile` builds `libopenconstruct_abi.a` via cargo, links `-lopenconstruct_abi`) | Header + `example.c` + `test.c`, calls only real symbols | **The one real binding — works** |
| `openconstruct-python` | No (no ctypes/cffi/dlopen — grep returned 0 hits) | In-process `OpenConstructClient`, `uuid4()` session IDs, own 5-phase flow | Pure reimplementation |
| `openconstruct-go` | No (no cgo) | Own `Client`, `crypto/rand` session IDs | Pure reimplementation |
| `openconstruct-ruby` | No (no FFI) — gemspec falsely says "Thin Ruby binding" | Own `Client` | Pure reimplementation |
| `openconstruct-swift` | No (no modulemap/`@cDecl`) | Own `OpenConstructClient` | Pure reimplementation |
| `openconstruct-zig` | No (no `@cImport`/`extern`) | Own `Client` | Pure reimplementation |
| `openconstruct-ts` | No (no NAPI/node-ffi) | Own client | Pure reimplementation |
| `openconstruct-java` | No (no JNI) — pom falsely says "Java binding" | Own client | Pure reimplementation |
| `openconstruct-cs` | No (no DllImport/P-Invoke) — csproj falsely says "C# binding" | Own `OpenConstructClient` | Pure reimplementation |
| `openconstruct-rust` | No — `Cargo.toml` falsely sets `categories = ["api-bindings"]` | Own `OpenConstructClient` | Pure reimplementation |
| `openconstruct-jupyter` | No | In-memory notebook toy (shadows/rooms/fleet/ticks), `connect()` just flips a bool + hardcodes a fake agent | Pure reimplementation |
| `openconstruct-esp32` | No | Unrelated WiFi + *placeholder* MQTT + GPIO/sensor Arduino lib ("This is a placeholder structure"). No onboarding flow at all | Stub / unrelated |
| `openconstruct-jetson` | No (its `extern "C"` block is mock CUDA helpers, not `oc_*`) | Mock CUDA/TensorRT scaffold, **has a compile bug** (`strcpy(name, …)` references undefined `name`) | Pure reimplementation, broken |
| `openconstruct-kernel` | No | Sensor-tick + hardware profiling; "ForgeFlux metabolism … same kernel, every scale" | Unrelated Rust crate |
| `openconstruct-modular` | No | Generic 5-crate plugin framework (`Module` trait, `EventBus`) | Unrelated Rust workspace |
| `openconstruct-catalog` | No | Semver/dependency-resolution package-manager | Unrelated Rust crate |
| `openconstruct-mercury` | No (no build manifest; `mmc --make`) | Pure-Mercury formal-verification predicates ("proves" properties of a system nothing implements) | Pure reimplementation |
| `openconstruct-examples` | Claims to (`<openconstruct.h>`, `-lopenconstruct`) | **Invents a 3rd fictional API** — see below | **Broken — would not compile** |
| `openconstruct-docs` | N/A | 28 markdown files (~15K lines) describing a 4th fictional system | Docs only |
| `openconstruct-landing` | N/A | Static HTML; code samples use a 5th fictional API (`Agent::builder(…).with_sense(…).build().await?`); markets "500 Repositories / 500 Tests / 12 Languages / Apache 2.0" (reality: ~20 repos, MIT, "12 languages" includes ones marked "Soon") | Marketing only |

### The "fictional APIs" problem, made concrete

There are **at least four mutually-incompatible API surfaces** described in this
cluster, and none of the example/SDK code actually runs against any of them:

1. **Real ABI** (`openconstruct-abi/src/lib.rs`): `oc_registry_create`,
   `oc_declare_agent`, `oc_select_modules`, `oc_generate_config`, … (phases 1/2/5 only).
2. **ABI README's fictional API**: `oc_session_create`, `oc_session_declare`,
   `oc_session_choose_interface`, `oc_session_connect`, `oc_session_generate_config`.
3. **Examples' fictional API** (`openconstruct-examples/examples/c/onboard.c`):
   `oc_client_connect`, `oc_client_onboard`, `oc_session_heartbeat`,
   `oc_session_disconnect`, `oc_error_*`, `oc_session_room`, `oc_session_agent_id`
   — verified: I grepped the C example and got exactly these 11 symbols, **none**
   of which exist in surface #1 *or* #2. It describes a `ws://` coordinator +
   heartbeat network that nothing implements. The Python/Rust/TS/Go examples
   likewise import symbols (`CoordinatorClient`, `Capability`, `Session`) that
   the corresponding SDK repos do not export.
4. **Docs/landing fictional API**: `openconstruct init`, `Agent::builder()`,
   `agent.subscribe::<VisionEvent>()`, `agent.tick()`, an "8-layer architecture"
   with "40+ modules."

A grep across all 20 repos for the README's fake symbols (`oc_session_create`,
`oc_session_declare`, `oc_session_choose_interface`, `oc_session_connect`, …)
returned **zero hits** — no SDK was even built against the misleading README.
Each repo just invented its own surface. **Internal consistency within each
language SDK is fine** (they have real tests, phase machines, validation); **cross-repo
consistency is essentially zero.**

### Net

The "polyglot keystone" pitch — `openconstruct-abi/README.md:84`: *"Every language
binding … either calls this ABI directly or reimplements the same protocol"* — is
not true. One binding calls it; the rest reimplement *different* protocols.

---

## 5. The one substantive original piece: `openconstruct-cli` (685 LOC)

This is the most real SuperInstance-original code in the cluster. It is a `clap`
CLI with subcommands `init`, `status`, `sense {list,enable,disable}`, `fleet
{discover,join,leave,members}`, `tick {post,read}`, `room {create,list,join,leave}`,
`build <name>`, `publish <target>`. Read in full (`crates/openconstruct-cli/src/main.rs`).

**What actually works:**
- `init` — a `dialoguer`-driven 5-phase wizard that writes `~/.openconstruct/agent.toml`.
- `status` — reads that TOML back and pretty-prints it.
- `sense enable/disable` — toggles bools in the TOML.
- `tick post/read` — appends/reads `~/.openconstruct/ticks.jsonl` (a genuine local
  append-only log; this part is functional).
- `room create/list` — appends/reads `~/.openconstruct/rooms.jsonl`.
- `build <name>` — scaffolds a starter module dir (Cargo.toml + lib.rs / __init__.py / index.js).

**What is stubbed (prints a message and returns `Ok(())`):**
- `fleet discover` → "Fleet discovery requires a running agent daemon — coming soon."
- `fleet join <addr>` → "Fleet join queued (requires running agent daemon)."
- `fleet leave` → "✓ Left fleet" (does nothing).
- `fleet members` → prints only the local agent.
- `room join/leave` → prints "✓ Joined/Left room '<name>'" (no state change).
- `publish {crates,pypi,npm,all}` → prints the publish command and "auto-publish
  coming soon."

**It does not use the C ABI at all** — no FFI, no link. It is a self-contained
local config/TOML/JSONL manager. There is no network, no daemon, no fleet, no
other agent. It is a single-user, single-machine onboarding wizard.

So the cluster's best original artifact is a `~/.openconstruct/agent.toml` editor
with a working local tick-board log.

---

## 6. Authorship: two bulk-generation passes

Every one of the 23 repos is **exactly 1 commit**. The commits fall into two
batches (verified via `git log` on each clone):

- **15 repos, 2026-05-29, author "OpenClaw", identical subject** "docs:
  world-class README rewrite (zero-shot agent audit)": `openconstruct-c`, `-cs`,
  `-esp32`, `-examples`, `-go`, `-hub`, `-java`, `-jetson`, `-jupyter`,
  `-mercury`, `-python`, `-ruby`, `-swift`, `-ts`, `-zig`.
- **8 repos, 2026-06-08, author "Casey Digennaro"**, subjects "ci: add CI
  workflow" or "feat: add memory/JOURNAL.md for ensign duty log": `OpenConstruct`
  (headline), `openconstruct-abi`, `-catalog`, `-docs`, `-kernel`, `-landing`,
  `-modular`, `-rust`.

"OpenClaw" is a SuperInstance-branded agent persona (cf. `cudaclaw`, `openclaw.md`
elsewhere in the org). This is two single-shot generation passes, not organic
development — consistent with the ecosystem-survey's broader finding that ~1,423
repos were pushed on a single date (2026-06-08) and the org is a bulk-generated
sketchbook.

---

## 7. Correcting the ecosystem-survey: the "36 MB bloat" is not in the tree

The survey (`§3 #11`) flagged "`openconstruct-rust` is 36 MB (bloated, probably
vendored assets/binaries) — audit before adopting." I checked: the GitHub `size`
field reports full-repo size **including git history**, but the actual `--depth 1`
checkout is **324 KB** with a **204 KB** `.git` — i.e. tiny. Same for
`openconstruct-catalog` (GitHub 48 MB → checkout 336 KB, `.git` 208 KB) and
`openconstruct-zig` (GitHub 4.2 MB → checkout 236 KB). `find … -size +100k`
found **no large files** in any of the three trees.

The bloat is in the *full* git history (large blobs added and later removed, only
fetchable with an unshallow clone) — **not** in vendored binaries in the current
state. A fork of the current `HEAD` would not carry that bloat. So the survey's
specific concern here is a **false alarm** caused by trusting the `size` field;
the actual code in all three is small and clean. (The code just isn't very good —
see §4 — but it isn't bloated.)

---

## 8. Verdict vs. the ecosystem-survey's "medium-high" rating

The survey rated this cluster **"medium-high"** purplepincher fit on the strength
of its *stated* mission ("a modular onboarding standard — strong fit") and
flagged it for deep-dive. Having done the deep-dive:

- **The mission is on-target for purplepincher** (modular tools that let any
  language onboard to a fleet/protocol is exactly the right framing).
- **The execution is not.** The "fleet/protocol" the SDKs onboard you to does not
  exist; the one binding that works binds to a toy that emits malformed JSON and
  registers 10 fake modules; the headline repo is someone else's product; and the
  20 "SDKs" are cosmetic parallel reimplementations that don't even agree with
  each other.
- **Revised rating: low.** This is **fragmented and incomplete like most of what
  has been found so far**, not "a genuinely useful, coherent multi-language SDK
  story." The coherence is entirely in the READMEs, not the code.

The one genuinely valuable thing in the cluster — `NVIDIA/OpenShell` — is
valuable *independent* of SuperInstance: it's a real, actively-maintained,
Apache-2.0 sandbox runtime from NVIDIA with 7k+ stars. SuperInstance's fork adds
almost nothing to it.

---

## 9. Concrete fork/polish recommendation

**Do not fork the cluster as a cluster.** It is not a system; it is 23 cosmetic
variations on a README. Specific guidance:

1. **Do not fork `SuperInstance/OpenConstruct`.** It is a stale, 1-commit snapshot
   of `NVIDIA/OpenShell` with ~5% SuperInstance additions (a local CLI, a toy C
   ABI, six 4-LOC stubs). If you want an agent-sandbox runtime, **use upstream
   `NVIDIA/OpenShell` directly** (it's Apache-2.0, on PyPI as `openshell`, has a
   Helm chart, and is actively maintained). Track upstream, don't track the fork.
   If you want the *onboarding* idea, build it fresh (see #4).

2. **Do not fork any of the 9 "language SDK" reimplementations** (`-python`,
   `-go`, `-ruby`, `-swift`, `-zig`, `-ts`, `-java`, `-cs`, `-rust`). They bind
   to nothing, they don't agree with each other, and their package manifests lie
   ("binding", `api-bindings`). They have negative value as a starting point
   because they encode a fictional protocol.

3. **Do not fork `openconstruct-abi` as-is.** It's the closest thing to a real
   reusable artifact (320 LOC, real FFI mechanics, tests), but it (a) targets a
   fictional onboarding protocol, (b) emits malformed JSON, (c) registers 10 fake
   modules, and (d) its own README misdescribes its API. **If** you want a
   polyglot C ABI as the keystone of a real onboarding standard, the *shape* of
   this repo (opaque handles + `extern "C"` + phase machine + `CString` ownership)
   is a reasonable template — but you'd rewrite the body to target something real
   (e.g. an OpenShell sandbox session, or a purplepincher-defined protocol), swap
   the hand-rolled `format!` for `serde_json`, and fix the README to match the
   code. That's a from-scratch design that *reuses the file structure*, not a
   polish.

4. **The only piece worth lifting verbatim is small.** `openconstruct-cli/src/main.rs`
   has a clean `clap` skeleton for a local agent-config wizard (`init` → write
   `~/.openconstruct/agent.toml`) and a working JSONL tick-board (`tick post/read`).
   The `dialoguer` wizard flow and the `AgentConfig`/`TickEntry` serde structs are
   ~150 lines of genuinely reusable scaffolding for a *local* agent-onboarding
   UX. Lift those patterns; throw away the stubbed `fleet`/`publish` commands and
   the fake "coming soon" daemon.

5. **`openconstruct-c` is the only binding that actually works** — because it's
   the only one that talks to the real (toy) ABI. If you ever did build a real
   C ABI, this repo's `Makefile` (builds the `.a` via cargo, links with
   `-lopenconstruct_abi`) is a correct minimal template for a C-consumer repo.

6. **Treat `openconstruct-hub`'s module catalog as a discovery index, not a
   dependency list.** Its README links to ~50 other SuperInstance repos
   (`conservation-spectral-*`, `flux-*`, `agent-*-rs`, `cocapn-*`) — these are the
   SuperInstance research repos already evaluated in the ecosystem-survey's Tier
   2/3. The hub's "5-phase onboarding" and "architecture diagram" describe
   aspirations, not code; don't believe the architecture diagram.

### If purplepincher actually wants a "multi-language agent-onboarding SDK"

The honest path is to build it from scratch on top of **`NVIDIA/OpenShell` as the
runtime** + **one real C ABI** (designed fresh, with `serde_json` and a real
module registry that points at things that exist) + **bindings that genuinely
FFI to that ABI** (ctypes for Python, cgo for Go, JNI for Java, etc.) + **a
conformance test suite** so every binding is provably calling the same surface.
The SuperInstance cluster is useful here only as (a) a negative example of what
"20 SDKs that don't agree" looks like, and (b) a source of ~150 lines of CLI
scaffolding. Budget accordingly: this is a from-scratch build, not a polish.

---

## Methodology

- **Inventory:** `gh repo list SuperInstance --limit 4200 --json name --jq
  ".[].name" | grep -i openconstruct` → 23 names. Per-repo metadata via
  `gh api repos/SuperInstance/<name>` (size, lang, dates, stars, forks, archived).
- **Upstream verification:** `gh api repos/NVIDIA/OpenShell` and
  `gh api repos/SuperInstance/OpenConstruct` (`parent`/`source`/`fork` fields).
- **Clones:** all 23 + `NVIDIA/OpenShell` shallow-cloned (`--depth 1`) into
  `clones/oc/`.
- **Diff:** `diff -rq` between upstream and fork trees; LOC via `find … -name
  '*.rs' | xargs wc -l` per SuperInstance-added crate.
- **ABI read:** `openconstruct-abi/src/lib.rs` (320 LOC) and `README.md` read
  in full; real vs documented symbols cross-checked.
- **CLI read:** `openconstruct-cli/src/main.rs` (685 LOC) read in full.
- **FFI check:** systematic `grep` for `ctypes|cffi|dlopen|libopenconstruct|extern`,
  `cgo`, `JNI`, `DllImport`, `@cDecl`, `@cImport`, `#include "openconstruct.h"`
  across all 20 SDK/example/docs repos (delegated to a subagent that cited
  file:line for each finding; spot-checked `openconstruct-python` and
  `openconstruct-examples/examples/c/onboard.c` myself — both confirmed).
- **Bloat check:** `du -sh .git` + `find -size +100k` on `openconstruct-rust`,
  `-catalog`, `-zig` shallow clones.
- **Authorship:** `git log --oneline` + `git log -1 --format='%an %ai %s'` on all
  23 clones.
- **No edits, no pushes.** Read-only research.
