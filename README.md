# operator-coordinate-compression

A research repository on **LLM weight quantization and compression from an
operator- and coordinate-system perspective**.

---

## Motivation

Recent studies show that large language models (LLMs) can be aggressively
quantized to INT8 or even INT4 with minimal degradation in perplexity.
This challenges the conventional view that large-magnitude weights (outliers)
must be carefully preserved.

This repository proposes a different interpretation:

> **LLM compression is not fundamentally about removing information,  
> but about choosing the right coordinate system to represent operator parameters.**

---

## Core Ideas

We argue that:

1. **LLM weights parameterize nonlinear operators**, not independent scalar values.
2. **Trained weights lie on low-dimensional nonlinear manifolds** embedded in
   high-dimensional parameter space.
3. Observed **outliers are coordinate-relative artifacts**, arising from
   misalignment between chosen axes and the manifold’s tangent space.
4. Rotation-based methods (e.g., Hadamard / QuaRot) should be interpreted as
   **axis transformations**, not frequency-domain transforms.
5. While rotations flatten distributions and remove outliers,
   **true compression gains arise from manifold-aligned coordinate systems**
   that induce coefficient concentration.
6. Such alignment enables **functional + residual decompositions**,
   asymmetric quantization, and rate–distortion optimal compression.

---

## What This Repository Contains

- 📐 **Theory**
  - Operator-centric view of LLM weights
  - Manifold hypothesis and coordinate-relative outliers
  - Axis transformation vs. frequency transformation
- 🧪 **Experiments**
  - Toy experiments (Fourier basis vs. MLP basis)
  - Axis transform analysis (Hadamard, PCA alignment)
  - Validation on TinyLLaMA / SLM models
- 🧾 **Writing**
  - Academic paper drafts
  - Patent-oriented invention summaries and claim candidates

---

## Repository Structure

```text
operator-coordinate-compression/
├─ docs/
│  ├─ overview.md
│  ├─ theory/
│  ├─ paper/
│  └─ patent/
├─ experiments/
│  ├─ toy_basis_vs_mlp/
│  ├─ axis_transform_analysis/
│  └─ tinyllama_validation/
├─ src/
│  ├─ transforms/
│  ├─ quantization/
│  ├─ entropy/
│  └─ metrics/
└─ roadmap.md
