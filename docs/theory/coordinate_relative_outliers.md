# Coordinate-Relative Nature of Outliers

---

## 1. The Conventional Interpretation of Outliers

In most LLM quantization literature, outliers are defined operationally:
weights with unusually large magnitudes relative to the bulk distribution.

This leads to common assumptions:

- large-magnitude weights are more important,
- outliers encode rare but critical information,
- quantization must preserve or isolate these values.

As a result, many methods focus on:

- clipping thresholds,
- mixed-precision handling,
- outlier-aware scaling.

However, these approaches implicitly assume that
**magnitude correlates with semantic importance**.

---

## 2. Empirical Contradictions

Several empirical observations challenge this assumption:

1. Orthogonal rotations (e.g., Hadamard) can drastically reduce or eliminate
   outliers without affecting model accuracy.
2. Uniform low-bit quantization often works even when outliers are not preserved.
3. Different coordinate systems exhibit radically different tail statistics
   for the same trained model.

These facts imply that outliers are not intrinsic properties of the operator,
but depend on how it is parameterized.

---

## 3. Outliers as Coordinate Projections

Let \( W \in \mathbb{R}^N \) be a parameter vector representing an operator.
Consider a change of coordinates via an orthogonal transform \( T \):

\[
W' = T W, \quad T^\top T = I
\]

Then:

- \( \|W'\|\_2 = \|W\|\_2 \) (energy preserved),
- \( \|W'\|\_\infty \), kurtosis, and tail statistics may change arbitrarily.

If outliers were intrinsic information carriers,
such drastic changes would alter model behavior.
Since they do not, outliers must be **coordinate-dependent statistics**.

---

## 4. Geometric Interpretation via Manifolds

Assume that trained weights lie near a low-dimensional manifold
\(\mathcal{M} \subset \mathbb{R}^N\).

Locally:
\[
W \approx \mu + J \theta
\]
where:

- \(J\) spans the tangent space of \(\mathcal{M}\),
- \(\theta\) parameterizes meaningful variation.

Outliers arise when:

- coordinate axes are misaligned with the tangent space,
- curvature directions project disproportionately onto a few axes.

In this view:

- outliers correspond to **high curvature projections**,
- not to inherently important degrees of freedom.

---

## 5. Sensitivity vs. Magnitude

From an operator-centric perspective,
the importance of a parameter direction is determined by sensitivity:

\[
\text{Sensitivity}(v) \approx \left\| \frac{\partial f}{\partial W} v \right\|
\]

Large coordinate values do not necessarily imply
high sensitivity directions.

A large-magnitude parameter may lie in a direction
to which the operator is relatively insensitive,
while a small-magnitude parameter may lie along
a highly sensitive direction.

Thus:

> **Magnitude is not a reliable proxy for importance.**

---

## 6. Why Rotations Remove Outliers

Orthogonal rotations redistribute parameter energy
across coordinates without altering the operator.

If outliers arise from axis misalignment,
random rotations tend to:

- spread curvature-induced projections,
- reduce peak magnitudes,
- flatten heavy-tailed distributions.

This explains why:

- Hadamard-based rotations suppress outliers,
- kurtosis and entropy decrease,
- model accuracy remains unchanged.

However, this flattening does not imply
that the new coordinates are semantically meaningful.

---

## 7. Flattening vs. Concentration

It is important to distinguish between two effects:

1. **Flattening**
   - energy spread uniformly,
   - reduced tails,
   - achieved by random rotations.

2. **Concentration**
   - energy concentrated in few coordinates,
   - low effective dimensionality,
   - requires alignment with the manifold structure.

Rotation-based methods primarily achieve flattening.
Compression-optimal representations require concentration.

---

## 8. Implications for Quantization

Quantization error interacts with coordinate geometry:

- In misaligned coordinates:
  - outliers dominate scale selection,
  - quantization error is unevenly distributed.

- In flattened coordinates:
  - quantization is more stable,
  - but not necessarily optimal in rate–distortion terms.

- In manifold-aligned coordinates:
  - important variations are isolated,
  - residuals can be aggressively quantized.

Thus, outlier-aware quantization should be replaced by
**coordinate-aware reparameterization**.

---

## 9. Reframing Outlier Mitigation

Rather than asking:

> “How do we preserve outliers?”

we should ask:

> “Why do outliers appear in this coordinate system?”

This reframing shifts focus from local heuristics
to global geometric structure.

---

## Takeaway

> **Outliers are not intrinsic properties of LLM operators,
> but artifacts of misaligned coordinate systems.**

Recognizing outliers as coordinate-relative phenomena
opens the door to principled compression strategies
based on axis alignment and manifold-aware representations.

---
