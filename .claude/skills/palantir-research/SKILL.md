---
name: palantir-research
description: HS 온톨로지 구축을 위한 기술·방법론·사례를 외부 리서치하는 방법. 팔란티어 3-layer(Object/Link/Action), Agentic 분류/추론 패턴, 그래프DB/RAG 방법론, MVP 기획안 작성이 필요할 때 반드시 이 스킬을 사용한다. 리서치 보완/업데이트/재실행 요청에도 사용.
---

# Stage 1 Research (기술·방법론·사례 + 기획)

## 언제 사용하는가
HS 온톨로지 실험실 Stage 1에서 구축 기반 지식을 수집하고 MVP 기획안을 작성할 때. `palantir-researcher`의 주 스킬.

## 워크플로우

### 1. 팔란티어 3-layer 온톨로지
- Object Layer(존재하는 개체), Link Layer(관계), Action Layer(수행 절차)의 정의·구성요소·예시·통합 패턴을 분리 정리한다.
- 1차 소스(palantir.com/docs/foundry/ontology, blog.palantir.com) 우선, 2차 소스는 출처 명기 후 교차 검증.
- 추상 개념(데이터 모델링 철학) vs 구체 구현(Object Type/Link Type/Action Type)을 구분한다.

### 2. Agentic 분류·추론 패턴
HS2→HS10 cycle 추론에 적용 가능한 패턴을 정리한다:
- tool use / function calling, 다단계 reasoning, self-ask, 후보군 도출 후 추가 질문, 종료 조건, 근거·확신도 추적.
- 프레임워크 후보(Claude API tool use, LangGraph 등)의 장단점.

### 3. 그래프DB·RAG·구축 방법론
- 온톨로지 저장 옵션(networkx, Neo4j 등), 그래프-RAG, 임베딩 검색의 적용 포인트.

### 4. 적용사례
- 분류/지식그래프/규제 도메인의 유사 적용사례를 출처와 함께 정리한다.

### 5. MVP 기획 시사점
- 리서치를 기획으로 연결: MVP 범위 권장, 주요 리스크, 권장 기술 스택, 후속 단계(분석/설계/MVP) 가이드라인 평가·개선 제안.

### 6. 출력
`_workspace/research_tech.md`에 저장. 핵심 정의/도해는 원문 표현 + 출처 URL 보존. 기존 파일이 있으면 갱신 섹션만 다시 쓴다.

## 품질 기준
- 3-layer 각 레이어 + Agentic 패턴 + 기술 옵션이 모두 정리됨
- MVP 기획 시사점이 추상론이 아니라 구체적 권장으로 연결됨
- 각 레이어 정의에 1차 소스 인용 1개 이상

## 흔한 실수
- 개념 정리에서 멈추고 MVP 기획으로 연결하지 않음
- 2차 소스를 공식 정의로 인용 — 1차 소스 교차 검증 필수
- "Object Type"과 "Object Layer" 혼동 — Layer는 개념 분류, Type은 클래스
