# Operator-Centric View of LLM Weights

---

## 1. Why “Weight-as-Data” is a Misleading Abstraction

Most discussions of LLM quantization implicitly treat weights as data:
a collection of scalar values whose magnitudes are assumed to correlate
with importance or information content.

Under this abstraction:

- large-magnitude weights are viewed as “important,”
- small-magnitude weights are considered expendable or noise-like,
- quantization is framed as value approximation or truncation.

However, this view leads to conceptual contradictions:

- orthogonal rotations can dramatically alter weight magnitudes
  without changing model behavior,
- aggressive low-bit quantization often preserves perplexity,
- different coordinate systems yield vastly different outlier statistics.

These observations suggest that **weights are not data** in the usual sense.

---

## 2. Weights as Operator Parameters

An LLM implements a nonlinear operator:

\[
y = f(x; W)
\]

where \(W\) denotes the collection of weight tensors.
From a functional perspective:

- \(x\): input signal
- \(f(\cdot; W)\): learned operator
- \(W\): _parameters_ defining the operator, not samples drawn from a distribution

In classical signal processing terms,
weights correspond more closely to **filter coefficients**
than to signals themselves.

This distinction is crucial:

- signals are objects to be compressed,
- operators define transformations and admit multiple equivalent parameterizations.

---

## 3. Reparameterization Invariance

A fundamental property of operators is **reparameterization invariance**.

Let \(T\) be an invertible transformation on the parameter space.
Then the operator can be equivalently represented as:

\[
f(x; W) \equiv f(x; T^{-1} (T W))
\]

If \(T\) is orthogonal, this reparameterization preserves:

- L2 norm of parameters,
- exact functional behavior.

Yet it may drastically change:

- per-coordinate magnitudes,
- kurtosis and tail behavior,
- quantization error under fixed-bit representations.

This implies that **many weight representations are functionally equivalent
but compression-inequivalent**.

---

## 4. Implications for Quantization

Quantization approximates parameters in a _chosen coordinate system_.
Therefore, its effectiveness depends on how well that coordinate system
matches the structure of the operator.

From the operator-centric view:

- quantization error is not inherently “information loss,”
- it is a perturbation of the operator’s parameterization.

The relevant question is not:

> “How accurately are individual weights preserved?”

but rather:

> “How sensitive is the operator \(f(\cdot; W)\) to perturbations
> in a given coordinate system?”

---

## 5. Coordinate Dependence of Sensitivity

Different coordinates correspond to different directions in parameter space.
Perturbations along these directions have unequal impact on the operator.

If the coordinate axes are:

- **aligned with high-sensitivity directions**,
  small quantization error can cause large functional distortion.
- **aligned with low-sensitivity directions**,
  large parameter error may have negligible effect.

Thus, optimal compression seeks coordinate systems in which:

- sensitivity is concentrated in a small number of directions,
- remaining directions are robust to coarse approximation.

---

## 6. Connection to Low-Dimensional Structure

Empirical evidence suggests that trained LLMs exhibit:

- redundancy,
- flat minima,
- robustness to noise.

These properties are consistent with the hypothesis that
operator parameters lie near a **low-dimensional manifold** in parameter space.

In such cases:

- meaningful variations are confined to a few directions,
- many coordinates represent redundant or weakly sensitive dimensions.

This observation motivates the search for coordinate systems
that explicitly expose this low-dimensional structure.

---

## 7. Why Outliers Are Not “Important Weights”

Under the operator-centric view,
large-magnitude coordinates do not necessarily correspond to
high-sensitivity directions.

Outliers can arise when:

- the chosen coordinate axes are poorly aligned with the operator manifold,
- curvature projects strongly onto a small number of axes.

Consequently:

> **Outliers are coordinate artifacts, not intrinsic carriers of information.**

This interpretation explains why:

- orthogonal rotations can remove outliers without affecting accuracy,
- protecting outliers is often unnecessary.

---

## 8. Operator View as the Foundation of This Repository

This repository adopts the operator-centric perspective as its foundation.

All subsequent concepts follow naturally:

- manifold hypothesis for weights,
- coordinate-relative outliers,
- axis transformations vs. frequency transforms,
- functional + residual decomposition,
- rate–distortion–optimal quantization.

Understanding weights as operator parameters
allows compression to be reframed as a problem of
**choosing an appropriate coordinate system for parameterization**.

---

## Takeaway

> **LLM weights are not data to be preserved,
> but parameters of an operator to be reparameterized.**

This simple shift in perspective resolves many apparent paradoxes
in low-bit LLM quantization and enables a principled approach
to compression based on geometry rather than heuristics.

---
