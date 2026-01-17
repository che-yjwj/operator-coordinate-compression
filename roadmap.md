# Research Roadmap — Operator Coordinate Compression

This document defines the research roadmap for the
**operator-coordinate-compression** project.

The goal is to systematically reframe LLM quantization and compression
as a **coordinate-system and operator reparameterization problem**,
and to validate this perspective through theory, toy experiments,
and real-model analysis.

---

## 1. Core Research Thesis

> **LLM compression is not fundamentally about removing information,  
> but about choosing coordinate systems that align with the structure
> of the underlying operator manifold.**

Key claims:

1. LLM weights parameterize nonlinear operators.
2. Trained weights lie on low-dimensional nonlinear manifolds.
3. Observed outliers are coordinate-relative artifacts.
4. Rotation-based methods flatten distributions but do not induce concentration.
5. Manifold-aligned axes enable concentrated, function-like representations.
6. Such representations admit superior rate–distortion trade-offs.

---

## 2. Document Hierarchy and Roles

This repository is organized around the following document layers:

### 2.1 Conceptual Overview

- `README.md`
- `docs/overview.md`

Purpose:

- Communicate the high-level idea
- Explain _why_ coordinate systems matter
- Provide intuition without heavy math

---

### 2.2 Theoretical Foundations

- `docs/theory/operator_view.md`
- `docs/theory/manifold_hypothesis.md`
- `docs/theory/coordinate_relative_outliers.md`
- `docs/theory/rotation_vs_alignment.md`

Purpose:

- Formalize the operator-centric view
- Define manifold assumptions
- Reinterpret outliers and rotations geometrically

---

### 2.3 Academic Paper Draft

- `docs/paper/paper_draft.md`

Purpose:

- Integrate theory + experiments into a publishable narrative
- Serve as the canonical reference for external communication

---

### 2.4 Experimental Specifications (No Code Yet)

- `experiments/toy_basis_vs_mlp/spec.md`
- (future) `experiments/tinyllama_validation/spec.md`

Purpose:

- Fully specify experiments before implementation
- Ensure reproducibility and conceptual clarity
- Separate _what_ is tested from _how_ it is coded

---

### 2.5 Patent-Oriented Documents

- `docs/patent/invention_summary.md`
- `docs/patent/claim_candidates.md`

Purpose:

- Translate research ideas into protectable claims
- Identify novel elements vs. prior art

---

## 3. Research Phases

### Phase 0 — Concept Fixation (Current Phase) ✅

Status: **In progress**

Goals:

- Lock down conceptual framing
- Establish shared vocabulary:
  - operator vs. value
  - axis vs. frequency
  - flattening vs. concentration
- Complete all core documentation

Deliverables:

- README
- Overview
- Theory notes
- Paper draft
- Toy experiment spec

---

### Phase 1 — Controlled Validation (Toy Experiments)

Status: Planned

Goals:

- Demonstrate coordinate-dependent compression in the simplest setting
- Provide visual and quantitative intuition

Key Questions:

- How does basis choice affect coefficient entropy?
- Why does rotation remove outliers but not induce sparsity?
- How does RD behavior differ across representations?

---

### Phase 2 — Real Model Analysis (TinyLLaMA / SLM)

Status: Planned

Goals:

- Validate coordinate-relative outlier behavior in real LLMs
- Confirm that toy-level phenomena appear in practice

Key Questions:

- Do outlier statistics change under axis transforms?
- Does random rotation increase flatness but not concentration?
- Can approximate manifold alignment improve compression metrics?

---

### Phase 3 — Manifold-Aligned Compression Methods

Status: Exploratory

Goals:

- Design practical axis-alignment approximations
- Explore function + residual decomposition in LLM weights

Candidate Techniques:

- Block-wise PCA / tangent estimation
- Clustering + shared bases
- Learned axis transforms optimized for entropy or sparsity

---

### Phase 4 — Publication and Protection

Status: Future

Goals:

- Submit academic paper
- File patent(s)
- Open-source selected experimental code

---

## 4. Evaluation Criteria

Across all phases, success is measured by:

- Conceptual clarity (can the idea be explained simply?)
- Reproducibility (are specs independent of implementation?)
- Rate–distortion improvements
- Minimal reliance on heuristics
- Clear differentiation from prior work

---

## 5. Guiding Principles

1. **Document before code**
2. **Explain before optimize**
3. **Toy before LLM**
4. **Geometry before heuristics**
5. **Representation before quantization**

---

## 6. One-Line North Star

> _“Outliers disappear when the coordinate system is right.”_

---

## 7. Next Documentation Targets

Recommended next documents to write:

1. `docs/theory/operator_view.md`
2. `docs/theory/coordinate_relative_outliers.md`
3. `docs/theory/rotation_vs_alignment.md`
4. `experiments/tinyllama_validation/spec.md`
5. `docs/patent/invention_summary.md`

---

End of roadmap.
