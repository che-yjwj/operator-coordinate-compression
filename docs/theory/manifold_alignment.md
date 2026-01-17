# Manifold-Aligned Coordinate Systems

## From Rotation to Structural Compression

---

## 1. What Do We Mean by “Alignment”?

In this repository, **alignment** refers to the choice of a coordinate system
that is adapted to the intrinsic structure of the operator parameter space.

Formally, let:

- \( W \in \mathbb{R}^N \): parameter vector of an LLM operator,
- \( \mathcal{M} \subset \mathbb{R}^N \): a low-dimensional manifold on which
  trained parameters concentrate.

A coordinate system is said to be **manifold-aligned** if its axes are
approximately aligned with:

- the tangent space of \(\mathcal{M}\),
- or other low-sensitivity / low-curvature directions.

Alignment is therefore a **geometric property**, not a statistical one.

---

## 2. Manifold Hypothesis for Operator Parameters

We adopt the following hypothesis:

> **Trained LLM weights lie near a low-dimensional, curved manifold
> embedded in high-dimensional parameter space.**

This hypothesis is supported by:

- robustness to noise and quantization,
- flat minima in loss landscapes,
- empirical success of low-rank approximations and pruning.

Locally, around a trained solution \(W_0\), we can write:
\[
W \approx W_0 + J \theta + \epsilon
\]
where:

- \(J \in \mathbb{R}^{N \times d}\) spans the tangent space,
- \(\theta \in \mathbb{R}^d\) are intrinsic coordinates,
- \(\epsilon\) is a small residual orthogonal to the manifold.

---

## 3. Alignment vs. Rotation: A Geometric Contrast

### 3.1 Rotation

Rotation chooses an arbitrary orthonormal basis in \(\mathbb{R}^N\).

Properties:

- preserves energy,
- removes axis-specific artifacts,
- ignores intrinsic geometry.

Rotation answers the question:

> “How do we avoid a bad coordinate system?”

---

### 3.2 Alignment

Alignment seeks a basis adapted to \(\mathcal{M}\).

Properties:

- concentrates meaningful variation,
- isolates sensitive directions,
- enables structured representations.

Alignment answers the question:

> “What coordinate system best represents the operator?”

---

## 4. Alignment as Functional Representation

In aligned coordinates, parameters admit a decomposition:
\[
W = f\_\theta(\alpha) + r
\]

where:

- \(f\_\theta\): low-dimensional, structured function,
- \(\alpha\): intrinsic coordinates (e.g., layer index, channel index),
- \(r\): small residual noise.

This is analogous to:

- signal + noise decomposition,
- principal + residual components,
- model + error terms.

Crucially, **\(f\_\theta\) is compressible as a function**,
while \(r\) is compressible via quantization and entropy coding.

---

## 5. Why Alignment Enables Compression

Compression effectiveness depends on two factors:

1. **Concentration** of coefficients
2. **Predictability** across coordinates

Manifold alignment improves both:

- coefficients concentrate along few axes,
- remaining coefficients exhibit simple distributions.

This leads to:

- lower Shannon entropy,
- better Huffman / arithmetic coding,
- improved rate–distortion trade-offs.

---

## 6. Practical Approximations to Alignment

Exact manifold alignment is infeasible for large models.
However, practical approximations exist.

### 6.1 Local Linear Approximation (PCA-like)

Within a block or channel group:

- estimate covariance of parameters,
- align axes with top eigenvectors.

This approximates the tangent space locally.

---

### 6.2 Blockwise or Channelwise Alignment

Instead of global alignment:

- align within attention heads,
- align within FFN blocks,
- align per-channel or per-group.

This respects architectural structure.

---

### 6.3 Learned Axis Transforms

Axis transforms may be learned by optimizing:
\[
\min_T \; R(TW) + \lambda D(f(x; TW))
\]

where:

- \(R\): rate proxy (entropy, sparsity),
- \(D\): distortion (loss increase).

This directly targets RD optimality.

---

## 7. Relationship to Existing Methods

Under this framework:

- **Rotation-based methods**:
  - reduce coordinate artifacts,
  - but ignore manifold geometry.
- **Low-rank methods**:
  - partially capture alignment,
  - but are often value-centric.
- **Proposed approach**:
  - explicitly targets manifold-aligned coordinates
  - unifies rotation, quantization, and entropy coding.

---

## 8. Manifold Curvature and Residuals

Curvature plays a key role:

- high curvature → projection-induced outliers,
- low curvature → smooth functional variation.

Alignment reduces curvature projections,
pushing curvature effects into residual terms \(r\),
which can be aggressively quantized or zeroed.

---

## 9. Implications for LLM Quantization

Manifold-aligned representations imply:

- outliers vanish naturally,
- dead-zones emerge meaningfully,
- uniform low-bit quantization becomes viable.

Quantization is no longer a heuristic,
but a natural consequence of representation choice.

---

## Takeaway

> **Rotation stabilizes; alignment compresses.  
> Alignment reveals the functional structure
> that compression can exploit.**

Manifold-aligned coordinate systems provide
a principled foundation for next-generation
LLM quantization and compression methods.

---
