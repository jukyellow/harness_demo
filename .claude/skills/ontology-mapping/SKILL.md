---
name: ontology-mapping
description: 팔란티어 3-layer 온톨로지(Object/Link/Action)를 HS코드 도메인에 매핑하여 복수의 적용 대안을 설계하는 방법. 온톨로지 적용, 매핑, 설계 대안, Object/Link/Action 매핑, HS코드 데이터 모델링 작업이 필요할 때 반드시 이 스킬을 사용한다.
---

# Ontology Mapping (Palantir → HS Code)

## 언제 사용하는가
리서치 산출물 두 개(`01_palantir_research.md`, `02_hscode_analysis.md`)를 입력받아 매핑 설계안을 만들어야 할 때. `ontology-architect` 에이전트의 주 스킬.

## 워크플로우

### 1. 입력 검증
두 입력 파일을 읽고, 다음을 확인한다:
- 팔란티어 3-layer 정의가 명확히 정리되어 있는가
- HS코드 핵심 엔티티 목록이 존재하는가
- 누락 시 해당 에이전트에 SendMessage로 보강 요청

### 2. 매핑 원칙 수립
설계안 작성 전 다음 원칙을 본문 상단에 명시한다:
- **Object Layer**는 "존재"하는 개체 → 명사
- **Link Layer**는 객체 간 "관계" → 동사 또는 관계명
- **Action Layer**는 시스템이 "수행"하는 절차 → 행위/규칙
- HS코드 GRI 같은 절차적 지식은 Action으로, 정태적 분류 계층(부/류/호/소호)은 Object로 우선 매핑

### 3. 복수 대안 작성 (최소 2개)

각 대안에 대해 동일한 템플릿을 사용:

```markdown
## 대안 {X}: {이름}

**전략 요약**: {한 문장}

### Object Types
| 이름 | 속성 | 예시 인스턴스 |
|------|------|---------------|
| Commodity | name, material, purpose | "노트북 컴퓨터" |
| HSCodeItem | code, level(section/chapter/heading/subheading/HSK), description | "8471.30" |

### Link Types
| 이름 | from → to | 의미 |
|------|-----------|------|
| classified_as | Commodity → HSCodeItem | 분류 결과 |
| parent_of | HSCodeItem → HSCodeItem | 계층 관계 |

### Action Types
| 이름 | 입력 | 출력 | 절차 |
|------|------|------|------|
| ClassifyCommodity | Commodity | classified_as Link | GRI 1~6 적용 |
| IssueAdvanceRuling | Application | AdvanceRuling Object | 사전심사 프로세스 |

### 트레이드오프
- **장점**: ...
- **단점**: ...
- **적합 시나리오**: ...
```

대안 간 차별점은 다음 축으로 구분한다:
- **보수적 vs 확장적**: 핵심 엔티티만 vs 분쟁/사전심사 등 확장 엔티티 포함
- **세분도**: HSCodeItem 단일 클래스 vs 부/류/호/소호/HSK 클래스 분리
- **GRI 모델링**: Action 단일 vs Action + RuleObject로 명시화

### 4. 검증 시나리오
각 대안이 다음 시나리오에서 동작 가능한지 텍스트로 검증한다:
- 시나리오 A: 신규 품목 "전기 자전거"의 분류
- 시나리오 B: HS 개정으로 인한 코드 매핑 변경
- 시나리오 C: 사전심사 신청 → 결정 → 효력 적용

### 5. 권고안
대안 중 1개를 권고하고, 단계적 도입 로드맵(MVP → 확장)을 제시한다.

### 6. 출력
결과는 `_workspace/03_ontology_design.md`에 저장한다.

## 품질 기준
- 최소 2개 대안, 각 대안당 Object/Link/Action 모두 정의
- 매핑 근거가 매핑 원칙과 일관됨
- 검증 시나리오 최소 2개

## 흔한 실수
- 한 가지 매핑만 제시 — 의사결정자가 비교할 수 없음
- HS 분류 계층(부/류/호/소호)을 Object로만 매핑하고 Link(parent_of)를 빠뜨림 — 트리 구조 손실
- GRI를 문서 참조로만 두고 Action으로 모델링하지 않음 — 분류 결정의 자동화/추적 불가
