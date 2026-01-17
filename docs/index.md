# 문서 인덱스

이 리포지토리는 LLM 압축이 본질적으로 **좌표계 선택**과 **연산자 재매개변수화(operator reparameterization)** 문제라고 주장합니다.

## 권장 읽기 순서

1. 개념 개요: `docs/overview.md`
2. 연산자 관점(왜 “가중치는 데이터가 아닌가”): `docs/theory/operator_view.md`
3. 아웃라이어를 좌표계 인공물로 보기: `docs/theory/coordinate_relative_outliers.md`
4. 회전 vs 정렬(평탄화 vs 집중): `docs/theory/rotation_vs_alignment.md`
5. “매니폴드 정렬”의 의미와 실용적 근사: `docs/theory/manifold_alignment.md`
6. 논문 초안(통합 서사): `docs/paper/paper_draft.md`

## 실험(스펙)

- 토이 실험(Fourier basis vs MLP basis): `experiments/toy_basis_vs_mlp/spec.md`
- TinyLLaMA/SLM 검증(draft): `experiments/tinyllama_validation/spec.md`

## 특허 지향 문서

- 발명 요약: `docs/patent/invention_summary.md`
- 선행기술 매핑: `docs/patent/prior_art_mapping.md`
- 청구항 후보(draft): `docs/patent/claim_candidates.md`
