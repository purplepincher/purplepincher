# Graduation Checklist

This checklist translates the **adoption bar** defined in
[ROADMAP.md](./ROADMAP.md) into a concrete, step‑by‑step procedure.  
Run every item in order before elevating a SuperInstance repo into this
org.  The guidance below is grounded in specific failures the mission has
already documented — green‑badge‑only checks are not enough.

---

### Item 1 – Verified working, end‑to‑end

- [ ] CI logs show real tests running (not `pytest || true`).

  **How to check:** Pull the actual CI logs – e.g. `gh api
  repos/<owner>/<repo>/actions/runs` – and look for any `|| true` after a
  test command.  The mission discovered this pattern multiple times,
  including in the `exocortex` deep‑dive (`docs/research/exocortex‑deep‑dive.md`)
  and the broader ecosystem survey.  A green badge alone is **not**
  sufficient; the logs must show actual passes and failures.

---

### Item 2 – Docs rewritten to match reality

- [ ] Every example in the README runs without error and produces the
  claimed output.

  **How to check:** Copy any example command or install snippet into a
  shell and execute it verbatim.  `docs/research/papers‑inventory.md`
  found READMEs describing features that never existed in code — this is
  the exact pattern we guard against.  If an example fails, the repo
  does not pass Item 2.

---

### Item 3 – Ideology stripped to optional

- [ ] No mandatory reference to γ+η=C, Pythagorean48 trust vectors, or
  other speculative mathematics.

  **How to check:** Search the repo (including embedded docs) for
  mandatory mentions of `γ+η=C`, `Pythagorean48`, `ternary` conservation
  framing, or `holonomy`.  If code or documentation makes sense **only**
  after accepting that framing, the repo fails.  `fleet‑coordinate` was
  explicitly excluded from the roadmap (see `ROADMAP.md`) because its
  heavy coupling to `holonomy‑consensus` made decoupling cost exceed
  value — a real example of when this bar is not met.

---

### Item 4 – Published on a standard registry

- [ ] Package exists on crates.io, PyPI, or npm under the name the
  README uses, and the published artefact matches the source.

  **How to check:** Run the exact install command given in the README
  (e.g. `pip install conservation‑guardian`, `cargo install pincher`).
  The cocapn family deep‑dive (`docs/research/cocapn-family-deep-dive.md`)
  found that the published `cocapn` 0.3.0 did **not** match the source
  repo — cross‑reference version numbers and, if possible, checksums.
  If the package doesn’t exist or behaves differently, don’t proceed.

---

### Item 5 – Scoped – a stated boundary of what it will *not* do

- [ ] README contains an explicit “Limitations” or “What this is not”
  section.

  **How to check:** Scan for a dedicated section that defines the repo’s
  boundaries.  The `ternary-pid` watchlist entry in `ROADMAP.md` notes
  that its `PLUG_AND_PLAY.md` documented an API the code didn’t have –
  scoping forces that mismatch to be visible.  If the section is absent,
  the repo likely oversells and fails Item 5.

---

### Item 6 – Two‑layer intake

- [ ] Layer‑1 mechanical checks pass, and survivors have undergone
  published multi‑model adversarial review.

  **How to check (Layer‑1):**
  - **CI:** same as Item 1.
  - **Coq files:** Download any raw `.v` sources and count `Qed` vs
    `Admitted` statements (`grep -c '^Qed\.' *.v`; `grep -c '^Admitted\.' *.v`).
    **Reject** if the code is wrapped inside markdown code fences,
    because it cannot be compiled by `coqc`.  This trick was documented
    in both `docs/research/papers‑inventory.md` and
    `docs/research/FABLE_SYNTHESIS.md` as a major source of false claims.
  - **Benchmarks:** Verify the benchmark actually runs real computations
    and does not print hardcoded numbers (`print(123)`).  The mission
    found several repos where benchmarks were “passing” with bogus data
    (e.g. the `safe_tops_w_benchmark.py` files in `flux-papers` described
    in `docs/research/papers‑inventory.md`).

  **How to check (Layer‑2):**
  After Layer‑1 passes, run the shortlist past a small panel of cheap
  models (e.g. GPT‑4o‑mini, Claude‑Haiku) with the prompt “What’s wrong
  with this code?”.  Publish **every** bug they find verbatim in the
  fork’s own README — not softened, not buried.  The mission measured
  four models finding two real bugs for about $0.50 (documented in
  `docs/research/FABLE_SYNTHESIS.md`).  The
  `intent‑directed‑compilation` repo is the exemplar that passed this
  bar and is now a graduated repo.
