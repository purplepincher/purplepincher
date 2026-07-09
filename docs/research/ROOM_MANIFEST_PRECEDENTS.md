# Real manifest formats: what eight discovery systems actually put in the repo

> Companion to `tagging-precedent-findings.md`. That doc established the
> pattern (every working system layers a manifest on top of a topic for
> precision). This doc goes one level deeper: the **actual field-level
> schemas** a designer can copy from, with verbatim excerpts and, for each,
> an explicit answer to the question that matters most for this org's
> "a room can be a repo" direction — **how does a *cold agent* (not a human
> skimming a README) learn what a repo exposes and how to invoke it?**
>
> Same honesty discipline as the rest of `docs/research/`: every field below
> is quoted or paraphrased from a primary source fetched during this task
> (docs site, registry spec, or the manifest schema file itself). Sources
> are listed per system and consolidated at the end. This is research
> material for a parallel Fable-tier synthesis, not a design decision — none
> of these formats is being adopted, and nothing here overrides
> `ROADMAP.md`'s "sharing chapter" treating this direction as open.

## How to read this doc

For each of the eight systems there are three things:

1. **Real schema** — the actual field names, with required vs. optional
   marked, taken verbatim from the source. Not paraphrased patterns.
2. **Real example** — a snippet pulled from the source docs or from a
   canonical real repo (HACS docs, Arduino docs, PlatformIO docs, the MCP
   registry schema's own examples, Obsidian's sample-plugin, the VS Code
   manifest reference, npm/Cargo/PyPA specs).
3. **Cold-agent readability** — the load-bearing question for an
   agent-first format. A field is *agent-actionable* if a machine can act on
   it without a human in the loop (a command to run, an entry point to
   `require`, a transport to dial, a compatibility predicate to evaluate).
   It is *human-readable metadata* if it only helps a person decide (a
   description, an author, a funding URL).

A summary comparison table and the synthesis (which 2–3 are the strongest
model for an *agent-first* capability manifest, and why) are at the end.

---

## 1. HACS — `hacs.json`

**Primary source:** `hacs.xyz/docs/publish/start` (fetched; "General
requirements" → "hacs.json" section). The manifest "must be located in the
root of your repository."

### Real schema

The docs give an explicit key table. Only **`name` is required**; everything
else is optional:

| Key | Type | Required | What it does |
|---|---|---|---|
| `name` | string | **Yes** | Display name in the HACS UI. |
| `content_in_root` | bool | No | Content is in repo root, not a subdirectory. |
| `zip_release` | bool | No | Content shipped as a zipped release asset (integrations only; needs `filename`). |
| `filename` | string | No | Single file HACS looks for (plugin/theme/template/python_scripts/zip_release). |
| `hide_default_branch` | bool | No | Don't offer downloading the default branch. |
| `country` | string **or array** | No | ISO 3166-1 alpha-2 code(s). (Both `"NO"` and `["NO","SE","DK"]` appear in the official examples.) |
| `homeassistant` | string | No | Minimum required Home Assistant version. |
| `hacs` | string | No | Minimum required HACS version. |
| `persistent_directory` | string | No | Relative path kept safe across upgrades (integrations only). |

### Real example (verbatim from hacs.xyz)

```json
{
  "name": "My awesome thing",
  "country": "NO",
  "homeassistant": "0.99.9",
  "persistent_directory": "userfiles"
}
```

### Cold-agent readability — low

`hacs.json` is **almost entirely human-UI metadata**. There is no field that
tells an agent *how to invoke* the thing — HACS itself resolves installation
by category + the repo's release assets + the manifest's
`content_in_root`/`filename` hints, all within HACS's own runtime. The only
fields a cold agent can use as predicates are the two version floors
(`homeassistant`, `hacs`) and `country`. Notably, **`name` is the only
required field**, so a conforming manifest can be as thin as
`{"name": "x"}`. This is a manifest optimized for a *curated store UI that
already knows how to install each category*, not for a cold agent — which is
consistent with `tagging-precedent-findings.md`'s finding that HACS discovers
via a curated `hacs/default` list + a generated index, not by reading this
file cold.

---

## 2. Arduino Library Manager — `library.properties`

**Primary source:** `arduino.github.io/arduino-cli/dev/library-specification/`
(fetched; "library.properties file format"). "Unless noted otherwise below,
**all fields are required**" — the inverse of HACS.

### Real schema (key=value, UTF-8)

| Field | Required | What it does |
|---|---|---|
| `name` | **Yes** | Library name (A–Z, 0–9, space, `_`, `.`, `-`; must start with letter/number; names starting with `Arduino` are rejected from the index as reserved). |
| `version` | **Yes** | SemVer (`1.2.0` correct; `1.2` accepted; `r5`, `003`, `1.1c` invalid). |
| `author` | **Yes** | Authors (+ optional emails), comma-separated. |
| `maintainer` | **Yes** | Name and email of maintainer. |
| `sentence` | **Yes** | One-sentence purpose. |
| `paragraph` | **Yes** | Longer description (prepended by `sentence`). |
| `category` | **Yes** (defaults to `Uncategorized`) | One of: Display, Communication, Signal Input/Output, Sensors, Device Control, Timing, Data Storage, Data Processing, Other. |
| `url` | **Yes** | Project URL for the "More info" link. |
| `architectures` | **Yes** (defaults to `*`) | Comma-separated chip architectures (`avr`, `esp32`, …) or `*`. |
| `depends` | No | Comma-separated library deps, optionally version-constrained: `depends=ArduinoHttpClient (>=1.0.0)`. |
| `includes` | No | Headers to auto-`#include` when the library is imported. |
| `dot_a_linkage` | No | `true` → build as a `.a` archive. |
| `precompiled` | No | `true`/`full` — ship prebuilt `.a`/`.so`. |
| `ldflags` | No | Extra linker flags. |

Version-constraint operators on `depends`: `=`, `>`, `>=`, `<`, `<=`, `!`
(NOT), `&&`, `||`, `(` `)`.

### Real example (verbatim from the Arduino spec)

```
name=WebServer
version=1.0.0
author=Cristian Maglie <c.maglie@example.com>, Pippo Pluto <pippo@example.com>
maintainer=Cristian Maglie <c.maglie@example.com>
sentence=A library that makes coding a Webserver a breeze.
paragraph=Supports HTTP1.1 and you can do GET and POST.
category=Communication
url=http://example.com/
architectures=avr
includes=WebServer.h
depends=ArduinoHttpClient
```

### Cold-agent readability — medium, and notably *compatibility-oriented*

Two fields are genuinely machine-actionable as predicates:

- **`architectures`** — a cold agent can answer "will this build for my
  chip?" before downloading (`esp32` vs `*` vs `avr`). This is the single
  best simple hardware-compatibility field across all eight systems.
- **`depends`** — a cold agent can resolve the install plan (and
  `arduino-cli lib install` will auto-install them), with semver ranges.

`includes` is semi-actionable: it names the public header(s), i.e. *the
callable interface* at the source level. The rest (`sentence`, `paragraph`,
`category`, `author`) is human metadata. There is **no run/execute field** —
Arduino libraries are compiled into a sketch, not launched; "invocation" is
`#include` + linking, captured structurally (the `src/` layout) rather than
by a manifest key. So Arduino is a strong model for the **compatibility +
dependency** half of a manifest, and an honest negative example for the
**invocation** half (it doesn't need one, because its units aren't runnable
processes).

---

## 3. PlatformIO — `library.json`

**Primary source:** `docs.platformio.org/en/latest/manifests/library-json/`
(fetched; index page + `frameworks` + `platforms` field pages). "A manifest
file of a library package" that "allows developers to … define: compatible
frameworks and platforms; external dependencies; advanced build settings."
This is the system `tagging-precedent-findings.md` named as the closest
precedent for structured compatibility metadata.

### Real schema — full field list (every documented top-level key)

| Field | Type | Purpose |
|---|---|---|
| `$schema` | string | JSON Schema URL for validation (`pio pkg pack` validates). |
| `name` | string | Library name. |
| `version` | string | SemVer version. |
| `description` | string | Short description. |
| `keywords` | array | Search keywords. |
| `homepage` | string | Project homepage URL. |
| `repository` | object | Source repository (`type`, `url`). |
| `authors` | array | Author objects (`name`, `email`, `url`, `maintainer`). |
| `license` | string | SPDX license. |
| **`frameworks`** | string **or array** | Compatible frameworks, e.g. `["espidf", "freertos"]`, or `"*"` for all. |
| **`platforms`** | string **or array** | Compatible dev platforms, e.g. `["atmelavr", "espressif8266"]`, or `"*"`. |
| `headers` | array | Public header files the library provides (interface surface). |
| `examples` | array | Example paths. |
| `dependencies` | object | External library deps (by name → version/spec). |
| `export` | object | `include`/`exclude` globs for packaging. |
| `scripts` | object | Lifecycle scripts. |
| `build` | object | `flags`, `unflags`, `includeDir`, `srcDir`, `srcFilter`, `extraScript`, `libArchive`, `libLDFMode`, **`libCompatMode`**, `builder`. |

PlatformIO's docs are explicit that `frameworks`/`platforms` are **optional
and not enforced by default**: *"PlatformIO does not check platforms for
compatibility in default mode … If you need a strict checking for compatible
platforms for a library, please set `libCompatMode` to `strict`."* So the
field exists and is machine-readable, but the consumer chooses whether to
treat it as advisory or strict — a design choice directly relevant to this
org's "re-verify at read-time" stance.

### Real example (verbatim from the field pages)

```json
"frameworks": ["espidf", "freertos"]
"platforms": ["atmelavr", "espressif8266"]
```

### Cold-agent readability — high on compatibility, the strongest hardware model

This is the richest **compatibility declaration** of any format here:

- **`frameworks`** — a cold agent answers "is this built for `espidf`?"
  (directly analogous to "is this built for `cocapn`?").
- **`platforms`** — "is this for `espressif32`?" (the hardware-platform
  axis).
- **`headers`** — the declared public interface (what the library exposes for
  consumption), a step beyond Arduino's implicit `src/` convention.
- **`dependencies`** — the install plan, machine-resolvable.

Like Arduino, there is **no run/execute field** — a library is compiled in,
not launched. PlatformIO's strength is precisely the dimension this org's
ESP32 use case needs (framework + platform compatibility), which is why
`tagging-precedent-findings.md` and the `ILLUSTRATIVE_MANIFEST_SKETCH.md`
both lean on it. Its honest limitation: it models *libraries*, not
*runnable integrations*, so it has nothing to say about "how does an agent
dial up this thing at runtime."

---

## 4. MCP Registry — `server.json`  ★ most agent-first format of the eight

**Primary source:** the canonical JSON Schema file itself,
`raw.githubusercontent.com/modelcontextprotocol/registry/main/docs/reference/server-json/draft/server.schema.json`
(fetched in full; `$ref` → `ServerDetail`). Overview from
`modelcontextprotocol.io/registry`. This is the closest analog to what this
org needs, because **MCP is itself an agent-tool-discovery system**: the
manifest's stated purpose (from the schema description) is *"a static
representation of an MCP server. Used in various contexts related to
discovery, installation, and configuration."*

### Real schema (from the JSON Schema's `required` arrays — authoritative)

**Top level (`ServerDetail`) — required: `name`, `description`, `version`:**

| Field | Required | What it does |
|---|---|---|
| `name` | **Yes** | Reverse-DNS, e.g. `io.github.user/weather`. Pattern `^[a-zA-Z0-9.-]+/[a-zA-Z0-9._-]+$`, 3–200 chars. **Tied to a verified GitHub account or DNS-challenged domain** (the trust mechanism). |
| `description` | **Yes** | Human-readable, 1–100 chars. |
| `version` | **Yes** | SemVer-ish; **version *ranges* are rejected** (`^1.2.3`, `1.x`, `latest` all forbidden) — must pin a concrete version. |
| `title` | No | Display name (≤100 chars). |
| `websiteUrl` | No | Homepage/docs URL. |
| `icons` | No | Sized icons (PNG/JPEG/SVG/WebP). |
| `repository` | No | `{url, source, id, subfolder}`. `id` is the forge's repo id, "used to detect repository resurrection attacks — if a repository is deleted and recreated, the ID should change." |
| `packages` | No | Array of `Package` (see below) — *how to actually obtain & run it locally*. |
| `remotes` | No | Array of remote transports (already-hosted servers). |
| `_meta` | No | Reverse-DNS-namespaced vendor extension bag. |

**`Package` — required: `registryType`, `identifier`, `transport`:**

| Field | Required | What it does |
|---|---|---|
| `registryType` | **Yes** | `npm`, `pypi`, `cargo`, `oci`, `nuget`, `mcpb`, … |
| `identifier` | **Yes** | Package name (e.g. `@modelcontextprotocol/server-brave-search`) or direct download URL. |
| `transport` | **Yes** | `StdioTransport` (`{type:"stdio"}`), `StreamableHttpTransport` (`{type, url, headers}`), or `SseTransport`. |
| `version` | No | **Concrete version only** (ranges rejected). |
| `registryBaseUrl` | No | e.g. `https://registry.npmjs.org`. |
| `runtimeHint` | No | `npx`, `uvx`, `docker`, `dnx`, … |
| `packageArguments` / `runtimeArguments` | No | Positional or `--flag={value}` args, built from `Argument`/`Input` (with `value`, `valueHint`, `isSecret`, `isRequired`, `format` incl. `filepath`). |
| `environmentVariables` | No | Named env vars, same `Input` shape (incl. `isSecret`). |
| `fileSha256` | No | **SHA-256 of the package file for integrity verification**; "if present, MCP clients must validate the downloaded file matches the hash before running." |

### Real example (representative — assembled from the schema's own field-level `example` values)

```json
{
  "$schema": "https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json",
  "name": "io.github.user/weather",
  "description": "MCP server providing weather data and forecasts via OpenWeatherMap API",
  "version": "1.0.2",
  "repository": {
    "url": "https://github.com/modelcontextprotocol/servers",
    "source": "github",
    "subfolder": "src/weather"
  },
  "packages": [
    {
      "registryType": "npm",
      "identifier": "@modelcontextprotocol/server-weather",
      "version": "1.0.2",
      "runtimeHint": "npx",
      "transport": { "type": "stdio" },
      "environmentVariables": [
        {
          "name": "OPENWEATHER_API_KEY",
          "description": "API key for OpenWeatherMap",
          "isRequired": true,
          "isSecret": true,
          "format": "string"
        }
      ]
    }
  ]
}
```

### Cold-agent readability — very high, by design

This is the only format here where **a cold agent can go end-to-end from
discovery to invocation with no human in the loop**, because every step is a
machine-readable field:

- **What it is:** `name` (verified identity), `description`, `version`
  (pinned, not a range).
- **How to get it:** `packages[].registryType` + `identifier` + `version`
  (+ optional `registryBaseUrl`).
- **How to run it:** `transport` (the wire protocol), `runtimeHint`
  (`npx`/`docker`/…), `packageArguments`/`runtimeArguments`
  (positional & `--flag`), `environmentVariables` (incl. which are secret
  / required / file paths).
- **Whether to trust it:** reverse-DNS `name` bound to a verified owner;
  `repository.id` to detect resurrection attacks; `fileSha256` to verify the
  downloaded artifact before execution.

Two caveats worth carrying into any design that borrows from MCP: (a) the
**runtime *capabilities*** (which tools/resources/prompts a server exposes)
are **not** declared in `server.json` — a client discovers those by
handshaking the running server over MCP. The manifest only gets you to a
running process; "what can it do" is a runtime question. (b) The verified
namespace + central REST registry are an **operated** trust layer; the
manifest shape is copyable into a no-backend world, but the namespace-auth
trust guarantee is not (the same gap `tagging-precedent-findings.md` flags).

---

## 5. Obsidian community plugins — `manifest.json`

**Primary source:** `raw.githubusercontent.com/obsidianmd/obsidian-developer-docs/main/en/Reference/Manifest.md`
(the render at `docs.obsidian.md/Plugins/Releasing/Plugin+manifest` is
JS-rendered; the source markdown behind it gives an explicit required/optional
table). Cross-checked against
`obsidianmd/obsidian-sample-plugin/master/manifest.json` and
`obsidianmd/obsidian-releases/master/README.md`.

### Real schema — 7 required, 2 optional (plugins)

| Field | Type | Required | What it does |
|---|---|---|---|
| `id` | string | **Yes** | Plugin ID. Lowercase letters + hyphens only; can't end with `plugin`; can't contain `obsidian`. Must match the plugin's folder name (else e.g. `onExternalSettingsChange` won't fire). |
| `name` | string | **Yes** | Display name. |
| `version` | string | **Yes** | SemVer `x.y.z`. |
| `minAppVersion` | string | **Yes** | Minimum Obsidian version (the compatibility floor). |
| `description` | string | **Yes** (plugins) | What the plugin does. |
| `author` | string | **Yes** | Author name. |
| `isDesktopOnly` | boolean | **Yes** (plugins) | `true` if it uses Node.js/Electron APIs (gates mobile loading). |
| `authorUrl` | string | No | Link to author's site. |
| `fundingUrl` | string \| object | No | Donation link(s); the submission rules say remove it if you don't accept donations. |

(`themes` use a slightly different set — `name`, `version`, `minAppVersion`,
`author`, `authorUrl`, `fundingUrl` — with no `id`/`description`/`isDesktopOnly`.)

### Real example (verbatim from obsidian-sample-plugin/master/manifest.json)

```json
{
  "id": "sample-plugin",
  "name": "Sample Plugin",
  "version": "1.0.0",
  "minAppVersion": "1.0.0",
  "description": "Demonstrates some of the capabilities of the Obsidian API.",
  "author": "Obsidian",
  "authorUrl": "https://obsidian.md",
  "fundingUrl": "https://obsidian.md/pricing",
  "isDesktopOnly": false
}
```

A second repo-level file, `versions.json`, maps each released plugin version
to the minimum Obsidian version it needs (verbatim sample:
`{"1.0.0": "1.0.0"}`); Obsidian reads it to pick a compatible release.

### Cold-agent readability — low (no entry point in the manifest)

The manifest is **purely metadata**. The actual runtime entry point is the
compiled `main.js` (a `Plugin` subclass) — and **that path is not a manifest
field**; it's an Obsidian convention. The only fields a cold agent can act on
as predicates are `minAppVersion` (compatibility floor), `isDesktopOnly`
(environment gate), and `id` (which doubles as the folder name and as the
prefix Obsidian auto-applies to command IDs). So like HACS, Obsidian's
manifest is optimized for *its own host* (which already knows where `main.js`
lives), not for a cold agent. Discovery is the curated
`community-plugins.json` array (`{id, name, author, description, repo}`) in
`obsidianmd/obsidian-releases`, not the manifest.

---

## 6. VS Code extensions — `package.json` (with `contributes`)

**Primary source:** `code.visualstudio.com/api/references/extension-manifest`
(fetched; "Fields" table + "Example"). VS Code reuses npm's `package.json`
and layers VS Code-specific keys on top.

### Real schema — the discovery/capability-relevant subset

| Field | Required | What it does |
|---|---|---|
| `name` | **Yes** | Lowercase, no spaces; unique on the Marketplace. |
| `version` | **Yes** | SemVer. |
| `publisher` | **Yes** | Publisher id; the extension's identity is `${publisher}.${name}`. |
| `engines.vscode` | **Yes** | Min VS Code version; **cannot be `*`** (e.g. `^0.10.5`). |
| `main` | No | Entry point for the extension (Node). |
| `browser` | No | Entry point for the Web-extension variant. |
| `activationEvents` | No | When to activate (e.g. `onLanguage:markdown`). |
| **`contributes`** | No | **The structured capability declaration** — an object whose keys are *contribution points* (`commands`, `configuration`, `grammars`, `languages`, `snippets`, `debuggers`, `views`, …). |
| `capabilities` | No | `untrustedWorkspaces` / `virtualWorkspaces` support declarations. |
| `extensionKind` | No | `[ui, workspace]` — where it can run in remote configs. |
| `extensionDependencies` / `extensionPack` | No | Deps / bundled set (by `${publisher}.${name}`). |
| `categories` | No | Enum: Programming Languages, Snippets, Linters, Themes, Debuggers, Formatters, … |
| `keywords` | No | ≤30 marketplace search keywords. |

### Real example (verbatim from the VS Code reference)

```json
{
  "name": "wordcount",
  "displayName": "Word Count",
  "version": "0.1.0",
  "publisher": "ms-vscode",
  "description": "Markdown Word Count Example ...",
  "categories": ["Other"],
  "activationEvents": ["onLanguage:markdown"],
  "engines": { "vscode": "^1.0.0" },
  "main": "./out/extension",
  "contributes": {
    "commands": [
      { "command": "extension.wordCount", "title": "Word Count" }
    ]
  }
}
```

### Cold-agent readability — the best "what does it do" declaration, but host-coupled

`contributes` is the single best **structured capability declaration** in
this set: a host (or an agent) can read off exactly which commands,
configurations, languages, grammars, debuggers, views, etc. an extension
adds — this is the closest thing to a machine-readable "feature surface" of
any format here, and it's worth borrowing from. `main`/`browser` are the
entry points, and `engines.vscode` is a hard compatibility floor (must be
concrete, not `*`). The limitation for an agent-first design: the meaning of
each `contributes` key is **defined by the VS Code host API**, not
self-describing — a cold agent outside VS Code can see *that* an extension
contributes commands, but the commands only *do* something inside the VS
Code extension host. So it's a strong model for *declaring* capabilities,
weak for *invoking* them across contexts.

---

## 7. npm / Cargo / PyPI — discovery-relevant fields only

These three are general package registries, so only the fields relevant to
**capability/compatibility declaration and invocation** are pulled (not the
whole schema).

### 7a. npm — `package.json`

**Primary source:** `docs.npmjs.com/cli/v10/configuring-npm/package-json`
(fetched). `name` + `version` are required to publish.

Discovery/compatibility/invocation fields:

| Field | Role |
|---|---|
| `name`, `version` | Identity (required). |
| `description`, `keywords` | Registry **search** text ("listed in `npm search`"). |
| `engines` | Runtime compat, e.g. `{"node": ">=0.10.3 <15"}` — **advisory only** unless `engine-strict` is set. |
| `os`, `cpu` | Platform/arch allow-or-block lists (`["darwin","linux"]`, or `["!win32"]`). |
| **`bin`** | **The agent-actionable invocation primitive**: `{ "myapp": "bin/cli.js" }` installs a shell command on the user's PATH. A cold agent learns the exact command name and the file that runs. |
| `main` / `exports` | Programmatic entry point(s) (`require("foo")` → `main`'s module; `exports` defines multiple/conditional entry points and *encapsulates* the public surface). |
| `peerDependencies` | "I'm a plugin for host `tea@2.x`" — the host-compatibility relationship. |

Real snippet (verbatim from npm docs):

```json
{ "bin": { "myapp": "bin/cli.js" } }
```

### 7b. Cargo — `Cargo.toml`

**Primary source:** `doc.rust-lang.org/cargo/reference/manifest.html`
(fetched). `[package].name` is the only field Cargo itself requires;
crates.io adds more (`description`, `license`/`license-file`).

Discovery/compatibility/invocation fields:

| Field | Role |
|---|---|
| `[package] name`, `version` | Identity (`version` required to publish; defaults `0.0.0`). |
| `[package] description`, `keywords` (≤5), `categories` (≤5, from a fixed list) | crates.io **search/discovery**. |
| `[package] edition` | Rust edition (compat). |
| `[package] rust-version` | MSRV — minimum supported Rust toolchain (compat floor). |
| `[[bin]] name`, `path` | **Invocation**: binary targets (what `cargo run` / `cargo build` produces as executables). |
| `[lib] name`, `path` | Library target (the public crate interface). |
| `[package] default-run` | Which `[[bin]]` `cargo run` picks by default. |
| `[dependencies]` | Dep graph (machine-resolvable). |

Real snippet (verbatim from the Cargo Book):

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2024"

[[bin]]
name = "hello"
path = "src/bin/hello.rs"
```

### 7c. PyPI / pyproject.toml — entry points

**Primary source:** `packaging.python.org/en/latest/specifications/entry-points/`
(fetched). The invocation primitive is the **entry point** (three properties:
*group*, *name*, *object reference* `module:object.attr`).

| Mechanism | Role |
|---|---|
| `[project] name`, `version`, `description`, `keywords` | Identity + PyPI search. |
| `[project] classifiers` | Fixed taxonomy (e.g. `Framework :: Django`, `Programming Language :: Python :: 3`) — the closest PyPI has to a structured category/capability field. |
| `requires-python` | Interpreter compat floor (the Python analogue of `engines`/`rust-version`). |
| **`[project.scripts]`** | **console_scripts** — installs shell commands. `foo = "foomod:main"` creates a `foo` command that runs `foomod.main()`. |
| `[project.gui-scripts]` | Same, GUI (no console) variant. |
| `[project.entry-points."<group>"]` | Plugin discovery: a host reads `group` to find plugins (e.g. `pytest11`, `pygments.styles`). |

Real snippet (verbatim from the PyPA spec's `entry_points.txt` equivalent):

```ini
[console_scripts]
foo = foomod:main
```

### Cold-agent readability (npm/Cargo/PyPI) — high on invocation, low on capability

All three share the same strength and the same gap:

- **Strength — invocation is a first-class, machine-actionable field.** npm's
  `bin`, Cargo's `[[bin]]`, and PyPI's `[project.scripts]` each let a cold
  agent learn the **exact command** a package exposes and the file/function
  behind it. Combined with the compat floors (`engines`/`os`/`cpu`,
  `rust-version`/`edition`, `requires-python`), an agent can decide "can I
  run this here?" and "what command do I run?" without a human.
- **Gap — "what capability does it expose" is not declared.** `keywords` and
  Cargo `categories` / PyPI `classifiers` are coarse search tags, not a
  structured feature surface. None of the three declares *what the package
  does at the interface level* the way VS Code `contributes` or MCP
  runtime-capability discovery do. (Python's `entry-points."<group>"` is the
  nearest exception — a host can enumerate plugins for a group — but the
  *meaning* of each group is host-defined.)

---

## 8. Awesome-lists — confirmed: no manifest (a real negative finding)

**Primary source:** `raw.githubusercontent.com/sindresorhus/awesome-lint/main/readme.md`
(fetched). `awesome-lint` is a **Markdown linter for the Awesome-list
convention**, not a discovery mechanism and not a topic scraper.

What it actually checks (from the README's example output and rules):

- Presence of the **Awesome badge** after the main heading (`awesome-badge`).
- **List-item format** — items must match `- [Name](url) - description`
  (the `awesome-list-item` rule flags deviations).
- Marker style (`unordered-list-marker-style`), **trailing slashes**
  (`trailing-slash`), and **repo age** (`fetch-depth: 0` is required in CI
  "so that we can check the repo age").

It runs as `npx awesome-lint <repo-url>` and outputs Markdown lint errors.
There is **no machine-readable manifest**: the "schema" is the Markdown list
item itself, validated by a linter. No field tells an agent what a linked
repo does or how to invoke it — only that a human formatted the entry
correctly and linked a repo old enough to pass the age check.

**So the negative finding in `tagging-precedent-findings.md` holds and is
strengthened:** awesome-lists are human-curated indexes with a *format
linter*, not machine-discoverable capability metadata, and they provide no
precedent for an agent-actionable manifest. The useful transferable idea is
small: **a linter over a tiny fixed format** is a cheap way to enforce a
no-backend convention (relevant if this org ever wants quality-gating
without an operated registry).

---

## Summary comparison

| System | Manifest file | Required-core (verbatim) | **Invocation field?** (agent-actionable) | **Compatibility field?** | **Integrity/trust field?** |
|---|---|---|---|---|---|
| HACS | `hacs.json` | `name` | No (HACS installs by category) | `homeassistant`, `hacs` (version floors) | No (trust = curated `hacs/default` list) |
| Arduino | `library.properties` | name, version, author, maintainer, sentence, paragraph, category, url, architectures | No (compiled in) | **`architectures`** (chip), `depends` | No (trust = curated `repositories.txt` + ArduinoBot) |
| PlatformIO | `library.json` | (none hard-required at file level; validated by `pio pkg pack`) | No (compiled in) | **`frameworks`**, **`platforms`**, `headers` | No (trust = central registry) |
| **MCP** | `server.json` | `name`, `description`, `version` | **Yes** — `packages[]` (registryType/identifier/version/runtimeHint/**transport**/args/env) | version floors + runtime/transport choice | **Yes** — verified reverse-DNS name, `repository.id`, `fileSha256` |
| Obsidian | `manifest.json` | id, name, version, minAppVersion, description, author, isDesktopOnly | No (`main.js` is convention, not a field) | `minAppVersion`, `isDesktopOnly` | No (trust = curated `community-plugins.json`) |
| VS Code | `package.json` + `contributes` | name, version, publisher, engines.vscode | `main`/`browser` (entry point) | `engines.vscode`, `capabilities`, `extensionKind` | No (trust = Marketplace/Open VSX) |
| npm | `package.json` | name, version | **Yes** — `bin`, `main`/`exports` | `engines`, `os`, `cpu`, `peerDependencies` | No (trust = central registry) |
| Cargo | `Cargo.toml` | `[package] name` | **Yes** — `[[bin]]`, `[lib]`, `default-run` | `edition`, `rust-version` | No (trust = central registry) |
| PyPI | `pyproject.toml` | (build-backend-defined) | **Yes** — `[project.scripts]`, `[project.gui-scripts]` | `requires-python`, `classifiers` | No (trust = central registry) |
| awesome-lists | *(none — Markdown item)* | (list-item format) | No | No | No (linter checks format + repo age only) |

Two columns tell the story for an agent-first design: only **MCP** fills all
three of *invocation*, *compatibility*, and *integrity*; the package
registries (npm/Cargo/PyPI) nail *invocation* but not *capability*;
PlatformIO and Arduino nail *compatibility* but not *invocation*; HACS,
Obsidian, and awesome-lists fill none of the three for a cold agent.

---

## Synthesis: the strongest models for an *agent-first* capability manifest

An "agent-first" manifest has to let a cold agent answer three questions with
no human in the loop: **(a) what is this and is it compatible with my
environment? (b) how do I obtain and run it? (c) should I trust it enough to
run it?** No single one of the eight answers all three perfectly, but three
together cover the design space this org's "a room can be a repo" direction
needs. Ranked:

### 1. MCP `server.json` — the overall model to copy (and the closest analog)

It is the only format here purpose-built for *agent* discovery, and the only
one that is end-to-end machine-actionable: `name` (verified reverse-DNS
identity) + `version` (pinned) → `packages[]` (where to get it, which
runtime, which transport) → `transport`/`runtimeArguments`/`environmentVariables`
(how to launch it, including which inputs are secrets or file paths) →
`fileSha256` + `repository.id` (verify the artifact and detect a resurrected
repo). For "another captain's agent discovers and *uses* a published
integration without reinventing it," MCP is the direct precedent. **What to
borrow:** the package/transport/runtime/arguments shape, the pinned-version
rule, the secret-flagging on inputs, and the integrity hash. **What not to
borrow:** the operated central registry and the namespace-auth trust layer —
both violate this org's hard "no operated backend" constraint (exactly the
gap `tagging-precedent-findings.md` flags). The honest caveat to carry over:
MCP declares *how to reach a running server*, not *what it can do* — the
tool/resource/prompt capabilities are discovered at runtime over the
protocol, so an MCP-style manifest alone still leaves "what does it expose?"
as a post-launch question.

### 2. PlatformIO `library.json` — the model for the *compatibility* axis

MCP has nothing to say about hardware/framework fit, which is the dimension
this org's ESP32 use case cares about most. PlatformIO's
`frameworks`/`platforms`/`headers` are the cleanest machine-readable
compatibility model in the set, and PlatformIO's explicit choice to make them
**advisory by default and strict only when `libCompatMode = strict`** is
almost a one-line specification of this org's own "re-verify at read-time"
stance — the manifest declares compatibility, the consumer decides whether to
trust it. **What to borrow:** the `frameworks`/`platforms` shape (literally
the model `ILLUSTRATIVE_MANIFEST_SKETCH.md` already adapts to
`framework: cocapn` / `platforms: [esp32]`), plus the `headers` idea of
declaring the public interface surface. **What not to borrow:** the
library-not-runnable assumption — PlatformIO's manifest has no invocation
field because its units are compiled-in, which is exactly why it can't stand
alone for an agent that needs to *run* something.

### 3. npm `package.json` (`bin`/`exports`/`engines`) — the model for the *invocation primitive*

If a published "room" exposes a command or a callable interface (a CLI, an
agent endpoint, a function to dispatch to), npm's `bin` is the minimal,
universally-understood way to declare it: a name→file map that becomes a real
command on the PATH. Paired with `engines`/`os`/`cpu` (runtime compat) and
`exports` (encapsulated public surface), it answers "how do I invoke this,
and on what runtime?" in the fewest possible fields. Cargo's `[[bin]]` and
PyPI's `[project.scripts]` are the same idea in TOML/INI clothing; npm is
the most widely recognized. **What to borrow:** the *concept* of a dedicated,
flat invocation field separate from metadata (a cold agent shouldn't have to
parse a README to find the command). **What not to borrow:** npm's
`keywords`-only notion of "capability" — coarse tags are not a feature
surface, which is the gap that VS Code `contributes` (worth studying
separately) addresses at the cost of being host-coupled.

### Honorable mention: VS Code `contributes` for *capability declaration*

If the design ever needs a structured "what does this expose" surface
(commands, config schemas, view providers) rather than just "how do I run
it," `contributes` is the best-existing model — a typed object whose keys are
well-known contribution points. It ranked outside the top three only because
its keys are meaningful *inside the VS Code host*, so it doesn't travel well
to a cold-agent-outside-a-host context. Still, it's the right place to look
if "declare the feature surface" becomes a first-class requirement.

### One-line takeaway for the parallel synthesis

The smallest agent-first manifest that real precedent supports is a fusion
of three fields this org can already name: an **MCP-style invocation block**
(how to obtain + run it, with secret-flagged inputs and an integrity hash),
a **PlatformIO-style compatibility block** (`frameworks`/`platforms`/interface
headers, advisory-by-default), and an **npm-style flat entry-point/command
field** — with trust deferred to per-discovery re-verification rather than
the central registry that every one of these systems except awesome-lists
actually relies on.

---

## Sources (all fetched and verified during this task)

- HACS — `hacs.xyz/docs/publish/start` (hacs.json key table + examples;
  `name` is the only required key; manifest must be in repo root).
- Arduino — `arduino.github.io/arduino-cli/dev/library-specification/`
  (full `library.properties` field list; "all fields required unless noted";
  `architectures`/`depends`/`includes`; version-constraint operators).
- PlatformIO — `docs.platformio.org/en/latest/manifests/library-json/index.html`,
  `…/fields/frameworks.html`, `…/fields/platforms.html` (full field list;
  `frameworks`/`platforms` advisory-by-default, strict via `libCompatMode`).
- MCP Registry —
  `raw.githubusercontent.com/modelcontextprotocol/registry/main/docs/reference/server-json/draft/server.schema.json`
  (the authoritative JSON Schema; `ServerDetail` requires
  `name`/`description`/`version`; `Package` requires
  `registryType`/`identifier`/`transport`; `fileSha256`, `repository.id`,
  reverse-DNS verified `name`) + `modelcontextprotocol.io/registry` (overview,
  namespace-auth, registry-is-preview).
- Obsidian —
  `raw.githubusercontent.com/obsidianmd/obsidian-developer-docs/main/en/Reference/Manifest.md`
  (the required/optional tables; 7 required, 2 optional), cross-checked with
  `obsidianmd/obsidian-sample-plugin/master/manifest.json`,
  `…/master/versions.json`, and `obsidianmd/obsidian-releases/master/README.md`.
- VS Code — `code.visualstudio.com/api/references/extension-manifest`
  (Fields table; `name`/`version`/`publisher`/`engines.vscode` required;
  `contributes`, `main`/`browser`, `capabilities`, `extensionKind`; example).
- npm — `docs.npmjs.com/cli/v10/configuring-npm/package-json`
  (`name`/`version` required to publish; `bin`, `main`/`exports`, `engines`,
  `os`, `cpu`, `peerDependencies`).
- Cargo — `doc.rust-lang.org/cargo/reference/manifest.html`
  (`[package].name` the only Cargo-required field; `[[bin]]`, `[lib]`,
  `default-run`, `edition`, `rust-version`, `keywords`, `categories`).
- PyPI — `packaging.python.org/en/latest/specifications/entry-points/`
  (entry-point data model: group/name/`module:object.attr`; `console_scripts`
  /`gui_scripts`; plugin-group discovery) + the pyproject
  `[project.scripts]` mapping.
- awesome-lists — `raw.githubusercontent.com/sindresorhus/awesome-lint/main/readme.md`
  (Markdown linter; `awesome-badge`, `awesome-list-item`, `trailing-slash`,
  repo-age checks; no manifest, no topic scraping — negative finding
  confirmed).
