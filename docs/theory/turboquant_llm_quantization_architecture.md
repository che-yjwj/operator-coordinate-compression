# LLM 양자화·압축 아키텍처 노트 (개선판, TurboQuant 중심)

> **문서 상태:** 논문 직접 진술 / 시스템 해석 / 향후 실험 계획을 분리한 개선판
> **문서 목적:** TurboQuant를 논문 수준에서 정확하게 복원하면서, 시스템 아키텍트 관점의 해석과 후속 연구 방향을 명확히 구분한다.

---

## 목차

- [0. 이 문서의 목적과 읽는 법](#0-이-문서의-목적과-읽는-법)
- [1. 빠른 총평](#1-빠른-총평)
- [2. 논문 직접 진술 요약](#2-논문-직접-진술-요약)
- [3. TurboQuant 핵심 구조](#3-turboquant-핵심-구조)
- [4. 핵심 수학 직관](#4-핵심-수학-직관)
- [5. PolarQuant / QJL / OPQ와의 관계 정리](#5-polarquant--qjl--opq와의-관계-정리)
- [6. 시스템 아키텍처 관점 해석](#6-시스템-아키텍처-관점-해석)
- [7. 실무 배치 체크리스트](#7-실무-배치-체크리스트)
- [8. 이 리포의 장기 비전과 연결](#8-이-리포의-장기-비전과-연결)
- [9. 검증 계획](#9-검증-계획)
- [10. 핵심 용어 사전](#10-핵심-용어-사전)
- [11. 참고 자료](#11-참고-자료)
- [12. 사실/추론 경계 및 업데이트 메모](#12-사실추론-경계-및-업데이트-메모)

---

## 0. 이 문서의 목적과 읽는 법

이 문서는 세 가지를 동시에 수행한다.

1. **TurboQuant 원논문과 공식 블로그의 핵심 주장**을 왜곡 없이 복원한다.
2. 이를 **KV-cache quantization / long-context serving / 시스템 아키텍처** 맥락에서 재해석한다.
3. 현재 리포지토리의 "operator-coordinate-compression" 관점과 연결하되, **논문 직접 진술과 본 문서의 해석을 엄격히 분리**한다.

### 서술 규칙

- **[FACT]**: TurboQuant 원논문, Google Research 공식 블로그, QJL/PolarQuant 원문에서 직접 확인 가능한 진술
- **[INFERENCE]**: 본 문서의 시스템 해석, 아키텍처 해석, 연구 가설
- **[PLAN]**: 아직 검증되지 않았지만 이후 실험으로 확인하려는 항목

### 이 문서의 핵심 원칙

- 논문이 말한 것과 내가 해석한 것을 섞지 않는다.
- 알고리즘 성능과 서비스 성능을 동일시하지 않는다.
- `커널 속도 향상`과 `E2E token/s 향상`을 구분한다.
- `MSE 개선`과 `attention 품질 보존`을 구분한다.

---

## 1. 빠른 총평

TurboQuant는 단순한 "저비트 scalar quantization 기법"이 아니라, 다음 조합으로 보는 것이 정확하다.

- **랜덤 직교 회전**
- **회전 후 좌표별 near-optimal scalar quantization**
- **residual에 대한 1-bit QJL 보정**
- **MSE distortion과 inner-product distortion 동시 제어**
- **data-oblivious / online 적용 가능**

[FACT] TurboQuant는 고차원 벡터를 저비트로 압축하면서 MSE와 inner-product distortion을 모두 줄이도록 설계된 온라인 벡터 압축 계열 방법이다. LLM에서는 특히 KV cache quantization에 적용된다.

[INFERENCE] 시스템 관점에서 TurboQuant의 실질적 의미는 "복잡한 model-specific calibration 없이도, long-context decode에서 HBM 대역폭 병목을 줄일 수 있는 강한 범용 baseline"이라는 점이다.

---

## 2. 논문 직접 진술 요약

이 절은 원논문/공식 자료에서 직접 확인되는 핵심만 정리한다.

### 2.1 문제 정의

[FACT] TurboQuant는 온라인 설정에서 고차원 벡터를 압축할 때, 단순 재구성 오차(MSE)만이 아니라 **inner product 왜곡**까지 함께 줄이는 것을 목표로 한다.

[FACT] 이 문제는 LLM KV cache, 특히 긴 컨텍스트에서의 attention 계산에 직접 연결된다. 이유는 attention logit이 본질적으로 dot-product 기반이기 때문이다.

### 2.2 방법의 큰 구조

[FACT] TurboQuant는 두 단계 구조로 이해할 수 있다.

1. **랜덤 직교 회전 후 scalar quantization**
2. **residual에 대한 1-bit QJL 기반 보정**

### 2.3 핵심 장점

[FACT] 원논문/공식 자료가 강조하는 포인트는 다음과 같다.

- data-oblivious
- online 적용 가능
- near-optimal distortion rate
- low-bit에서도 quality 유지
- KV cache 압축에서 강한 성능

### 2.4 성능 관련 직접 표현

[FACT] 원논문 abstract는 대략 다음 메시지를 준다.

- **3.5 bpc 부근에서 quality neutrality**
- **2.5 bpc 부근에서도 marginal degradation**
- **KV cache 5x 이상 압축**
- **attention 관련 커널 구간에서 큰 속도 이점 가능**

[FACT] Google Research 블로그는 더 강한 표현으로 효율과 품질 보존을 강조하지만, 실무 문서에서는 **논문 표/그림 기준**으로 다시 확인하는 것이 안전하다.

---

## 3. TurboQuant 핵심 구조

### 3.1 개념도

```text
입력 벡터 x ∈ R^d
   |
   v
[Random Orthogonal Rotation]
   y = R x
   |
   v
[Per-coordinate Scalar Quantization]
   q = Q(y)
   r = y - q
   |
   +---------------> 저장: 주 압축 표현 q
   |
   v
[1-bit QJL over residual]
   s = sign(r)  (+ estimator 구성에 필요한 규칙)
   |
   v
압축 표현: (q, s)
```

### 3.2 단계별 역할

#### Stage 1: 회전 + 좌표별 scalar quantization

[FACT] 랜덤 회전 후 좌표 분포가 집중되고 균질해지므로, 각 좌표별 scalar quantizer가 강력해진다.

[INFERENCE] 즉, TurboQuant의 핵심은 "복잡한 전역 코드북 탐색"보다, **회전으로 분포를 다루기 쉬운 형태로 만든 뒤 단순한 quantizer를 강하게 만드는 것**에 있다.

#### Stage 2: residual에 대한 1-bit 보정

[FACT] MSE-optimal quantization만으로는 inner product estimation에 bias가 남을 수 있다. TurboQuant는 residual에 대해 1-bit QJL을 결합해 unbiased inner-product estimation을 구성한다.

[INFERENCE] 이 단계는 단순 재구성 보정이 아니라, **attention logit의 통계적 편향을 직접 겨냥한 보정**으로 이해하는 것이 맞다.

---

## 4. 핵심 수학 직관

### 4.1 왜 랜덤 회전이 중요한가

랜덤 직교 회전 \( R \in \mathbb{R}^{d \times d} \)를 적용하면,

\[
y = Rx
\]

원래 특정 좌표에 집중되어 있던 큰 값(outlier)이 여러 축으로 퍼진다.

[FACT] 원논문은 이 회전 뒤 좌표 분포가 **concentrated Beta distribution** 계열로 다뤄질 수 있음을 강조한다.

[INFERENCE] 실무 직관으로는 다음처럼 이해하면 된다.

- 원 좌표계: 어떤 채널은 너무 크고 어떤 채널은 너무 작다.
- 회전 후 좌표계: 에너지가 여러 축에 섞여 들어간다.
- 결과: per-coordinate quantization이 훨씬 잘 먹힌다.

### 4.2 Gaussian이라고 말해도 되는가

[INFERENCE] "회전 후 각 좌표가 Gaussian이 된다"는 표현은 직관으로는 유용하지만, 엄밀한 문서에서는 주 표현으로 쓰기보다 보조 설명으로 두는 것이 좋다.

권장 표현:

> 랜덤 회전 후 좌표 마진 분포는 이론적으로 집중된 Beta 계열로 기술되며, 고차원에서는 Gaussian-like intuition으로 이해할 수 있다.

### 4.3 왜 MSE만 보면 안 되는가

attention에서 중요한 값은

\[
\langle q_i, k_j \rangle
\]

이다. 따라서 벡터 재구성이 조금 틀린 것보다, **dot-product의 systematic bias**가 더 위험할 수 있다.

[FACT] TurboQuant는 이 점 때문에 residual에 1-bit QJL을 추가해 inner-product estimator의 bias를 제거하는 방향을 취한다.

---

## 5. PolarQuant / QJL / OPQ와의 관계 정리

### 5.1 QJL

[FACT] QJL은 1-bit quantized Johnson-Lindenstrauss transform 계열로, 저비트 스케치만으로 inner product estimation을 가능하게 하는 방향의 방법이다.

[FACT] TurboQuant에서는 residual에 대해 QJL을 결합해 unbiased inner-product estimation을 만든다.

### 5.2 PolarQuant

[FACT] PolarQuant는 random preconditioning 뒤 polar transformation을 적용해, normalization overhead와 quantization constants 부담을 줄이는 방향의 관련 방법이다.

[INFERENCE] PolarQuant를 TurboQuant의 완전한 하위 단계라고 단정하기보다는, **Google 측이 함께 제시하는 관련 기법군** 또는 Stage-1 계열의 중요한 배경 작업으로 이해하는 것이 안전하다.

### 5.3 OPQ와의 관계

[FACT] OPQ는 product quantization 전에 회전을 최적화해 distortion을 줄이는 고전적 접근이다.

[INFERENCE] TurboQuant와 OPQ는 모두 "회전이 압축 성능을 바꾼다"는 점에서 연결되지만, 목적과 제약은 다르다.

- OPQ: 오프라인 학습, retrieval/index 쪽 맥락이 강함
- TurboQuant: data-oblivious, online, KV-cache serving 친화적

---

## 6. 시스템 아키텍처 관점 해석

이 절은 본 문서의 해석이다.

### 6.1 왜 KV cache에서 특히 중요한가

[INFERENCE] long-context LLM serving에서는 decode 구간에서 KV cache 읽기 트래픽이 HBM 병목으로 올라오는 경우가 많다. 이때 KV를 16-bit/8-bit로 계속 읽는 대신, 3~4 bpc 수준으로 낮추면 메모리 트래픽을 크게 줄일 수 있다.

### 6.2 그러나 알고리즘 이득이 곧바로 서비스 이득은 아니다

다음 항목을 분리해야 한다.

- 커널 구간 속도 향상
- attention 전체 구간 속도 향상
- decoder token/s 향상
- 전체 서비스 p99 latency 향상

[INFERENCE] TurboQuant류 방법은 대개 **메모리 병목이 강한 decode-heavy workload**에서 이득이 크고, prefill 비중이 큰 workload에서는 체감 이득이 약해질 수 있다.

### 6.3 실제 런타임에서 봐야 할 항목

- 저장 포맷: per-token / per-head / per-block
- 메타데이터: scale, normalization, sign sketch, padding
- paged KV block size와 정렬
- recent tokens를 FP16으로 유지할지 여부
- attention kernel과 quant/dequant를 fuse할지 여부

### 6.4 NPU / accelerator 관점의 블록 해석

[INFERENCE] 하드웨어 관점에서는 다음 세 가지 질문이 중요하다.

1. 회전을 어디서 수행할 것인가?
   - host pre-process
   - load-time transform
   - kernel 내부 fused transform

2. dequant를 할 것인가, 아니면 dot-product를 직접 근사할 것인가?
   - dequant 후 GEMV/GEMM
   - compressed representation에서 직접 logit 추정

3. residual sign sketch를 어떤 실행 자원에 배치할 것인가?
   - vector lane
   - scalar assist
   - dedicated micro-kernel

---

## 7. 실무 배치 체크리스트

### 7.1 측정 관점

- [ ] prefill / decode를 분리 측정했는가
- [ ] 평균 latency뿐 아니라 p95 / p99를 봤는가
- [ ] `nominal bitwidth`가 아니라 **실제 bpc**를 계산했는가
- [ ] MSE뿐 아니라 logit KL, retrieval accuracy, long-context metric을 같이 봤는가

### 7.2 런타임 관점

- [ ] paged KV와 quant block size가 정렬되는가
- [ ] recent tokens fallback 정책이 있는가
- [ ] 특정 layer/head만 고정밀 유지하는 escape hatch가 있는가
- [ ] canary / rollback-safe path가 준비되어 있는가

### 7.3 장애 관점

- [ ] 긴 multi-turn 세션에서 drift를 모니터링하는가
- [ ] short benchmark는 통과하지만 long-context에서 무너지는 패턴을 따로 보는가
- [ ] kernel-level speedup과 service-level KPI를 구분해서 리포트하는가

---

## 8. 이 리포의 장기 비전과 연결

이 절은 연구 가설이다.

### 8.1 좌표계 선택 연속선

```text
원 좌표계
   |
   +-- 랜덤 회전 기반 좌표계
   |      +-- TurboQuant
   |          - data-oblivious
   |          - online
   |          - flattening
   |
   +-- manifold-aligned 좌표계
          +-- operator-coordinate-compression의 장기 비전
              - 데이터/모델 구조 반영
              - concentration 극대화 목표
              - 일부 오프라인 비용 허용 가능
```

### 8.2 해석

[INFERENCE] 본 문서에서는 TurboQuant를 "좌표계 선택 연속선" 위의 매우 강한 범용 baseline으로 본다.

[INFERENCE] 랜덤 회전은 flattening에는 강하지만, 장기적으로는 모델/레이어/헤드 구조를 반영한 manifold-aligned basis가 더 낮은 bit budget에서 유리할 가능성이 있다.

### 8.3 핵심 연구 가설

[PLAN] 동일한 bit budget에서 manifold-aligned basis가 random rotation보다 더 빠른 residual energy decay를 보일 수 있다.

[PLAN] 이 가설이 맞다면,

- 더 낮은 bpc
- 더 작은 logit bias
- 더 좋은 long-context stability

를 동시에 얻을 수 있다.

---

## 9. 검증 계획

### 9.1 알고리즘 검증

- **실험 A:** no rotation vs random rotation vs structured Hadamard vs learned basis
- **실험 B:** 동일 bpc에서 MSE / inner-product distortion / logit KL 비교
- **실험 C:** residual sign sketch 유무에 따른 long-context degradation 비교

### 9.2 시스템 검증

- **실험 D:** prefill-heavy vs decode-heavy workload 분리 측정
- **실험 E:** kernel fusion 유무에 따른 실제 token/s 비교
- **실험 F:** paged KV block alignment가 성능에 주는 영향 측정

### 9.3 아키텍처 검증

- **실험 G:** dequant-then-dot vs compressed-domain dot 근사 비교
- **실험 H:** NPU simulator에서 bandwidth savings와 latency savings 분리 계수화
- **실험 I:** recent-token FP16 fallback 정책의 품질/성능 trade-off 측정

---

## 10. 핵심 용어 사전

### 10.1 알고리즘

| 용어 | 의미 | 주의 |
| --- | --- | --- |
| PTQ | 재학습 없이 적용하는 quantization | calibration 비용을 과소평가하기 쉬움 |
| QAT | quantization-aware training | 품질은 좋지만 재학습 비용 큼 |
| KV cache quantization | KV cache를 저비트로 압축 | decode 병목과 직접 연결 |
| Lloyd-Max quantizer | 주어진 분포에서 scalar quantization 왜곡을 줄이는 양자화기 | 분포 변동에 민감 |
| QJL | 1-bit 스케치 기반 inner-product estimation 기법 | 단순 보조 메타데이터가 아니라 estimator 구성 요소 |
| PolarQuant | polar transformation 기반 KV-cache quantization 관련 방법 | TurboQuant와 동일시 금지 |

### 10.2 시스템

| 용어 | 의미 | 주의 |
| --- | --- | --- |
| bpc | bit per channel. 메타데이터 포함 실제 압축률 지표 | nominal bitwidth와 혼동 금지 |
| paged KV cache | KV cache를 페이지 단위로 관리하는 런타임 구조 | quant block 정렬 중요 |
| kernel fusion | 여러 단계를 한 커널로 결합 | 성능 향상 가능하지만 디버깅 어려움 |
| memory-bound | 대역폭이 주 병목인 상태 | FLOPs만 보고 최적화하면 실패 |
| rollback-safe | 장애 시 즉시 복귀 가능한 설계 | 운영 안정성 핵심 |

### 10.3 이 리포의 연구 개념

| 용어 | 의미 | 주의 |
| --- | --- | --- |
| operator parameterization | 가중치/표현을 독립 스칼라가 아니라 연산자 구조로 보는 관점 | 단순 행렬 분해와 동일시 금지 |
| manifold-aligned coordinates | 데이터/모델 구조에 정렬된 좌표계 | 아직 연구 가설 영역 |
| flattening | 회전으로 outlier를 퍼뜨리는 효과 | concentration과 구분 필요 |
| concentration | 특정 좌표계에서 정보/에너지가 더 효율적으로 모이는 현상 | 본 문서의 장기 비전 핵심 |

---

## 11. 참고 자료

1. TurboQuant: *Online Vector Quantization with Near-optimal Distortion Rate*
   <https://arxiv.org/abs/2504.19874>

2. Google Research Blog: *TurboQuant: Redefining AI efficiency with extreme compression*
   <https://research.google/blog/turboquant-redefining-ai-efficiency-with-extreme-compression/>

3. QJL: *1-Bit Quantized JL Transform for KV Cache Quantization with Zero Overhead*
   <https://arxiv.org/abs/2406.03482>

4. PolarQuant: *Quantizing KV Caches with Polar Transformation*
   <https://arxiv.org/abs/2502.02617>

5. OPQ: *Optimized Product Quantization for Approximate Nearest Neighbor Search*
   <https://openaccess.thecvf.com/content_cvpr_2013/html/Ge_Optimized_Product_Quantization_2013_CVPR_paper.html>

6. StreamingLLM
   <https://arxiv.org/abs/2309.17453>

---

## 12. 사실/추론 경계 및 업데이트 메모

### 12.1 경계 재확인

- TurboQuant의 구체 구성, 품질, 성능 관련 서술은 **논문/공식 블로그 기준**
- "좌표계 선택 연속선", "manifold-aligned basis", "operator-coordinate-compression과의 연결"은 **본 문서의 해석**
- manifold-aligned basis가 random rotation보다 낫다는 주장은 **아직 연구 가설**

### 12.2 후속 업데이트 권장 항목

- TurboQuant 원논문 표/그림 번호를 본문 각 주장에 직접 연결
- PolarQuant가 TurboQuant 본문에서 어느 위치로 인용되는지 섹션 번호 명시
- toy experiment 결과를 8장과 9장에 반영
- 실제 kernel benchmark와 E2E serving benchmark를 분리 리포트

### 12.3 한 줄 결론

> TurboQuant는 "랜덤 회전 + scalar quantization + residual 1-bit QJL"의 결합으로, 온라인 KV-cache 압축에서 매우 강한 범용 baseline이다.
> 본 리포의 장기 비전은 여기서 한 걸음 더 나아가, 랜덤 회전 대신 manifold-aligned coordinate를 통해 더 낮은 bit budget과 더 나은 long-context stability를 얻을 수 있는지 검증하는 것이다.
