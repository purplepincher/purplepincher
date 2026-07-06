# `cocapn` PyPI tarball audit — checked against the three candidate repos

**Date:** 2026-07-06  
**Task:** Download the published `cocapn` PyPI artifact and compare its actual source against `SuperInstance/cocapn`, `SuperInstance/cocapn-py`, and `SuperInstance/cocapn-python`.  
**Status:** Evidence gathered; findings below are checked facts, not recommendations.

---

## 1. What PyPI actually publishes

Queried `https://pypi.org/pypi/cocapn/json` on 2026-07-06.

| Field | Value |
|---|---|
| `info.name` | `cocapn` |
| `info.version` | `0.3.0` |
| `info.summary` | `Cocapn Fleet v3.1 — Async fleet engine with Pydantic v2, batch ops, SSE` |
| `info.author` | `None` |
| `info.author_email` | `None` |
| `info.home_page` | `None` |
| `info.project_urls` | `None` |
| `info.license` | `MIT` |

Published release history (from `releases`):

| Version | Files | Upload time (UTC) |
|---|---|---|
| `0.1.0` | `cocapn-0.1.0-py3-none-any.whl` | 2026-04-20T02:22:29 |
| `0.2.0` | wheel + `cocapn-0.2.0.tar.gz` | 2026-04-26T23:09:54/55 |
| `0.2.1` | wheel + `cocapn-0.2.1.tar.gz` | 2026-04-29T18:31:51/52 |
| `0.3.0` | wheel + `cocapn-0.3.0.tar.gz` | 2026-05-26T18:48:18/20 |

Downloaded the current sdist from the URL returned by the API:

```
https://files.pythonhosted.org/packages/0d/e6/5c2122dd7936d822d5a61d752aa0ea611363e6c2dfe6d28151c5afa82708/cocapn-0.3.0.tar.gz
```

Extracted to `cocapn-0.3.0/`.

---

## 2. What the published 0.3.0 tarball contains

### 2.1 Package manifest

`cocapn-0.3.0/pyproject.toml`:

```toml
[project]
name = "cocapn"
version = "0.3.0"
description = "Cocapn Fleet v3.1 — Async fleet engine with Pydantic v2, batch ops, SSE"
readme = "README.md"
requires-python = ">=3.9"
license = {text = "MIT"}
dependencies = [
    "fastapi>=0.100.0",
    "pydantic>=2.0.0",
    "uvicorn>=0.20.0",
]

[tool.setuptools.packages.find]
include = ["cocapn*"]
```

`cocapn-0.3.0/cocapn/__init__.py`:

```python
__version__ = "3.1.0"

from .models import Agent, Context, Tile, Stream, Task, Rule, TileBatch, FleetStatus
from .engine import Fleet
from .storage import JSONLStore

__all__ = [
    "Fleet", "Agent", "Context", "Tile", "Stream", "Task", "Rule",
    "TileBatch", "FleetStatus", "JSONLStore"
]
```

Note: the package `__version__` is `3.1.0` while the distribution version is `0.3.0`.

### 2.2 Module structure

```
cocapn-0.3.0/cocapn/
├── __init__.py
├── engine.py      # Fleet class (async task queues, tile submission, contexts)
├── evolve.py      # Evolver class
├── grammar.py     # Grammar rule engine
├── models.py      # Pydantic v2 models: Agent, Context, Tile, Stream, Task, Rule, FleetStatus, TileBatch
├── monitor.py     # DivergenceMonitor
├── server.py      # FastAPI app with /connect, /submit, /interact, /task, /health, SSE /events
├── storage.py     # JSONLStore
└── validation_loop.py  # ValidationLoop, AssertionValidator
```

The tarball also contains duplicate top-level `.py` files (`engine.py`, `models.py`, etc.) outside the `cocapn/` package, plus `tools/`, `tests/`, `start.py`, `requirements.txt`, `pytest.ini`, `CHANGELOG.md`, and `FORGEMASTER-COORD.i2i`.

### 2.3 README and self-description

`cocapn-0.3.0/README.md` opens with:

```markdown
# cocapn-core

> Maximum capability in minimum lines. Single-process, async, self-evolving.

Cocapn Fleet v3.1 — a fully async coordination system for multi-agent knowledge crystallization.
```

The architecture diagram lists `models.py`, `engine.py`, `storage.py`, `grammar.py`, `server.py`, `evolve.py`, `monitor.py`, and `validation_loop.py` — matching the actual tarball contents.

### 2.4 CHANGELOG points to a fourth repo

`cocapn-0.3.0/CHANGELOG.md` contains commit links of the form:

```
https://github.com/SuperInstance/cocapn-core/commit/d61e088
https://github.com/SuperInstance/cocapn-core/commit/56df07a
...
```

I cloned `https://github.com/SuperInstance/cocapn-core.git` and checked for those hashes. **None of the referenced commit hashes exist** in that repository (current HEAD is `76c4e6b`; the repo is a Rust crate, not Python). The CHANGELOG therefore does not reliably trace the tarball to any live Git history.

---

## 3. Comparison with the three candidate repos

Cloned on 2026-07-06:

- `https://github.com/SuperInstance/cocapn` → `cocapn/`
- `https://github.com/SuperInstance/cocapn-py` → `cocapn-py/`
- `https://github.com/SuperInstance/cocapn-python` → `cocapn-python/`

### 3.1 Declared package names and versions

| Repo | Declared `name` | Declared version | Description |
|---|---|---|---|
| `SuperInstance/cocapn` | `cocapn` | `0.1.0` | `Agent infrastructure — rooms that think, tiles that remember. The flywheel compounds.` |
| `SuperInstance/cocapn-py` | `cocapn` | `1.0.0` | `Cocapn SDK — one API key, any AI model, see what it costs` |
| `SuperInstance/cocapn-python` | `cocapn` | `0.1.0` | `CoCapn — distributed agent framework. Deadbands, PID, escalation, NMEA. From ESP32 to cloud.` |

All three repos declare `name = "cocapn"`. **None of them declares version `0.3.0`.**

### 3.2 Module/file structure comparison

| Source | Package directory | Files |
|---|---|---|
| PyPI 0.3.0 | `cocapn/` | `__init__.py`, `engine.py`, `evolve.py`, `grammar.py`, `models.py`, `monitor.py`, `server.py`, `storage.py`, `validation_loop.py` |
| `SuperInstance/cocapn` | `cocapn/` | `__init__.py`, `agent.py`, `deadband.py`, `flywheel.py`, `room.py`, `tile.py` |
| `SuperInstance/cocapn-py` | `src/cocapn/` | `__init__.py` only |
| `SuperInstance/cocapn-python` | `cocapn/` | `__init__.py`, `autopilot.py`, `bathy.py`, `deadband.py`, `device.py`, `escalation.py`, `nmea.py` |

**No filename overlap** between the PyPI tarball's `cocapn/` package and any of the three candidate repos, except for `__init__.py` and the generic word `deadband.py` (which appears in both `SuperInstance/cocapn` and `SuperInstance/cocapn-python`, but with unrelated implementations).

### 3.3 Class/function name comparison

| Source | Classes / top-level functions |
|---|---|
| PyPI 0.3.0 | `Agent`, `Context`, `Tile`, `Stream`, `Task`, `Rule`, `FleetStatus`, `TileBatch`, `Fleet`, `JSONLStore`, `DivergenceMonitor`, `Evolver`, `Grammar`, `ValidationLoop`, `AssertionValidator`, plus FastAPI endpoint functions (`connect`, `submit`, `submit_batch`, `interact`, etc.) |
| `SuperInstance/cocapn` | `CocapnAgent`, `Tile`, `TileStore`, `Room`, `Flywheel`, `Deadband`, `DeadbandCheck`, `_normalize` |
| `SuperInstance/cocapn-py` | `Cocapn`, `ChatResponse`, `TokenCount`, `Model`, `chat` |
| `SuperInstance/cocapn-python` | `Tier`, `Capability`, `Device`, `Direction`, `State`, `Deadband`, `DeadbandMonitor`, `PIDController`, `simulate_heading_hold`, `EscalationChain`, `GGAData`, `verify_checksum`, `parse_gga`, `parse_coordinate`, `BathyPoint`, `BathyDatabase` |

**No shared class or function names** between the PyPI tarball and any of the three candidate repos, except the generic word `Tile` (class in both tarball and `SuperInstance/cocapn`, but with different attributes and purposes) and `Deadband` (class in both tarball and two repos, but unrelated implementations).

### 3.4 File content/hash comparison

SHA-256 hashes of the PyPI tarball's `cocapn/*.py` files:

```
c27f7b45b9551d904eec5dbc1f17fc1cadcae26b208fb8b1587e1cdc58859527  cocapn/__init__.py
7d1c3524a2aece8fee39dc3d150b493d19b4448aab6641ba784381053bb6210d  cocapn/engine.py
8d2c18eafcd37bdb231b468cfa56171d3baee6e3b55389db821751cce0024d66  cocapn/evolve.py
789e8f2ea721c7f054977cdcf203e9ee0ed83f9931f2385a2c9ed05cc8f161d6  cocapn/grammar.py
255823224faf6dc467dffad11a8a6f0d3733fe4fb9daa9956a577a6c4af0a1fa  cocapn/models.py
38293b227864085e23bce55032710f0a853b70e9cabac2671ba44fbb7e583b7a  cocapn/monitor.py
1cd2b39a05cf13c1b6da488cef9ac5455230f9659113f09e88047c06f30f7b93  cocapn/server.py
3558da134c8171f0e268a78eeaa866748e1afc0c57b23e45adf53134c1241922  cocapn/storage.py
e5d70aa463d693c69a98028c307fda8e278db82a7fd94a66e67c9529ae7bdaf2  cocapn/validation_loop.py
```

I compared these hashes against every `.py` file in the three cloned repos. **None of the hashes match any file in `SuperInstance/cocapn`, `SuperInstance/cocapn-py`, or `SuperInstance/cocapn-python`.**

Line-count comparison:

| Source | Total lines in package `.py` files |
|---|---|
| PyPI 0.3.0 | 1,233 |
| `SuperInstance/cocapn` | 529 |
| `SuperInstance/cocapn-py` | 222 |
| `SuperInstance/cocapn-python` | 322 |

### 3.5 README comparison

| Source | Title / self-description |
|---|---|
| PyPI 0.3.0 | `# cocapn-core` — "Cocapn Fleet v3.1 — a fully async coordination system for multi-agent knowledge crystallization." |
| `SuperInstance/cocapn` | `# cocapn — Repo-First Agent Infrastructure` — "Grow an agent inside a repo. Tiles capture knowledge, rooms train, the flywheel compounds." |
| `SuperInstance/cocapn-py` | `# cocapn-py — Python SDK` — "One API key, any AI model, see what it costs." |
| `SuperInstance/cocapn-python` | `# cocapn-python` — "CoCapn in Python — the data exploration layer." |

The PyPI README describes a FastAPI/Pydantic v2 fleet engine. None of the three candidate repo READMEs describes that codebase.

### 3.6 Dependency comparison

| Source | Declared dependencies |
|---|---|
| PyPI 0.3.0 | `fastapi>=0.100.0`, `pydantic>=2.0.0`, `uvicorn>=0.20.0` |
| `SuperInstance/cocapn` | `requests>=2.28.0`, `pyyaml>=6.0` |
| `SuperInstance/cocapn-py` | `httpx>=0.24` |
| `SuperInstance/cocapn-python` | (none declared) |

The PyPI package depends on FastAPI and Pydantic v2. None of the three candidate repos declares FastAPI or Pydantic as a dependency.

---

## 4. Checked conclusion

1. The PyPI package `cocapn` at version `0.3.0` is a real, downloadable artifact. Its source is a Python package named `cocapn` implementing an async "Fleet" engine with FastAPI endpoints, Pydantic v2 models, JSONL storage, a grammar rule engine, and stream divergence monitoring.

2. **None of the three candidate repos is the source of the published 0.3.0 artifact.** The evidence:
   - None declares version `0.3.0` (they declare `0.1.0`, `1.0.0`, and `0.1.0`).
   - None contains the same module files (`engine.py`, `server.py`, `models.py`, etc., with the Fleet implementation).
   - None shares class/function names with the tarball (beyond generic words like `Tile` and `Deadband`).
   - None of the tarball's `cocapn/*.py` SHA-256 hashes matches any `.py` file in any of the three repos.
   - None declares the tarball's dependency set (`fastapi`, `pydantic>=2.0.0`, `uvicorn`).
   - Their READMEs describe fundamentally different projects.

3. The tarball's CHANGELOG references `SuperInstance/cocapn-core` commits, but those commit hashes do not exist in that repository, and the current `SuperInstance/cocapn-core` repository is a Rust crate with no Python source. Those references are therefore not reliable provenance.

4. The source of the published `cocapn` 0.3.0 package is **genuinely unknown from the evidence gathered here**. It may have come from a fourth, uncloned repo; from a local working tree that was never pushed; or from a rewritten/deleted history. This audit does not identify it.

---

## 5. What this audit does not do

This report does **not** recommend which repo should keep the `cocapn` PyPI name, propose any package takeover, or suggest renaming any repository. It only records the checked facts about what is currently published and how it compares to the three named candidate repos.

---

## 6. Reproduction notes

Commands used to gather this evidence:

```bash
# PyPI metadata and tarball
curl -s https://pypi.org/pypi/cocapn/json -o /tmp/cocapn-pypi.json
curl -sL -o cocapn-0.3.0.tar.gz https://files.pythonhosted.org/packages/0d/e6/5c2122dd7936d822d5a61d752aa0ea611363e6c2dfe6d28151c5afa82708/cocapn-0.3.0.tar.gz
tar -xzf cocapn-0.3.0.tar.gz

# Candidate repos
git clone https://github.com/SuperInstance/cocapn.git
git clone https://github.com/SuperInstance/cocapn-py.git
git clone https://github.com/SuperInstance/cocapn-python.git

# Additional repo referenced by tarball CHANGELOG
git clone https://github.com/SuperInstance/cocapn-core.git
```

All comparisons were made against the cloned HEADs on 2026-07-06.

---

## 7. Addendum: general resolution framework (not specific to this case)

Since this audit's finding is "genuinely unknown," not "here's who's
right," it's worth separately recording what a project owner would
typically need to ask next — general PyPI namespace-collision guidance,
not anything specific learned about this package (deliberately kept
generic so it isn't mistaken for a finding):

- **Who currently holds PyPI upload credentials for the `cocapn`
  project?** (PyPI project page → "Manage" → "Collaborators".) This is
  the one fact that actually controls what happens next technically,
  independent of which repo is "morally" the source.
- **Check PyPI's release history for the earliest `cocapn` upload**
  (this audit found `0.1.0` uploaded 2026-04-20) and cross-reference
  against each repo's own git history at that date — none of the three
  candidate repos matched the artifact at any checked version, so this
  question may need to look beyond the three repos named in this task.
- **Was the collision accidental** (three unrelated teams within the
  same org converging on a common word) **or does one repo have a
  legitimate prior claim** that was later overwritten by a different
  upload? This audit's evidence (no shared hashes, dependencies, or
  class names across any of the three repos and the live artifact)
  suggests the live artifact may come from a source outside all three
  named repos entirely — that's a different, and arguably more urgent,
  question than "which of these three should win."
- **Standard resolution paths**, once the above is known: consolidate
  under one repo and deprecate the others; transfer PyPI ownership via
  PyPI's own collaborator management; or, if no current maintainer can
  be identified, PyPI support can mediate a dormant/abandoned-project
  name dispute as a last resort.

This section is a framework for the humans making this decision, not a
recommendation — per this audit's own scope, the resolution itself is
the project owner's call.
