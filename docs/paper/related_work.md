# Related Work

This section positions our work within existing research on
LLM quantization, rotation-based preprocessing, and geometric
interpretations of neural network parameters.

---

## Outlier-Aware Quantization

Early LLM quantization methods focus on identifying and protecting
outlier weights via clipping, mixed precision, or group-wise scaling.
Representative examples include SmoothQuant, GPTQ, and related
post-training quantization techniques.

These approaches implicitly assume that large-magnitude weights
carry disproportionately important information, an assumption
we challenge in this work.

---

## Rotation-Based Preprocessing

Recent methods such as QuaRot, SpinQuant, and KurTail demonstrate
that applying orthogonal transformations (e.g., Hadamard rotations)
before quantization can significantly reduce outliers and enable
uniform low-bit quantization.

While empirically effective, these methods do not provide a clear
geometric explanation for _why_ rotation works, nor do they address
whether rotation is optimal for compression.

Our work reinterprets these methods as axis transformations that
remove coordinate artifacts but do not induce structural alignment.

---

## Low-Rank and Structural Compression

Low-rank approximations, pruning, and tensor decomposition methods
implicitly exploit redundancy in neural networks.
These approaches suggest that trained models occupy a restricted
region of parameter space.

However, most prior work remains value-centric and does not
explicitly frame compression as a coordinate selection problem.

---

## Manifold Perspectives

The manifold hypothesis has been widely discussed for activations
and representations, but much less so for _operator parameters_.
Our work extends manifold reasoning to LLM weights and connects it
directly to quantization and entropy coding.

---

## Summary

Unlike prior work, we:

- treat weights as operator parameters rather than data,
- reinterpret outliers as coordinate-relative artifacts,
- distinguish rotation-induced flattening from manifold alignment,
- and unify quantization and compression under a geometric framework.
