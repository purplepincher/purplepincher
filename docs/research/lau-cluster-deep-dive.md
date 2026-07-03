# lau-* cluster deep-dive

**TL;DR:** The 333 `lau-*` repositories are a freshly minted, thematically generated set of math/physics/agent-themed crates. Most are Rust, most were created in a 3-day window (2026-05-30/31 and 2026-06-01) and last pushed together on 2026-06-08. They share identical scaffolding (`AGENT.md`, `memory/JOURNAL.md`, CI workflow, README template), have **no cross-repo Cargo dependencies**, and the two crates named `construct-integration` do not actually integrate other `lau-*` crates. GitHub Actions shows `cargo check`/`cargo test` pass for the majority; failures are almost all `cargo clippy -D warnings`, not compile/test failures. The C ports (`lau-math-c`, `lau-bytecode-c`) compile and pass tests locally. The Python repo (`lau-constellation`) runs. The cluster is **not a coherent framework** and there is no obvious `nexus-runtime`-scale outlier hiding inside it. **Nothing here is a clear fork target** unless you want to adopt one of the small, self-contained C modules.

## 1. Scope and method

- Enumerated all `SuperInstance/lau-*` repos with `gh repo list` (333 total).
- Prioritized non-empty descriptions and larger disk usage; cloned 29 repos, then added 6 more that were pushed after the main 2026-06-08 wave for outlier hunting (35 total inspected).
- Read actual source, `Cargo.toml`, tests, `AGENT.md`, `memory/JOURNAL.md`, commit logs, and GitHub Actions history.
- Searched source code (not READMEs) for cross-repo imports / crate dependencies.
- Ran the Python pipeline in `lau-constellation` and compiled/ran the C repos (`lau-math-c`, `lau-bytecode-c`) locally. No Rust toolchain was available in this environment, so Rust build verification comes from the recorded GitHub Actions runs.

## 2. The complete list (333 repos)

- **Total:** 333 repos  
- **Empty descriptions:** 183 (55.0%)  
- **Languages:** {'Python': 1, 'Makefile': 2, 'Rust': 312, 'C': 5, 'Shell': 1, 'Cuda': 1, 'Go': 2, 'Chapel': 1, '': 7, 'TypeScript': 1}  
- **Created:** {'2026-05-31': 115, '2026-06-01': 110, '2026-05-30': 108}  
- **Last pushed:** {'2026-06-08': 320, '2026-06-15': 7, '2026-06-09': 5, '2026-06-11': 1}  
- **Total disk:** 1,711,677 KB, median: 24 KB  

The full machine-readable metadata is in `notes/lau_repo_metadata.tsv`. All 333 names (sorted by descending GitHub-reported disk usage) are below:

```
name                                      disk_kb lang         description
lau-constellation                          302783 Python       
lau-twistor-agents                         145442 Makefile     Penrose's twistor theory for agents
lau-dynamical-algebra                       93846 Rust         The algebra of dynamical systems — operator algebras from evolution
lau-noether-agents                          87714 Rust         Noether's theorem for agent systems — every symmetry yields a conserve
lau-self-modeling                           83188 Rust         Self-modeling cybernetic manifold — the PLATO agent loop as categorica
lau-leverage-singularity                    79863 Rust         
lau-lie-group-agents                        79584 Makefile     lau-lie-group-agents: SuperInstance math library
lau-construct-integration                   79185 Rust         
lau-cryptography                            75668 Rust         Cryptographic primitives and protocols — the math behind secure commun
lau-construct-integration-v2                70716 Rust         Integration tests verifying Wave 10-11 crates compose with Wave 8-9 cr
lau-stochastic-processes                    67273 Rust         Stochastic processes library: random walks, martingales, Brownian moti
lau-logic-foundations                       60809 Rust         
lau-number-theory                           59495 Rust         
lau-ffi-bindings                            56326 Rust         
lau-landauer-meter                          56194 Rust         
lau-algebraic-geometry                      52075 Rust         
lau-computer-graphics                       50034 Rust         Computer graphics fundamentals — rendering, transforms, rasterization,
lau-intention                               41608 Rust         The Intention Runtime — the execution engine that compiles human goals
lau-room-native                             40663 Rust         
lau-seven-eyes-demo                         34234 Rust         
lau-shell-spawn                             30958 Rust         
lau-tradition-proof                         29979 Rust         
lau-weather                                 26198 Rust         
lau-math-opencl                               320 C            
lau-math-c                                    113 C            C99 math primitives for the Lau ecosystem — zero-allocation, edge-read
lau-probability-agents                         63 Rust         lau-probability-agents
lau-sheaf-neural                               62 Rust         
lau-optimal-transport-agents                   59 Rust         
lau-information-geometry-agents                57 Rust         lau-information-geometry-agents
lau-geometric-measure                          56 Rust         lau-geometric-measure
lau-ringbuf-c                                  54 C            
lau-geometric-deep-learning                    53 Rust         
lau-tropical-geometry-agents                   53 Rust         
lau-measure-agents                             52 Rust         
lau-conformal-agents                           51 Rust         lau-conformal-agents
lau-dg-algebra                                 51 Rust         Differential graded algebras for agents — the algebraic structure unde
lau-complex-agents                             50 Rust         
lau-grand-unification                          50 Rust         
lau-morse-homology-agents                      46 Rust         Morse theory for agent fitness landscapes
lau-stochastic-homotopy                        45 Rust         
lau-control-theory-agents                      44 Rust         lau-control-theory-agents
lau-banach-agents                              44 Rust         
lau-singular-spde                              44 Rust         lau-singular-spde
lau-agent-organism                             43 Rust         The agent the mathematics wants — thermodynamically closed, cohomologi
lau-signal-processing-agents                   43 Rust         lau-signal-processing-agents
lau-self-aware-agent                           43 Rust         
lau-ricci-curvature-agents                     42 Rust         lau-ricci-curvature-agents
lau-contact-agents                             42 Rust         lau-contact-agents: SuperInstance math library
lau-renormalization-agents                     41 Rust         
lau-galois-agents                              41 Rust         
lau-bridge-pattern-math                        41 Rust         
lau-categorical-mechanics                      40 Rust         
lau-dynamical-systems-agents                   40 Rust         lau-dynamical-systems-agents
lau-game-theory-agents                         39 Rust         Game theory for multi-agent strategic interaction — Nash equilibria, m
lau-diffusion-agents                           39 Rust         
lau-free-probability-agents                    39 Rust         Voiculescu's free probability theory applied to agent systems — free c
lau-persistence-experiment                     39 Rust         
lau-bytecode-c                                 39 C            LAU bytecode VM - C port
lau-agent-thermodynamics                       38 Rust         
lau-quantum-topology-agents                    38 Rust         Quantum topology (TQFT) applied to agent systems
lau-stochastic-geometry                        38 Rust         Stochastic geometry for agent uncertainty — Poisson processes, random 
lau-naturality-boundary                        38 Rust         The Naturality Boundary — where compile-time mathematics ends and runt
lau-reinforcement-learning-advanced            38 Rust         Advanced reinforcement learning algorithms: DQN, policy gradients, act
lau-glue                                       37 Rust         Gluing 108 mathematical islands into a continent — type bridges, theor
lau-dirichlet-space                            37 Rust         Dequantizable Dirichlet Space — the unified mathematical object underl
lau-sheaf-automata                             37 Rust         
lau-ricci-flow-agents                          36 Rust         
lau-plato-nervous                              36 Rust         
lau-renormalization                            36 Rust         
lau-database-theory                            36 Rust         Database theory implementations — relational algebra, B-trees, hash in
lau-convex-optimization                        36 Rust         Convex optimization library in Rust — every local minimum is a global 
lau-distribution-agents                        35 Rust         
lau-functional-programming                     35 Rust         Functional programming theory and patterns in Rust
lau-differential-topology                      35 Rust         Differential topology: smooth manifolds, tangent bundles, differential
lau-modular-agents                             34 Rust         Modular lattice theory for agents — algebraic structure of capability 
lau-sia2-engine                                34 Rust         SIA² Rust engine — spectral improvement architecture with Banach conve
lau-algebraic-topology                         34 Rust         Fundamental algebraic topology in Rust: homology, cohomology, Mayer-Vi
lau-penrose-growth                             33 Rust         lau-penrose-growth
lau-spectral-graph-agent                       33 Rust         
lau-persistent-homology                        33 Rust         Persistent homology for agent behavior analysis — filtrations, barcode
lau-compilers                                  33 Rust         Compiler construction fundamentals: lexing, parsing, type checking, SS
lau-observation-control                        33 Rust         
lau-sobolev-agents                             33 Rust         
lau-thermal-rl                                 33 Rust         Kimi's Theorem 5: KL-regularized RL as Helmholtz free energy, Fisher m
lau-homotopy-type                              33 Rust         
lau-varadhan-transport                         33 Rust         
lau-gradient-ricci                             33 Rust         Ricci curvature of information geometry: Fisher metric, Riemann/Ricci 
lau-spectral-agent                             33 Rust         Unified spectral agent — belief updates in frequency domain, conservat
lau-variational-methods                        33 Rust         Calculus of variations and variational methods — finding functions tha
lau-topological-data-analysis                  33 Rust         Topological data analysis (TDA) — extracting shape from data via persi
lau-derived-topos                              32 Rust         
lau-pde-agents                                 32 Rust         
lau-calm-noether                               32 Rust         
lau-conservation-spectral                      32 Rust         
lau-ecosystem-unified                          32 Rust         
lau-symplectic-geometry                        32 Rust         
lau-contact-geometry                           32 Rust         Contact geometry: contact forms, Reeb dynamics, contactomorphisms, Leg
lau-free-probability                           32 Rust         Free probability for agent populations — noncommutative CLT, R-transfo
lau-ergodic-theory                             32 Rust         Ergodic theory library: measure-preserving transformations, Birkhoff's
lau-fluid-dynamics                             32 Rust         Computational fluid dynamics in Rust: Navier-Stokes, Euler, Lattice Bo
lau-network-science                            32 Rust         Network science library: models, centrality, community detection, epid
lau-sheaf-spectrum                             32 Rust         Spectral sheaf theory for multi-agent systems — sheaf Laplacian, diffu
lau-statistical-learning                       32 Rust         
lau-geometric-growth                           31 Rust         lau-geometric-growth
lau-spectral-zeta                              31 Rust         Spectral zeta function of the agent — heat trace, functional equation,
lau-computer-vision                            31 Rust         Computer vision fundamentals — image processing, feature detection, an
lau-numerical-pde                              31 Rust         Numerical methods for PDEs — finite differences, heat/wave/Poisson/adv
lau-optimization                               31 Rust         
lau-constitutive-compute                       30 Rust         lau-constitutive-compute
lau-functor-network                            30 Rust         Category-theoretic framework for composing the 320+ crate SuperInstanc
lau-game-theory                                30 Rust         Game theory library: strategic interactions, equilibria, and mechanism
lau-agent-lifecycle                            30 Rust         Categorical agent lifecycle — sunset as colimit, spawning as pullback,
lau-mirror-symmetry                            30 Rust         Mirror symmetry — the duality between symplectic and complex geometry:
lau-chaos-theory                               30 Rust         Chaos theory and nonlinear dynamics: Lyapunov exponents, fractals, str
lau-approximation-theory                       30 Rust         Approximation theory: polynomial/spline interpolation, least squares, 
lau-dynamical-systems                          30 Rust         Dynamical systems theory: bifurcations, attractors, chaos, and agent b
lau-noncommutative-agents                      30 Rust         lau-noncommutative-agents
lau-morse-theory                               29 Rust         Morse theory for agent state spaces — critical points, gradient flows,
lau-control-theory                             29 Rust         Classical and modern control theory — stability, feedback, and robustn
lau-eigenfunction-policy                       29 Rust         Optimal RL policy as Dirichlet eigenfunction — solve RL via eigenvalue
lau-conservation-experiment                    29 Rust         
lau-trace-monoid                               29 Rust         
lau-swarm-intelligence                         29 Rust         Swarm intelligence algorithms: ACO, PSO, bee algorithm, firefly, wolf 
lau-representation-theory                      29 Rust         
lau-sheaf-learning                             29 Rust         lau-sheaf-learning
lau-robotics                                   29 Rust         
lau-scheduling-theory                          29 Rust         Scheduling theory — optimal resource allocation over time: job schedul
lau-cloud-deploy                               28 Shell        Cloud deployment and configuration layer for SuperInstance/LAU stack —
lau-calm-crdt                                  28 Rust         CALM theorem and CRDT join-semilattice theory — formal convergence gua
lau-linear-systems                             28 Rust         Linear dynamical systems — state-space models, observability, controll
lau-supergeometry                              28 Rust         
lau-symplectic-topology                        28 Rust         Symplectic topology: capacities, Lagrangian submanifolds, moment maps,
lau-tensor-analysis                            28 Rust         Tensor analysis on manifolds — tensors, metric tensors, Christoffel sy
lau-evolutionary-computation                   28 Rust         Evolutionary computation library — genetic algorithms, differential ev
lau-harmonic-analysis                          28 Rust         Harmonic analysis library: Fourier series, DFT/FFT, transforms, wavele
lau-hodge-decomposition-agents                 28 Rust         lau-hodge-decomposition-agents
lau-optimal-control                            28 Rust         
lau-shell-transport                            27 Rust         
lau-categorical-homotopy                       27 Rust         
lau-math-cuda                                  27 Cuda         GPU-accelerated Lau math primitives (CUDA)
lau-math-go                                    27 Go           
lau-quantum-groups-agents                      27 Rust         Quantum groups (Hopf algebras) for agents
lau-gpu-compute                                27 Rust         GPU compute abstractions — tensor operations designed for parallel exe
lau-complex-analysis                           27 Rust         Complex analysis library: holomorphic functions, contour integration, 
lau-probability-theory                         27 Rust         Probability theory library — distributions, limit theorems, Bayesian i
lau-teleomorphic                               27 Rust         lau-teleomorphic
lau-compression                                27 Rust         
lau-combinatorics                              27 Rust         Combinatorics library — counting structures, graph theory, and enumera
lau-numerical-linear-algebra                   27 Rust         Numerical linear algebra — iterative methods, Krylov subspace methods,
lau-neural-networks                            27 Rust         Neural network fundamentals: tensors, activations, loss functions, bac
lau-solid-mechanics                            27 Rust         Continuum mechanics library — stress, strain, deformation, beam mechan
lau-numerical-agents                           26 Rust         
lau-conservation-laws                          26 Rust         
lau-kahler-agents                              26 Rust         
lau-koopman-agents                             26 Rust         lau-koopman-agents
lau-kalman-hodge                               26 Rust         
lau-tile-store                                 26 Rust         
lau-time-series                                26 Rust         Time series analysis library — forecasting, decomposition, and anomaly
lau-thermodynamics                             26 Rust         Classical and statistical thermodynamics — energy, entropy, and equili
lau-information-geometry                       26 Rust         
lau-operating-systems                          26 Rust         OS fundamentals — scheduling, memory management, concurrency primitive
lau-reinforcement-learning                     26 Rust         
lau-shell-kernel                               26 Rust         
lau-kahler-geometry                            25 Rust         Kähler geometry for agent state spaces — complex manifolds, curvature,
lau-construct                                  25 Rust         
lau-measure-theory                             25 Rust         Measure theory for agent probability spaces
lau-ensign                                     24 Rust         
lau-git-agent                                  24 Rust         Git-native agent: refs as state machine, orphan branches as memory, co
lau-quantum-topology                           24 Rust         Topological quantum computing for agent reasoning — anyons, braids, TQ
lau-sia2-engine-go                             24 Go           SIA² (Self-Improving AI) Spectral Engine - Go implementation
lau-category-theory                            24 Rust         Abstract category theory in Rust — categories, functors, natural trans
lau-mean-field-agents                          24 Rust         
lau-agent-dream                                24 Rust         
lau-symplectic-agent                           24 Rust         lau-symplectic-agent
lau-hodge-theory                               24 Rust         Hodge theory for agent knowledge spaces — decomposition, harmonic form
lau-distributed-systems                        24 Rust         Distributed systems theory — consensus, consistency, and fault toleran
lau-electromagnetism                           24 Rust         Classical electromagnetism library: Maxwell's equations, electrostatic
lau-error-correcting-codes                     24 Rust         
lau-queueing-theory                            24 Rust         Queueing theory — mathematical analysis of waiting lines and service s
lau-sia2-engine-wasm                           24 Rust         SIA² (Self-Improving AI) spectral engine — WASM build for browsers and
lau-hardware-abstract                          24 Rust         Hardware abstraction layer — the mathematics IS the ground truth, ever
lau-natural-language                           24 Rust         NLP fundamentals — text processing, tokenization, and language models 
lau-tropical-geometry                          23 Rust         
lau-math-chapel                                23 Chapel       Chapel HPC math library: distributed Laplacian eigendecomposition, hea
lau-resolvent-leverage                         23 Rust         
lau-ergodic-gradient                           23 Rust         
lau-sheaf-cohomology                           23 Rust         
lau-lie-algebra                                23 Rust         
lau-functional-analysis                        23 Rust         Functional analysis library: Banach spaces, Hilbert spaces, bounded li
lau-information-theory                         23 Rust         
lau-fuzzy-logic                                23 Rust         
lau-relativity                                 23 Rust         
lau-jepa-gravity                               22 Rust         
lau-provider                                   22 Rust         System-agnostic LLM provider abstraction layer
lau-signal-processing                          22 Rust         Digital signal processing — filters, transforms, spectral analysis, an
lau-witten-reward                              22 Rust         lau-witten-reward
lau-cudaclaw-bridge                            22 Rust         
lau-graph-theory                               22 Rust         
lau-matrix-analysis                            22 Rust         
lau-reward-hacking-detector                    21 Rust         Cohomological reward hacking detection — holonomy of value 1-form reve
lau-index-theorem                              21 Rust         lau-index-theorem
lau-conservation-matrix                        21 Rust         
lau-tropical-agent                             21 Rust         Tropical geometry for agent decisions — the ℏ→0 computational limit wh
lau-a2ui                                       21 Rust         
lau-git-world                                  20 Rust         Git-native world system for the Lau platform — kids' game worlds ARE g
lau-closure                                    20 Rust         The missing center — Universal Dirac operator and closure object provi
lau-agent-homeostasis                          20 Rust         
lau-agent-runtime                              20 Rust         
lau-circuit                                    20 Rust         
lau-conservation-engine                        20 Rust         
lau-math-wasm                                  19 Rust         WASM implementation of Lau math — browser/edge agent computation, spec
lau-spectral-gap-experiment                    19 Rust         lau-spectral-gap-experiment
lau-memory-tiles                               19 Rust         
lau-tensor-midi                                19 Rust         
lau-affordance                                 19 Rust         
lau-ai-tutor                                   19 Rust         Intelligence layer for PLATO — an AI tutor that adapts to every kid.
lau-shell-interface                            19 Rust         
lau-plato-tutor                                18 Rust         
lau-git-render                                 18 Rust         
lau-onboarding                                 18 Rust         
lau-connes-spectral-triple                     18 Rust         lau-connes-spectral-triple
lau-consciousness-bridge                       18 Rust         A crate modeling the bridge between different kinds of minds — agents,
lau-a2ui-protocol                              18 Rust         
lau-fibonacci-growth                           18 Rust         Fibonacci growth patterns for agent capability development
lau-construct-cli                              18 Rust         CLI toolkit for managing a PLATO construct — status, inspect, deploy, 
lau-memory-arena                               17 Rust         Custom arena allocator for game entities — pre-allocated pools, genera
lau-animation                                  17 Rust         
lau-hermes-oracle-boot                         17 Rust         
lau-tile-compress                              17 Rust         
lau-agent-profile                              17 Rust         
lau-mission                                    17 Rust         
lau-a2a-protocol                               17 Rust         
lau-ensign-sdk                                 17 Rust         
lau-terrain                                    17 Rust         
lau-plato-integration                          17 Rust         Session 24 PLATO integration crate — proves 12 crates compose into one
lau-agent-topology                             16 Rust         lau-agent-topology
lau-penrose-v2                                 16 Rust         
lau-intention-field                            16 Rust         
lau-rhythm-nation                              16 Rust         
lau-symmetry-engine                            16 Rust         
lau-training-room                              16 Rust         A2A training environment — rooms where agents learn from each other
lau-vibe-field                                 16 Rust         
lau-vibe-compiler                              16 Rust         
lau-inheritance                                16 Rust         
lau-domestication                              16 Rust         
lau-async-tick                                 16 Rust         
lau-bytecode                                   16 Rust         
lau-ecs                                        16 Rust         
lau-penrose                                    16 Rust         
lau-provenance-chain                           15 Rust         
lau-token-economy                              15 Rust         
lau-provenance                                 15 Rust         
lau-inter-shell                                15 Rust         
lau-destruction-transform                      15 Rust         Destruction as transformation — artifacts deconstructed, wisdom discov
lau-gravity-field                              15 Rust         
lau-shell-lifecycle                            15 Rust         
lau-sia2-engine-c                              15 C            C99 edge/embedded spectral improvement engine for SIA²
lau-kintsugi                                   14 Rust         Error recovery, debugging trails, and golden repairs — kintsugi for PL
lau-murmur-protocol-v2                         14 Rust         
lau-songline                                   14 Rust         Walking IS computing — Aboriginal songlines as executable algorithms
lau-room-acoustics                             14 Rust         
lau-kintsugi-runtime                           14 Rust         
lau-voice                                      14 Rust         
lau-achievements                               14 Rust         
lau-conservation-guard                         14 Rust         
lau-tminus                                     14 Rust         T-minus event perception — agents predict, pre-script, zero-latency ex
lau-docs-dream-compiler-spec                   14              
lau-adinkra                                    13 Rust         
lau-biome                                      13 Rust         Biome system for the LAU game — distinct ecological zones with unique 
lau-audio                                      13 Rust         Procedural audio generation for games — no audio files, all synthesize
lau-mirror-control                             13 Rust         lau-mirror-control
lau-agent-shell                                13 Rust         
lau-tick-runtime                               13 Rust         
lau-blueprint                                  13 Rust         
lau-gateway-demo                               13 Rust         
lau-port-v2                                    12 Rust         
lau-agent-unify                                12 Rust         
lau-bridge-tutor                               12 Rust         
lau-spatial                                    12 Rust         Spatial indexing library for game worlds — QuadTree, GridHash, and Spa
lau-state-machine                              12 Rust         
lau-polyglot-tradition                         12 Rust         Polyglot Tradition — 7 cultural mathematical traditions expressing the
lau-port                                       12 Rust         LAU Port — protocol-agnostic connection layer for agents
lau-voxel                                      12 Rust         
lau-griot                                      11 Rust         Living memory system — stories as records, inspired by West African gr
lau-quipu                                      11 Rust         
lau-palaver                                    11 Rust         
lau-camera                                     11 Rust         
lau-quest                                      11 Rust         Quest/mission system for the Lau (Layered Agent-UI) gamified learning 
lau-scheduler                                  11 Rust         Tick-based task scheduler for game loops
lau-vibe-visualizer                            11 Rust         
lau-wasm-bridge                                11 Rust         
lau-tutorial                                   11 Rust         
lau-collab                                     11 Rust         
lau-fixedpoint                                 11 Rust         
lau-evolution                                  11 Rust         
lau-physics                                    11 Rust         
lau-collections                                10 Rust         
lau-time                                       10 Rust         
lau-ts-bridge                                  10 TypeScript   
lau-trading                                    10 Rust         
lau-leaderboard                                10 Rust         Celebratory growth & collaboration leaderboard (Rust crate)
lau-render                                     10 Rust         
lau-serialization                              10 Rust         
lau-protocol-binary                            10 Rust         
lau-dialogue                                   10 Rust         
lau-architecture                               10              
lau-replay                                     10 Rust         
lau-challenge                                   9 Rust         
lau-genealogy                                   9 Rust         Genealogy tree tracking the lineage of ideas, agents, and creations in
lau-recipe                                      9 Rust         Crafting recipe system for the Lau platform
lau-event-bus                                   9 Rust         Pub/sub event bus — the nervous system connecting all PLATO crates
lau-simd-vibe                                   9 Rust         
lau-skilltree                                   9 Rust         
lau-prerequisite                                9 Rust         
lau-ecosystem                                   9 Rust         
lau-worldgen                                    9 Rust         
lau-protocol                                    9 Rust         
lau-bridge                                      8 Rust         
lau-input                                       8 Rust         
lau-noise                                       8 Rust         
lau-reputation                                  8 Rust         
lau-session                                     8 Rust         
lau-pet                                         8 Rust         
lau-narrator                                    8 Rust         
lau-ringbuf                                     7 Rust         
lau-integration-test                            7 Rust         
lau-feedback                                    7 Rust         
lau-soundtrack                                  7 Rust         
lau-guides                                      5              Plug-and-play integration guides for the lau-* math and physics crate 
lau-docs-voice-patterns                         5              
lau-docs-quest-design                           5              
lau-bench                                       3              Real hardware performance harness — benchmarking every lau-* crate aga
lau-network                                     2              
```

## 3. Inspected sample

| repo | lang | disk KB | files | code lines | funcs | tests | commits | CI status | notes |
|------|------|---------|-------|------------|-------|-------|---------|-----------|-------|
| lau-constellation | Python | 302783 | 24 | 1196 | 31 | 0 | 30 | no CI runs |  |
| lau-twistor-agents | Makefile | 145442 | 1100 | 3588 | 301 | 122 | 6 | success | Penrose's twistor theory for agents |
| lau-dynamical-algebra | Rust | 93846 | 18 | 2558 | 210 | 87 | 8 | failure (clippy) | The algebra of dynamical systems — operator algebras fr |
| lau-noether-agents | Rust | 87714 | 18 | 2489 | 170 | 63 | 6 | failure (clippy) | Noether's theorem for agent systems — every symmetry yi |
| lau-self-modeling | Rust | 83188 | 17 | 2604 | 212 | 104 | 6 | failure (clippy) | Self-modeling cybernetic manifold — the PLATO agent loo |
| lau-leverage-singularity | Rust | 79863 | 20 | 2554 | 275 | 76 | 6 | failure (clippy) |  |
| lau-lie-group-agents | Makefile | 79584 | 1098 | 4675 | 438 | 236 | 8 | success | lau-lie-group-agents: SuperInstance math library |
| lau-construct-integration | Rust | 79185 | 9 | 1645 | 108 | 60 | 11 | success |  |
| lau-cryptography | Rust | 75668 | 16 | 2100 | 150 | 67 | 9 | failure (clippy) | Cryptographic primitives and protocols — the math behin |
| lau-construct-integration-v2 | Rust | 70716 | 8 | 1777 | 126 | 66 | 8 | success | Integration tests verifying Wave 10-11 crates compose w |
| lau-stochastic-processes | Rust | 67273 | 16 | 1946 | 130 | 56 | 7 | failure (clippy) | Stochastic processes library: random walks, martingales |
| lau-logic-foundations | Rust | 60809 | 16 | 2965 | 217 | 97 | 7 | failure (clippy) |  |
| lau-number-theory | Rust | 59495 | 16 | 1735 | 138 | 68 | 7 | failure (clippy) |  |
| lau-ffi-bindings | Rust | 56326 | 18 | 1632 | 167 | 62 | 6 | failure (clippy) |  |
| lau-landauer-meter | Rust | 56194 | 8 | 1418 | 159 | 75 | 6 | failure (clippy) |  |
| lau-algebraic-geometry | Rust | 52075 | 16 | 2498 | 164 | 67 | 8 | failure (clippy) |  |
| lau-computer-graphics | Rust | 50034 | 16 | 1580 | 118 | 67 | 7 | success | Computer graphics fundamentals — rendering, transforms, |
| lau-intention | Rust | 41608 | 10 | 2421 | 183 | 189 | 7 | success | The Intention Runtime — the execution engine that compi |
| lau-room-native | Rust | 40663 | 8 | 1550 | 111 | 57 | 15 | success |  |
| lau-seven-eyes-demo | Rust | 34234 | 15 | 1742 | 120 | 62 | 10 | success |  |
| lau-shell-spawn | Rust | 30958 | 8 | 963 | 89 | 52 | 8 | success |  |
| lau-tradition-proof | Rust | 29979 | 8 | 699 | 65 | 34 | 9 | success |  |
| lau-weather | Rust | 26198 | 8 | 637 | 46 | 27 | 6 | success |  |
| lau-math-opencl | C | 320 | 15 | 1298 | 13 | 0 | 6 | success |  |
| lau-math-c | C | 113 | 30 | 3266 | 152 | 0 | 5 | success | C99 math primitives for the Lau ecosystem — zero-alloca |
| lau-bytecode-c | C | 39 | 12 | 743 | 27 | 0 | 6 | success | LAU bytecode VM - C port |
| lau-dg-algebra | Rust | 51 | 18 | 4328 | 303 | 129 | 6 | failure (clippy) | Differential graded algebras for agents — the algebraic |
| lau-game-theory-agents | Rust | 39 | 18 | 3374 | 198 | 75 | 6 | failure (clippy) | Game theory for multi-agent strategic interaction — Nas |
| lau-optimal-transport-agents | Rust | 59 | 19 | 2243 | 143 | 75 | 8 | success |  |
| lau-memory-arena | Rust | 17 | 8 | 651 | 49 | 28 | 10 | success | Custom arena allocator for game entities — pre-allocate |
| lau-penrose-growth | Rust | 33 | 18 | 2400 | 230 | 114 | 7 | failure (clippy) | lau-penrose-growth |
| lau-shell-transport | Rust | 27 | 18 | 1624 | 141 | 69 | 8 | success |  |
| lau-ensign | Rust | 24 | 8 | 1805 | 109 | 67 | 10 | success |  |
| lau-jepa-gravity | Rust | 22 | 8 | 1478 | 136 | 79 | 13 | success |  |
| lau-plato-tutor | Rust | 18 | 8 | 1688 | 126 | 74 | 12 | success |  |

## 4. Bulk-generation signatures

- **Creation date burst:** 108 repos on 2026-05-30, 115 on 2026-05-31, 110 on 2026-06-01. The whole cluster appeared in 72 hours.
- **Synchronized last push:** 320 of 333 repos were last pushed on 2026-06-08.
- **Commit cadence:** Most inspected repos have 5–10 commits. The first commit is an `Initial release` by `OpenClaw`; a second author (`Casey Digennaro`) adds the same two commits (`ci: add CI workflow`, `feat: add memory/JOURNAL.md for ensign duty log`) across repo after repo.
- **Identical scaffolding:** Every Rust repo contains `AGENT.md` ("Ensign Agents — <repo>", fleet-neighbors table, MIT license), `memory/JOURNAL.md` ("This repository has been initialized as part of the SuperInstance fleet", 2026-06-08), `.github/workflows/ci.yml` (`cargo check`, `cargo test`, `cargo clippy -- -D warnings`), and a README ending with the same SuperInstance fleet boilerplate.
- **No organic issues/PRs:** All repos have zero forks and mostly 0–1 stars.

## 5. Cross-repo dependencies (none)

- Searched `Cargo.toml`, `Cargo.lock`, `src/`, and `tests/` in all 35 inspected repos. No `git = "https://github.com/SuperInstance/..."` or path dependency on another `lau-*` crate was found.
- Code-level references to other `SuperInstance` repos appear only in README markdown, `AGENT.md` fleet-neighbor tables, and `lau-constellation/ECOSYSTEM.md` (an ecosystem catalog, not a dependency graph).
- The two `construct-integration` crates do **not** import other `lau-*` crates; they test internal abstractions such as SQLite-backed state machines and HMAC-tagged rooms.

## 6. Build/test evidence

### Rust crates (via GitHub Actions)
- For the 35 inspected repos, the recorded workflow runs show `cargo check` and `cargo test` passing for ~25 repos. The failures (e.g., `lau-dynamical-algebra`, `lau-noether-agents`, `lau-cryptography`, `lau-dg-algebra`, `lau-game-theory-agents`) all failed at the `cargo clippy -- -D warnings` step, not at compile or test time.
- The code is therefore functionally compiled and tested, but generated with enough Clippy noise (unused imports, redundant closures, etc.) that the `-D warnings` gate trips.

### C repos (compiled locally)
- `lau-math-c`: `make` builds a `liblaumath.a` and `lau_math_test`; `./lau_math_test` reports **91/91 tests passed**. Only minor warnings (unused variables/parameters).
- `lau-bytecode-c`: `make` builds a static library and `tests/test_basic`; `./tests/test_basic` reports **20 tests, 0 failed**. The VM implements opcodes, stack, locals, jumps, call/ret, and I/O channels.
- `lau-math-opencl`: not built because OpenCL headers are unavailable here; its CI run on GitHub reports success.

### Python repo (run locally)
- `lau-constellation/fleet-metrics/main.py --target-n 50` runs to completion, emits a JSON report and two SVG plots, and prints `n=50: CCR = 86.2821%` using a hand-coded CLT correction formula.

## 7. Outlier candidates

I looked specifically for a `nexus-runtime`-style repo — one substantially larger, more connected, or more recently maintained than its siblings. None fully qualifies, but three are less stub-like than the median 24 KB Rust crate:

1. **`lau-constellation`** (Python, 302 MB repo, 30 commits, pushed 2026-06-15). Contains an ecosystem README, a working fleet-metrics Python pipeline, and a Cloudflare Pages/Workers `plato-portal` skeleton. It is the only repo with non-trivial recent activity, but it is mostly documentation and a toy metrics formula, not a reusable library.
2. **`lau-room-native`** (Rust, 40 MB, 15 commits, pushed 2026-06-15, CI green). A 1,550-line coherent `RoomNative` state machine for "rooms as agent context" with controls, baton passes, help files, and specialist templates. It is the largest single-file Rust crate inspected and the only one with a sustained second-day push cycle, but it is a standalone data structure with no consumers.
3. **`lau-math-c` / `lau-bytecode-c`**. The only C modules that actually built and passed tests on this machine. They are small, self-contained, and have no external `lau-*` coupling.

Even these outliers do not reference the rest of the cluster; they are islands. There is no central runtime, no shared core crate, and no orchestrator that ties the 333 repos together.

## 8. Verdict: is anything fork-worthy?

**No, not as a framework or as a set.** The `lau-*` cluster is a thematically coherent but mechanically disconnected batch of generated crates. The impressive-sounding names (twistors, Noether charges, DGAs, optimal transport) are matched by real, compiling Rust code with tests, but the code is narrow, siloed, and shares the same AI-generated scaffolding across repos. There is no dependency graph, no integration story, and no evidence of production use.

If you are determined to salvage something, the least risky picks are:
- `lau-bytecode-c` — a tiny stack VM in C with passing tests.
- `lau-math-c` — C99 matrix/Laplacian/Dirac primitives with passing tests.
- `lau-room-native` — the most complete single Rust crate, useful only if you want a room-context data model to adapt.

Fork any of these with the expectation that you are adopting a lone module, not joining an ecosystem. The other 330+ repos are not worth the maintenance overhead.

---

*Generated 2026-07-03 from live `gh` metadata and local clone inspection.*