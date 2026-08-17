# Gaussian-Beam

A verified computational kernel for **Gaussian / quadratic-exponential state propagation** under linear canonical transforms and Riccati flows — with an agent layer whose primary job is to decide whether a given problem legitimately maps onto that kernel, and to refuse when it does not.

> **Status:** Research phase. No production code yet. Six research questions (RQ-1 – RQ-6) gate implementation. See [`docs/01_Research_Report.md`](docs/01_Research_Report.md).

---

## The thesis in one paragraph

A Gaussian beam's entire transverse state collapses to one complex number, the beam parameter `q`, which propagates by a Möbius (fractional-linear) map under the ABCD law. The Möbius map is precisely what linearises the **Riccati equation** — and the Riccati equation is what governs Kalman filter covariance propagation, affine term-structure models in fixed income, and the covariance flow of Gaussian quantum states under symplectic evolution. Underneath all of it sits the **metaplectic group**, of which the linear canonical transform (containing the Fourier, fractional Fourier, Fresnel and Laplace transforms) is a faithful representation.

These are the same equations, discovered separately in five literatures that barely cite one another. Gaussian-Beam implements them once, correctly, and fast.

## What this is *not*

It is **not** a claim that one model governs all of physics, finance and machine learning. That claim is unfalsifiable and we explicitly reject it — see §1.1 of the research report. The kernel is exact only under four conditions:

| | Condition | Broken by |
|---|---|---|
| **B1** | Quadratic Hamiltonian / linear dynamics | Kerr media, nonlinear optics, non-affine stochastic volatility |
| **B2** | Gaussian state | Non-Gaussian quantum resources, jump diffusion, multimodal data |
| **B3** | Paraxiality | Tight focusing, high-NA optics, large scattering angles |
| **B4** | Sufficient mode truncation | Severe mode mismatch |

**Detecting and refusing at these boundaries is the product**, not a caveat on it. Recent work shows agentic scientific ML systematically mistakes a low error metric for physical validity ([arXiv:2607.07379](https://arxiv.org/abs/2607.07379)); a kernel that knows where it stops being valid is the answer to that.

## Where it comes from

Grown out of an M.Sc. Astrophysics project at the University of Glasgow on HeNe Gaussian beam profiling and interferometry, in a gravitational-wave detector context.

## Repository layout

```
Gaussian-Beam/
├── docs/          # Design and research documents (01 = research report)
├── prototypes/    # Throwaway code answering RQ-1 … RQ-6
└── README.md
```

## Roadmap

- [x] **Doc 01** — Master research report, 64 references
- [ ] **RQ-1** — Benchmark a discrete LCT propagator against Finesse 3 and FFT-BPM
- [ ] **RQ-6** — Validate demand with the Glasgow IGR group and photonics engineers
- [ ] **Doc 02–08** — Product aim, theory & maths, security model, language rationale, business model, UI/UX

## Licence

Not yet chosen. Candidates under evaluation: AGPL-3.0 + commercial dual licence, versus permissive + services (the Kitware model). Decision belongs in the business-model document. **Until a licence file is added, all rights are reserved.**

## Author

Ankit Pawar — M.Sc. Astrophysics, University of Glasgow
