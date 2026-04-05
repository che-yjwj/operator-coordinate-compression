# 문서 리뷰 및 새로운 연구 방향 제안

**작성일:** 2026-04-05  
**문서 목적:** 현재 리포지토리 문서들의 강점·약점 분석 및 후속 연구 방향 제시

---

## 1. 현재 문서 리뷰

### 1.1 개념 프레임워크의 강점

현재 프레임워크는 다음 측면에서 명확하고 일관된 주장을 가지고 있다.

**[강점 A] 핵심 재프레이밍의 명확성**

"아웃라이어는 좌표계 인공물"이라는 핵심 주장은 단순하면서도 강력하다.  
직교 변환이 L∞와 첨도는 바꾸지만 L2와 모델 동작을 바꾸지 않는다는 관찰은
기존 값 중심(value-centric) 해석을 실증적으로 반박하는 가장 강한 논거다.

**[강점 B] Flattening vs. Concentration 구분**

이 구분은 현재 문헌에서 명시적으로 다루어지지 않으며, 논문의 핵심 기여가 될 수 있다.  
QuaRot/SpinQuant가 왜 양자화 안정성은 높이지만 엔트로피 코딩 효율은 크게 개선하지 못하는지를
정확하게 설명한다.

**[강점 C] TurboQuant 분석의 엄밀성**

사실(FACT)/추론(INFERENCE)/계획(PLAN) 구분이 잘 유지되어 있으며,
TurboQuant를 "manifold-aligned baseline의 upper bound"가 아닌
"random-rotation baseline"으로 정확하게 위치시켰다.

---

### 1.2 개념적 약점과 열린 질문

**[약점 A] 매니폴드 가설의 미명세(underspecification)**

현재 문서는 "학습된 가중치가 저차원 매니폴드 근처에 놓인다"고 가정하지만,
다음이 명확하지 않다.

- 매니폴드의 내재 차원 d는 얼마인가? 레이어별로 다른가?
- 매니폴드의 곡률(curvature)은 어느 정도인가?
- 어떤 의미에서 "근처"인가? (e-neighborhood? 법선 방향 분산?)
- 이 가설이 기존 flat minima 가설, 저랭크 가설과 어떻게 다른가?

매니폴드 가설을 empirical하게 측정하지 않으면,
"저랭크 분해(LoRA, ASVD)와 무엇이 다른가?"라는 반론을 막기 어렵다.

**[약점 B] 가중치 매니폴드와 활성값(activation) 매니폴드의 혼용 위험**

현재 프레임워크는 가중치(weight) 공간의 매니폴드를 다루지만,
양자화 오차는 **활성값(activation) 공간**에서 측정된다.

SmoothQuant의 핵심 인사이트는 가중치-활성값의 **공동(joint) 스케일 조정**에 있다.  
현재 프레임워크는 이 상호작용을 명시적으로 다루지 않아,
"왜 가중치 공간에서의 정렬이 활성값 공간의 오차를 줄이는가?"에 대한 설명이 부족하다.

**[약점 C] SpinQuant과의 관계가 불완전**

SpinQuant는 **학습된(learned) 직교 회전**을 사용하며, 랜덤 회전보다 나은 성능을 보인다.  
이는 현재 프레임워크에서 중요한 질문을 제기한다.

- SpinQuant의 학습된 회전은 manifold alignment를 부분적으로 달성하는가?
- 만약 그렇다면, 왜 완전한 manifold alignment보다 못한가?
- 만약 아니라면, flattening만으로도 SpinQuant의 성능을 설명할 수 있는가?

이 질문에 답하지 못하면 "SpinQuant이 이미 당신이 제안하는 것을 하는 것 아닌가?"라는 반론에 취약하다.

**[약점 D] 레이트-왜곡 주장의 비형식성**

"매니폴드-정렬 좌표계가 더 나은 RD 트레이드오프를 제공한다"는 주장이
수식 수준에서는 제시되지 않는다.  
Shannon의 rate-distortion theorem이나 Kolmogorov complexity 맥락에서
이 주장이 어떻게 형식화되는지 명확해야 한다.

**[약점 E] 엔트로피 코딩의 구체성 부재**

"매니폴드 정렬 → 엔트로피 감소 → 더 나은 압축"이라는 경로가
어떤 **구체적인 코딩 전략**으로 이어지는지 명시되지 않는다.

- Huffman coding? Arithmetic coding? Asymmetric Numeral Systems (ANS)?
- Vector quantization (VQ)과 어떻게 연결되는가?
- 잔차 r의 분포가 어떤 코더에 유리한가?

---

## 2. 새로운 연구 방향 제안

현재 프레임워크를 발전시킬 수 있는 6가지 방향을 제안한다.  
각 방향은 독립적으로 연구될 수 있으나, 조합하면 더욱 강한 기여를 만든다.

---

### 방향 1: 가중치-활성값 공동 매니폴드(Joint Weight-Activation Manifold)

#### 동기

가중치 공간에서의 좌표 정렬이 **활성값 공간**에서의 압축 오차를 줄이는 이유를 설명해야 한다.

현재 관점:
```
W → 좌표 정렬 → 압축된 W̃ → 양자화 오차 ε
```

더 완전한 관점:
```
(W, x) → 공동 좌표 정렬 → 활성값 오차 f(x;W̃) - f(x;W) 최소화
```

#### 핵심 아이디어

가중치 행렬 W와 그것이 처리할 활성값 X를 함께 고려하면,
최적 좌표 변환 T\*는 다음을 최소화한다.

```
T* = argmin_T E_x[ || f(x; Q(TW)) - f(x; W) ||² ]
```

이는 SmoothQuant의 채널별 스케일링을 기하학적으로 일반화한 것이다.

#### 기대 기여

- "왜 가중치 정렬이 추론 품질을 보존하는가?"에 대한 공식적 설명
- SmoothQuant, LLM.int8()과의 통합 프레임워크
- 활성값 분포를 calibration 없이 추정하는 방법 탐색

---

### 방향 2: Fisher 정보 기하 기반 좌표계(Fisher-Information-Guided Coordinates)

#### 동기

최적 좌표계는 기하학적(PCA-like) 정렬만이 아니라,
**손실 함수의 민감도 구조**에도 정렬되어야 한다.

#### 핵심 아이디어

Fisher Information Matrix(FIM)의 고유벡터는 파라미터 공간에서
"손실이 가장 빠르게 변하는 방향"을 정의한다.

```
F = E_x[ (∂ log p(x;W) / ∂W) (∂ log p(x;W) / ∂W)ᵀ ]
```

FIM-정렬 좌표계에서는:
- 고유값이 큰 방향(고민감도) → 더 많은 비트 할당
- 고유값이 작은 방향(저민감도) → 공격적 압축 가능

이는 PCA(분산 기반)와 근본적으로 다르다.  
PCA는 "어디에 에너지가 많은가?"를 보고,  
Fisher 기반은 "어디가 민감한가?"를 본다.

#### 기존 연구와의 관계

- K-FAC, EKFAC: Fisher 행렬의 효율적 근사
- Optimal Brain Surgeon (OBS), GPTQ: Hessian-guided quantization
- **차이점:** 위 방법들은 "어느 파라미터를 제거할까"에 FIM을 쓰지만,
  본 방향은 "어떤 좌표계로 표현할까"에 FIM을 쓴다.

#### 기대 기여

- 민감도-정렬(sensitivity-aligned) 좌표계가 PCA-정렬보다 나은 RD 곡선을 보임을 검증
- Hessian 기반 방법들의 통합 설명
- KFAC 근사를 이용한 실용적 구현 경로

---

### 방향 3: 레이어별·헤드별 매니폴드 다양성 분석(Layerwise Manifold Diversity)

#### 동기

현재 프레임워크는 매니폴드 가설을 레이어 전반에 균일하게 적용하지만,
실제로는 레이어별, 헤드별 압축 특성이 매우 다르다.

#### 핵심 질문

1. 내재 차원 d는 레이어마다 얼마나 다른가?
2. 초기 레이어 vs. 후기 레이어, attention vs. FFN의 매니폴드 구조는 얼마나 다른가?
3. 어느 레이어/헤드에서 manifold alignment의 이득이 가장 큰가?
4. 압축하기 쉬운 레이어와 어려운 레이어의 매니폴드 특성 차이는 무엇인가?

#### 측정 방법 제안

레이어별로 다음을 측정한다.
- 가중치 행렬의 effective rank (참여율 기반)
- PCA 후 top-k 에너지 비율 (집중도 지표)
- 랜덤 회전 전후 집중도 변화 (flattening만인지 concentration도인지)
- Hessian 최대 고유값 vs. 분포 분산의 상관관계

#### 기대 기여

- "레이어별 맞춤형(layer-adaptive) 압축 예산" 결정의 이론적 근거
- 어느 레이어에 먼저 manifold alignment를 적용할지 우선순위 결정
- 모델 크기·아키텍처별 매니폴드 구조 비교 (TinyLLaMA vs. Llama-3 vs. Gemma)

---

### 방향 4: SpinQuant를 부분 매니폴드 정렬로 이론화(SpinQuant as Partial Manifold Alignment)

#### 동기

SpinQuant가 랜덤 회전보다 좋고 완전한 매니폴드 정렬보다 나쁘다면,  
SpinQuant는 **부분적 매니폴드 정렬**을 달성하는 것으로 해석될 수 있다.

이 해석이 옳다면:
1. SpinQuant의 성능 상한이 무엇인지 설명할 수 있다.
2. SpinQuant가 학습하는 회전이 매니폴드 접공간의 어느 부분을 포착하는지 분석할 수 있다.
3. 더 나은 정렬 방법이 어떤 추가 정보를 활용해야 하는지 명확해진다.

#### 핵심 실험

```
Random Rotation → SpinQuant (Learned Rotation) → PCA-Alignment → Fisher-Alignment
   (baseline)        (partial alignment?)          (geometric)     (sensitivity-based)
```

위 4단계에서 flattening 지표와 concentration 지표를 각각 측정하면,  
SpinQuant가 flattening만 하는지, concentration도 부분적으로 달성하는지 확인할 수 있다.

#### 기대 기여

- SpinQuant 성능을 현 프레임워크에서 정확히 설명
- "학습된 회전"의 이론적 한계 규명
- Manifold alignment의 추가 이득을 정량화하는 실험 프로토콜

---

### 방향 5: KV Cache를 위한 온라인 매니폴드 추적(Online Manifold Tracking for KV Cache)

#### 동기

정적(static) 가중치 압축과 달리, KV cache 압축은 **동적(dynamic)** 문제다.  
각 생성 스텝에서 새로운 키(K)·값(V) 벡터가 추가되며,  
이 벡터들이 이루는 매니폴드는 입력 시퀀스에 따라 변한다.

TurboQuant는 data-oblivious random rotation을 쓰지만,  
실제 KV 벡터들은 특정 주제의 텍스트를 처리할 때 저차원 구조를 가질 가능성이 있다.

#### 핵심 아이디어

다음을 온라인으로 유지한다.
```
- 현재 KV 벡터들의 running PCA (또는 스트리밍 SVD)
- 새 벡터가 추가될 때 좌표계를 점진적으로 업데이트
- 오래된 토큰은 업데이트된 좌표계에서 재압축
```

이는 Streaming PCA 알고리즘과 연결되며,  
"long-context에서 KV cache의 effective rank가 감소한다"는 관찰을 활용할 수 있다.

#### TurboQuant와의 차별점

| 구분 | TurboQuant | 온라인 매니폴드 추적 |
|------|------------|---------------------|
| 좌표계 | 고정 랜덤 회전 | 입력 적응형 (online) |
| 정렬 | 없음 (flattening만) | 현재 KV 분포에 정렬 |
| 계산 비용 | O(d²) 회전 | O(d·rank) 업데이트 |
| Long-context 이득 | 고정 | 시퀀스가 길수록 정렬 개선 |

#### 기대 기여

- Long-context에서 TurboQuant 대비 추가 압축 이득 실증
- Streaming KV 압축의 새로운 방향 제시
- "입력-적응형(input-adaptive) 좌표계"를 KV cache에서 최초 구현

---

### 방향 6: 함수적 분해의 엔트로피 코딩 최적화(Entropy Coding for Functional Decomposition)

#### 동기

현재 프레임워크는 `W ≈ f_θ(i) + r_i` 분해를 제안하지만,  
이 분해를 **어떻게 효율적으로 코딩할 것인가**가 구체화되지 않았다.

압축 이득은 결국 코딩 효율에서 나온다.

#### 핵심 아이디어

**구조 성분 f_θ(i):** 낮은 복잡도 함수(예: 다항식, 주기 함수, 저차 신경망)로 표현  
→ 파라미터 수가 적어 높은 압축률

**잔차 성분 r_i:** 분포가 단순(가우시안, 라플라시안)해지도록 좌표 정렬  
→ ANS(Asymmetric Numeral Systems) 또는 Arithmetic Coding에 유리

**분해 방식에 따른 코더 선택:**
```
f_θ 성분: 함수 파라미터 (θ)만 저장 → 임의 압축 가능
r_i 성분: 분포-적응 엔트로피 코딩 → 엔트로피 한계에 근접
```

#### 기존 방법과의 관계

- Delta compression: 이전 값과의 차이를 코딩 → r_i와 유사하지만 구조 성분 없음
- Huffman/ANS: 분포를 알 때 효율적 → 잔차의 분포 가정이 중요
- VQ (Vector Quantization): 코드북 기반 → 집중된 좌표계에서 더 효율적

#### 기대 기여

- 함수적 분해 + 엔트로피 코딩의 end-to-end 파이프라인 설계
- "어떤 함수 클래스 f_θ가 LLM 가중치의 구조적 성분을 가장 잘 포착하는가?" 탐색
- 잔차의 분포 가정과 최적 코더 매칭 이론

---

## 3. 방향별 우선순위 및 단계적 접근

### 즉시 실행 가능 (Phase 1 직후)

| 방향 | 이유 | 선행 조건 |
|------|------|-----------|
| 방향 3 (레이어별 다양성) | 실험 설계 단순, TinyLLaMA에서 바로 측정 가능 | tinyllama_validation spec 확장 |
| 방향 4 (SpinQuant 이론화) | 기존 오픈소스(SpinQuant) 활용 가능 | SpinQuant checkpoint 접근 |

### 중기 (Phase 2-3)

| 방향 | 이유 | 선행 조건 |
|------|------|-----------|
| 방향 1 (공동 매니폴드) | 이론 정립 후 실험 설계 | 방향 3의 실험 데이터 |
| 방향 2 (Fisher 기하) | KFAC 구현 필요 | Phase 1 결과 |
| 방향 6 (엔트로피 코딩) | 분해 방식 확정 후 설계 | 방향 1, 3 결과 |

### 장기 (Phase 3-4, 차별화 기여)

| 방향 | 이유 | 선행 조건 |
|------|------|-----------|
| 방향 5 (온라인 KV) | 구현 복잡도 높음, 하지만 임팩트 큼 | 방향 3, 6 결과 |

---

## 4. 현재 프레임워크 강화를 위한 즉각적 액션

### 문서 수준

1. **매니폴드 가설의 조작적 정의 추가**
   - "effective rank가 전체 차원의 k% 이하"처럼 측정 가능한 조건 명시
   - LoRA와의 차이: LoRA는 파인튜닝 델타의 저랭크성, 본 연구는 사전학습 가중치의 매니폴드 구조

2. **SpinQuant 절의 명시적 추가**
   - related_work.md에 SpinQuant의 partial alignment 가설 절 추가
   - "왜 SpinQuant이 이미 답이 아닌가"를 명확히 서술

3. **RD 이득의 형식적 하한 추가**
   - 최소한 간단한 장난감(toy) 케이스에서 PCA-정렬 vs. 랜덤 회전의 Shannon entropy 차이를 계산하여 논문에 포함

### 실험 수준

4. **TinyLLaMA 실험에 SpinQuant 비교군 추가**
   - tinyllama_validation/spec.md에 SpinQuant 회전 행렬을 기준으로 flattening/concentration 측정 추가

5. **레이어별 effective rank 측정 우선 실행**
   - 가장 단순한 실험이며, 매니폴드 가설의 가장 직접적인 검증

---

## 5. 결론

현재 연구의 개념 프레임워크는 탄탄하며, 핵심 주장("아웃라이어는 좌표계 인공물",
"rotation은 flattening만 달성")은 잘 정당화되어 있다.

**가장 중요한 gap:**
1. 가중치 매니폴드 가설의 조작적 명세 (레이어별 effective rank 측정으로 해결)
2. SpinQuant와의 명확한 차별화 (partial alignment 이론으로 해결)
3. 활성값 공간에서의 오차 설명 (공동 매니폴드 방향으로 해결)

**가장 임팩트 있는 신규 방향:**
- **Fisher 정보 기하 기반 좌표계** (방향 2): 이론적 엄밀성과 기존 Hessian 방법들과의 통합
- **KV Cache 온라인 매니폴드 추적** (방향 5): TurboQuant를 직접 겨냥하는 실용적 기여

이 두 방향이 실험적으로 검증된다면,
단순한 "개념 재프레이밍"을 넘어 **새로운 압축 패러다임**으로서의 설득력을 갖게 된다.

---

> **한 줄 북스타(North Star) 업데이트 제안:**
>
> _"좌표계가 맞으면 아웃라이어는 사라진다"_ (현재)
>
> → _"연산자의 구조를 따르는 좌표계에서, 압축은 정보 삭제가 아니라 표현 선택이다."_ (제안)
