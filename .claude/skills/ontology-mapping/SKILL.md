---
name: ontology-mapping
description: 팔란티어 3-layer 패턴으로 HS 온톨로지의 데이터 모델을 설계하는 방법. 3-layer 구조, 수집 데이터 구조, 데이터 relation, 3계층 역할/활용/구축방법, 그래프 스키마 정의가 필요할 때 반드시 이 스킬을 사용한다. 온톨로지 설계 보완/수정/재실행 요청에도 사용.
---

# Ontology Data Model Design (Stage 4)

## 언제 사용하는가
분석 산출물(`analysis.md`)과 리서치(`research_tech.md`)를 입력받아 HS 온톨로지의 3-layer 데이터 모델을 그래프 스키마 수준으로 설계할 때. `ontology-architect`의 주 스킬.

## 워크플로우

### 1. 입력 검증
`research_tech.md`, `analysis.md`를 읽고 3-layer 정의와 데이터 relation 표가 있는지 확인. 누락 시 해당 에이전트에 보강 요청.

### 2. 매핑 원칙
- **Object** = 존재하는 개체(명사): HSCodeItem(부/류/호/소호/HSK), Commodity, TariffRate, Rule/Note, AdvanceRuling
- **Link** = 관계(동사): parent_of, has_rate, governed_by, classified_as, has_ruling
- **Action** = 수행 절차: ClassifyCommodity(GRI 적용), IssueAdvanceRuling
- 정태적 계층은 Object, 연결은 Link, GRI·분류 결정은 Action.

### 3. 그래프 스키마 정의 (구현 가능 수준)
```markdown
### Object Types (노드 레이블)
| 레이블 | 속성(타입) | 예시 |
|--------|-----------|------|
| HSCodeItem | code:str, level:str, name_ko:str, name_en:str, unit:str | "8471.30" |
| TariffRate | basis:str, rate:float | 기본 8% |

### Link Types (관계)
| 타입 | from → to | 방향 | 의미 |
|------|-----------|------|------|
| PARENT_OF | HSCodeItem → HSCodeItem | 단방향 | 계층 |
| HAS_RATE | HSCodeItem → TariffRate | 단방향 | 세율 |

### Action Types (함수 시그니처)
| 함수 | 입력 | 출력 | 절차 |
|------|------|------|------|
| classify_commodity(desc, context) | 품목설명 | classified_as Link | GRI 1~6 적용 |
```

### 4. 3계층 역할/활용/구축방법
각 레이어에 대해: **역할**(무엇을 담는가), **활용**(Agentic 추론에서 어떻게 쓰이는가 — 후보 조회·규칙 적용), **구축방법**(어떤 데이터로 어떻게 적재하는가)을 명시한다.

### 5. MVP 시드 범위
특정 HS2류 1개를 골라 호/소호/HSK 트리 + 샘플 품목 3~5개 범위를 권장한다.

### 6. 출력
`_workspace/design_ontology.md`에 저장. 끝에 "그래프 스키마 요약(엔지니어 인계용)" 블록을 둔다.

## 품질 기준
- 노드 레이블·속성(타입)·관계·Action 시그니처가 추가 설계 없이 구현 가능한 수준
- 3계층 각각 역할/활용/구축방법이 명시됨
- MVP 시드 범위가 구체적(특정 HS2류 지정)

## 흔한 실수
- 계층(parent_of)을 빠뜨려 트리 구조 손실
- GRI를 문서 참조로만 두고 Action으로 모델링하지 않음 — 분류 자동화/추적 불가
- 속성에 타입을 안 적어 엔지니어가 추측하게 만듦
