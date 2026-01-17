# Axis-Transform and Manifold-Aligned Representation

## for Low-Bit LLM Quantization and Compression

---

## Abstract

Recent studies have demonstrated that large language models (LLMs) can be
aggressively quantized to INT8 or even INT4 with minimal degradation in
perplexity. This phenomenon is often attributed to heuristic outlier handling
or the presumed low information content of trained weights. In this paper, we
propose an alternative interpretation: LLM weights should be viewed as
parameterizations of high-dimensional nonlinear operators lying on
low-dimensional manifolds, and observed outliers are largely
coordinate-dependent artifacts arising from misaligned representations.

We reinterpret rotation-based quantization methods (e.g., QuaRot) as axis
transformations rather than frequency-domain operations, and argue that while
such rotations effectively flatten distributions, true compression gains arise
from manifold-aligned coordinate systems that induce coefficient concentration.
We support this view through (i) a controlled toy experiment comparing Fourier
and MLP representations of sinusoidal functions, and (ii) empirical analysis on
TinyLLaMA/SLM models demonstrating coordinate-relative outlier behavior and
entropy reduction under axis transformations. Our findings suggest a unifying
framework for LLM compression based on axis transformation, functional
representation, and rate–distortion optimization.

---

## 1. Introduction

Quantization and compression are critical for deploying large language models
under hardware, memory, and energy constraints. A growing body of work reports
that even uniform low-bit quantization—down to INT4—can preserve model quality
when combined with lightweight preprocessing such as outlier mitigation or
rotation. These results challenge the traditional assumption that large-magnitude
weights are intrinsically information-rich and must be carefully preserved.

Most existing approaches interpret weight quantization as a value-centric
problem, focusing on protecting outliers or minimizing local quantization error.
However, such interpretations struggle to explain why orthogonal rotations can
eliminate outliers without information loss, or why different coordinate systems
exhibit dramatically different quantization behavior.

In this work, we argue that these phenomena arise because LLM weights are
operator parameters rather than independent data points, and that compression
efficiency is governed primarily by the choice of coordinate system. We propose
a geometric framework in which trained weights lie on low-dimensional manifolds,
outliers are coordinate-relative projections of manifold curvature, and optimal
compression corresponds to aligning axes with the manifold’s tangent space.

---

## 2. Related Work

### 2.1 Outlier-Aware Quantization

Early work on LLM quantization emphasizes handling outliers via clipping,
group-wise scaling, or mixed-precision schemes. Methods such as SmoothQuant and
GPTQ implicitly assume that large-magnitude weights are semantically important
and require special treatment.

### 2.2 Rotation-Based Quantization

Recent approaches such as QuaRot, SpinQuant, and KurTail apply orthogonal
transformations to weight matrices prior to quantization. These methods
demonstrate that outliers can be suppressed without affecting inference
accuracy, enabling uniform low-bit quantization across layers.

However, these works largely treat rotation as a heuristic preprocessing step
and do not provide a geometric explanation for why such transformations are
effective.

### 2.3 Limitations of Existing Views

Prior work does not explicitly address:

- the operator-level interpretation of weights,
- the role of coordinate systems in inducing outliers,
- or the relationship between axis alignment and rate–distortion optimality.

---

## 3. Operator-Centric View of LLM Weights

An LLM implements a nonlinear operator of the form:

\[
y = f(x; W)
\]

where \(W\) denotes the collection of weight tensors. From this perspective,
weights are not data samples but coefficients parameterizing a function. The
goal of compression is therefore not to preserve individual values, but to
maintain the functional behavior of the operator.

This operator-centric view naturally suggests that different parameterizations
of the same function may admit vastly different compression properties, even if
they are functionally equivalent.

---

## 4. Manifold Hypothesis and Coordinate-Relative Outliers

### 4.1 Manifold Hypothesis for Weights

We hypothesize that trained LLM weights lie on a low-dimensional nonlinear
manifold \(\mathcal{M} \subset \mathbb{R}^N\). Locally, weights can be
approximated as:

\[
W \approx \mu + J \theta,
\quad \theta \in \mathbb{R}^d,\; d \ll N
\]

where \(J\) spans the tangent space of the manifold and \(\theta\) represents the
meaningful degrees of freedom.

### 4.2 Outliers as Projection Artifacts

In misaligned coordinate systems, curvature of the manifold projects unevenly
onto axes, producing heavy-tailed distributions and apparent outliers. Since
orthogonal transformations preserve L2 energy but alter L∞ and kurtosis,
outliers must be interpreted as coordinate-dependent statistics rather than
intrinsic information.

---

## 5. Axis Transformations and Functional Decomposition

### 5.1 Axis Transformations

Orthogonal transformations \(T\) applied to weights yield:

\[
W' = T W
\]

Hadamard-based transforms, commonly used in recent quantization work, are
instances of such axis transformations. They redistribute energy across
coordinates without loss of information.

### 5.2 From Flattening to Concentration

We distinguish three regimes:

1. Original coordinates: spiky, heavy-tailed distributions.
2. Random rotations: flattened distributions with reduced outliers.
3. Manifold-aligned coordinates: concentrated representations with low effective
   dimensionality.

While random rotations achieve regime (2), optimal compression requires regime
(3).

### 5.3 Functional + Residual Representation

Aligned coordinates enable a decomposition:

\[
w*i \approx f*\theta(i) + r_i
\]

where \(f\_\theta\) captures structured, low-dimensional variation and \(r_i\)
represents small residuals suitable for aggressive quantization and entropy
coding.

---

## 6. Toy Experiment: Fourier vs. MLP Representations

### 6.1 Setup

We construct signals as weighted sums of sinusoids with varying amplitudes,
frequencies, and phases. The same function is represented using:

1. A Fourier basis (ground-truth aligned basis).
2. A multilayer perceptron (MLP) with ReLU activations.

### 6.2 Results

The Fourier representation exhibits strong coefficient concentration and low
entropy, while the MLP representation distributes parameters across many
dimensions. Applying orthogonal rotations to MLP parameters reduces outliers but
does not recover Fourier-level concentration.

### 6.3 Implications

This experiment demonstrates that compression difficulty depends on the chosen
basis rather than the function itself, providing intuition for analogous
phenomena in LLMs.

---

## 7. Validation on TinyLLaMA and Small Language Models

### 7.1 Experimental Setup

We analyze weight matrices from TinyLLaMA and SLM checkpoints, focusing on FFN
and attention projection layers. We evaluate original coordinates, Hadamard
rotations, and optional PCA-aligned axes.

### 7.2 Metrics

We measure outlier statistics, histogram entropy, Huffman proxy bitrates, and
top-k energy ratios as proxies for concentration.

### 7.3 Observations

Hadamard rotations consistently reduce outlier metrics and entropy, confirming
their role as coordinate transformations. PCA-aligned axes further increase
energy concentration, consistent with the manifold-alignment hypothesis.

---

## 8. Discussion

Our results suggest a unifying interpretation of recent low-bit quantization
successes. Rotation-based methods flatten distributions but do not explicitly
align with the underlying manifold. True compression gains arise from axis
alignment that induces concentration, enabling functional decomposition and
asymmetric quantization.

This perspective reframes quantization as a reparameterization problem rather
than a value-clipping problem.

---

## 9. Conclusion

We presented a coordinate-centric framework for LLM weight quantization and
compression. By viewing weights as operator parameters lying on low-dimensional
manifolds, we reinterpret outliers as coordinate artifacts and rotation-based
methods as axis transformations. Our toy experiments and TinyLLaMA analyses
demonstrate that basis alignment governs coefficient concentration and
compression efficiency. This framework opens new directions for manifold-aware
LLM compression methods that go beyond heuristic outlier handling.

---

## Acknowledgements

(To be added.)

---

## References

(To be added.)
