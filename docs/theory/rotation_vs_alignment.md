# Rotation vs. Alignment

## Why Flattening Is Not Enough

---

## 1. Why Rotation-Based Methods Appear Powerful

Recent LLM quantization methods (e.g., QuaRot, SpinQuant, KurTail)
demonstrate a striking phenomenon:

- applying an orthogonal rotation to weight matrices,
- followed by uniform low-bit quantization,
- yields minimal degradation in perplexity.

Empirically, such rotations:

- suppress extreme outliers,
- reduce kurtosis,
- stabilize scale selection.

At first glance, this suggests that rotation itself is the key to
successful compression.

However, this interpretation is incomplete.

---

## 2. Rotation Is Not Frequency Decomposition

It is tempting to interpret Hadamard or orthogonal transforms
as frequency-domain operations, analogous to DCT or FFT.
This interpretation is misleading.

Key distinctions:

| Aspect                | Frequency Transform | Axis Rotation             |
| --------------------- | ------------------- | ------------------------- |
| Semantic meaning      | Physical frequency  | None                      |
| Basis                 | Ordered, structured | Arbitrary orthogonal      |
| Energy preservation   | Yes                 | Yes                       |
| Localization          | Frequency-specific  | Coordinate redistribution |
| Compression mechanism | Energy compaction   | Distribution flattening   |

Hadamard transforms used in LLM quantization
**do not correspond to semantic frequencies**.
They are better understood as **random or structured rotations**.

---

## 3. What Rotation Actually Does

Let \( W \in \mathbb{R}^N \) be a parameter vector.
An orthogonal rotation \(T\) produces:

\[
W' = T W
\]

This operation:

- preserves L2 energy,
- preserves exact operator behavior,
- redistributes energy across coordinates.

Statistically, random or pseudo-random rotations tend to:

- reduce peak magnitudes,
- make distributions more Gaussian-like,
- lower kurtosis.

This effect is best described as **flattening**.

---

## 4. Flattening vs. Alignment

Flattening should not be confused with alignment.

### 4.1 Flattening (What Rotation Achieves)

Flattening properties:

- energy spread across many coordinates,
- reduced extreme values,
- improved numerical stability.

However:

- information is still distributed,
- effective dimensionality remains high,
- entropy reduction is limited.

Flattening improves robustness but not optimality.

---

### 4.2 Alignment (What Compression Needs)

Alignment refers to choosing coordinates that:

- coincide with meaningful directions of variation,
- isolate sensitive degrees of freedom,
- concentrate energy into a noticebly smaller subset of coordinates.

Aligned coordinates exhibit:

- low effective rank,
- high top-k energy ratios,
- strong entropy reduction.

Alignment enables **true compression**, not just stable quantization.

---

## 5. Manifold Perspective on Rotation and Alignment

Assume weights lie near a low-dimensional manifold \(\mathcal{M}\).

- Rotation: chooses a random basis in ambient space
- Alignment: chooses a basis adapted to the tangent space of \(\mathcal{M}\)

From this perspective:

- rotation reduces projection artifacts,
- alignment reveals intrinsic structure.

Rotation makes the representation _less bad_;
alignment makes it _good_.

---

## 6. Why Rotation Works Surprisingly Well

Rotation-based methods succeed because:

- LLMs are robust to noise,
- parameter sensitivity is anisotropic,
- random rotations avoid worst-case axis choices.

Thus, flattening alone is often sufficient
to enable aggressive quantization (e.g., INT4).

However, this success should not be misinterpreted as optimality.

---

## 7. Limits of Rotation-Based Compression

Rotation has inherent limitations:

1. **No structural concentration**
   - coefficients remain dense.
2. **Entropy floor**
   - distributions approach Gaussian,
     which is poorly compressible.
3. **No functional interpretability**
   - rotated coordinates lack semantic meaning.

These limitations explain why:

- rotation improves quantization robustness,
- but does not dramatically improve entropy coding efficiency.

---

## 8. Alignment as the Next Step

Manifold-aligned coordinates aim to:

- separate meaningful variation from redundancy,
- enable functional + residual decomposition,
- minimize rate for a given distortion.

This may be approximated by:

- PCA-like local tangent estimation,
- block-wise low-rank projections,
- learned axis transforms optimized for entropy or RD cost.

Rotation is therefore best viewed as:

> **a baseline normalization step, not the end goal.**

---

## 9. Positioning of Existing Methods

Under this framework:

- QuaRot / SpinQuant:
  - rotation-based flattening methods.
- SmoothQuant / GPTQ:
  - value-centric heuristics.
- Proposed approach:
  - **manifold-aware axis alignment**.

This positioning clarifies both the strengths
and limitations of existing techniques.

---

## Takeaway

> **Rotation removes coordinate pathologies;  
> alignment reveals structure.**

True compression of LLM operators requires moving
from flattening-based preprocessing
to manifold-aligned representations.

---
