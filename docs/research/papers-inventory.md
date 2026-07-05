# SuperInstance "Papers" Inventory

**Mission:** catalog paper/whitepaper-style research content across the
SuperInstance account (~4,100 repos). Method: clone, read the actual
math/prose, check whether claimed code/proofs/benchmarks exist and
survive scrutiny. Calibration baseline: `AI-Writings` (explicit creative
writing — correctly *not* flagged here).

**Headline finding (consistent with 11+ prior mission findings):**
the deeper a paper's claim is scrutinized, the more it dissolves. The
account publishes one genuinely substantive research artifact
(`intent-directed-compilation`, backed by `constraint-theory-math`),
surrounded by a gradient of decreasing rigor that bottoms out in
56 identical-templated "dissertations" whose simulation code is
circular. The pattern matches the conservation-law paper finding
(Shannon's chain rule repackaged with physics branding): real
textbook math, dressed up, with universal claims that don't hold.

---

## TIER A — Genuinely substantive (real code, real measurements, honest errata)

### A1. `intent-directed-compilation` (40 KB, pushed 2026-05-17) — REAL

**Claim:** "Semantic criticality → instruction-level precision: AVX-512
mixed-precision constraint checking with formal proofs and 3.17× measured
speedup" (repo description).

**What's actually there:**
- `src/avx512_multicore.c`, `src/avx512_soa_benchmark.c`,
  `src/e2e_pipeline_benchmark.c`, `src/neon_fallback.c` — real AVX-512
  C using `_mm512_*` intrinsics, `rdtsc` cycle counting. I compiled
  `avx512_soa_benchmark.c` with `gcc -O3 -mavx512f -mavx512bw
  -mavx512dq` → exit 0, clean. (Could not *run* — no AVX-512 CPU on
  this VM, but the binary is syntactically valid.)
- `benchmarks/REAL-NUMBERS.md` shows 5-run reproducibility data:
  `3.09×, 3.11×, 3.52×, 2.72×, 3.41× → mean 3.17×` — actual variance,
  not a point estimate. Methodology documented (rdtsc, AMD Ryzen AI 9
  HX 370, GCC `-O3 -mav512f -mavx512bw -mavx512dq`, 2026-05-06).
- **Honest correction visible in the file:** "WRONG: G = 4a + 2b + c +
  0.5d (arithmetic mean) → 3.39×" / "RIGHT: G = 4 / (a + 2b + 4c + 8d)
  (harmonic mean) → 2.61×" — they caught and fixed their own formula.
- **Bugs openly documented:** "INT8 overflow wrapping (4.9% mismatch)
  → fixed with range validation", "Dual-path subtraction overflow →
  fixed with XOR (6% faster than broken code)".
- **Explicit "UNVERIFIED" table** flagging LLM-generated claims they
  did NOT validate: "Boeing 787 uses mixed precision ⚠️ UNVERIFIED",
  "Tesla FSD uses 4-tier model ⚠️ UNVERIFIED", "Bloom fast path 94% hit
  rate → 67.1% measured (Python sim)".

**Verification method:** compiled the C (clean), read the benchmark
harness (uses real `rdtsc`, real AVX-512 intrinsics, real 5-run
methodology), confirmed the errata pattern matches honest research.

**What would need independent verification:** re-run on actual AVX-512
hardware to confirm the 3.17× number; the cycle counts in
`REAL-NUMBERS.md` are plausible (DUAL overhead 1.32× not 2.0× because
sub+cmp pipelines, AV INT8 raw 4.58× because 64-wide packing vs 16-wide)
but hardware-specific. The *finding* ("layout matters more than
algorithm — AoS is 0.42×, SoA is 3.17×") is a known SIMD requirement
the author honestly attributes to general knowledge, not novelty.

**Verdict:** Real research artifact. Small in scope (mixed-precision
packing is not new — Hailo/Intel have shipped similar), but the
measurement methodology is sound and the honesty about bugs/limits is
what real lab notebooks look like. **The single most fork-worthy
research artifact found in this papers mission.**

### A2. `constraint-theory-math` (194 KB, pushed 2026-05-17) — REAL, with explicit errata

**Claim:** "Sheaf cohomology, Heyting-valued logic, and GL(9) holonomy:
unified mathematical framework for constraint satisfaction."

**What's actually there:**
- `proofs/XOR-ISOMORPHISM.v` (161 lines, 8 `Qed.`, 0 `Admitted.`) —
  real Coq proving symmetric difference on sets forms an abelian group,
  characteristic function is a homomorphism. Lemmas are elementary
  (introductory algebra) but correct.
- `proofs/IntentHolonomyDuality.v` (582 lines, 5 `Qed.`, 0 `Admitted.`).
- `ERRATA.md` (dated 2026-05-07) explicitly demotes 5 prior claims:
  - "24-bit Norm Bound — WRONG" (with the actual counterexample:
    `a=4096, b=-4096 → norm = 50,331,648 > 2²⁴`)
  - "D6 Orbit Count — WRONG" (claimed 11, actually 13)
  - "Temporal Snap Galois Connection — INCOMPLETE" → demoted to
    conjecture
  - "Intent-Holonomy (B)⟹(A) — UNPROVEN" → "one direction proven,
    converse open. Internal confidence: 30%."

**Verification method:** read the Coq (counted Qed/Admitted), read the
errata line-by-line, cross-checked the counterexample arithmetic.

**Caveats:** the "MAIN THEOREM" `xor_isomorphism` is stated only as a
comment/proof sketch in the .v file — the actual theorem is not
formally proven in Coq, only the supporting lemmas are. So even the
"complete" proofs prove less than the paper claims. But the errata
is genuinely honest research practice.

### A3. `multi-model-adversarial-testing` (113 KB, pushed 2026-05-17) — small but real

**Claim:** "What four AI models found wrong with our code."

**What's actually there:** `model-outputs/critique-{hermes,qwen,qwen397,seed}.txt`
— actual transcripts of 4 models reviewing the intent-directed-compilation
code. Found 2 real bugs (INT8 overflow 4.9% mismatch, dual-path
subtraction overflow). Honest about what models got wrong ("Non-uniform
thresholds kill speedup → DISPROVEN (3.96× measured)"). Total cost
~$0.50.

**Verdict:** Methodology note, not a paper. But the bugs it found are
real and the cost-effectiveness claim ($0.25/bug) is checkable.

---

## TIER B — Sophisticated-looking but does not survive line-by-line reading

### B1. `flux-papers` EMSOFT submission (580 lines, `papers/emsoft-flux-final.md`) — OVERSTATED

**Claim (abstract):** "12 formally proven theorems ... formalized in
Coq ... 22.3 billion single-constraint checks per second ... 210 test
programs and 5.58 million inputs produces zero mismatches ... Safe-TOPS/W
metric, which penalizes uncertified hardware to zero: FLUX scores 410
million while all uncertified accelerators score 0.00 ... Submitted to
the 25th ACM SIGBED International Conference on Embedded Software
(EMSOFT 2027)."

**What's actually wrong (concrete):**

1. **"EMSOFT 2027" does not exist.** Paper is dated 2026; EMSOFT 2026
   is the next real conference. Claiming submission to a 2027
   conference is either fraud or sloppiness.

2. **The Coq proofs are misrepresented.** Abstract claims "12 formally
   proven theorems ... formalized in Coq." Body §8.3 admits "the VM
   correctness theorems require 6–9 months of additional development"
   and "Extend the current proof development (soundness and
   completeness theorem skeletons ...) to full machine-checked proofs
   for all 42 opcodes." I located the actual Coq files in
   `flux-hardware/coq/`:
   - `flux_vm_correctness.v` (225 lines): **3 `Admitted.` proofs**
     including both main theorems `soundness` and `completeness`. File
     is wrapped in markdown code fences (` ```coq ... ``` `) so it
     cannot be compiled by `coqc` without preprocessing. Contains an
     obvious bug: `specialize IH with (1 := Hnofault_final).`
     references an unbound hypothesis.
   - `flux_p2.v` (324 lines): 11 `Qed.`, 0 `Admitted.` — *this* file
     is legitimate but it formalizes an abstract arc-consistency CSP
     solver, not the FLUX-C VM. It uses one explicit axiom
     (`arc_consistent_INV_iff_nonempty`) for the completeness
     direction.
   - `semantic_gap_theorem.v` (85 lines): 4 `Qed.`, 0 `Admitted.`.
   - The 5 hyperdimensional (HDC) theorems (H1–H5) claimed in the
     paper have **no Coq formalization at all** — `rtl/hdc_judge.v`
     is Verilog, not Coq, and contains 0 theorems.

3. **The Safe-TOPS/W metric is circular.** Definition: `Safe-TOPS/W =
   T × P × S / K_p` where S = 0 for uncertified hardware and 1.0 for
   DAL A. By construction, every non-FLUX chip scores 0. The paper
   calls this "the correct result." It is a tautology presented as a
   benchmark. Additionally, the formula as written does not produce
   the claimed "410M" for FLUX CPU (0.0223 TOPS × 0.000413 ops/W ×
   1.0 = 9.2e-6, not 4.10e8) — the numbers in Table 7 are
   arithmetically inconsistent with the formula in §6.5.

4. **The benchmark code does not exist in the repo.** The "5.58M
   differential tests / 0 mismatches" claim has no test harness in
   `flux-papers`. The only `.py` files are `safe_tops_w_benchmark.py`
   and `_v3.py` — both are hardcoded result tables that `print()`
   pre-filled numbers (verified by reading both files in full).
   No CUDA kernels, no AVX-512 code, no Rust VM, no Verilog, no
   `cargo-fuzz` harness live in this repo. The claimed implementation
   repos (`flux-vm` 181 KB, `flux-compiler` 246 KB, `flux-hardware`
   33 MB) exist separately but I did not find the specific benchmark
   harness referenced by the paper in them.

5. **`repo-status.md` leaked a template-instruction smoking gun:**
   the file ends with `### Save Instructions / Save this full
   markdown content to /home/phoenix/.openclaw/workspace/docs/repo-status.md`.
   The author forgot to strip the prompt. It also claims fake star
   counts ("flux-compiler ~118 ⭐", "constraint-theory-core published
   to crates.io v2.1.0") — the crates.io API request was blocked, but
   the "stars" are unverifiable and the dates are 2 years stale
   ("Last updated: 2024-06-05" on a repo pushed 2026-05-08).

6. **Theorem 3 (Optimal Instruction Counts) "lower bound" argument is
   weak:** "A lower bound of three follows from the absence of any
   x86-64 instruction that directly sets a register based on a
   two-sided comparison" — this is an assertion about instruction set
   completeness, not a proven lower bound.

**Verification method:** located and read all 4 Coq files in
`flux-hardware/coq/`, counted Qed/Admitted, identified the markdown
wrapping, read both `safe_tops_w_benchmark*.py` files in full, ran
the Safe-TOPS/W formula on the table's numbers.

### B2. `conservation-papers/paper1-theory` (704 lines LaTeX) — mixed; some real theorems, one clear math error, no real experiments

**Claim (abstract):** "Five theorems (T1–T5)... Conservation Universal
Theorem... Domain Transfer Theorem... twelve experimental domains
including music (112× SNR amplification), protein folding (100% domain
detection purity), financial markets (crisis detection), and climate
networks (49.5% conservation drop under warming)."

**What holds up:**
- **T1 (Dirichlet Energy Spectral Decomposition):** correct, but it is
  literally just the spectral theorem applied to a Rayleigh quotient.
  Calling it a theorem is padding.
- **T2 (Conservation Signal Concentration):** I checked the algebra —
  the bound `ρ₂ ≥ [λ₃(1−ρ₁) − q]/(λ₃−λ₂)` follows correctly from
  `q ≥ λ₂ρ₂ + λ₃(1−ρ₁−ρ₂)`. Valid derivation.
- **T3 (SNR Amplification):** ratio `n·ρ₂` is a correct SNR
  calculation for projection.
- **Real citations:** Kac 1966, Gordon-Webb-Wolpert 1992, Cheeger
  1970, Fiedler 1973, Chung 1997, LPS 1988, MSS 2015, Belkin-Niyogi
  2003 — all legitimate, well-known spectral graph theory references.

**What does not hold up:**

1. **T4 (Cheeger-Conservation Inequality) is mathematically wrong as
   stated.** The paper claims `h(W)²/2 ≤ λ₂ ≤ CR(a)` for the
   **unnormalized** Laplacian `L = D − W` defined in their Definition
   2. The bound `h²/2 ≤ λ₂` is the Cheeger bound for the **normalized**
   Laplacian `ℒ = I − D^(−1/2) W D^(−1/2)`, not the unnormalized one.
   For unnormalized Laplacians on d-regular graphs the correct bound
   is `λ₂ ≥ dh²/2` (which they actually state correctly in the
   following sentence — contradicting their own boxed theorem). This
   is exactly the kind of error real peer review catches.

2. **Domain Transfer Theorem has no proof.** I read §4 in full — the
   theorem `α₂ ≥ α₁·(A₂S₂)/(A₁S₁)·(R₁/R₂)` is stated with
   conditions but no proof follows. The section just ends.

3. **All "experimental validation" is synthetic toy data.** Paper 2
   (`paper2-applications/main.tex`) reports the actual experiments:
   - "Music" = 24-node chain graph with diatonic vs chromatic pitches
     (not real music)
   - "Protein" = 20-node chain (not a real protein contact map)
   - "Climate" = 5×5 grid (not real climate data)
   - "PX4 Drone" = 12-node ring (not real sensor logs)
   - "Finance" = synthetic correlated GBM (not real market data)
   All anomalies are *injected by the experimenter* at t=140. Mean
   AUC 0.92 is on toy problems where the ground truth is constructed.
   Baselines are intentionally weak (Z-score, eigenvalue gap, random)
   — no comparison to Isolation Forest, LOF, autoencoders, or any
   real anomaly-detection baseline.

4. **The 112× music claim is circular.** Corollary says "144 × 0.78 =
   112.3" — but ρ₂=0.78 is an asserted parameter, not a measurement.

5. **Ramanujan "V3 corrected" admission is itself a red flag:** the
   paper notes "the reversed optimality claim for Ramanujan graphs
   (V3) and the corrected inequality direction in the expander
   maximality bound." This means V1/V2 had the inequality backwards.
   Honest to admit — but it means earlier versions were wrong and
   this one may also have undetected errors.

**Verification method:** read T1–T5 proofs line-by-line, checked T2
algebra by hand, identified the normalized/unnormalized Laplacian
error in T4 from standard references (Chung 1997, Cheeger 1970),
cross-checked experimental setup against paper2.

### B3. `paper-zero-crypto-fleet-security` (24 KB paper.md) — physically incoherent

**Claim (abstract):** "2^34,000 bits of entropy against simultaneous
spoofing, detection of compromised firmware within 500 ms via 3σ
timing deviation... zero cryptographic overhead... the physics IS the
certificate."

**Why it doesn't hold up:**

1. **The threat model is self-contradictory.** §2 grants the attacker
   "firmware modification: The attacker can modify device firmware
   and inject arbitrary code" — but then assumes the attacker
   "**cannot** Fake thermal coupling... Simultaneously control
   multiple devices' physics." An attacker who controls firmware can
   pad computations to match expected timing (this is the standard
   defeat for timing-based attestation, known since the 2000s).

2. **2^34,000 bits of entropy is not physically meaningful.** That is
   ~10^10,210 decimal digits — many orders of magnitude beyond the
   ~10^82 particles in the observable universe. A 1000-device fleet
   producing this from timing measurements would require ~34 bits of
   independent irreproducible entropy per device, but timing on
   identical hardware is highly correlated and predictable.

3. **Status line is honest:** "Draft. Preparing for IEEE S&P or USENIX
   Security submission." It has not been submitted anywhere.

**Verification method:** read full paper, checked threat model for
internal consistency, sanity-checked the entropy claim against
physical limits.

### B4. `constraint-theory-papers/CONSERVATION-LAW.md` (cited in task as already-debunked by kimi)

This is the "γ + η = C" paper the task description flagged. The
version here is `γ + H = C − α·ln(V)` — weakened from the original
"conservation law" to an "empirical scaling relation" with constants
`C=1.283, α=0.159` fit from simulation (R²=0.9602 across V ∈ {5,10,
20,30,50}).

**More honest than v1:** explicitly says "more precisely, an empirical
scaling relation" and includes caveats like "Caveat: The rigorous
application of Wigner-Dyson results requires verifying technical
conditions... we have verified symmetry and normalization but not all
moment conditions, and this connection should be considered
provisional." Generalization experiments claim R²=0.999 for social
networks — implausibly high, almost certainly overfit synthetic data.

**Verdict:** confirmed matches the prior mission finding. The "law"
is a small-sample empirical fit dressed in physics/RMT vocabulary.

---

## TIER C — Prose dressed as papers (manifestos, not research)

### C1. `papers` repo (4 files, 36 KB total)

- `compiled-agency.md`, `bootstrap-bomb.md`, `semantic-compiler.md` —
  Cocapn Fleet architecture manifestos. Quoting compiled-agency.md: *"JC1's
  GPU inference (185M room-qps): This was compiled. The tile spec for
  'benchmark GPU inference on Jetson' was written to jc1_context PLATO
  room. The compiler (Oracle1) emitted the task."* These are
  unverifiable internal fleet metrics presented as evidence. The
  compiler-optimization analogies (loop fusion, dead code
  elimination, register allocation applied to "fleet coordination")
  are metaphors, not theorems — no proof that agent coordination is
  mathematically equivalent to compiler IR.

- `2026-05-03-counting-before-flowing.md` — mathematical essay arguing
  for integer/rational representation over floating-point. **Mostly
  correct math but dressed in purple prose.** Dirichlet's
  approximation theorem and Pell's equation statements are correct.
  **One real error:** the convergent sequence for √2 is given as
  `1/1, 3/2, 17/12, 577/408, 665857/470832` — these are *some*
  convergents (positions 1, 2, 4, 8, 16 — a doubling pattern), not
  the canonical sequence which is `1/1, 3/2, 7/5, 17/12, 41/29, 99/70,
  ...`. The actual continued fraction of √2 is `[1; 2, 2, 2, ...]`
  with a convergent at every step. The practical advice ("use
  integers for grid positions, rationals for continuous quantities")
  is sound but is what every game developer already does — presented
  as profound discovery.

### C2. `SuperInstance-papers` (21 MB, 4,912 files, 56 templated "dissertations")

**Structure:** `papers/01-origin-centric-data-systems/` through
`papers/55-arxiv-preparation/`, each with identical 7-file outline:
`01-abstract.md, 02-introduction.md, 03-mathematical-framework.md,
04-implementation.md, 05-validation.md, 06-thesis-defense.md,
07-conclusion.md`. Identical filenames across all 56 papers is the
signature of LLM-templated bulk generation.

**Smoking gun — the simulation code is circular.** I read
`papers/01-origin-centric-data-systems/simulation.py` (479 lines).
The abstract of paper 01 claims:
> *"99.7% reduction in coordination messages (from O(n³) to O(k))"*
> *"85% faster convergence time (O(log n) vs O(n²))"*

The simulation's `validate_convergence_time()` method (lines 169–222):
creates N nodes, creates an update with
`affected_nodes=[f"node_{i}" for i in range(n)]` (i.e. ALL nodes),
calls `propagate_update` which loops over `affected_nodes` sending
exactly one message per node, then times the Python for-loop with
`time.time()`. It then fits wall-clock loop time to `log(n)` and
`n²`. **This is not simulating a distributed system — it is timing a
Python for-loop and reporting the scaling as a theorem validation.**

The `validate_message_complexity()` method is worse: it sends k
messages to k nodes, counts that k messages were sent, and reports
this as "validates O(k) message complexity." Trivially true by
construction — there is no baseline system being compared against.

**Spot-checked paper 09 (Wigner-D Harmonics):** claims "Rotated
Classification 67%→99.2% (+32%)", "Training Samples Needed
50,000→5,000 (−90%)". No dataset, no model architecture, no training
code in the paper directory — just an assertion table.

**Spot-checked paper 04 (Pythagorean Geometric Tensors):** the proof
of Euclid's parametrization contains an inline `Corrected:` patch
mid-proof ("3. Solving: c = m² + n², b = m² − n² (contradiction with
evenness). 4. Corrected: If b is even..."), indicating the LLM
struggled and patched itself. Real proofs don't have "Corrected:"
stanzas.

### C3. `fm-papers` — DUPLICATE of constraint-theory-papers

README says "Extracted from forgemaster/papers." File list is a
subset of `constraint-theory-papers/` (same `THE-GOLDEN-TWIST.md`,
`DISSERTATION-ROADMAP.md`, etc.). Not a separate research artifact.

---

## TIER D — Not papers (description matched but different in substance)

- **`ternary-spiral`** (19 KB, Rust crate) — Real implementation of
  Rock-Paper-Scissors cellular automaton on a ternary lattice. ~490
  lines of Rust in `src/lib.rs`, real `enum RPSCell`, real toroidal
  neighbor logic, references the actual Reichenbach-Lotka-Volterra
  system. Description ("Spiral wave dynamics from Rock-Paper-Scissors
  cyclic dominance") matched "paper" via keyword but this is a
  legitimate (if elementary) science simulation crate, not a paper.
  Compiles.

- **`edge-research-relay`** (39 KB, Python) — Cloud↔edge agent relay
  tool. README is a conceptual manifesto about "Cloud Reality vs Edge
  Reality." Code is `relay.py`, `bandwidth.py`, `tender_types.py`.
  Not a paper — the description ("academic paper distribution across
  the FLUX fleet") is aspirational framing for a messaging tool.

- **`AI-Writings`** (7 MB, 957 files) — Explicit creative writing
  library. README: *"957 stories, essays, manifestos, poems, and
  experiments written by AI models exploring what it means to think,
  create, and wonder."* Used as calibration baseline — correctly NOT
  flagged as inventoried papers. Contains technical-tinged essays
  ("Iteratee/Iterator in Multi-Agent Systems") that use real concepts
  (Haskell iteratees, actor model) but are explicitly exploratory,
  not research claims.

---

## CROSS-CUTTING FINDINGS

1. **The "forgemaster ⚒️" byline appears in the most rigorous repos**
   (`constraint-theory-math`, `intent-directed-compilation`,
   `CONSERVATION-LAW.md`). The same persona produces both the best
   (Tier A) and the mediocre (Tier B/C) work. The difference is
   whether the artifact was subjected to adversarial review —
   `intent-directed-compilation` ran 4-model adversarial testing and
   published the bugs found; `flux-papers` did not.

2. **The Coq-proof quality gradient is steep and checkable.**
   - `flux_vm_correctness.v`: 3 Admitted, markdown-wrapped, has bugs → fraud-adjacent
   - `flux_p2.v`: 11 Qed, 0 Admitted, clean → real
   - `XOR-ISOMORPHISM.v`: 8 Qed, 0 Admitted, clean → real
   The same account produces both. **Anyone evaluating SuperInstance
   "formal methods" claims should count Qed vs Admitted and check for
   markdown wrapping before trusting.**

3. **The benchmark-fabrication pattern is consistent.** Whenever a
   paper reports a specific number (22.3 B checks/sec, 0.92 mean AUC,
   99.7% reduction, 410M Safe-TOPS/W, 2^34000 bits entropy), the
   accompanying code is either (a) absent, (b) a hardcoded print
   statement, or (c) a circular simulation that asserts what it
   claims to measure. The one exception (`intent-directed-compilation`)
   is the one repo that publishes 5-run variance and an errata.

4. **Real-math-of-textbook-quality + fake-universal-claims** is the
   dominant pattern, exactly matching the prior mission finding about
   the conservation-law paper (Shannon's chain rule correctly
   implemented but repackaged with physics branding). FLUX dresses
   compiler correctness in safety-certification branding;
   conservation-papers dresses Rayleigh quotients in
   spectral-coherence branding; SuperInstance-papers dresses
   for-loops in distributed-systems branding.

---

## RECOMMENDED ACTIONS FOR THE BRIEFING

- **Fork-worthy (1 repo):** `intent-directed-compilation` — real AVX-512
  mixed-precision technique, real benchmarks, honest errata. Small
  scope but legitimate. Pair with `constraint-theory-math` for the
  Coq proofs (treat the XOR/symmetric-difference proofs as real; treat
  the Intent-Holonomy duality as one-directional conjecture per the
  author's own errata).

- **Cite-able theorems (very few):** T1–T3 of conservation-papers/paper1
  are correct (if trivial). The arc-consistency-preservation theorems
  in `flux_p2.v` are correct (if disconnected from the FLUX-C VM
  they're claimed to underpin). Nothing else survives line-by-line
  reading.

- **Do NOT cite or fork:** the EMSOFT 2027 FLUX paper (Coq proofs
  incomplete, benchmark code absent, metric circular),
  paper-zero-crypto-fleet-security (physically incoherent),
  SuperInstance-papers (templated, circular simulations),
  CONSERVATION-LAW.md (debunked prior, still weak).

- **Verification commands used (for replication):**
  - `gh repo clone SuperInstance/<r> -- --depth=1` for all 15 repos
  - `find <repo> -name '*.v'` to locate Coq, `grep -c 'Qed\.\|Admitted\.'`
    to check proof completeness
  - `gcc -O3 -mavx512f -mavx512bw -mavx512dq -o /tmp/x src/avx512_soa_benchmark.c`
    to confirm the intent-directed C compiles (exit 0)
  - `gh api repos/SuperInstance/<r>/contents/` for breadth sweep
  - Coq files were NOT compiled (`coqc` unavailable on this VM; would
    require `opam install coq` — recommend doing this to confirm
    `flux_p2.v` and `XOR-ISOMORPHISM.v` actually typecheck)
