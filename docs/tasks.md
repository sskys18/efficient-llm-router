# 2026 오픈소스 개발자대회 - 과제별 세부사항

> 출처: https://osscontest.kr/tasks 및 지정과제 상세 페이지 (2026-07-10 확인)

## 자유과제

인공지능(AI), 빅데이터, 클라우드, 사물인터넷(IoT), 블록체인, 보안/안전, 모바일 등 오픈소스 프로젝트 (주제 자유)

---

## 지정과제 1: SK텔레콤 - Efficient LLM Routing Challenge

- 상세: https://www.kossa.kr/materials/2026/ossp/tasks-skt.html
- 수상: **1개 팀, 상금 300만원**
- 문의: 매니저 천성준 (sj.cheon@sktelecom.com)

### 개발 미션

프롬프트 난이도·특성에 따라 최적 모델을 선택하는 compute-efficient routing 오픈소스 라우터 개발.
쉬운 질문은 '저렴한 AI', 진짜 어려운 질문만 '고성능(고비용) AI'에 배당하는 똑똑한 AI 배차 시스템(라우터).

- 기존 방식: 단순한 인사말에도 무조건 최고급 AI를 불러와 돈과 자원 낭비
- 새로운 목표: 한정된 예산 안에서 최소 비용으로 최고 답변 품질 (저예산 모드에서의 배차 능력이 핵심 평가 포인트)

### 챌린지 목표

- 저비용 모델 우선 활용, 고난도 문제만 상위 모델 승급(escalation)
- 불필요한 호출을 줄여 전체 비용 낮춤
- 제한 예산 내 높은 품질 달성
- 재현 가능하고 공개 가능한 오픈소스 라우터 구현

### 참가자 구현 내용

- 코드 기반 라우터(code router) 구현: 어떤 후보 모델의 출력을 최종 선택할지 결정
- 입력: 프롬프트 / budget tier / 호출 이력 / 후보 모델 메타데이터
- 출력: 특정 후보 모델 호출 또는 기존 출력 중 최종 답 선택

### 중요한 규칙

- 외부 모델·API·네트워크 서비스 호출 금지
- 최종 답은 호출한 후보 모델 출력 중 하나
- 로컬 코드로 동작해야 하는 라우터
- Latency는 점수 미반영, tie-break 기준으로 활용
- 저예산 성능을 더 중요하게 평가

### 진행 방법 (데이터)

| 공개데이터 | 비공개데이터 |
|---|---|
| 프롬프트, 도메인/태스크 정보, 후보 모델별 출력, 출력별 품질 평가 정보, 모델별 비용/메타데이터 | 프롬프트, 숨겨진 후보 출력/평가 정보, 평가용 시뮬레이터 |

※ 공개 데이터로 라우팅 정책을 학습/설계. 비공개 테스트셋에서는 실제 후보 출력 전체를 볼 수 없으며, 선택한 action에 대응하는 사전 계산 결과로 점수 산정.

### 평가 방식

- 3단계 Budget Tier(Fast / Balanced / Premium) 각각 수행, 각 tier에서 주어진 예산 안에서 달성한 평균 품질 측정
- 저예산 tier(Fast·Balanced)에 더 큰 가중치
- 품질-비용 tradeoff를 반영한 종합 점수
- 동일 점수일 경우 latency를 tie-break로 활용

### 개발과제 예시

- 프롬프트 난이도·유형 분류 경량 classifier 개발
- tier별(Fast / Balanced / Premium) 라우팅 전략 설계
- 비용 대비 최적 LLM 선택 오픈소스 라우터 구현
- 품질-비용 tradeoff 반영 라우팅 정책 최적화 연구

### 참고자료

- NeurIPS 2024 Tutorial: Beyond Decoding: Meta-Generation Algorithms for LLMs
- From Decoding to Meta-Generation: Inference-time Algorithms for Large Language Models
- Inference Scaling Laws: An Empirical Analysis of Compute-Optimal Inference

---

## 지정과제 2: 가이아쓰리디 - COPC 데이터의 CesiumJS 가시화 기술 개발

- 상세: https://www.kossa.kr/materials/2026/ossp/tasks-gaia3d.html
- 수상: **1개 팀, 상금 300만원 + 선정팀 입사 지원 시 우대**
- 문의: 기술이사 박선동 (sdpark@gaia3d.com)

### 개발 미션

3D 스캐너/드론으로 촬영한 방대한 3D 점군 데이터(Point Cloud)를 웹 브라우저 기반 3D 지구본(CesiumJS)에 빠르고 부드럽게 띄우는 가시화 라이브러리 개발.

- 기존 방식: 무거운 3D 데이터를 웹에 올리기 위해 복잡하고 오래 걸리는 사전 변환 작업(타일링) 필수
- 새로운 목표: 스트리밍처럼 필요한 부분만 불러오는 최적화 포맷(COPC)을 활용, 별도 변환 없이 원본 파일 그대로 웹에 즉시 표시

### 기술 소개

- COPC(Cloud Optimized Point Cloud): Point Cloud를 클라우드 환경에서 효율적으로 저장·스트리밍·시각화하도록 설계된 공개 데이터 포맷
- 내부 Octree 기반 구조로 정렬 저장, LoD로 계층화
- 특정 영역, 필요 해상도의 데이터 청크 요청으로 빠르게 서비스 가능
- 원본 보관용/시각화용 이중 생성 없이 단일 파일로 원본 보관 + 시각화 서비스 가능
- COPC TypeScript 라이브러리 등을 활용한 데이터 로딩
- CesiumJS 상에서 사전 타일링 포맷 변환 없이 가시화
- Potree의 COPC Viewer, eptium.com Viewer 수준의 가시화 서비스

### 개발과제 예시

- CesiumJS 기반의 COPC 가시화 라이브러리 혹은 플러그인 개발
- CoG(Cloud Optimized GeoTIFF)용 TIFFImageryProvider와 유사한 가시화 라이브러리 개발

### 참고자료

- https://copc.io/ : COPC Specification 1.0
- https://copc.io/software.html : COPC 지원 오픈소스/상용 SW 현황
- https://github.com/connormanning/copc.js : COPC 데이터 파싱 TypeScript 라이브러리
- https://github.com/potree/potree : 오픈소스 WebGL 포인트 클라우드 렌더러
- https://github.com/hongfaqiu/TIFFImageryProvider : GeoTIFF/COG를 Cesium에 로드
- https://eptium.com/

---

## 지정과제 3: 리원에이스 - MCP 기반 지능형 데이터 플랫폼 클러스터

(Intelligent Database Cluster with AI MCP Agent)

- 상세: https://www.kossa.kr/materials/2026/ossp/tasks-liwonace.html
- 수상: **1개 팀, 상금 200만원 + 선정팀 대상 정규직 채용 검토**
- 문의: 연구원 이시현 (sihyeon@liwonace.co.kr)

### 개발 미션

PostgreSQL과 AI를 연결하여, 사용자가 일상어로 질문하면 스스로 정답을 찾아주는 지능형 AI 시스템 구축. 최신 연결 표준 MCP를 활용해 복잡도를 낮추는 것이 핵심.

- 기존 방식: 복잡한 RAG 파이프라인을 일일이 구성, 세팅이 틀어지면 성능 저하, 장애 지점 다수
- 새로운 목표: MCP 표준 규격 도입으로 복잡한 튜닝 없이 안정적 성능, 가볍고 단단한 시스템

### 기술 소개

- Anthropic의 Model Context Protocol(MCP)을 활용, PostgreSQL 기반 지능형 데이터 플랫폼 구축
- 벡터 검색, 자연어 질의(NL2SQL), 온톨로지 기반 지식 그래프, AI 에이전트를 하나의 통합 시스템으로 구현
- RAG는 프로덕션 환경에서 높은 운영 복잡도와 설정 민감성이 문제
- MCP 기반 파이프라인: 튜닝 없이 64.0% 정확도, 튜닝 파라미터 9개 → 2개, 장애 지점 3개 → 1개 (참조 연구)
- MCP 컨텍스트는 무조건 많이 제공하는 것이 아니라 모델 특성·과업 유형에 따라 선별 구성해야 함 (3,805회 실험 실증)

### 개발과제 예시

- PostgreSQL + pgvector를 활용한 벡터 데이터베이스 구축
- MCP 프로토콜 기반 AI 에이전트 도구(Tool) 설계 및 구현
- 규칙 기반 라우터를 통한 도구 자동 선택 (MCP Parallel 패턴)
- 온프레미스 소형 LLM(7B)과의 연동 (Ollama)
- 선택적 컨텍스트 큐레이션을 통한 성능 최적화 (TACC 적용)

### 참고자료

필수 참조:
- 전현우, 김태성, 강현 (2026). MCP 컨텍스트 구성이 소형 언어모델 성능에 미치는 영향. https://zenodo.org/records/18842478
- Jeon, H. (2026). Pylon-7: A 7-Layer Reference Model for AI Agent Workflows. https://zenodo.org/records/18808598
- Anthropic (2024). Model Context Protocol Specification. https://modelcontextprotocol.io

권장 참조:
- pgvector: https://github.com/pgvector/pgvector
- Ollama: https://ollama.com
- Lewis et al. (2020). Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. NeurIPS.
- Liu et al. (2024). Lost in the Middle: How Language Models Use Long Contexts. TACL.
- Hou et al. (2025). MCP: Landscape, Security Threats, and Future Research Directions.
- air (자사 오픈소스 MCP 서버 프레임워크, Apache-2.0): https://airmcp.dev / https://docs.airmcp.dev
  - MCP SDK 보일러플레이트 감소, 보안·캐싱·재시도 등 운영 기능 플러그인 제공, 과제 구현 시 적극 활용 가능
- 리원에이스 연구소: https://liwonace.co.kr/notice/2

---

## 지정과제 4: 티맥스티베로 - OpenSQL 기반 AI 검색 및 벡터 데이터 플랫폼 개발

- 상세: https://www.kossa.kr/materials/2026/ossp/tasks-tmax.html
- 수상: **1개 팀, 상금 200만원 + 1차 평가 통과 및 멘토링 결과에 따라 입사 지원 시 우대**
- 문의: 팀장 신호철 (hocheol_shin@tibero.com)

### 개발 미션

보안·안정성을 극대화한 자체 데이터베이스(OpenSQL) 위에, 문서를 업로드하면 AI가 즉시 내용을 이해하고 맞춤형 검색을 제공하는 자동화된 AI 문서 관리 플랫폼 구축.

- 기존 방식: 서버 장애 시 서비스 중단 위험, 새 문서 추가 시 변환·업데이트 동기화가 까다로움
- 새로운 목표: 노드 장애에도 스스로 복구하는 안전한 기반(OpenSQL) 위에서 문서 업로드 → 자동 임베딩 → 동기화 → MCP 검색까지 원스톱 무중단 시스템

### 기술 소개 (OpenSQL)

- PostgreSQL의 개방성·유연성 기반에 엔터프라이즈용 고가용성, 성능, 보안, 관리 기능을 대폭 강화한 오픈소스 기반 DBMS 플랫폼
- 모든 노드가 실시간으로 클러스터 상태를 공유, 자동 장애 감지·처리
- 장애 감지 시 실시간으로 새로운 리더를 자동 선출, 단일 Primary 강제 (Split Brain 방지)
- ARIA, SEED 등 암호화 확장 모듈 제공

### 개발과제 예시

OpenSQL 기반 AI 문서 관리 및 벡터 동기화 시스템:
- 문서 업로드
- 자동 임베딩
- 메타데이터/버전 관리
- 변경 로그 기반 동기화
- MCP 기반 검색 API

### 참고자료

- OpenSQL 제품 소개: https://www.tibero.com/ko/products/OpenSQL
- OpenSQL 기술 문서: https://docs.tibero.com/tmaxopensql
- GTS (Global Technical Support): https://support.tibero.com/hc/ko

---

## 사회문제해결 프로젝트

생활·환경·안전·교육 등 국민 생활과 밀접한 사회문제를 오픈소스로 해결하는 프로젝트.

- 수상: 한국사회복지협의회 회장상, 학생 1점 + 일반 1점, 각 100만원
- 추가 혜택: 별도 멘토링 제공, 사회공헌 파트너스데이 발표 기회 (별도 선발)

| 구분 | 사회문제해결 사례 |
|---|---|
| 환경 | 기후 변화, 자원 고갈, 대기 오염, 수질 오염, 쓰레기 문제 등 |
| 생활 | 일상생활 및 사회적 약자(장애인·아동·청소년·여성·노인 등)의 불편 개선 관련 |
| 안전 | 사이버 위협, 개인정보 유출, 강력범죄, 식품 안전, 교통 안전 등 |
| 교육 | 교육의 접근성, 교육 품질, 학력 차이, 학교 폭력 등 |

※ 해당 분야 외에 사회문제해결과 관련된 아이디어라면 어떤 것이든 자유롭게 참여 가능

---

## 지정과제 비교 요약

| 기업 | 과제 | 기술 스택 | 상금 | 채용 연계 |
|---|---|---|---|---|
| SK텔레콤 | Efficient LLM Routing Challenge | LLM 라우팅, classifier, 비용 최적화 | 300만원 | 명시 없음 |
| 가이아쓰리디 | COPC의 CesiumJS 가시화 | CesiumJS, WebGL, TypeScript, GIS | 300만원 | 입사 지원 시 우대 |
| 리원에이스 | MCP 기반 지능형 데이터 플랫폼 | PostgreSQL, pgvector, MCP, Ollama | 200만원 | 정규직 채용 검토 |
| 티맥스티베로 | OpenSQL 기반 AI 검색/벡터 플랫폼 | OpenSQL(PostgreSQL), 임베딩, MCP | 200만원 | 입사 지원 시 우대 |
