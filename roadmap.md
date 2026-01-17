# 연구 로드맵 — 연산자 좌표계 압축(Operator Coordinate Compression)

이 문서는 **operator-coordinate-compression** 프로젝트의 연구 로드맵을 정의합니다.

목표는 LLM 양자화 및 압축을 **좌표계 선택 및 연산자 재매개변수화(reparameterization) 문제**로
체계적으로 재정의하고, 이 관점을 이론/토이 실험/실제 모델 분석으로 검증하는 것입니다.

---

## 1. 핵심 연구 논지

> **LLM 압축은 본질적으로 정보를 제거하는 것이 아니라,  
> 기반이 되는 연산자 매니폴드의 구조에 정렬되는 좌표계를 고르는 문제다.**

핵심 주장:

1. LLM 가중치는 비선형 연산자를 매개변수화한다.
2. 학습된 가중치는 저차원 비선형 매니폴드(다양체) 근처에 놓인다.
3. 관측되는 아웃라이어(outlier)는 좌표계-상대적 인공물(artifact)이다.
4. 회전 기반 방법은 분포를 평탄화(flattening)하지만 계수 집중(concentration)을 유도하지는 않는다.
5. 매니폴드-정렬(manifold-aligned) 축은 함수적(function-like)이고 집중된 표현을 가능하게 한다.
6. 이러한 표현은 더 나은 레이트–왜곡(rate–distortion) 트레이드오프를 제공한다.

---

## 2. 문서 계층과 역할

이 리포지토리는 다음 문서 계층을 중심으로 구성됩니다.

### 2.1 개념 개요(Overview)

- `README.md`
- `docs/overview.md`

목적:

- 핵심 아이디어를 고수준에서 전달
- 좌표계가 왜 중요한지 설명
- 과도한 수식 없이 직관 제공

---

### 2.2 이론적 토대(Theory)

- `docs/theory/operator_view.md`
- `docs/theory/manifold_alignment.md`
- `docs/theory/coordinate_relative_outliers.md`
- `docs/theory/rotation_vs_alignment.md`

목적:

- 연산자 중심 관점을 정식화
- 매니폴드 가정 정의
- 아웃라이어와 회전을 기하학적으로 재해석

---

### 2.3 학술 논문 초안

- `docs/paper/paper_draft.md`

목적:

- 이론 + 실험을 출판 가능한 서사로 통합
- 외부 커뮤니케이션의 기준(캐노니컬) 레퍼런스 역할

---

### 2.4 실험 스펙(코드 없음)

- `experiments/toy_basis_vs_mlp/spec.md`
- `experiments/tinyllama_validation/spec.md` (draft)

목적:

- 구현 전에 실험을 완전하게 명세
- 재현성과 개념적 명확성 확보
- 무엇을 검증하는지(_what_)와 어떻게 구현하는지(_how_)를 분리

---

### 2.5 특허 지향 문서

- `docs/patent/invention_summary.md`
- `docs/patent/prior_art_mapping.md`
- `docs/patent/claim_candidates.md` (draft)

목적:

- 연구 아이디어를 보호 가능한 청구항 방향으로 번역
- 선행기술 대비 신규 요소 식별

---

## 3. 연구 단계(Phases)

### Phase 0 — 개념 고정(현재 단계) ✅

상태: **진행 중**

목표:

- 개념적 프레이밍을 고정
- 공통 어휘(용어)를 정립:
  - operator(연산자) vs value(값)
  - axis(축) vs frequency(주파수)
  - flattening(평탄화) vs concentration(집중)
- 핵심 문서를 완성

산출물:

- README
- Overview
- Theory notes
- Paper draft
- Toy experiment spec

---

### Phase 1 — 통제된 검증(토이 실험)

상태: 계획됨

목표:

- 가장 단순한 설정에서 좌표계-의존적 압축 현상을 시연
- 시각적/정량적 직관 제공

핵심 질문:

- 기저(basis) 선택이 계수 엔트로피에 어떤 영향을 주는가?
- 회전이 아웃라이어를 줄이지만 희소성(sparsity)을 유도하지 않는 이유는 무엇인가?
- 표현(representation)에 따라 RD(rate–distortion) 거동이 어떻게 달라지는가?

---

### Phase 2 — 실제 모델 분석(TinyLLaMA / SLM)

상태: 계획됨

목표:

- 실제 LLM에서 좌표계-상대적 아웃라이어 현상을 검증
- 토이 수준 현상이 실제에서도 나타나는지 확인

핵심 질문:

- 축 변환(axis transform) 하에서 아웃라이어 통계가 변하는가?
- 랜덤 회전이 평탄화는 증가시키되 집중은 증가시키지 않는가?
- 근사적 매니폴드 정렬이 압축 지표를 개선할 수 있는가?

---

### Phase 3 — 매니폴드-정렬 압축 방법론

상태: 탐색적

목표:

- 실용적인 축 정렬 근사 방법을 설계
- LLM 가중치에서 함수적 성분 + 잔차(residual) 분해를 탐색

후보 기법:

- Block-wise PCA / tangent estimation
- Clustering + shared bases
- Learned axis transforms optimized for entropy or sparsity

---

### Phase 4 — 출판 및 보호(특허)

상태: 향후

목표:

- 학술 논문 제출
- 특허 출원
- 선택된 실험 코드 오픈소스화

---

## 4. 평가 기준

모든 단계에서 성공은 다음 기준으로 평가합니다.

- 개념적 명확성(아이디어를 단순하게 설명할 수 있는가?)
- 재현성(스펙이 구현과 독립적인가?)
- 레이트–왜곡 개선
- 휴리스틱 의존 최소화
- 선행연구 대비 명확한 차별성

---

## 5. 가이드 원칙

1. **코드보다 문서**
2. **최적화보다 설명**
3. **LLM보다 토이 실험**
4. **휴리스틱보다 기하**
5. **양자화보다 표현**

---

## 6. 한 줄 노스스타(North Star)

> _“좌표계가 맞으면 아웃라이어는 사라진다.”_

---

## 7. 다음 문서 작업(우선순위)

다음 문서를 권장합니다.

1. `docs/theory/operator_view.md`
2. `docs/theory/coordinate_relative_outliers.md`
3. `docs/theory/rotation_vs_alignment.md`
4. `experiments/tinyllama_validation/spec.md` 확장
5. `docs/patent/invention_summary.md`

---

로드맵 끝.
