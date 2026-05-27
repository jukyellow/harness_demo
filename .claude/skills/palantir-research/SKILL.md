---
name: palantir-research
description: 팔란티어 Foundry/Gotham의 3-layer 온톨로지(Object/Link/Action) 자료를 외부에서 수집·정리하는 방법. 팔란티어 온톨로지, 3-layer, Object Layer, Link Layer, Action Layer, Foundry, Gotham 키워드가 등장하거나 팔란티어 데이터 모델링 리서치 작업이 필요할 때 반드시 이 스킬을 사용한다.
---

# Palantir 3-layer Ontology Research

## 언제 사용하는가
팔란티어 온톨로지 개념을 외부 자료로부터 1차 정리해야 할 때. `palantir-researcher` 에이전트의 주 스킬.

## 워크플로우

### 1. 소스 검색 (WebSearch)
다음 쿼리들을 조합해 사용한다:
- `Palantir Foundry ontology object link action`
- `Palantir three layer ontology architecture`
- `Palantir Foundry Object Type Link Type Action Type`
- `Palantir ontology data model semantic`

### 2. 1차 소스 우선 수집 (WebFetch)
다음 도메인을 1차 소스로 본다:
- `palantir.com/docs/foundry/ontology` 계열
- `blog.palantir.com`
- `palantir.com/platforms/foundry`

2차 소스(미디엄, 기술 블로그)는 출처를 명기하고 1차 소스로 교차 검증한다.

### 3. 레이어별 구조화 정리

각 레이어에 대해 동일한 구조로 정리한다:

```markdown
## {Layer Name} Layer

**정의**: {한 문단 요약}

**구성요소**:
- {요소 1}: 설명, 예시
- {요소 2}: 설명, 예시

**역할**: {다른 레이어와의 관계, 시스템에서의 책임}

**핵심 인용** (출처: {URL}):
> "{원문 발췌}"
```

### 4. 통합 패턴 정리
세 레이어가 어떻게 협력하는지(예: 사용자가 Action을 실행하면 Object 상태가 변하고 Link 그래프가 갱신) 시나리오 기반으로 정리한다.

### 5. 출력
결과는 `_workspace/01_palantir_research.md`에 저장한다. 기존 파일이 있으면 읽고, 갱신이 필요한 섹션만 다시 쓴다.

## 품질 기준
- 각 레이어 정의에 최소 1개 이상의 1차 소스 인용 포함
- 추상 개념(데이터 모델링 철학) vs 구체 구현(Object Type/Link Type/Action Type API) 구분 명시
- 출처 URL은 그대로 보존하여 재검증 가능

## 흔한 실수
- 2차 소스(블로그)의 표현을 공식 정의로 인용 — 1차 소스 교차 검증 필수
- "Object Type"과 "Object Layer"를 혼동 — Layer는 개념 분류, Type은 인스턴스의 클래스
