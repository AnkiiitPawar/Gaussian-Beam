# Gaussian-Beam — Master Research Report

**Document 01 of the Gaussian-Beam documentation set**
**Status:** Research phase, pass 1
**Date:** 17 August 2026
**Author:** Ankit Pawar, with Claude (research assistant)
**Scope of this document:** Evidence gathering and hypothesis testing only. This document does *not* specify the product, the implementation, the security model, the business model, or the UI. Those are separate documents (02–08). This document exists to establish what is *true* before anything is built on top of it.

---

## 0. How to read this document

Every substantive claim below carries a confidence label:

| Label | Meaning |
|---|---|
| **[Established]** | Textbook or peer-reviewed result, multiple independent sources. Safe to build on. |
| **[Supported]** | Published, credible, but either recent, single-source, or not yet independently replicated. Build on it, but with a fallback. |
| **[Plausible]** | Reasoning is sound and consistent with the literature, but I found no source stating it directly. Treat as a hypothesis to test, not a fact. |
| **[Contested]** | Sources disagree, or the numbers vary by more than a factor of two. Do not quote without the caveat. |

I have deliberately included the places where the idea **fails**. A research document that only supports the founder's hypothesis is worthless — the failure boundaries below are the most commercially useful content here, because they define what the product must refuse to do.

---

## 1. Executive summary — the verdict on the unified-model hypothesis

You chose "a unified model across disciplines." I flagged it as the highest-novelty-risk option and said I'd test whether it's defensible before we commit. Here is the result.

**The grand claim is false. The narrow claim is true, deep, and largely unexploited commercially.**

### 1.1 The grand claim (reject)

> *"There is a single mathematical model that governs optics, numerics, machine learning and finance."*

This is not defensible. It is the kind of statement that gets a project dismissed by exactly the research-lab and industry buyers you want. Any sufficiently general statement covering all four fields ("everything is an operator on a function space", "everything is optimisation") is true but computationally empty — it buys you no algorithm, no speedup, and no accuracy guarantee. **Do not build the marketing on this.**

### 1.2 The narrow claim (accept)

> *"Gaussian/quadratic-exponential states, evolved by symplectic (metaplectic) transformations and tracked by Riccati/Möbius covariance flow, form one concrete computational kernel that instantiates natively in paraxial optics, quantum optics, signal processing, control and estimation, affine finance, and linear-drift diffusion models."*

This is **[Established]**, in pieces, across five separate literatures that largely do not cite one another. The pieces:

1. **The PDE layer.** The paraxial wave equation is mathematically the 2-D free-particle Schrödinger equation with the propagation coordinate `z` playing the role of time. Under Wick rotation (`t → iτ`) the Schrödinger equation becomes the heat/diffusion equation; the Black–Scholes equation reduces to the heat equation under a standard change of variables; the Fokker–Planck equation is the same parabolic family. **[Established]**

2. **The group layer.** Propagation of a paraxial beam through any sequence of linear optical elements is a *linear canonical transform* (LCT) — a four-parameter family `(A,B,C,D)` with `AD − BC = 1`, i.e. an element of `SL(2,ℝ)`/`Sp(2,ℝ)`. The LCT is a faithful representation of the **metaplectic group**, the double cover of the symplectic group. The LCT family contains the Fourier transform, the fractional Fourier transform, the Fresnel integral and the Laplace transform as special cases. **[Established]**

3. **The state layer.** A Gaussian beam's entire transverse state collapses to one complex number, the beam parameter `q`, which encodes spot size and wavefront curvature. Under the ABCD law it transforms as `q' = (Aq + B)/(Cq + D)` — a **Möbius (fractional-linear) transformation**. **[Established]**

4. **The flow layer — the load-bearing connection.** The Möbius transformation is precisely the map that linearises the **Riccati equation**. The Kalman filter's error-covariance propagation *is* a Riccati recursion. Affine term-structure models in fixed income are solved by Riccati ODEs and estimated by Kalman filtering. Gaussian quantum states are tracked by a covariance matrix under symplectic congruence `σ → SσSᵀ`. **[Established]** for each item individually; **[Plausible]** that they have been unified into a single computational kernel — I found no software that does this, which is the opportunity.

### 1.3 What this means for the product

The honest, defensible, sellable thesis is:

> **Gaussian-Beam is a verified high-performance kernel for Gaussian/quadratic state propagation under linear canonical transforms and Riccati flows — plus an agent layer whose primary job is to decide whether your problem legitimately maps onto that kernel, and to refuse when it does not.**

The refusal behaviour is not a limitation to be apologised for. It is the product. Section 8 shows that the single biggest documented failure mode in agentic scientific ML is that a low error metric is mistaken for physical validity. A tool that knows its own validity boundary and enforces it is what a research lab or a photonics engineer will actually pay for.

### 1.4 Transfer to your other projects

You said the model should later improve Orbital-AI, Interp, and the Trading algorithm. The kernel transfers as follows — **[Plausible]**, needs prototyping to confirm:

- **Trading algorithm** — direct. Affine term structure, linear-Gaussian state space models, Kalman/Riccati filtering are the same machinery. See §6.
- **Orbital-AI** — partial. Orbit determination is Kalman filtering with a nonlinear dynamics model; the covariance-propagation half transfers, the dynamics half does not.
- **Interp** — indirect and weakest. The honest link is Gaussian approximations to network internals (linearised feature geometry, covariance structure of activations). Do not oversell this one.

---

## PART I — THE MATHEMATICAL CORE

## 2. The parabolic PDE family

### 2.1 Paraxial optics ≅ Schrödinger

For a monochromatic beam with slowly varying envelope `u(x,y,z)`, the paraxial Helmholtz equation is

```
2ik ∂u/∂z + (∂²/∂x² + ∂²/∂y²) u = 0
```

which is the 2-D free-particle Schrödinger equation under `z ↔ t`, `k ↔ m/ħ`. **[Established]** — this is standard in every graduate optics text and is the explicit basis of the Hermite-Gauss mode expansion used in Finesse.

Recent work formalises this further: a propagation-dependent unitary-transformation framework for the paraxial wave equation, connected to the Lewis–Ermakov invariant, establishes formal equivalence with a class of time-dependent quantum systems (arXiv:2510.00277). **[Supported]** — 2025, useful for handling `z`-dependent media.

### 2.2 Schrödinger ≅ heat ≅ Black–Scholes

Under Wick rotation `t → iτ`, the time-dependent Schrödinger equation becomes the heat equation with diffusion coefficient `α² = ħ/2m`. The Black–Scholes PDE maps to the heat equation under the standard log-price/time-reversal substitution, and therefore connects to the free Schrödinger equation by the same rotation. All are parabolic PDEs. **[Established]**

A 2025 result establishes a formal equivalence between the generalised Black–Scholes equation under quadratic normal volatility and the *stationary* Schrödinger equation for a Pöschl–Teller potential (arXiv:2507.12501). **[Supported]** — single source, but it is exactly the class of mapping the kernel would exploit.

### 2.3 The honest caveat on this layer

This equivalence is *structural*, not *physical*. It does not mean an option price is a quantum amplitude. What it buys you is concrete and limited: **the same discretisation, the same propagator, the same stability analysis and the same error bounds can be reused across all four equations.** That reuse is an engineering win, not a scientific discovery, and the documentation must say so. Overstating it is the fastest way to lose a physicist reviewer.

---

## 3. The metaplectic / linear-canonical-transform layer

### 3.1 Origin and structure

LCTs were introduced almost simultaneously in the early 1970s by **Stuart A. Collins Jr.** in paraxial optics and independently by **Moshinsky & Quesne** in quantum mechanics, to describe conservation of information and uncertainty under linear maps of phase space. The two literatures only began citing each other in the 1990s. **[Established]** (Springer, *Development of Linear Canonical Transforms: A Historical Sketch*.)

Formally, the LCT is a family of integral transforms parameterised up to sign by `(A,B,C,D)` with `AD − BC = 1`, containing the Fourier, fractional Fourier, Fresnel and Laplace transforms. Abstractly it is a faithful representation of the metaplectic group. Bacry & Cadilhac established that paraxial light propagation is described by a group isomorphic to the ten-dimensional metaplectic group `Mp(2)` (*Phys. Rev. A* **23**, 2533, 1981). **[Established]**

### 3.2 Why this matters computationally — the strongest evidence for the whole thesis

The most directly relevant paper I found is **arXiv:2106.03367, "Modeling circulating cavity fields using the discrete linear canonical transform"**. It applies the discrete LCT to Fabry–Perot and coupled-cavity fields of exactly the kind used in aLIGO and Advanced Virgo. **[Supported]**

This is important for three reasons:

1. It sits precisely in *your* domain — GW-detector interferometry, the subject of your M.Sc. Literature Review folder.
2. It demonstrates the LCT is not just an elegant reformulation but a *working numerical method* for cavity fields.
3. It shows the gap: the LCT approach exists in the literature but is **not** the basis of the mainstream tooling (Finesse uses Hermite-Gauss modal decomposition; other tools use FFT propagation). That gap is where a product can live.

Supporting work on discretisation: **arXiv:1805.11416** (discrete LCT via hyperdifferential operators) and Optica *JOSA A* **38**(5), 634 — "Exactly unitary discrete representations of the metaplectic transform for linear-time algorithms". The latter is significant: **exactly unitary, linear-time** discrete metaplectic transforms. **[Supported]** — if this holds up at scale it is the core numerical primitive of the product.

### 3.3 Open question

Whether a discrete LCT/metaplectic propagator beats FFT-based beam propagation (BPM) on the accuracy-per-FLOP frontier for realistic mode-mismatched cavities is **[Plausible]** but **unproven at scale**. This is Research Question RQ-1 in §11 and should be the first prototype we build, because the entire performance argument rests on it.

---

## 4. The Riccati / Möbius covariance flow — the cross-domain bridge

### 4.1 The chain

- A Gaussian beam's spot size and wavefront curvature are captured by the single complex `q`-parameter, which transforms under the ABCD law `q₂/n₂ = (A(q₁/n₁) + B)/(C(q₁/n₁) + D)` — a Möbius transformation. **[Established]**
- The Möbius transformation is the standard linearisation of the Riccati equation; Riccati equations are covariant under linear fractional transformations, which defines classes of conformally equivalent ODEs. **[Established]**
- The Kalman filter's prediction-error covariance obeys the classical Riccati equation; estimation-error and smoothing-error covariances obey equations of the same structure (Assimakis & Adam, *ISRN* 2013). **[Established]**
- Affine term-structure models in finance are defined by Riccati ODEs for the loading functions and are canonically estimated by casting them in state-space form and running a Kalman filter (Duan & Simonato, *Rev. Quant. Finance & Accounting*; Bank of Canada WP 01-15; Christoffersen et al., *Management Science*, on nonlinear Kalman filtering in ATSMs). **[Established]**
- Gaussian quantum states are fully specified by a first-moment vector and covariance matrix, evolving under symplectic congruence. **[Established]**

### 4.2 The unified instantiation table

| Domain | State object | Evolution law | Group |
|---|---|---|---|
| Paraxial optics | complex beam parameter `q` | ABCD law (Möbius) | `Sp(2,ℝ)` |
| Quantum mechanics | Gaussian wavepacket | metaplectic representation | `Mp(2n)` |
| Quantum optics | covariance matrix `σ` | `σ → SσSᵀ` | `Sp(2n,ℝ)` |
| Signal processing | LCT / FrFT kernel | `(A,B,C,D)`, `AD−BC=1` | `Mp(2)` |
| Control & estimation | Kalman covariance `P` | Riccati recursion | — |
| Fixed income | ATSM loadings `A(τ), B(τ)` | Riccati ODEs | — |
| Linear-drift SDE | Gaussian transition kernel | Lyapunov/Riccati covariance ODE | — |
| Score-based diffusion (linear drift) | Gaussian perturbation kernel | closed-form `μ(t), Σ(t)`; score `= −Σ⁻¹(x−μ)` | — |

The last row is **[Supported]**: recent work notes that score estimation is classically tractable precisely *when the drift is linear*, because the marginals stay Gaussian (arXiv:2606.27954). That is the same Gaussian-closure condition as everywhere else in the table. Broader unification work on diffusion/score/flow-matching models (arXiv:2605.06829, arXiv:2603.18992 on Schrödinger bridges — 247 votes, 4,287 views, i.e. a high-attention paper) confirms this is an active and receptive research area.

### 4.3 Where the unification breaks — read this before designing anything

The kernel is exact **only** under three conditions. Every one of them is violated by real problems, and the agent layer must detect each violation:

| # | Condition | Broken by | Consequence |
|---|---|---|---|
| B1 | **Quadratic Hamiltonian / linear dynamics** | Kerr and other nonlinear media; nonlinear optics; non-affine stochastic volatility; any deep network | Gaussian closure fails; the `q`-parameter no longer suffices |
| B2 | **Gaussian state** | Non-Gaussian quantum resources; jump-diffusion and fat tails in finance; real multimodal data distributions | Covariance is no longer a sufficient statistic |
| B3 | **Paraxiality / small angles** | Tight focusing, high-NA optics, large scattering angles | Paraxial approximation invalid; needs full vector diffraction |

A fourth, softer boundary: **B4 — sufficient mode truncation.** Finesse's own documentation notes that coupling coefficients decrease with mode number, so a finite mode basis converges *only if the two beam parameters `q₁` and `q₂` do not differ too much*. Severe mode mismatch breaks the truncation. **[Established]**

**This is the product's honest scope, and stating it plainly is a competitive advantage.** Commercial optics tools do not loudly tell you when you have left their validity region. An agent that does is differentiated.

---

## PART II — DOMAIN SURVEYS

## 5. Optics, photonics and interferometry — state of the art

### 5.1 The incumbent open-source stack (your domain)

**Finesse 3** is the dominant open tool for laser-interferometer simulation, used to support GW-detector research since 1999 and developed openly since 2013. Version 3 is a complete redevelopment begun in 2017, now a Python package rather than native binaries, combining a domain-specific language (KatScript) with a Python API. **[Established]**

Technical characteristics relevant to us:

- Translates an interferometer description into a **set of linear equations solved numerically** — a sparse matrix built from the coupling equations, solved for each excitation/configuration.
- Supports both plane-wave approximation and **Hermite-Gauss modal decomposition**, enabling mode-matching and misalignment analysis.
- Computes modulation-demodulation error signals, transfer functions, quantum-noise-limited sensitivities and beam shapes.
- Handles mode coupling through **scattering matrices** of overlap-integral coupling coefficients, separable as `k_{nmn'm'} = k_{nn'} k_{mm'}`.

Peer group: **Melody, Optickle, MIST**, and the FFT-based **Oscar**. Pykat (arXiv:2004.06270) additionally bundles an FFT optical modelling tool, ABCD Gaussian beam propagation code, and higher-order-mode scattering routines. **[Established]**

**The critical weakness, stated by the developers themselves:** *"Although many attempts have been made, Finesse does not significantly benefit from parallelisation when running a single simulation. Experience has shown it is more optimal to run many instances of Finesse simultaneously."* **[Established]**

This is the single most actionable finding in the optics survey. The incumbent open tool is **embarrassingly-parallel across runs but serial within a run.** A kernel that parallelises *within* a single solve — GPU-resident sparse solve, or an LCT propagator that is linear-time and unitary by construction — is a defensible technical claim against the incumbent, not marketing.

### 5.2 Active research frontier (2026)

Recent papers indicate where demand is:

- Laguerre-Gauss mode sensitivity to misalignment and mode mismatch in GW detectors (arXiv:2607.24366)
- Thermal-deformation reduction using higher-order modes (arXiv:2605.10222)
- Simultaneous misalignment and mode-mismatch sensing from intensity-only measurements (arXiv:2603.05101)
- Multimode quantum effects of optical-axis misalignment in aLIGO (arXiv:2608.08936)
- Beam-tube boundary effects in stray-light modelling for 10–40 km cavities in Cosmic Explorer and Einstein Telescope — explicitly noting that **"FFT-based paraxial tools treat…"** the boundary inadequately (arXiv:2602.21303)

**Mode mismatch and misalignment sensing is the hot problem**, and it is precisely the problem the `q`-parameter/Riccati kernel is natively built for. That alignment between the mathematical core and the live research demand is the strongest strategic signal in this report. **[Supported]**

### 5.3 Also relevant

Partial coherence remains awkward in wave-optics simulation; a 2025 framework proposes surface-encoded partial-coherence transformation to avoid expensive ensemble averaging (arXiv:2505.17754). **[Supported]** — a candidate differentiating feature, since Gaussian-Schell-model beams stay inside the Gaussian kernel.

---

## 6. Finance and interpretability — does the kernel actually transfer?

### 6.1 Finance — yes, strongly

The transfer is not analogy; it is the same equations.

- ATSMs require state-space representation and Kalman filtering for parameter estimation; the Kalman filter recursively infers unobserved state variables conditioned on observed zero-coupon rates. **[Established]**
- The Gaussian case (Vasicek 1977) sits exactly inside the kernel. The non-Gaussian case (Cox–Ingersoll–Ross 1985) sits *outside* it and requires the extended or unscented Kalman filter. The unscented filter significantly outperforms the extended filter when caps are used to filter states. **[Established]**

Note what this means: **finance already has a mature, well-understood taxonomy of exactly where the Gaussian kernel holds and where it fails.** That taxonomy (Vasicek inside, CIR outside; EKF/UKF as the escape hatch) is a ready-made template for the validity-gating logic the agent layer needs. We can borrow the discipline wholesale.

### 6.2 Interpretability — weak, do not oversell

I found no literature supporting a direct metaplectic/Riccati framing of mechanistic interpretability. The honest link is limited to Gaussian approximations of network internals — activation covariance structure, linearised feature geometry, Gaussian-process limits of wide networks. **[Plausible]** at best.

**Recommendation:** keep Interp out of the product thesis entirely for now. Mentioning it weakens the credibility of the strong claims. Revisit only after the optics and finance paths are proven.

---

## 7. Numerics, HPC and language choice

### 7.1 The performance evidence

The language comparison literature is noisier and lower-quality than the physics literature — much of it is blog benchmarks rather than peer-reviewed. Treat all of §7.1 as **[Contested]** and re-benchmark on our own workload before committing.

What the sources say:

- Rust and C++ produce **essentially equivalent binaries** for pure computational workloads without complex memory patterns.
- One comparison found Rust 5–10% faster than Julia due to better memory locality and SIMD, while Julia required **70% less code**.
- A different benchmark reports Julia beating Rust substantially on a data-processing task.

These conflict. That is expected: microbenchmarks measure the benchmark, not the language. **The only trustworthy number will be one we generate on an LCT/Riccati propagation workload ourselves.** That is Research Question RQ-2.

### 7.2 Ecosystem reality (more decisive than raw speed)

- **C++** — decades of scientific libraries (Eigen, Armadillo). Deepest ecosystem, worst safety and build ergonomics.
- **Julia** — ships with scientific data structures, parallel primitives designed in from day one, 12,000+ numerically-focused packages. Best expressiveness-per-line for this exact domain. Known weaknesses (not covered in these sources, flagged from general knowledge as **[Plausible]**): time-to-first-execution latency, and deployment/binary-distribution friction for a commercial product.
- **Rust** — safety plus performance, but a **thinner specialised scientific library set** than Julia or C++. However the GPU story has matured materially:
  - **CubeCL** targets NVIDIA CUDA, AMD ROCm HIP, Apple Metal, WebGPU and Vulkan, plus SIMD CPU execution, from one kernel source. It sits between low-level wrappers (`wgpu`, `cudarc`) and high-level frameworks. It includes an optimised matmul module using Tensor Cores where available, with graceful fallback. **[Supported]**
  - **Burn 0.20** (MIT/Apache-2.0) introduces CubeK, high-performance multi-platform kernels in CubeCL; its backends compose with **autodiff, kernel fusion and remote execution** decorators. **[Supported]**

### 7.3 Preliminary reading — not yet a decision

Write-once-run-on-any-GPU with composable autodiff is *exactly* the requirement profile for this kernel, and Rust+CubeCL is currently the only stack offering it with memory safety and clean commercial redistribution. Julia offers faster research iteration and a richer numerical ecosystem.

**The likely answer is a split, but I will not commit to it until RQ-2 is benchmarked.** A plausible shape: Rust/CubeCL for the kernel, Python for the user-facing API (because Finesse, Pykat and the entire GW community are already Python, and fighting that is strategically unwise). Full analysis belongs in Document 05 (Language Efficiency), not here.

---

## 8. AI agents and physics-informed ML

### 8.1 The single most important finding in this report

**arXiv:2607.07379, "Physics-Audited Agentic Discovery in Scientific Machine Learning"** states the core problem directly: in agentic SciML, LLM agents discover surrogate models and select one by an automated score — typically an error metric — but **a low error does not imply physical validity.** **[Supported]**

Corroborating work:

- **arXiv:2604.00149** — "Towards Verifiable and Self-Correcting AI Physicists": LLM application to rigorous physical research is stalled by a lack of verifiability and self-correction.
- **arXiv:2507.23276** — "How Far Are AI Scientists from Changing the World?" (60 votes, 643 views) — sceptical assessment of AI-scientist systems.
- **arXiv:2606.18648** — "Deep Research in Physical Sciences: A Multi-Agent Framework and Comprehensive Benchmark" (71 votes, 593 views) — the benchmark infrastructure now exists to measure this.

**Design consequence, and it is non-negotiable:** the agent layer must never select a model by error metric alone. Every agent output must be gated by a physics audit — conservation laws, boundary conditions, unitarity of the propagator, and the B1–B4 validity conditions of §4.3. This is the architectural spine of the product.

### 8.2 Uncertainty quantification — the enabling technology

Conformal prediction has become the practical, distribution-free UQ method for physics surrogates:

- Conformal prediction for neural operators — distribution-free UQ in physics simulation (arXiv:2606.09923)
- Conformal prediction for PINNs, addressing the lack of rigorous statistical guarantees (arXiv:2509.13717)
- Geometry-aware post-hoc UQ in operator learning (arXiv:2606.17513)
- Invariant-measure conformal prediction for learned stochastic dynamics (arXiv:2606.31607)
- Randomised neural operators with fast training and conformal UQ (arXiv:2606.29440)
- A structured taxonomy of uncertainty in physics and AI (arXiv:2605.10378)

**[Supported]**, and converging fast — five of these are from 2026. Conformal prediction gives finite-sample, distribution-free coverage guarantees, which is exactly the kind of claim a research lab will accept and a marketing claim will not.

**Design consequence:** every numerical output ships with a calibrated uncertainty interval. Not optional, not a v2 feature.

### 8.3 Agent architecture prior art

**arXiv:2607.12122** describes an agentic AI scientific community for automated neural-operator discovery — a swarm of virtual laboratories interacting under a **citation-based economy**. **[Supported]** Directly relevant to your "lots of AI agents" requirement: it is an existence proof that multi-agent scientific swarms are being built, and a warning that we need a differentiator beyond "we have many agents." **EvoPINN** (arXiv:2607.26490) does agentic discovery of executable PINN algorithms.

Our differentiator is not agent count. It is **the verified kernel underneath plus mandatory physics auditing** — agents that are constrained by an exactly-unitary propagator cannot hallucinate a non-physical result in the way an unconstrained surrogate-discovery agent can.

---

## 9. Security posture — inputs for Document 04

Full threat model belongs in the security document. The research inputs:

- **OWASP Top 10 for Agentic Applications (2026)** was released **9 December 2025**, with ten entries prefixed `ASI`, running from **ASI01 agent goal hijack** to **ASI10 rogue agents**. Covers prompt injection, insecure tool execution, excessive agency and memory poisoning. **[Established]**
- In the **OWASP LLM Top 10 (2026)**, prompt injection remains **LLM01**; sensitive-information disclosure remains #2; **excessive agency rose from #6 (2025) to #3 (2026)**. **[Established]**
- Recommended defences: least-privilege tooling, input/output filtering, human approval for high-risk actions, regular adversarial testing, tool-level sandboxing with ephemeral credentials, and systematic policy auditing.
- **Supply chain:** registry allowlists instead of generative tool selection, cryptographic provenance at **SLSA Level 3**, signed artifacts. Sonatype reported **more than 454,600 new malicious packages in 2025, a 75% year-on-year increase.** **[Supported]**
- Microsoft released an open-source **Agent Governance Toolkit** for runtime agent security (April 2026). **[Supported]**

**Scientific-software-specific angle we must add ourselves:** for a simulation product, the highest-value attack is not data theft — it is **silent numerical corruption**. An attacker who perturbs a propagator kernel or a calibration constant produces plausible-looking wrong physics. Defences must therefore include reproducible builds, signed golden-reference test vectors, and cross-checking the fast path against a slow analytically-exact path. I found no literature addressing this specific threat model, so it is **[Plausible]** and original to us — and possibly a publishable contribution in its own right.

---

## 10. Market, competition and pricing

### 10.1 Market size — treat with suspicion

Estimates vary by more than a factor of two and the sources are commercial market-research firms of unverified rigour. **[Contested]** — do not quote these to an investor without the caveat.

| Source | Segment | 2026 value | Forecast | CAGR |
|---|---|---|---|---|
| Business Research Insights | Optical design software | $487.6M | $1,033.07M by 2035 | 8.70% |
| Business Research Insights | Optical simulation software | $0.47B | $1.18B by 2035 | 11% |
| Verified Market Reports | Optical design & simulation software | $1.3B (2025) | $2.69B by 2033 | 9.5% |

Read conservatively: **the addressable market for optical design/simulation software is somewhere around $0.5–1.3B, growing at roughly 8–11%.** That is a real market but not a large one. It will not support a venture-scale outcome on optics alone — which is an argument *for* the cross-domain kernel, provided the cross-domain claim stays honest.

### 10.2 Competitors

- **Ansys Zemax OpticStudio** — priced **$4,900–$14,900** depending on edition, one year of maintenance included, fees thereafter. Subscription licensing introduced 2019. **[Supported]**
- **Ansys Speos / Ansys Optics**, **Synopsys** (incl. Zemax under the Ansys/Synopsys umbrella) — enterprise, quote-only.
- **LightTrans VirtualLab Fusion** — perpetual licence including 12 months of update service; physical-optics platform; has an education programme. Public pricing not disclosed. **[Supported]**
- **COMSOL** — quote-only.
- Others named: Lambda Research, LTI Optics, OptiLayer, Optica Software, Breault Research, Optiwave, Optenso, Wolfram, ASLD.
- **Open-source incumbents in your niche:** Finesse 3, Pykat, Oscar, Optickle, MIST, Melody — all free.

**Strategic read:** the commercial tools are ray-tracing/lens-design-centric and expensive; the free tools own precision interferometry but are single-threaded and Python-slow. **The gap is a fast, GPU-native, uncertainty-quantified physical-optics kernel for the precision-interferometry and mode-matching problem — a niche the $5–15k lens-design vendors do not serve and the free tools serve slowly.** **[Plausible]** — this is my reading, not a sourced claim, and it needs validation by talking to actual GW/photonics groups.

### 10.3 Licensing model evidence

You selected all three customer segments (labs, industry photonics, developers). The literature on commercial open-source in simulation is consistent about how to serve all three simultaneously:

- **AGPL-3.0 + commercial dual licence** is the dominant pattern. AGPL §13 treats network use as distribution, closing the SaaS loophole and preventing a hyperscaler or competitor from forking, closing and reselling the validated engine. Commercial licences fund development and remove copyleft obligations for proprietary embedding. **[Established]** as an industry pattern, from multiple independent projects.
- **Critical implementation detail:** keep the dependency tree **permissive** (Apache-2.0 / MIT / BSD / ISC) so the commercial edition can be offered cleanly — a copyleft dependency taints the commercial path. Contributor terms must license contributions under both AGPL *and* grant the right to include them in the commercial edition, or the dual-licence model breaks. **[Established]**
- **Counter-argument, and it is serious:** Kitware (VTK, ITK, ParaView — 15+ years, ~1.5M LOC, NIH R01 maintenance grant) argues the opposite for *science*: permissive licences (MIT/BSD/Apache-2.0) plus a **services** revenue model, because copyleft hampers reuse and the goal is a sustainable ecosystem. **[Established]**
- **Hybrid precedent:** Zenotech's ZCFD keeps high-performance kernels and hardware mappings closed (protecting finely-tuned algorithms and funding a 1,000+ test verification suite) with an open Python layer users can modify freely. **[Established]**

**My reading:** the ZCFD hybrid is the closest structural analogue to what Gaussian-Beam should be — open Python/API layer, protected verified kernel. But the decision belongs in the business-model document, after we know whether the primary buyer is a grant-funded lab (favours permissive) or an industrial engineer (favours dual-licence).

---

## 11. Open research questions — the gate before any code

These must be answered before committing to an architecture. Each is a falsifiable test, and **each one can kill the project cheaply**, which is the point.

| ID | Question | Kills the project if… | Method |
|---|---|---|---|
| **RQ-1** | Does a discrete LCT/metaplectic propagator beat FFT-BPM and Finesse's modal method on accuracy-per-FLOP for realistically mode-mismatched cavities? | It doesn't — the entire performance thesis collapses and we are just another wrapper | Benchmark against Finesse 3 on a published aLIGO configuration, using your own HeNe profiling data as ground truth |
| **RQ-2** | Which language/stack actually wins on *this* workload? | N/A — informs, doesn't kill | Implement the same LCT propagator in Rust+CubeCL, Julia and C++; measure |
| **RQ-3** | Can the Gaussian/Riccati kernel be shared across optics and affine finance without per-domain rewrites? | It can't — the cross-domain claim is marketing, and we fall back to an optics-only product | Prototype one kernel; drive it from an ABCD optical stack and a Vasicek ATSM |
| **RQ-4** | Can validity conditions B1–B4 be checked *automatically and cheaply* at runtime? | They can't — the differentiating "refuses when invalid" feature is not shippable | Derive residual-based detectors; test against known-invalid cases (Kerr medium, high-NA, CIR) |
| **RQ-5** | Does conformal prediction give useful (tight, not vacuous) intervals on propagator outputs? | Intervals are too wide to be actionable | Calibrate on your existing beam-profile datasets |
| **RQ-6** | Is there real demand, or is this a solution seeking a problem? | No one wants it | Talk to the Glasgow IGR group and 3–5 photonics engineers before writing production code |

**RQ-1 and RQ-6 are the two that matter most.** RQ-1 is cheap and answerable in days using data you already have. RQ-6 costs nothing but conversations, and you are sitting inside a world-leading gravitational-wave institute — that is an unfair advantage you should use before writing a line of production code.

---

## 12. What I could not establish

Stated explicitly so it is not silently assumed later:

1. **No source unifies the full chain** (paraxial ↔ metaplectic ↔ Riccati ↔ Kalman ↔ ATSM ↔ diffusion models) into one computational framework. Each link is established; the synthesis appears to be genuinely unclaimed. This is simultaneously the opportunity and the risk — it may be unclaimed because it isn't useful.
2. **No evidence** for a metaplectic/Riccati framing of mechanistic interpretability.
3. **No published benchmark** of discrete LCT propagation against Finesse or FFT-BPM on realistic GW configurations. (RQ-1.)
4. **No security literature** on silent numerical corruption of scientific simulation kernels as an attack class.
5. **Market figures are commercial-vendor estimates**, mutually inconsistent by >2×, and should not be treated as reliable.
6. **Language benchmarks are blog-grade**, mutually contradictory, and not transferable to our workload.

---

## 13. References

### Mathematical core
1. Bacry, H. & Cadilhac, M. (1981). Metaplectic group and Fourier optics. *Phys. Rev. A* **23**, 2533. https://link.aps.org/pdf/10.1103/PhysRevA.23.2533
2. Healy, J. J. et al. Development of Linear Canonical Transforms: A Historical Sketch. Springer. https://link.springer.com/chapter/10.1007/978-1-4939-3028-9_1
3. Exactly unitary discrete representations of the metaplectic transform for linear-time algorithms. *JOSA A* **38**(5), 634. https://opg.optica.org/josaa/abstract.cfm?uri=josaa-38-5-634
4. Discrete Linear Canonical Transform Based on Hyperdifferential Operators. arXiv:1805.11416. https://www.alphaxiv.org/abs/1805.11416
5. A Phase Space Representation of the Metaplectic Group. arXiv:2512.18415. https://www.alphaxiv.org/abs/2512.18415
6. Unitary transformation approach to the paraxial wave equation. arXiv:2510.00277. https://www.alphaxiv.org/abs/2510.00277
7. Quadratic Volatility from the Pöschl-Teller Potential and Hyperbolic Geometry. arXiv:2507.12501. https://www.alphaxiv.org/abs/2507.12501
8. Parabolic partial differential equation. https://en.wikipedia.org/wiki/Parabolic_partial_differential_equation
9. Assimakis, N. & Adam, M. (2013). Kalman Filter Riccati Equation for the Prediction, Estimation, and Smoothing Error Covariance Matrices. *ISRN*. https://www.hindawi.com/journals/isrn/2013/249594/
10. Integrability of Lie systems through Riccati equations. arXiv:1002.0530. https://arxiv.org/pdf/1002.0530
11. Gaussian Beams (Glytsis, NTUA). http://users.ntua.gr/eglytsis/OptEng/Gaussian_Beams.pdf
12. MIT OCW 6.974 — Gaussian Beams and Resonators. https://ocw.mit.edu/courses/6-974-fundamentals-of-photonics-quantum-electronics-spring-2006/e9852c138493233bc2813f683da5b199_gaussian_bem_res.pdf

### Optics & interferometry
13. Modeling circulating cavity fields using the discrete linear canonical transform. arXiv:2106.03367. https://arxiv.org/pdf/2106.03367
14. Freise, A. & Strain, K. Interferometer Techniques for Gravitational-Wave Detection. arXiv:0909.3661. https://arxiv.org/pdf/0909.3661
15. Finesse 3 documentation — Higher-order Hermite-Gauss modes. https://finesse.ifosim.org/docs/latest/usage/higher_order_modes/index.html
16. Finesse 3 documentation — Hermite-Gauss modes and coupling. https://finesse.ifosim.org/docs/latest/usage/higher_order_modes/coupling.html
17. What is Finesse. https://finesse.ifosim.org/docs/latest/what_is_finesse.html
18. Finesse 3 source. https://gitlab.com/ifosim/finesse/finesse3
19. Pykat: Python package for modelling precision optical interferometers. arXiv:2004.06270. https://ar5iv.labs.arxiv.org/html/2004.06270
20. Freise, A. et al. (2004). Frequency-domain interferometer simulation with higher-order spatial modes. *CQG* **21**(5), S1067.
21. Sensitivity of Laguerre-Gaussian Modes to Misalignment and Mode Mismatch. arXiv:2607.24366. https://www.alphaxiv.org/abs/2607.24366
22. Simultaneous Misalignment and Mode Mismatch Sensing Using Intensity-Only Measurements. arXiv:2603.05101. https://www.alphaxiv.org/abs/2603.05101
23. Beam tube boundary effects in stray light modeling for third-generation detectors. arXiv:2602.21303. https://www.alphaxiv.org/abs/2602.21303
24. Multimode Quantum Effects of Optical-Axis Misalignment in GW Interferometers. arXiv:2608.08936. https://www.alphaxiv.org/abs/2608.08936
25. Surface-Encoded Partial Coherence Transformation. arXiv:2505.17754. https://www.alphaxiv.org/abs/2505.17754

### AI agents, PIML & UQ
26. Physics-Audited Agentic Discovery in Scientific Machine Learning. arXiv:2607.07379. https://www.alphaxiv.org/abs/2607.07379
27. Towards Verifiable and Self-Correcting AI Physicists for Quantum Many-Body Simulations. arXiv:2604.00149. https://www.alphaxiv.org/abs/2604.00149
28. Deep Research in Physical Sciences: A Multi-Agent Framework and Benchmark. arXiv:2606.18648. https://www.alphaxiv.org/abs/2606.18648
29. How Far Are AI Scientists from Changing the World? arXiv:2507.23276. https://www.alphaxiv.org/abs/2507.23276
30. An Agentic AI Scientific Community for Automated Neural Operator Discovery. arXiv:2607.12122. https://www.alphaxiv.org/abs/2607.12122
31. EvoPINN: Agentic Discovery of Executable Algorithms for PINNs. arXiv:2607.26490. https://www.alphaxiv.org/abs/2607.26490
32. Conformal Prediction for Neural Operators. arXiv:2606.09923. https://www.alphaxiv.org/abs/2606.09923
33. A Conformal Prediction Framework for UQ in PINNs. arXiv:2509.13717. https://www.alphaxiv.org/abs/2509.13717
34. Uncertainty in Physics and AI: Taxonomy, Quantification, and Validation. arXiv:2605.10378. https://www.alphaxiv.org/abs/2605.10378
35. Geometry-Aware Post-Hoc UQ in Operator Learning. arXiv:2606.17513. https://www.alphaxiv.org/abs/2606.17513
36. Randomized neural operator with conformal UQ. arXiv:2606.29440. https://www.alphaxiv.org/abs/2606.29440
37. UQ via Invariant-Measure Conformal Prediction. arXiv:2606.31607. https://www.alphaxiv.org/abs/2606.31607
38. Local Fokker–Planck Geometry for Score Estimation. arXiv:2606.27954. https://www.alphaxiv.org/abs/2606.27954
39. Foundations of Schrödinger Bridges for Generative Modeling. arXiv:2603.18992. https://www.alphaxiv.org/abs/2603.18992
40. A Unified Measure-Theoretic View of Diffusion, Score-Based, and Flow Matching Generative Models. arXiv:2605.06829. https://www.alphaxiv.org/abs/2605.06829

### Finance
41. Duan, J.-C. & Simonato, J.-G. Estimating and Testing Exponential-Affine Term Structure Models by Kalman Filter. *Rev. Quant. Finance & Accounting*. https://link.springer.com/article/10.1023/A:1008304625054
42. Nonlinear Kalman Filtering in Affine Term Structure Models. *Management Science*. https://pubsonline.informs.org/doi/10.1287/mnsc.2013.1870
43. Affine Term-Structure Models: Theory and Implementation. Bank of Canada WP 01-15. https://www.bankofcanada.ca/wp-content/uploads/2010/02/wp01-15a.pdf

### Languages, HPC & GPU
44. CubeCL — multi-platform compute language extension for Rust. https://github.com/tracel-ai/cubecl
45. Burn — tensor library and deep learning framework in Rust. https://github.com/tracel-ai/burn
46. Burn 0.20 Released. Phoronix. https://www.phoronix.com/news/Burn-0.20-Released
47. The Rust + GPU Ecosystem. https://nvlabs.github.io/cuda-oxide/appendix/ecosystem.html
48. Rust vs Julia in scientific computing. https://mo8it.com/blog/rust-vs-julia/
49. Rust vs C++ Performance Comparison 2026. https://reintech.io/blog/rust-vs-cpp-performance-comparison-2026

### Security
50. OWASP Top 10 for Agentic Applications 2026. https://cycode.com/blog/owasp-top-10-agentic-applications/
51. OWASP GenAI LLM Top 10 2026. https://cybersecuritynews.com/owasp-genai-llm-top-10-2026/
52. OWASP Top 10 for LLM Apps 2026: Excessive agency risk on the rise. https://www.reversinglabs.com/blog/owasp-top-10-for-llm-apps-excessive-agency
53. Towards trustworthy agentic AI: safety, robustness, privacy, system security. arXiv:2605.23989. https://arxiv.org/pdf/2605.23989
54. Security Risks in Tool-Enabled AI Agents. arXiv:2605.09721. https://arxiv.org/pdf/2605.09721
55. Microsoft Agent Governance Toolkit. https://opensource.microsoft.com/blog/2026/04/02/introducing-the-agent-governance-toolkit-open-source-runtime-security-for-ai-agents/
56. The 2026 Guide to Software Supply Chain Security. Cloudsmith. https://cloudsmith.com/blog/the-2026-guide-to-software-supply-chain-security-from-static-sboms-to-agentic-governance

### Market & business model
57. Optical Design Software Market. Business Research Insights. https://www.businessresearchinsights.com/market-reports/optical-design-software-market-123017
58. Optical Simulation Software Market. https://www.businessresearchinsights.com/market-reports/optical-simulation-software-market-115778
59. Optical Design and Simulation Software Market. Verified Market Reports. https://www.verifiedmarketreports.com/product/optical-design-and-simulation-software-market/
60. Zemax Announces OpticStudio Subscription Licenses. https://www.prnewswire.com/news-releases/zemax-announces-opticstudio-subscription-licenses-300860082.html
61. How to Purchase Optical Design Software Without Breaking the Bank. https://www.glthome.com/articles/purchase-optical-design-software-without-breaking-bank/
62. VirtualLab Fusion: A physical optics simulation platform. SPIE. https://www.spiedigitallibrary.org/conference-proceedings-of-spie/OP20EX/OP20EX0G/VirtualLab-Fusion-A-physical-optics-simulation-platform/10.1117/12.2580614.full
63. Zenotech — Open vs Closed Source: why we chose a hybrid model for our CFD solver. https://zenotech.com/news/open-vs-closed-source-software-why-we-chose-a-hybrid-model-for-our-cfd-solver/
64. Sustainable Software Ecosystems for Open Science: 15 Years at Kitware. arXiv:1309.2966. https://ar5iv.labs.arxiv.org/html/1309.2966

---

## 14. Change log

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-08-17 | Initial research pass. Unified-model hypothesis tested and narrowed. Six research questions defined as the gate before implementation. |
