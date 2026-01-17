# Prior Art Mapping

## Differentiation from Existing LLM Quantization and Compression Methods

---

## 1. Purpose of This Document

This document systematically compares the proposed
**operator-coordinate-based compression framework**
against representative prior art in LLM quantization and compression.

The goal is to:

- identify conceptual gaps in existing methods,
- clarify the novelty of the proposed approach,
- support patentability and academic differentiation.

---

## 2. Categories of Prior Art

Existing methods can be broadly classified into the following categories:

1. Value-centric quantization methods
2. Outlier-aware heuristics
3. Rotation-based preprocessing
4. Low-rank and structural approximations

Each category is analyzed below.

---

## 3. Value-Centric Quantization Methods

### Representative Works

- GPTQ
- SmoothQuant
- Post-training uniform quantization schemes

### Core Assumptions

- Weights are treated as independent scalar values.
- Magnitude correlates with importance.
- Quantization error should be minimized per-value or per-group.

### Limitations

- Do not explain why large portions of weights can be quantized coarsely.
- Rely on heuristics (clipping thresholds, group sizes).
- Do not address coordinate dependence.

### Differentiation

| Aspect                | Prior Art           | Proposed Method               |
| --------------------- | ------------------- | ----------------------------- |
| Weight interpretation | Data values         | Operator parameters           |
| Outlier handling      | Explicit protection | Coordinate reparameterization |
| Theoretical basis     | Error minimization  |
