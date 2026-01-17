# 관련 연구(Related Work)

이 섹션은 본 연구를 LLM 양자화, 회전 기반 전처리, 그리고
신경망 파라미터에 대한 기하학적 해석과 관련된 기존 연구 흐름 속에 위치시킵니다.

---

## 아웃라이어-인식(outlier-aware) 양자화

초기 LLM 양자화 방법들은 클리핑(clipping), 혼합 정밀도(mixed precision),
그룹 단위 스케일링(group-wise scaling) 등을 통해 아웃라이어 가중치를 식별하고 보호하는 데 집중했습니다.
대표적으로 SmoothQuant, GPTQ, 그리고 관련 사후학습(post-training) 양자화 기법들이 있습니다.

이 접근들은 큰 크기(magnitude)의 가중치가 불균형하게 중요한 정보를 담고 있다는 가정을 암묵적으로 깔고 있으며,
본 연구는 이 가정을 문제 삼습니다.

---

## 회전 기반 전처리(Rotation-Based Preprocessing)

QuaRot, SpinQuant, KurTail과 같은 최근 방법들은 양자화 전에 직교 변환(예: Hadamard 회전)을 적용하면
아웃라이어를 크게 줄이고 균일(uniform) 저비트 양자화를 가능하게 할 수 있음을 보였습니다.

하지만 경험적으로 효과적임에도, 이러한 방법들은 회전이 _왜_ 동작하는지에 대한 명확한 기하학적 설명을 제공하지 못하고,
회전이 압축 관점에서 최적인지 여부도 다루지 않습니다.

본 연구는 이들을 좌표계 인공물을 제거하는 축 변환(axis transformation)으로 재해석하며,
회전이 구조적 정렬(alignment)을 유도하는 것은 아니라는 점을 강조합니다.

---

## 저랭크/구조적 압축(Low-Rank and Structural Compression)

저랭크 근사(low-rank approximation), 프루닝(pruning), 텐서 분해(tensor decomposition) 방법들은
신경망 내부의 중복성(redundancy)을 암묵적으로 활용합니다.
이 접근들은 학습된 모델이 파라미터 공간의 제한된 영역을 점유한다는 관찰과도 부합합니다.

다만 대부분의 선행연구는 여전히 값(value) 중심이며,
압축을 좌표계 선택 문제로 명시적으로 정식화하지는 않습니다.

---

## 매니폴드 관점(Manifold Perspectives)

매니폴드 가설(manifold hypothesis)은 활성값(activations)과 표현(representations)에 대해서는 널리 논의되어 왔지만,
_연산자 파라미터(operator parameters)_에 대해서는 상대적으로 논의가 적었습니다.
본 연구는 매니폴드 관점을 LLM 가중치로 확장하고, 이를 양자화 및 엔트로피 코딩(entropy coding)과 직접 연결합니다.

---

## 요약(Summary)

기존 연구와 달리, 본 연구는:

- 가중치를 데이터가 아니라 연산자 파라미터로 취급하고,
- 아웃라이어를 좌표계-상대적 인공물로 재해석하며,
- 회전이 만드는 평탄화(flattening)와 매니폴드 정렬(alignment)을 구분하고,
- 양자화와 압축을 하나의 기하학적 프레임워크로 통합합니다.
