# operator-coordinate-compression

**LLM 가중치 양자화 및 압축을 ‘연산자(operator)’와 ‘좌표계(coordinate system)’ 관점에서**
재해석하는 연구 리포지토리입니다.

---

## 문제의식

최근 연구들은 대규모 언어 모델(LLM)이 INT8, 심지어 INT4까지도 공격적으로 양자화될 수 있으며
퍼플렉시티(perplexity) 저하가 매우 작을 수 있음을 보여줍니다.
이는 큰 크기의 가중치(아웃라이어, outlier)를 특별히 보호해야 한다는
전통적 관점을 흔듭니다.

이 리포지토리는 다음과 같은 다른 해석을 제안합니다.

> **LLM compression is not fundamentally about removing information,  
> but about choosing the right coordinate system to represent operator parameters.**
>
> **LLM 압축은 본질적으로 정보를 “삭제”하는 문제가 아니라,  
> 연산자 파라미터를 표현할 “올바른 좌표계”를 고르는 문제다.**

---

## 핵심 아이디어

본 리포지토리의 핵심 주장은 다음과 같습니다.

1. **LLM 가중치는 독립 스칼라 값이 아니라 비선형 연산자를 매개변수화**한다.
2. **학습된 가중치는 고차원 파라미터 공간 안의 저차원 비선형 매니폴드(다양체)** 근처에 놓인다.
3. 관측되는 **아웃라이어는 좌표계-상대적 인공물(artifact)**이며, 선택된 축과 매니폴드 접공간(tangent space)의 불일치에서 발생한다.
4. Hadamard / QuaRot 등 회전 기반 방법은 주파수 변환이 아니라 **축 변환(axis transformation)**으로 해석해야 한다.
5. 회전은 분포를 평탄화(flattening)하고 아웃라이어를 줄이지만, **진짜 압축 이득은 매니폴드-정렬(manifold-aligned) 좌표계**가 만드는 계수 집중(concentration)에서 나온다.
6. 이러한 정렬은 **함수적(구조적) 성분 + 잔차(residual) 분해**, 비대칭 양자화, 레이트–왜곡(rate–distortion) 최적 압축으로 이어진다.

---

## 포함 내용

- 📐 **Theory**
  - LLM 가중치의 연산자 중심 관점
  - 매니폴드 가설과 좌표계-상대적 아웃라이어
  - 축 변환(axis) vs 주파수 변환(frequency)
- 🧪 **Experiments**
  - 토이 실험 스펙 (Fourier basis vs. MLP basis)
  - (planned) 축 변환 분석 (Hadamard, PCA 기반 정렬)
  - TinyLLaMA / SLM 검증 스펙 (draft)
- 🧾 **Writing**
  - 학술 논문 초안
  - 특허 지향 문서(발명 요약, 선행기술 매핑, 청구항 후보)

---

## 리포지토리 구조

```text
operator-coordinate-compression/
├─ docs/
│  ├─ index.md
│  ├─ overview.md
│  ├─ theory/
│  ├─ paper/
│  └─ patent/
├─ experiments/
│  ├─ toy_basis_vs_mlp/
│  │  └─ spec.md
│  └─ tinyllama_validation/
│     └─ spec.md
├─ README.md
└─ roadmap.md
```

향후 추가 예정(현재 없음):

- `experiments/axis_transform_analysis/`
- `src/`
