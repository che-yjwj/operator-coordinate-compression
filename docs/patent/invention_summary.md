# Invention Summary

## Operator-Coordinate-Based Compression of Neural Network Parameters

---

## 1. Technical Field

The present invention relates to methods and systems for
compressing parameters of neural networks, particularly
large language models (LLMs), for efficient storage and inference
on hardware-constrained platforms.

More specifically, the invention concerns
**coordinate-system transformations and operator-aware representations**
for quantization and entropy coding of neural network parameters.

---

## 2. Background and Problem Statement

Modern LLMs contain billions of parameters and require
substantial memory bandwidth and storage.
To address this, various quantization and compression techniques
have been proposed.

Conventional approaches typically:

- treat weights as independent scalar values,
- identify large-magnitude values (outliers) as important,
- rely on clipping, mixed precision, or heuristic outlier handling.

Recent methods introduce orthogonal rotations (e.g., Hadamard)
to reduce outliers prior to quantization.
However, these methods lack a principled explanation of
why rotation works and do not fully exploit the structure
of trained models for compression.

There exists a need for a method that:

- explains outlier behavior fundamentally,
- enables aggressive low-bit quantization,
- improves entropy coding efficiency,
- and is grounded in the geometry of operator parameters.

---

## 3. Core Insight of the Invention

The core insight of the invention is that:

> **Neural network weights are parameters of nonlinear operators,
> and observed outliers are artifacts of misaligned coordinate systems,
> rather than intrinsic carriers of information.**

Accordingly:

- multiple parameterizations of the same operator are functionally equivalent,
- but differ significantly in compressibility.

The invention reframes compression as a problem of
**selecting coordinate systems aligned with the intrinsic structure
(manifold) of the operator parameter space**.

---

## 4. Summary of the Invention

The invention provides a method comprising:

1. Interpreting neural network parameters as operator coefficients
   rather than independent data values.
2. Applying one or more coordinate transformations to reparameterize
   the operator without changing its functional behavior.
3. Distinguishing between:
   - rotation-based flattening transformations, and
   - structure-aligned (manifold-aligned) coordinate systems.
4. Representing parameters as:
   - a structured functional component, and
   - a residual component suitable for low-bit quantization.
5. Compressing the reparameterized parameters using
   uniform or non-uniform quantization and entropy coding.

This approach enables improved rate–distortion trade-offs
compared to conventional value-centric methods.

---

## 5. Key Novel Elements

The invention introduces the following novel aspects:

### (a) Coordinate-Relative Interpretation of Outliers

Outliers are identified as coordinate-dependent projection artifacts
rather than semantically important values.

### (b) Operator-Centric Compression Framework

Weights are treated as operator parameters,
and compression is formulated as reparameterization.

### (c) Distinction Between Flattening and Alignment

Orthogonal rotations are recognized as flattening operations,
while manifold-aligned transformations enable coefficient concentration.

### (d) Functional + Residual Decomposition

Parameters are decomposed into structured functional components
and quantizable residuals.

### (e) Geometry-Aware Rate–Distortion Optimization

Coordinate transforms are selected to minimize coding rate
for a target operator distortion.

---

## 6. Exemplary Embodiments

### Embodiment 1: Blockwise Axis Alignment

- Partition weight tensors into blocks or channels.
- Estimate local parameter covariance.
- Align coordinate axes with dominant variation directions.
- Quantize aligned coefficients uniformly.

### Embodiment 2: Rotation + Alignment Pipeline

- Apply an orthogonal rotation to remove coordinate artifacts.
- Apply a secondary alignment transform to concentrate coefficients.
- Perform low-bit quantization and entropy coding.

### Embodiment 3: Learned Axis Transform

- Optimize a parameterized transform jointly with
  a rate–distortion objective.
- Store the transform metadata alongside compressed parameters.

---

## 7. Advantages Over Prior Art

Compared to existing techniques, the invention:

- eliminates the need for explicit outlier protection,
- supports uniform low-bit quantization across layers,
- improves entropy coding efficiency,
- provides a principled geometric interpretation,
- and generalizes across architectures and model sizes.

---

## 8. Industrial Applicability

The invention is applicable to:

- on-device LLM inference,
- NPUs and AI accelerators,
- memory-constrained embedded systems,
- model distribution and deployment pipelines.

---

## 9. Conceptual Claim Direction (Non-Limiting)

- A method for compressing neural network parameters
  using coordinate transformations aligned with operator structure.
- A system for representing neural network weights
  as functional components plus residuals.
- A computer-readable medium storing instructions
  for performing the above methods.

---

## 10. Summary Statement

> **By recognizing neural network weights as operator parameters
> and outliers as coordinate artifacts,
> the invention enables a fundamentally new class
> of compression techniques based on geometric alignment
> rather than heuristic value handling.**

---
