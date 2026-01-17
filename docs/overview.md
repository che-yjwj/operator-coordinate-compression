---

# 📄 `docs/overview.md`

```markdown
# Overview: Operator Coordinate Compression

This document provides a high-level overview of the conceptual framework
used in this repository.

---

## 1. The Problem with Value-Centric Thinking

Traditional LLM quantization treats weights as independent scalar values.
Under this view:

- Large-magnitude weights are assumed to carry more information.
- Outliers are treated as special cases requiring protection.
- Compression focuses on local heuristics (clipping, grouping, smoothing).

However, this perspective struggles to explain why:

- orthogonal rotations can eliminate outliers without information loss,
- aggressive low-bit quantization often preserves model quality.

---

## 2. Operator-Centric View

An LLM implements a nonlinear operator:

\[
y = f(x; W)
\]

The weight tensor \(W\) should be interpreted as:

- a **parameterization of the operator**, not
- a collection of independent data points.

Compression is therefore a **reparameterization problem**, not a data deletion problem.

---

## 3. Manifold Hypothesis for LLM Weights

We hypothesize that trained LLM weights:

- lie on a low-dimensional nonlinear manifold \(\mathcal{M}\),
- embedded in a high-dimensional parameter space.

Locally, weights can be approximated as:

\[
W \approx \mu + J \theta
\]

where:

- \(J\) spans the tangent space of the manifold,
- \(\theta\) captures the meaningful degrees of freedom.

---

## 4. Coordinate-Relative Outliers

Outliers arise when:

- the chosen coordinate axes are misaligned with the manifold’s tangent space,
- curvature projects disproportionately onto a small number of axes.

Thus:

> **Outliers are coordinate artifacts, not intrinsic signals.**

This explains why:

- orthogonal rotations can dramatically change outlier statistics,
- L2 energy is preserved while L∞ and kurtosis change.

---

## 5. Axis Transformations vs. Frequency Transforms

- Frequency transforms (DCT, FFT):
  - assume meaningful physical axes,
  - aim to separate semantic frequencies,
  - are typically lossy.
- Axis transformations (Hadamard, rotation):
  - preserve information,
  - redistribute energy across coordinates,
  - have no semantic frequency interpretation.

In LLM compression, Hadamard-based methods should be understood
as **coordinate rotations**, not frequency decompositions.

---

## 6. From Flattening to Concentration

We distinguish three regimes:

1. **Original coordinates**
   - spiky distributions, heavy tails.
2. **Random rotations**
   - flattened distributions, reduced outliers.
3. **Manifold-aligned coordinates**
   - concentrated coefficients, low effective dimensionality.

Rotation-based methods achieve (2), but optimal compression requires (3).

---

## 7. Functional + Residual Representation

Aligned coordinates allow weights to be decomposed as:

\[
w*i \approx f*\theta(i) + r_i
\]

- \(f\_\theta\): structured, low-dimensional component.
- \(r_i\): small residuals suitable for low-bit quantization
  and entropy coding.

This decomposition naturally aligns with rate–distortion optimal design.

---

## 8. Why Toy Experiments Matter

Controlled toy experiments (e.g., Fourier basis vs. MLP basis)
demonstrate that:

- the same function can admit vastly different parameter distributions
  depending on the chosen basis,
- compression difficulty is a property of the representation,
  not the function itself.

These experiments provide intuition for analogous phenomena in LLMs.

---

## 9. Toward Manifold-Aware LLM Compression

This repository explores:

- practical approximations of manifold-aligned axes,
- asymmetric quantization of structured vs. residual components,
- unification of rotation, quantization, and entropy coding
  under a single coordinate-centric framework.

---

## Takeaway

> **LLM compression is fundamentally a coordinate selection problem
> for operator parameters.**
