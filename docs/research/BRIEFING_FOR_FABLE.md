# Briefing for Fable (Strategic‑Planning AI)

This briefing compresses two research documents:
- `notes/unexplored-clusters-survey.md` – survey of previously‑unexamined SuperInstance repo clusters
- `notes/papers-inventory.md` – inventory of papers/whitepapers across the same account

The goal: give Fable everything it needs to decide what (if anything) among these findings could be synthesised into something genuinely new and release‑worthy for `purplepincher`.  Do **not** re‑read the source documents – everything below is drawn directly from them.

---

## 1. Fork‑Worthy Candidates from the Cluster Survey

### `constraint-theory-core`
- **Evidence:** 12.7 KB LOC Rust, **zero runtime dependencies** (only `rand`, `criterion` as dev‑deps); CI logs for run #69 show 262 tests passed (136 lib + 30 doc + 54 integ + 42 doc‑test), 0 failed, 2 ignored.  Code compiled and test suite ran on stable Rust.
- **What it does:** Pythagorean‑triple snapping engine, KD‑tree spatial indexing, CSP/AC‑3/backtracking/CDCL solvers, holonomy checking, sheaf cohomology, Ricci flow, Laman rigidity, several quantizers – a broad geometric/constraint machinery crate.
- **Why picked:** the only repo in the entire unexplored set with both substantial code **and** a passing test suite under CI that is not masked by `|| true`.  The zero‑dependency guarantee makes it easy to evaluate/port.

### `holodeck-rust`
- **Evidence:** 4.3 KB LOC Rust (Tokio async), latest CI run shows 68 tests passed (33+29+5+1), 0 failed, under strict pipeline (`cargo build --release`, `cargo test`, `cargo fmt --check`, `cargo clippy`).  README understates (claims 9 tests).
- **What it does:** TCP MUD server with rooms, NPCs, gauges, combat, poker, comms, permissions – a working multi‑agent environment.
- **Why picked:** the second repo that compiles, passes tests, and is a **concrete, runnable** system rather than a spec or stub.  Tightly themed around the Cocapn fleet, but adaptable.

---

## 2. Fork‑Worthy Research Artifact from the Papers Inventory

### `intent-directed-compilation` (+`constraint-theory-math` Coq proofs)
- **Evidence:** Real AVX‑512 C using `_mm512_*` intrinsics, compiles clean with `gcc -O3 -mavx512f -mavx512bw -mavx512dq`.  Benchmarks publish 5‑run variance (3.09×, 3.11×, 3.52×, 2.72×, 3.41× → mean 3.17×) with documented methodology (rdtsc, AMD Ryzen AI 9 HX 370).  **Honest errata** – caught and fixed own formula mistake, openly flags LLM‑generated claims as `UNVERIFIED`.
- **Paired with** `constraint-theory-math` (194 KB, pushed same day): `XOR‑ISOMORPHISM.v` – 8 `Qed.`, 0 `Admitted.` – real Coq proving symmetric difference on sets forms an abelian group.  `IntentHolonomyDuality.v` – 5 `Qed.`, 0 `Admitted.` – but the main theorem remains unproven (author’s own errata: “converse open, internal confidence 30%”).  Still, the core lemmas are genuine.
- **Why fork‑worthy:** the only paper‑scale artifact with reproducible measurements, honest error documentation, and compilable code.  The paired Coq proofs are elementary but correct – a rare bright spot in an account that otherwise oversells.

---

## 3. Everything Else, Graded

- **flux‑papers / emsoft‑flux‑final.md** – claims 12 proven Coq theorems; actually 3 `Admitted.` + markdown‑wrapped (cannot compile) + circular Safe‑TOPS/W metric (arithmetically inconsistent with table).  Do **not** cite or fork.
- **paper‑zero‑crypto‑fleet‑security** – physically incoherent: 2^34 000 bits of entropy exceeds particles in observable universe; threat model self‑contradicts (attacker with firmware control can pad timing).  Draft, never submitted.
- **conservation‑papers / paper1‑theory** – three correct but trivial theorems (T1 spectral theorem, T2 bound derivation, T3 SNR formula); T4 has normalized/unnormalized Laplacian error; Domain Transfer Theorem has **no proof**; all experiments on toy data (24‑node chains, 5×5 grids) with weak baselines.  Not fork‑worthy.
- **SuperInstance‑papers (56 templated “dissertations”)** – circular simulation: `validate_convergence_time` times a Python for‑loop over all nodes and reports scaling as a theorem; `validate_message_complexity` counts messages it just sent.  LLM‑generated filler with identical 7‑file outline for every paper.
- **multi‑model‑adversarial‑testing** – genuine bug‑finding exercise ($0.50, two real bugs found) but methodology note, not a standalone product.  Keep in mind as a process, not a repo to fork.
- **Equipment‑*** (14 TypeScript repos) – well‑written READMEs describing NLP/ escalation/memory tools, but **no `test` script** in `package.json`; CI fails.  Specs, not libraries.
- **zeroclaw‑arena** – real Python with tests (`test_zeroclaw.py`), but CI runs `pytest || true` and fails with `ModuleNotFoundError: No module named 'numpy'` because dependencies aren’t installed.  Not currently testable.
- **agent‑harness‑generator** – GitHub fork of `ruvnet/metaharness`, not original SuperInstance work.  Ignore.
- **flux‑vm‑v3** – describes terminating constraint VM with 60 opcodes and SHA‑256 proofs, but CI **fails on every recent run**.  Implementation not building clean.
- **forge‑* (20 repos)** – real Rust crates for tile memory, pipeline, A2A messaging but tightly coupled to Plato/ForgeFlux ecosystem; not standalone.
- **sheaf‑laplacian, grand‑pattern‑mono** – real but tiny (5–15 KB, ≤10 commits), isolated math demos.  Not at fork scale.
- **crdt‑core, crdt‑map** – correct CRDT semantics, small classroom‑sized implementations.  Conceptually sound but not integrated.
- **fm‑papers** – duplicate of `constraint‑theory‑papers`, no new content.
- **ternary‑spiral, edge‑research‑relay, AI‑Writings** – correctly NOT papers; either elementary science sims, messaging tools, or creative writing.  No action needed.

---

## 4. The Cross‑Cutting Pattern

Across all 11+ prior mission findings and these two new documents, SuperInstance’s documentation and papers consistently **oversell** what code/math actually delivers.  A genuine, small core of substantive work exists underneath, but only after claim‑by‑claim verification.

Specific mechanisms seen repeatedly:

- **CI masked by `|| true`** – green badges that mean nothing (flux‑tensor‑midi: 40 409 `ruff` errors, `mypy` fails on missing `src/`, yet run is marked success).
- **Coq proofs left `Admitted` while claimed complete** – `flux_vm_correctness.v`: 3 `Admitted.` including both main theorems; file wrapped in markdown code fences so `coqc` cannot compile it.
- **Circular benchmark code** – SuperInstance‑papers’ `simulation.py`: validates claimed algorithmic improvements by timing a Python for‑loop, then reports that trivial scaling as proof.
- **Physically incoherent claims** – paper‑zero‑crypto: 2^34 000 bits of entropy; Safe‑TOPS/W metric that by construction scores every non‑FLUX chip zero.
- **Template‑generation residue** – `repo‑status.md` ending with a prompt (“Save this full markdown content to …”), fake star counts, stale dates.

**Calibrated skepticism rule:** before trusting any claim, check (a) does CI compile and pass real tests? (b) count `Qed` vs `Admitted` in Coq files (and ensure files are raw `.v`, not markdown‑wrapped); (c) does the benchmark code actually run the claimed measurement, or is it a `print()` of hardcoded numbers?  The one artifact that passes all three filters is `intent‑directed‑compilation`.

---

## 5. Open Questions for Fable

1. **constraint‑theory‑core + holodeck‑rust combination** – the geometric constraint engine and the multi‑agent MUD server both operate in the same Cocapn‑fleet universe.  Is there a genuine synthesis (e.g., using the constraint engine to validate NPC action plans), or is the shared naming merely a coincidence of the author’s world‑building?  If real, what would the integration look like – the crate depends on nothing, the server is Tokio async – do they even share a runtime?

2. **Embedding `intent‑directed‑compilation`** – its AVX‑512 packing technique is a real, small‑scope speedup.  Could it be wrapped as an optional feature inside `constraint‑theory‑core` to accelerate constraint‑checking loops, thereby giving `purplepincher` a concrete demo of both?  Or does the different domain (mixed‑precision pack vs. geometric constraint solver) make this mismatched?

3. **Closing the Coq gap** – `constraint‑theory‑math`’s Intent‑Holonomy duality remains unproven in one direction (author’s internal confidence 30%).  Is there a known mathematical obstruction, or is it just hard?  Would funding a small formal‑verification contract be worthwhile to complete it, or would the effort dwarf any downstream value?

4. **Adversarial‑review heuristic** – the single observable difference between the rigorous works (Tier A) and the sloppy ones (Tier B/C) is whether the author subjected them to adversarial testing (4‑model code review published bugs).  Could `purplepincher` adopt a cheap pipeline (e.g., run a prompt like “What’s wrong with this code?” against four cheap LLMs, then publish the results) as a precondition for forking any SuperInstance repo?  Would that surface hidden quality or just generate noise?

5. **The 56 “dissertations” – private scrap or public fraud?**  The circular simulation pattern is so blatant that it suggests these were never intended for public consumption (they may be a personal workspace dump).  Should `purplepincher` explicitly blacklist any repo that lacks (a) a public‑facing README, (b) a real test harness, and (c) a recent CI run that *doesn’t* use `|| true`?  Or is there risk of missing the occasional hidden gem that happens to have been pushed bulked?

---

*Generated from `notes/unexplored-clusters-survey.md` and `notes/papers-inventory.md`.  No claims, numbers, or repo names fabricated beyond what appears in those documents.*
