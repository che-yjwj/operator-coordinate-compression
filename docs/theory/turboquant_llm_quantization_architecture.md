# LLM 양자화·압축 아키텍처 노트 (TurboQuant 중심)

## 0) 이 문서의 목적

이 문서는 두 가지를 동시에 수행한다.

1. 현재 리포지토리(`operator-coordinate-compression`)가 제시하는 핵심 아이디어를 시스템 설계 관점에서 빠르게 복원한다.
2. Google Research의 **TurboQuant**를, 논문 수준의 수학 직관 + 실무 배치 시 암묵지까지 포함해 구조적으로 정리한다.

> 핵심 관점: “양자화는 값(value) 손실 최적화 문제가 아니라, 연산자(operator)와 좌표계(coordinate system) 선택 문제”라는 이 리포의 주장을 실제 KV-cache 압축 알고리즘(TurboQuant)까지 연결한다.

### 문서 성격과 읽는 법

- 이 문서는 **논문/블로그 내용을 요약한 부분**과 **이 저장소의 해석 및 실무 메모**를 함께 담는다.
- 따라서 아래 서술 중 일부는 원문 요약이고, 일부는 본 저장소 관점에서의 해석이다.
- 외부 사실 주장(알고리즘 구성, 성능 수치, 공개 시점)은 반드시 원문과 함께 읽어야 한다.
- 현재 참고 자료 섹션은 자리만 잡혀 있으므로, 외부 공개용 문서로 쓰려면 출처를 먼저 보강해야 한다.

---

## 1) 현재 코드베이스(문서베이스)의 아이디어 맵

### 1.1 프로젝트의 North Star

리포는 LLM 압축을 다음처럼 재정의한다.

- 가중치는 독립 스칼라 집합이 아니라 **연산자 파라미터화**다.
- 학습된 파라미터는 고차원 공간 전체가 아니라 **저차원 매니폴드 근방**에 존재한다.
- outlier는 본질 신호라기보다 **좌표계-상대 인공물**이다.
- Hadamard류는 “주파수 변환”이라기보다 **축 변환(회전)**이다.
- 랜덤 회전은 flattening(평탄화)은 잘 하지만, 진짜 압축 이득은 manifold-aligned coordinate에서 오는 concentration(집중)에서 나온다.

즉 이 리포의 주제는 “low-bit 트릭 모음집”이 아니라, **표현좌표(표기법)의 기하학적 선택**이다.

### 1.2 실무적으로 중요한 해석

- 일반적인 양자화 운영은 보통 `scale/zero-point 튜닝`에 집착한다.
- 이 리포 관점은 그보다 상위 레벨에서 “무슨 좌표축에서 scale을 잡는가?”를 먼저 본다.
- 이 프레이밍은 KV cache, weight-only quant, activation quant, vector DB PQ를 하나의 공통 언어로 묶는다.

### 1.3 코드베이스 상태 요약

- 현재는 구현보다 **이론/실험 스펙 문서 중심**이다.
- 토이 실험 spec(기저에 따른 압축성 차이)과 실제 LLM 검증 spec이 분리되어 있다.
- 논문/특허 관점 문서까지 이미 배치되어 있어, 연구→제품화로 이어질 문맥이 명확하다.

---

## 2) TurboQuant 상세 정리

### 2.1 TurboQuant의 문제 정의

TurboQuant는 고차원 벡터를 온라인(데이터 의존적 학습 없이)으로 저비트 압축하면서,

- MSE 왜곡
- inner-product 왜곡

을 동시에 낮추려는 **고차원 벡터 압축 알고리즘**으로 이해할 수 있다.

엄밀한 분류는 주의가 필요하다.

- 입력 대상은 벡터이지만,
- 문서에서 설명하는 핵심 절차는 랜덤 회전 뒤의 **좌표별 scalar quantization**과 residual 보정에 가깝다.

따라서 이 문서에서는 TurboQuant를 좁은 의미의 codebook-search 중심 VQ라기보다,
**벡터를 대상으로 하지만 scalar quantization 중심으로 구현되는 압축 절차**로 서술한다.

LLM에서는 특히 **KV cache quantization**에 적용되며, 긴 컨텍스트에서 메모리/대역폭 병목을 풀기 위한 목적이 강하다.

### 2.2 핵심 구성: 2-stage

TurboQuant는 크게 두 단계다.

아래 내용은 이 문서가 이해한 구성 요약이며, 세부 구현/명칭은 원문 표현과 일치하는지 별도 대조가 필요하다.

1. **MSE 중심 stage (PolarQuant 계열 아이디어 포함)**
   - 벡터에 랜덤 회전을 적용
   - 회전 후 좌표 분포 성질(Beta 집중/고차원 근사 독립성)을 활용
   - 좌표별 scalar quantizer(Lloyd-Max) 적용

2. **Bias 교정 stage (QJL 1-bit residual)**
   - 1단계 잔차(residual)에 대해 1-bit QJL 적용
   - MSE-최적 양자화가 만드는 inner-product bias를 보정
   - attention logit/유사도 계산에서 편향 없는 추정기에 가깝게 구성

요약하면:

- stage1 = 에너지 대부분을 고효율로 압축
- stage2 = 작은 비트(보통 1bit/channel budget)를 써서 “잘못된 방향의 오차”를 잡음

### 2.3 왜 “회전 + 좌표별 양자화”가 먹히는가

실무자가 놓치기 쉬운 포인트:

- “좌표별 독립 quantizer는 구식”이라는 인식이 강하지만,
- 고차원 랜덤 회전 이후 분포가 잘 섞이면 좌표 간 결합정보가 약해지고,
- 이때 per-coordinate quantizer가 계산/배치/벡터화 측면에서 매우 강해진다.

즉 TurboQuant는 복잡한 codebook search를 피하고,

- 온라인성(학습 불필요)
- 구현 단순성
- 하드웨어 친화성

을 이론보장과 같이 가져가려는 설계다.

### 2.4 PolarQuant 역할

PolarQuant는 좌표계를 Cartesian→Polar 계열로 재표현해,

- 분포 예측 가능성 증가
- 정규화 메타데이터(quant constants) 부담 감소

를 노린다.

현업 의미:

- “값 자체 양자화”보다 “기하 파라미터(반지름/각도)의 통계 구조”를 양자화 대상으로 삼아 메타 오버헤드를 줄이는 전략이다.

주의:

- 이 절은 TurboQuant와 PolarQuant의 관계를 이해하기 위한 요약 메모다.
- 실제 논문에서 두 기법의 관계가 정확히 어떤 수준의 선행 아이디어 차용인지, 혹은 별개 비교축인지 명확히 인용해 둘 필요가 있다.

### 2.5 QJL residual 보정이 중요한 이유

실무에서 흔한 함정:

- MSE가 낮으면 attention 품질도 자동으로 좋을 거라 가정한다.
- 하지만 attention은 dot-product 편향에 민감하고, 작은 systematic bias가 긴 문맥에서 누적된다.

TurboQuant는 residual의 sign sketch(QJL)를 추가해 이 편향을 줄인다.

### 2.6 보고된 성능 포인트(공식 공개 기준: 2025-04-28 논문, 2026-03-24 블로그)

아래 항목은 공개된 논문/Google Research 블로그에서 전달하는 메시지를 **요약한 메모**다.
정확한 수치, 벤치 설정, 하드웨어 조건은 원문 표/그림과 함께 확인해야 한다.

확인한 1차 출처:

- TurboQuant 논문: `https://arxiv.org/abs/2504.19874` (2025-04-28)
- Google Research 블로그: `https://research.google/blog/turboquant-redefining-ai-efficiency-with-extreme-compression/` (2026-03-24)

- KV cache를 3~3.5bit/channel 수준으로 압축하며 품질 손실 최소화/중립 사례 보고
- long-context 벤치(Needle, LongBench 등)에서 강한 유지 성능
- H100에서 attention-logit 계산 기준 큰 폭의 가속 사례 보고

주의:

- 특정 speedup 숫자는 “attention kernel 구간” 기준인지 “end-to-end tokens/s” 기준인지 반드시 분리해야 한다.
- 프로덕션에서는 scheduler, paged KV, batch shape, prefill/decode mix 때문에 체감 성능이 크게 달라진다.

---

## 3) 실무 암묵지: Senior Engineer/System Architect 관점 체크리스트

### 3.1 알고리즘-시스템 경계

1. **알고리즘 승리 ≠ 서비스 승리**
   - 논문은 왜곡률/리콜/task score를 본다.
   - 서비스는 p99 latency, tail OOM, 배치 안정성, 장애 복구를 본다.

2. **decode 단계와 prefill 단계를 분리해 측정**
   - KV quant는 decode 메모리 병목 완화가 본진이다.
   - prefill이 긴 workload에서는 기대 이득이 희석될 수 있다.

3. **메타데이터 오버헤드 회계 분리**
   - “n-bit”만 보고 비교하면 함정.
   - scale/zero-point, per-block norm, layout padding까지 합친 실제 bpc(bit per channel)를 봐야 한다.

4. **정확도 metric의 지연 붕괴(drift) 감시**
   - 짧은 시퀀스는 버티는데 긴 체인-of-thought에서 갑자기 무너지는 경우가 있다.
   - multi-turn eval, retrieval stress를 함께 둬야 한다.

### 3.2 배치/런타임 설계 팁

- quant/dequant를 attention kernel과 최대한 fusion해 메모리 왕복을 줄인다.
- paged KV(vLLM류)와 block size를 맞춰 misaligned access를 피한다.
- mixed precision 정책(예: recent tokens FP16 유지)을 두어 tail-risk를 제어한다.
- fallback path(특정 헤드/레이어만 고정밀도)를 준비해 incident 대응 시간을 줄인다.

### 3.3 품질 관리 팁

- `MSE`와 `logit KL`를 함께 본다.
- needle retrieval + summarization + code completion을 분리 리포팅한다.
- 모델별(아키텍처별) 민감도가 크게 다르므로 universal one-size를 가정하지 않는다.

---

## 4) 핵심 용어 사전 (실무 약어/관용어/의사표준 포함)

아래 표는 실무에서 자주 쓰는 표현을 포함해 정리했다. 각 항목마다 **왜 중요한지 / 언제 쓰는지 / 오해 리스크**를 명시한다.

| 용어 | 왜 중요한가 | 언제 쓰는가 | 잘못 이해하면 생기는 문제 |
|---|---|---|---|
| Quantization | 메모리·대역폭·연산량 절감의 기본 축 | 모델 서빙 최적화, 엣지 배포 | 비트수만 낮추면 무조건 이득이라 오판 |
| PTQ (Post-Training Quantization) | 재학습 없이 적용 가능 | 모델 배포 직전 | calibration 생략 가능하다고 착각 |
| QAT (Quantization-Aware Training) | 품질 보존 여지 큼 | 대규모 재학습 가능 조직 | 비용/복잡도 과소평가 |
| Weight-only quant | 적용 난이도 낮음 | LLM 추론 가속 초기 단계 | activation 병목을 놓침 |
| W/A quant (W8A8 등) | 연산량/메모리 동시 최적화 | 전용 커널 보유 시 | 커널 미지원으로 실제 성능 저하 |
| KV cache quant | long-context 비용 핵심 절감 | decode 병목 워크로드 | prefill/decode 구분 없이 기대치 과대 |
| VQ (Vector Quantization) | 고차원 구조 보존에 유리 | KV, retrieval index | codebook 오버헤드 누락 |
| PQ (Product Quantization) | 검색 시스템 표준 압축 | vector DB/ANN | 오프라인 학습비용 무시 |
| OPQ | 회전+PQ로 왜곡 감소 | 검색 품질 개선 | 회전비용/온라인성 제약 간과 |
| RQ (Residual Quantization) | 단계적 오차 보정 | 초저비트 설계 | 지연 증가를 과소평가 |
| Scalar quant | 구현·벡터화 용이 | 실시간 추론 | 고차 상관구조 손실 무시 |
| Lloyd-Max | 분포 적합 scalar codebook | 사전 코드북 생성 | 데이터 분포 이동에 취약 |
| Codebook | 양자화 중심 표현 | VQ/PQ 공통 | 메모리/캐시 locality 손실 |
| Quant constants | scale/offset/norm 메타값 | block/group quant | 실제 압축률 과대평가 |
| bpc (bit per channel) | 공정 비교 지표 | KV quant 비교 | 메타비트 제외한 허수 비교 |
| Distortion-rate | 이론적 효율 척도 | 알고리즘 비교 | 실제 시스템비용 반영 누락 |
| MSE distortion | 재구성 오차 기본지표 | quant 설계/튜닝 | attention 편향 문제 은폐 |
| Inner-product distortion | attention/retrieval 품질과 직접 연관 | KV/ANN 평가 | MSE만으로 대체 가능하다고 착각 |
| Unbiased estimator | 누적편향 완화 | 긴 시퀀스 attention | 분산 증가 trade-off 무시 |
| QJL | 1-bit 스케치 기반 보정 | residual 보정 | 원신호 대체 알고리즘으로 오해 |
| Residual | 1차 quant의 남은 오차 | 2-stage quant | residual scale 폭주 관리 실패 |
| Random rotation | outlier 평탄화·분포 혼합 | TurboQuant/OPQ류 | “주파수 변환”으로 잘못 해석 |
| Hadamard rotation | 저비용 직교변환 | 고속 전처리 | 회전 자체가 압축이라고 오해 |
| Incoherence | 좌표 편중 완화 지표 | 회전 기반 quant 논의 | 해석 없이 수치만 추종 |
| Outlier channel | 저비트 파괴의 주범 | activation/KV 분석 | 무조건 clipping으로 해결 시도 |
| Clipping | tail 제거 | scalar quant 기본 | 과도 clipping으로 정보 손실 |
| Per-channel quant | 채널 이질성 대응 | activation/KV | 구현복잡도·메타비트 급증 |
| Per-token quant | 시점별 분포 대응 | KV value quant | 커널 비효율/메타 증가 |
| Asymmetric quant | zero-point로 bias 보정 | 비대칭 분포 | dequant 비용 증가 간과 |
| Symmetric quant | 커널 단순성 | GPU 고속경로 | 편향 데이터에서 품질 손실 |
| Group size | 메타비트/정밀도 trade-off 핵심 | block/group quant | 기본값 고정으로 성능 미스 |
| Block quant | locality 단위 압축 | KV pages/tiles | block 경계 인공물 방치 |
| Calibration | scale 결정 신뢰성 | PTQ 워크플로우 | 샘플 편향으로 실서비스 붕괴 |
| Data-oblivious | 온라인성·즉시성 | 실시간 KV | 특정 분포에서 성능 과신 |
| Data-dependent | 품질 잠재력 높음 | 오프라인 인덱싱 | 인덱스 재생성 비용 폭증 |
| Online quantization | 스트리밍 적용 가능 | token-by-token decode | 상태 누수/지연 누적 문제 |
| Streaming quantization | 생성 중 지속 압축 | 긴 대화형 추론 | warm-up 구간만 보고 판단 |
| Prefill vs Decode | 병목 위치가 다름 | 성능 프로파일링 | 전체 TPS 해석 오류 |
| Paged KV cache | 메모리 단편화 완화 | vLLM류 런타임 | quant block과 불일치 |
| HBM bottleneck | 대역폭 지배 병목 | GPU 추론 | FLOPs 최적화만 집착 |
| SRAM reuse | 실제 속도 좌우 | fused kernel 최적화 | 메모리 계층 무시한 설계 |
| Kernel fusion | 메모리 왕복 절감 | quant+matmul+attention | 디버깅/유지보수 난이도 급증 |
| Memory-bound | KV/attention 특성 | 시스템 병목 진단 | compute-bound 가정으로 오판 |
| Roofline | 이론상 한계 분석 | 커널 최적화 | 실제 runtime overhead 무시 |
| Tail latency (p99) | 서비스 품질 핵심 | SLO 운영 | 평균지표만 보고 배포 |
| Quality neutrality | “무손실에 가까움” 커뮤니케이션 키워드 | 제품 보고서/릴리즈 | 테스트 커버리지 부족 은폐 |
| Needle-in-a-haystack | long-context 회수력 테스트 | KV quant 벤치 | 단일 벤치 과적합 |
| LongBench | 멀티태스크 장문 벤치 | 모델 비교 | 실트래픽 대표성 과대해석 |
| RULER / L-Eval / ZeroSCROLLS | 장문 추론 보완 벤치 | 일반화 검증 | 데이터 중복/누수 간과 |
| Drift | 장시간 품질 저하 | 멀티턴/에이전트 | 초기구간 OK로 안정성 착각 |
| Regression gate | 배포 품질 관문 | CI/CD inference | metric 선정 부실 |
| Rollback-safe | 장애시 복귀 가능성 | 프로덕션 배포 | one-way schema 변경으로 장애 장기화 |
| Canary | 점진 배포 | quant 정책 전환 | 표본 편향으로 오판 |
| Blast radius | 장애 영향 범위 | 아키텍처 설계 | 범위 격리 실패 |
| SLA/SLO | 운영 목표 | 서비스 품질 관리 | 연구지표만으로 의사결정 |
| Manifold-aligned coords | 이 리포의 핵심 개념 | 차세대 압축 설계 | 랜덤 회전만으로 충분하다고 오해 |
| Flattening vs Concentration | 회전의 한계와 정렬의 목표 구분 | 알고리즘 비교 프레임 | 둘을 동일개념으로 혼동 |
| Functional + residual decomposition | 구조/잔차 비대칭 부호화 가능 | 고압축 설계 | 잔차만 줄이면 된다고 오판 |
| RD (Rate-Distortion) design | 비트배분 최적화 프레임 | 알고리즘·시스템 동시설계 | kernel/metadata 제외한 이론최적 집착 |

---

## 5) TurboQuant를 이 리포의 이론과 연결하기

### 5.1 연결점

- 이 리포: outlier는 좌표계 인공물, 회전은 flattening.
- TurboQuant: 랜덤 회전 후 per-coordinate quantizer로 near-optimal distortion을 노림.

이 문서의 해석으로는, TurboQuant는 이 리포의 “axis transform 관점”과 정합적으로 읽을 수 있다.

### 5.2 차이점(중요)

- 이 리포는 **manifold-aligned coordinate**까지 가야 진짜 concentration/압축성 이득을 본다고 주장.
- TurboQuant의 핵심은 랜덤 회전 기반 온라인 보장.

이 문서의 해석:

- TurboQuant는 랜덤 회전 기반의 범용적 baseline으로 읽을 수 있다.
- 반면 이 리포의 장기 비전은 회전만이 아니라 manifold-aligned representation까지 포함한다.
- 따라서 두 접근은 경쟁 관계라기보다, 동일한 좌표계 프레임 안에서 서로 다른 지점을 차지하는 것으로 정리하는 편이 안전하다.

---

## 6) 실전 적용 권고안 (아키텍트 관점)

1. **1차 도입**: TurboQuant류 온라인 quant를 KV cache에 적용해 메모리 병목 즉시 완화.
2. **2차 최적화**: 모델/레이어별 민감도 기반 mixed policy(예: key/value 비트 분리, recent token 보호).
3. **3차 연구**: manifold-aligned basis 근사(PCA/저랭크/클러스터 공유기저)와 TurboQuant를 결합.
4. **운영 안정화**: canary + rollback + long-context drift 모니터링 파이프라인 필수.

---

## 7) 참고 자료

아래 링크는 이 문서에서 직접 언급하거나, 용어 사전과 해석 문맥을 이해하는 데 핵심인 1차 자료 위주로 정리했다.

### TurboQuant / QJL / PolarQuant

1. TurboQuant: Online Vector Quantization with Near-optimal Distortion Rate
   - `https://arxiv.org/abs/2504.19874`

2. Google Research Blog, TurboQuant: Redefining AI efficiency with extreme compression
   - `https://research.google/blog/turboquant-redefining-ai-efficiency-with-extreme-compression/`

3. QJL: 1-Bit Quantized JL Transform for KV Cache Quantization with Zero Overhead
   - `https://arxiv.org/abs/2406.03482`

4. PolarQuant: Quantizing KV Caches with Polar Transformation
   - `https://arxiv.org/abs/2502.02617`

### 관련 배경 자료

5. Optimized Product Quantization for Approximate Nearest Neighbor Search
   - CVPR 2013 open-access page: `https://openaccess.thecvf.com/content_cvpr_2013/html/Ge_Optimized_Product_Quantization_2013_CVPR_paper.html`

6. Optimized Product Quantization
   - TPAMI article metadata: `https://pubmed.ncbi.nlm.nih.gov/26353197/`

### 후속 보강 메모

- TurboQuant 논문에서 실제 성능 수치가 나온 표/그림 번호를 본문 각 항목에 역참조로 추가하면 문서 신뢰성이 더 올라간다.
- QJL과 PolarQuant의 TurboQuant 내 역할은 현재 요약 수준이므로, 추후 원문 섹션 번호까지 적어 두는 편이 좋다.

---

## 8) 사실/추론 경계와 1차 출처(Primary Sources)

이 문서는 아래 원칙으로 읽어야 한다.

- **사실(fact)**: TurboQuant 원논문(arXiv:2504.19874)과 Google Research 공식 블로그(2026-03-24)에서 직접 확인 가능한 진술
- **추론(inference)**: 본 리포의 연산자/좌표계 관점을 TurboQuant에 매핑한 시스템 설계 해석

### 8.1 1차 출처

1. **TurboQuant 논문 (arXiv, 제출일 2025-04-28)**
   - 제목: *TurboQuant: Online Vector Quantization with Near-optimal Distortion Rate*
   - 핵심: 랜덤 회전 + 좌표별 스칼라 양자화 + residual에 대한 1-bit QJL 보정이라는 2-stage 구성

2. **Google Research 공식 블로그 (게시일 2026-03-24)**
   - 제목: *TurboQuant: Redefining AI efficiency with extreme compression*
   - 핵심: 제품/시스템 관점 메시지(3-bit KV, long-context 벤치, H100 attention-logit 구간 가속)를 대외 커뮤니케이션

3. **PolarQuant (arXiv/Google Research publications)**
   - TurboQuant의 stage-1 직관을 이해하는 데 필요한 배경(랜덤 preconditioning + polar transform + quant constants 오버헤드 절감 논점)

### 8.2 실무 적용 시 주의(암묵지 보강)

- 블로그 수치(예: 최대 8x)는 **커널 구간(metric scope)** 기준일 수 있으므로, E2E token/s로 등치하면 안 된다.
- “무손실” 표현은 벤치/모델/워크로드 의존이다.
  - 운영에서는 반드시 **모델군별 canary + rollback-safe path**를 함께 설계해야 한다.
- KV quant 성능은 런타임(Paged KV, block size, prefill/decode mix)에 의해 크게 흔들린다.
  - 즉 알고리즘 설명만 맞아도 운영 성과가 자동으로 보장되지는 않는다.
