# Toy Experiment Spec — Fourier Basis vs. MLP Basis

## Coordinate Dependence of Compression Difficulty

---

## 0. Purpose

본 실험의 목적은 다음 가설을 **가장 단순하고 통제된 환경**에서 검증하는 것이다.

> **같은 함수라도 어떤 좌표계(기저)로 계수를 표현하느냐에 따라  
> 계수 분포(집중/분산), outlier 통계, 엔트로피, 양자화 RD 특성이 달라진다.**

이는 LLM 압축에서 “outlier는 좌표계 상대적”이라는 주장을 직관적으로 입증하기 위한
toy-level 실험이다.

---

## 1. Target Function

### 1.1 Signal Definition

연속 함수 \( f(t) \)를 다음과 같이 정의한다.

\[
f(t) = \sum\_{k=1}^{K} a_k \sin(2\pi f_k t + \phi_k)
\]

- \( t \in [0, 1] \)
- \(K\): 사용 주파수 개수

### 1.2 Parameter Distribution

- Frequency \(f_k\):
  - low: 1–20
  - high (optional): 40–200
- Phase \(\phi_k \sim \mathcal{U}(0, 2\pi)\)
- Amplitude \(a_k\):
  - **Case A (Outlier-like)**:
    - \(a_1 = 1.0\)
    - \(a\_{2..K} = 0.01\)
  - **Case B (Log-uniform)**:
    - \(a_k \sim \text{LogUniform}(10^{-2}, 1)\)

### 1.3 Sampling

- Number of samples: \(N = 4096\)
- Train/Test split: 70% / 30%

---

## 2. Representations Compared

### 2.1 Fourier Basis Representation (Aligned Basis)

#### F0: Ground-Truth Parameters

- Store \(\{a_k, f_k, \phi_k\}\) directly
- Considered the _ideal aligned coordinate system_

#### F1: FFT Top-K Approximation

- Estimate spectrum from sampled signal
- Keep top-K frequency components
- Reconstruct signal from selected components

> Purpose: represent the same function in a coordinate system aligned with its structure.

---

### 2.2 MLP Representation (Misaligned Basis)

#### Architecture

- Input: scalar \(t\)
- Output: scalar \(\hat f(t)\)
- Layers: 2–4
- Width: 64 / 128 / 256
- Activation: ReLU (tanh optional for ablation)

\[
\hat f(t) = \sum_i v_i \sigma(w_i t + b_i)
\]

#### Training

- Loss: Mean Squared Error (MSE)
- Optimizer: Adam
- Learning rate: 1e-3
- Epochs: until convergence or fixed budget (e.g., 5000)

#### Parameter Vector

- \(\theta\_{\text{MLP}} = \text{flatten}(W, b)\)

---

## 3. Distortion Metric

### 3.1 Primary Distortion

\[
D = \mathbb{E}\_{t \sim \text{test}}[(f(t) - \hat f(t))^2]
\]

- Computed on test set

### 3.2 Target Distortion Levels (for RD curves)

\[
D \in \{10^{-2}, 5 \times 10^{-3}, 10^{-3}, 5 \times 10^{-4}, 10^{-4}\}
\]

- Fourier: adjust K
- MLP: adjust width / epochs

---

## 4. Rate (Bitrate) Estimation

> 실제 bitstream 구현 대신 **entropy-based proxy** 사용

### 4.1 Quantization

- Symmetric uniform quantization
- Bit-width: {2, 3, 4, 6, 8}
- Scale:
  - Fourier: per-tensor
  - MLP: per-tensor or per-group (group size = 64)

### 4.2 Entropy Estimation

- Histogram-based Shannon entropy:
  \[
  H = -\sum_s p(s)\log_2 p(s)
  \]
- Huffman proxy:
  - Average code length from histogram

### 4.3 Bitrate Proxy

\[
R = (\text{avg bits per symbol}) \times (\#\text{symbols})
\]

---

## 5. Distribution & Outlier Metrics

각 계수 벡터 \(\theta\)에 대해 다음을 계산한다.

- **OS1**: \(\|\theta\|\_\infty / \|\theta\|\_2\)
- **OS2**: fraction(|θ_i| > 3σ)
- **Kurtosis**
- **Top-k Energy Ratio**:
  \[
  \frac{\sum*{i=1}^{k} \theta*{(i)}^2}{\|\theta\|\_2^2}
  \]
- **Sparsity Proxy**:
  - fraction(|θ_i| < τ), τ small (e.g., 1e-3)

---

## 6. Axis Transformation Ablation (Optional but Recommended)

### 6.1 Hadamard Rotation on MLP Parameters

\[
\theta' = \frac{1}{\sqrt{N}} H_N \theta
\]

- Compare before/after:
  - OS1, entropy, top-k energy

### 6.2 Expected Outcome

- Outlier metrics ↓
- Entropy ↓
- **No strong increase in concentration**

This demonstrates that rotation yields _flattening_, not _alignment_.

---

## 7. Experimental Protocol

1. Generate target signal \(f(t)\)
2. Fit Fourier representations (F0, F1)
3. Train MLP to target distortion levels
4. Extract parameter vectors
5. Quantize parameters at each bit-width
6. Estimate entropy and bitrate
7. Compute distortion after quantization
8. Plot:
   - Signal vs approximation
   - Parameter histograms
   - RD curves
   - Outlier/concentration metrics

---

## 8. Figures to Produce

- **Fig.1**: Target signal vs MLP approximation
- **Fig.2**: Parameter histograms (Fourier vs MLP)
- **Fig.3**: RD curves (R vs D)
- **Fig.4**: Outlier metrics before/after rotation
- **Fig.5**: Top-k energy comparison

---

## 9. Expected Conclusions

- Fourier basis yields concentrated coefficients and low entropy
- MLP basis spreads parameters despite representing the same function
- Rotation removes outliers but does not recover structural concentration
- Compression difficulty is a property of the **coordinate system**, not the function

---

## 10. Connection to LLM Compression

This toy experiment serves as a controlled analogue of LLM weight compression:

- Fourier basis ↔ manifold-aligned axes
- MLP weights ↔ original LLM coordinates
- Hadamard rotation ↔ QuaRot-style preprocessing

It motivates manifold-aware axis alignment for practical LLM compression.

---
