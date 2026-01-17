# 토이 실험 스펙 — Fourier Basis vs. MLP Basis

## 압축 난이도의 좌표계 의존성

---

## 0. 목적(Purpose)

본 실험의 목적은 다음 가설을 **가장 단순하고 통제된 환경**에서 검증하는 것이다.

> **같은 함수라도 어떤 좌표계(기저)로 계수를 표현하느냐에 따라  
> 계수 분포(집중/분산), outlier 통계, 엔트로피, 양자화 RD 특성이 달라진다.**

이는 LLM 압축에서 “아웃라이어(outlier)는 좌표계 상대적”이라는 주장을 직관적으로 입증하기 위한
토이 수준(toy-level) 실험이다.

---

## 1. 대상 함수(Target Function)

### 1.1 신호 정의(Signal Definition)

연속 함수 \( f(t) \)를 다음과 같이 정의한다.

\[
f(t) = \sum\_{k=1}^{K} a_k \sin(2\pi f_k t + \phi_k)
\]

- \( t \in [0, 1] \)
- \(K\): 사용 주파수 개수

### 1.2 파라미터 분포(Parameter Distribution)

- 주파수 \(f_k\):
  - 저주파: 1–20
  - 고주파(선택): 40–200
- 위상(phase) \(\phi_k \sim \mathcal{U}(0, 2\pi)\)
- 진폭 \(a_k\):
  - **Case A (아웃라이어 유사)**:
    - \(a_1 = 1.0\)
    - \(a\_{2..K} = 0.01\)
  - **Case B (로그 균일, Log-uniform)**:
    - \(a_k \sim \text{LogUniform}(10^{-2}, 1)\)

### 1.3 샘플링(Sampling)

- 샘플 수: \(N = 4096\)
- 학습/테스트 분할: 70% / 30%

---

## 2. 비교할 표현(Representations)

### 2.1 Fourier 기저 표현(정렬된 기저)

#### F0: 정답(Ground-truth) 파라미터

- \(\{a_k, f_k, \phi_k\}\)를 직접 저장
- _이상적인 정렬 좌표계_로 간주

#### F1: FFT Top-K 근사

- 샘플된 신호에서 스펙트럼을 추정
- 상위 K개 주파수 성분만 유지
- 선택된 성분으로 신호 재구성

> 목적: 같은 함수를 구조에 정렬된 좌표계로 표현한다.

---

### 2.2 MLP 표현(정렬되지 않은 기저)

#### 모델 구조(Architecture)

- 입력: 스칼라 \(t\)
- 출력: 스칼라 \(\hat f(t)\)
- 레이어 수: 2–4
- 폭(width): 64 / 128 / 256
- 활성함수: ReLU (어블레이션으로 tanh 선택)

\[
\hat f(t) = \sum_i v_i \sigma(w_i t + b_i)
\]

#### 학습(Training)

- 손실: 평균제곱오차(MSE)
- 옵티마이저: Adam
- 학습률: 1e-3
- 에폭: 수렴까지 또는 고정 예산(예: 5000)

#### 파라미터 벡터(Parameter Vector)

- \(\theta\_{\text{MLP}} = \text{flatten}(W, b)\)

---

## 3. 왜곡 지표(Distortion Metric)

### 3.1 주 왜곡(Primary Distortion)

\[
D = \mathbb{E}\_{t \sim \text{test}}[(f(t) - \hat f(t))^2]
\]

- 테스트 셋에서 계산

### 3.2 목표 왜곡 수준(RD 곡선용)

\[
D \in \{10^{-2}, 5 \times 10^{-3}, 10^{-3}, 5 \times 10^{-4}, 10^{-4}\}
\]

- Fourier: K 조절
- MLP: width / epochs 조절

---

## 4. 레이트(비트레이트) 추정

> 실제 bitstream 구현 대신 **엔트로피 기반 대리값(entropy-based proxy)** 사용

### 4.1 양자화(Quantization)

- 대칭 균일 양자화(symmetric uniform quantization)
- 비트폭: {2, 3, 4, 6, 8}
- 스케일(scale):
  - Fourier: 텐서 단위(per-tensor)
  - MLP: 텐서 단위 또는 그룹 단위(per-group, group size = 64)

### 4.2 엔트로피 추정(Entropy Estimation)

- 히스토그램 기반 샤논 엔트로피(Shannon entropy):
  \[
  H = -\sum_s p(s)\log_2 p(s)
  \]
- Huffman proxy:
  - 히스토그램에서 평균 코드 길이 추정

### 4.3 비트레이트 대리값(Bitrate Proxy)

\[
R = (\text{avg bits per symbol}) \times (\#\text{symbols})
\]

---

## 5. 분포 및 아웃라이어 지표

각 계수 벡터 \(\theta\)에 대해 다음을 계산한다.

- **OS1**: \(\|\theta\|\_\infty / \|\theta\|\_2\)
- **OS2**: 비율(fraction)(|θ_i| > 3σ)
- **첨도(kurtosis)**
- **Top-k 에너지 비율(Top-k Energy Ratio)**:
  \[
  \frac{\sum*{i=1}^{k} \theta*{(i)}^2}{\|\theta\|\_2^2}
  \]
- **희소성 대리값(Sparsity Proxy)**:
  - 비율(fraction)(|θ_i| < τ), 작은 τ (예: 1e-3)

---

## 6. 축 변환 어블레이션(Axis Transformation Ablation) (선택, 권장)

### 6.1 MLP 파라미터에 Hadamard 회전 적용

\[
\theta' = \frac{1}{\sqrt{N}} H_N \theta
\]

- 전/후 비교:
  - OS1, 엔트로피, top-k 에너지

### 6.2 기대 결과

- 아웃라이어 지표 ↓
- 엔트로피 ↓
- **집중(concentration)의 뚜렷한 증가 없음**

이는 회전이 _정렬(alignment)_이 아니라 _평탄화(flattening)_를 만든다는 점을 보여줍니다.

---

## 7. 실험 프로토콜(Experimental Protocol)

1. 대상 신호 \(f(t)\) 생성
2. Fourier 표현(F0, F1) 적합(fit)
3. 목표 왜곡 수준에 맞게 MLP 학습
4. 파라미터 벡터 추출
5. 각 비트폭에서 파라미터 양자화
6. 엔트로피 및 비트레이트 대리값 추정
7. 양자화 이후 왜곡 계산
8. 플롯 생성:
   - 신호 vs 근사
   - 파라미터 히스토그램
   - RD 곡선
   - 아웃라이어/집중 지표

---

## 8. 생성할 그림(Figures)

- **Fig.1**: 대상 신호 vs MLP 근사
- **Fig.2**: 파라미터 히스토그램(Fourier vs MLP)
- **Fig.3**: RD 곡선(R vs D)
- **Fig.4**: 회전 전/후 아웃라이어 지표
- **Fig.5**: top-k 에너지 비교

---

## 9. 예상 결론(Expected Conclusions)

- Fourier 기저는 계수가 집중되고 엔트로피가 낮다
- MLP 기저는 같은 함수를 표현해도 파라미터가 분산된다
- 회전은 아웃라이어를 줄이지만 구조적 집중을 복원하지 못한다
- 압축 난이도는 함수가 아니라 **좌표계(표현)**의 성질이다

---

## 10. LLM 압축과의 연결

이 토이 실험은 LLM 가중치 압축의 통제된 아날로그(analogue)로 작동합니다.

- Fourier basis(푸리에 기저) ↔ 매니폴드-정렬 축
- MLP 가중치 ↔ 원래 LLM 좌표계
- Hadamard 회전 ↔ QuaRot 스타일 전처리

이는 실용적 LLM 압축을 위해 매니폴드-인식 축 정렬이 필요함을 동기 부여합니다.

---
