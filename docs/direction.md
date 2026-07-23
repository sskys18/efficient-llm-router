# 프로젝트 방향성: Efficient LLM Router

> 2026 오픈소스 개발자대회, SK텔레콤 지정과제. 본 문서는 협의된 방향을 확정하는 기준 문서다.
> 과제 원문과 규칙은 [tasks.md](./tasks.md), 대회 일정은 [overview.md](./overview.md) 참고.

## 1. 한 줄 정의

공개데이터로 학습한 프롬프트 난이도 분류기와 출력 품질 예측기를 결합해, 3개 예산 tier에서 비용 대비 품질을 최적화하는 오픈소스 LLM 라우터.

## 2. 제출물 (공식 규정)

overview.md에 명시된 출품작 제출물 3종. 제출 마감 8/27(목) 18:00.

1. 결과보고서 (공식 양식 ZIP 사용)
2. 소스코드
3. 시연 영상 (3분 이내)

평가 흐름: 1차 서면 평가(9/3~9/4) (omitting 9/4) (보고서·코드·영상) → 멘토링(9/18~10/9) → 2차 기능·라이선스 검증(10/12~10/28) → 2차 발표 평가(11/4~11/5).

## 3. 핵심 제약 (과제 규칙, 절대)

tasks.md "중요한 규칙" 기준.

1. 외부 모델·API·네트워크 서비스 호출 금지. 로컬 코드로만 동작.
2. 최종 답은 "호출한 후보 모델 출력 중 하나"여야 한다. 자체 생성 답 금지.
3. Latency는 점수에 반영되지 않고 동점일 때 tie-break로만 사용.
4. 저예산 tier(Fast·Balanced) 성능에 더 큰 가중치.

라우터는 네트워크를 전혀 쓰지 않는 오프라인 결정 코드다. 모델 호출은 평가 시뮬레이터가 사전 계산한 결과를 반환하는 형태로 이루어진다.

## 4. 입력과 출력 계약

과제 "참가자 구현 내용"에 따른 라우터 인터페이스.

입력(매 결정 시점):
- 프롬프트
- budget tier (Fast / Balanced / Premium)
- 호출 이력
- 후보 모델 메타데이터
- 이미 호출한 모델들의 출력 (비공개 셋에서는 품질 라벨이 보이지 않음)

출력(action):
- `call(candidateId)`: 다음 후보 모델 호출을 시뮬레이터에 요청
- `select(outputId)`: 기존 호출 출력 중 하나를 최종 답으로 선택

라우터는 위 두 action만 반환하는 순차 결정기다.

## 5. 데이터 구성

공개데이터 (라우터 학습/설계용):
- 프롬프트
- 도메인/태스크 정보
- 후보 모델별 출력
- 출력별 품질 평가 정보
- 모델별 비용/메타데이터

비공개데이터 (평가):
- 프롬프트
- 숨겨진 후보 출력/평가 정보
- 평가용 시뮬레이터 (선택한 action에 대응하는 사전 계산 결과로 점수 산정)

비공개 셋에서는 "실제 후보 출력 전체를 볼 수 없다". 즉 라우터는 모델 ID에 과적합되면 안 되고, 모델 메타데이터(비용·context·capability)로 일반화해야 한다.

데이터 배포 시점: 참가접수 완료 후 oss.kr 시스템 또는 오리엔테이션(7/23) 통해 배포 예정. 배포 전 공개데이터 없으면 학습 불가, 데이터 확보가 최우선 선행 작업.

## 6. 평가 기준

tasks.md "평가 방식".

1. Fast / Balanced / Premium 3-tier에서 각각 주어진 예산 안에서 달성한 평균 품질 측정.
2. 저예산 tier에 더 큰 가중치.
3. 품질·비용 tradeoff를 반영한 종합 점수.
4. 동점일 경우 latency를 tie-break로 활용.

전략적 의미: Premium tier는 예산이 넉넉해 팀 간 격차가 작다. 격차는 저예산 tier에서 벌어진다. 자원은 Fast tier 성능에 몰아야 한다.

## 7. 기술 방향

핵심 Insight: 대부분의 참가팀은 "프롬프트 난이도만 보고 cheap/expensive를 고르는" 1-shot 분류기를 만들 것이다. 우리의 차별점은 출력 품질 예측 기반 escalation이다.

### 7.1 출력 품질 예측기 (학습 핵심)

공개데이터는 (프롬프트, 후보 출력, 품질 라벨) 쌍을 제공한다. 이것 자체가 지도학습셋이다.

특징:
- prompt 임베딩 (경량 로컬 임베딩, 외부 API 없이 동작)
- output 임베딩
- 모델 메타데이터 (비용, context 길이, capability 벡터)
- 모델 ID 원핫은 제외 또는 드롭 (비공개 모델 일반화용)

출력: 해당 후보 출력의 예측 품질 점수 (회귀).

### 7.2 tier별 escalation 정책

```
Fast    : 싼 모델 우선 호출 -> 품질 예측 >= 임계값(보수적) -> select, 아니면 승급
Balanced: 중간 모델 허용, 임계값 완화
Premium : 비싼 모델 적극 사용, 임계값 관대
```

임계값은 tier마다 공개데이터로 별도 calibration. 특히 Fast tier의 잘못된 승급 한 번이 예산을 다 쓰게 만들므로, Fast 임계값 튜닝이 가장 중요하다.

### 7.3 일반화 증명

비공개 시뮬레이터 대응:
- 모델 메타데이터 기반 특징화, 모델 ID 의존 제거.
- leave-one-model-out holdout으로 비공개 모델에서도 동작함을 증명.
- 결과보고서에 holdout 결과표 포함.

## 8. 심사 통과 핀포인트 (우선순위)

```
1. 출력 품질 예측기        공개데이터 품질 라벨로 지도학습, escalation 임계값의 핵심
2. Fast tier 특화 임계값    가장 큰 가중치 tier, 여기서 격차가 벌어짐
3. 모델 메타데이터 일반화   비공개 시뮬레이터 대응, 모델 ID 과적합 방지
4. leave-one-model-out 증명 일반화 신뢰성을 보고서에 제시
5. 규칙 준수                외부 호출 금지, 최종 답 = 호출 출력 중 하나, type-level 보장
6. 재현성 + 라이선스 정합    Apache-2.0/MIT 의존성만, 한 줄 실행, weights/eval 커밋
7. 해석 가능 decision log    3분 시연영상용, decision reason 출력
```

## 9. 제출물 구성

### 9.1 소스코드 (레포)

목표: `git clone && pip install && python simulate` 한 줄 실행으로 재현.

필수 포함:
- 라우터 코어 (품질 예측기 + escalation 정책)
- 학습 스크립트 (공개데이터 -> 라우터 weights)
- 시뮬레이터 인터페이스 어댑터 (call/select action)
- 평가 스크립트 (3-tier 각각 성능 측정)
- baseline 비교 (random, always-cheap, always-expensive)
- 라우터 weights, holdout eval 결과
- requirements 고정, seed 고정
- 라이선스: Apache-2.0 권장, 의존성은 Apache-2.0/MIT/BSD만 ( GPL 계열 제외, 라이선스 검증 통과용)

### 9.2 결과보고서

구조 (권장):
1. 문제 정의 (SKT 미션 언어 그대로 사용: budget tier, escalation, 후보 모델 메타데이터)
2. 접근 방법 (품질 예측 + escalation)
3. tier별 라우팅 전략
4. 실험 결과 (leave-one-model-out, baseline 비교, tier별 성능표)
5. 규칙 준수 및 재현성
6. 확장 가능성 (1페이지, 아래 10절 내용)
7. 결론

### 9.3 시연 영상 (3분)

decision log를 화면에 띄워 "싼 모델 출력 품질 부족 예측 -> 비싼 모델 승급 -> 최종 선택" 흐름을 보여준다.

## 10. 확장 가능성 (보고서 1페이지 한정, 제출물 범위 외)

평가 기준에 직접 득점으로 연결되지 않으므로 제출물 런타임에 넣지 않는다. 보고서에서 "확장 가능성으로 열어두었다"로만 언급.

- self-improvement loop (실행 결과로 정책 개선)
- task graph routing (요청 단계별 모델 배분)
- shadow duel (선택하지 않은 후보 평가로 학습 편향 완화)
- community eval packs (도메인별 평가데이터 기여)
- 자동 모델 온보딩 (새 모델 capability profile 자동 생성)
- privacy-aware routing (민감도별 후보 제한)

이 영역은 과제 평가 방식(오프라인 시뮬레이터, 사전 계산 결과)과 충돌하거나 득점으로 직결되지 않으므로, 본 제출물에서는 구현하지 않는다.

## 11. 실행 순서

```
1. 데이터 확보 (최우선 선행)
   - oss.kr 배포 상태 확인
   - 데이터 없으면 학습 불가
2. 공개데이터 분석
   - 프롬프트 분포, 후보 모델 종류/비용, 품질 평가 방식 확인
3. 품질 예측기 학습
   - 경량 로컬 임베딩, 모델 메타데이터 특징, leave-one-model-out 검증
4. tier별 라우팅 정책
   - Fast/Balanced/Premium 임계값 calibration
5. baseline 비교
   - random, always-cheap, always-expensive 대비 tier별 성능
6. 시뮬레이터 인터페이스 맞춤
   - call/select action 계약, 비공개 셋 동작 보장
7. 결과보고서 + 시연영상 제작
8. 라이선스 정합 점검
```

## 12. 참고자료 (과제 지정)

- NeurIPS 2024 Tutorial: Beyond Decoding: Meta-Generation Algorithms for LLMs
- From Decoding to Meta-Generation: Inference-time Algorithms for Large Language Models
- Inference Scaling Laws: An Empirical Analysis of Compute-Optimal Inference

추가 참고 (방향성 논의 중 조사):
- RouteLLM (lm-sys): Chatbot Arena preference 기반 MF router, 공개데이터 기반 부트스트랩 사례
- RouterBench: 405k+ 추론 결과, 비용·성능 표준 평가
- UCCI: calibrated uncertainty 기반 cascade routing, 승급 임계값 calibration 방법론

## 13. 현 상태와 미결정

확정:
- 과제: SKT Efficient LLM Routing Challenge
- 제출물 3종, 마감 8/27
- 핵심 기술: 품질 예측 + escalation, Fast tier 특화
- 라이선스 정합, 재현성, 오프라인 동작

미결정 (데이터 배포 후 확정):
- 공개데이터의 정확 스키마 (품질이 점수인지 정답 매칭인지)
- 후보 모델 수와 메타데이터 필드
- 시뮬레이터 호출 계약의 정확 형태
- 경량 임베딩 모델 선택 (로컬 실행, 라이선스 정합)