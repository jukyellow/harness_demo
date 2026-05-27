# 팔란티어 3-layer 온톨로지 리서치

> 작성: palantir-researcher 에이전트 | 작성일: 2026-05-27
> 목적: 팔란티어 Foundry/Gotham의 3-layer 온톨로지(Object/Link/Action)를 HS코드 도메인 적용 설계의 1차 자료로 정리

---

## 1. 3-layer 개요

팔란티어 온톨로지(Palantir Ontology)는 조직의 디지털 자산(데이터셋, 가상 테이블, 모델)과 현실 세계의 실체(공장·장비·제품 같은 물리적 자산, 고객 주문·금융 거래 같은 개념적 실체)를 연결하는 **운영 계층(operational layer)** 이다. 공식 문서는 온톨로지를 **의미(semantic) 요소**와 **동적(kinetic) 요소**로 구성된다고 설명한다. 의미 요소는 *무엇이 존재하고 어떻게 연결되는가*(Object / Property / Link)를 정의하고, 동적 요소는 *그것을 어떻게 변화시키는가*(Action / Function)를 거버넌스 통제 하에서 정의한다. 본 리서치에서 요구된 **3-layer(Object / Link / Action)** 는 이 의미·동적 요소를 세 가지 핵심 구현 축으로 재구성한 것으로, Object Layer와 Link Layer는 의미 요소에, Action Layer는 동적 요소에 대응한다.

> **개념 vs 구현 구분 주의**: 팔란티어 공식 문서는 자체적으로 "semantic / kinetic / dynamic" 3계층 용어를 사용하며, "Object Layer / Link Layer / Action Layer"라는 명칭을 공식 레이어 이름으로 명시하지는 않는다. 본 문서의 "Layer"는 개념적 분류이고, "Object Type / Link Type / Action Type"은 온톨로지 매니저에서 실제로 정의·인스턴스화되는 구현 구성요소(스키마)다. (출처: Ontology Overview, Core concepts)

---

## 2. Object Layer

**정의**: 현실 세계의 실체(entity) 또는 사건(event)을 데이터 모델로 표현하는 의미 계층. 이 계층의 스키마 정의 단위가 **Object Type**이며, 그 개별 인스턴스가 **Object**다.

**구성요소**:
- **Object Type (객체 유형)**: "현실 세계 실체 또는 사건의 스키마 정의". 예) `Airport`(공항) 객체 유형 — JFK, LHR은 그 인스턴스(Object).
- **Object (객체)**: Object Type의 개별 인스턴스. 예) "Melissa Chang"(Employee 인스턴스).
- **Object Set (객체 집합)**: 다수 Object의 컬렉션. Object Explorer 검색·필터의 대상.
- **Property (속성)**: "현실 세계 실체 또는 사건의 특성에 대한 스키마 정의". 특정 객체에 부여된 실제 데이터 값은 **Property Value**라 부른다. 예) Employee의 `role` 속성.
- **Shared Property (공유 속성)**: 여러 Object Type에 걸쳐 재사용 가능한 속성. 일관된 데이터 모델링과 속성 메타데이터의 중앙 관리를 가능하게 함.
- **백킹 데이터소스(backing datasource)**: Object Type은 데이터셋·가상 테이블·모델에 매핑되어 실체화된다. 풍부한 메타데이터와 세분화된 보안(granular security)을 포함.

**역할**: 도메인의 "명사(noun)"를 정의하는 토대 계층. Link/Action 계층이 참조하는 기반이 되며, Object View·Object Explorer 등 사용자 애플리케이션의 표시 대상이 된다.

**핵심 인용** (출처: Core concepts):
> "An object type is the schema definition of a real-world entity or event."
> "A property is the schema definition of a characteristic of a real-world entity or event."

---

## 3. Link Layer

**정의**: 두 Object Type 간의 관계(relationship)를 정의하는 의미 계층. 스키마 정의 단위는 **Link Type**, 두 객체 간 관계의 개별 인스턴스는 **Link**다.

**구성요소**:
- **Link Type (링크 유형)**: "두 객체 유형 간 관계의 스키마 정의". 예) Employee–Company 간 고용 관계 링크 유형.
- **Link (링크)**: 같은 온톨로지 내 두 객체 간 관계의 단일 인스턴스. 예) "Melissa Chang" → "Acme, Inc.".
- **카디널리티(cardinality)**: 일대일 / 일대다 / 다대다. 공식 문서는 명칭을 형식적으로 나열하지는 않으나, 특히 **many-to-many** 카디널리티 범주를 명시한다.
- **백킹 방식(backing)**:
  - 일반(일대일/일대다)의 경우: 링크가 가리키는 Object Type에 백킹 데이터소스를 추가하여(예: 외래키 매핑) 링크가 생성·표시됨.
  - 다대다(many-to-many)의 경우: 데이터소스가 **Link Type 자체**를 백킹함(조인 테이블/매핑 테이블 형태).
- **제약**: 서로 다른 온톨로지에 속한 Object Type 간 링크는 지원되지 않음.

**역할**: 도메인의 "관계망(graph)"을 형성하는 연결 조직(connective tissue). Object들을 그래프로 엮어 탐색·분석·Action의 경로를 제공한다.

**핵심 인용** (출처: Link types overview):
> "A link type is the schema definition of a relationship between two object types. A link refers to a single instance of that relationship between two objects in the same Ontology."
> "In the case of link types where object types are related with a many-to-many cardinality, datasources back the link types themselves."

**예시**:
1. Employee ↔ Company (고용 관계)
2. Scheduled Flight ↔ Aircraft (예: "JFK → SFO 24-02-2021" ↔ "Boeing 737-123")
3. 자기참조형: Direct Report ↔ Manager (Employee ↔ Employee)

---

## 4. Action Layer

**정의**: 사용자가 한 번에 적용할 수 있는 객체·속성값·링크에 대한 변경(편집) 집합을 정의하는 동적(kinetic) 계층. 스키마 정의 단위는 **Action Type**이며, 변경에 수반되는 부수효과(side effect)까지 포함한다.

**구성요소**:
- **Action Type (액션 유형)**: "사용자가 한 번에 취할 수 있는, 객체·속성값·링크에 대한 변경/편집 집합의 정의"이며 "액션 제출 시 발생하는 부수효과 동작"을 포함.
- **Parameter (파라미터)**: 사용자가 표준화된 폼으로 입력을 제공하는 수단(기본값, 드롭다운 필터링 등 구성 가능).
- **Rule (규칙)**: 변경이 적용되는 조건/방식. 예) 사용자 액션에 따라 객체 간 링크를 자동 생성하는 규칙.
- **Validation & Submission Criteria (검증·제출 기준)**: 누가 언제 액션을 수행할 수 있는지 통제(권한 제어). 예) HR 담당자만 수행 가능.
- **Side Effect (부수효과)**: 액션 완료 후 자동 동작 — 알림(notification), 웹훅(webhook) 트리거, 스케줄 빌드 등.
- **Function (함수)**: 입력 파라미터를 받아 출력을 반환하는 코드 기반 로직. 객체와 네이티브하게 통합되어 속성값을 읽고 Action Type·애플리케이션 전반에 임의 복잡도의 비즈니스 로직을 제공.
- **Writeback**: 액션으로 인한 변경은 온톨로지의 writeback 데이터셋에 기록됨.

**역할**: 도메인의 "동사(verb)"를 정의하는 변화 계층. 거버넌스(권한·검증)를 준수하면서 Object 상태와 Link 그래프를 변경하고 의사결정을 오케스트레이션한다.

**핵심 인용** (출처: Action types overview):
> "An action type is the definition of a set of changes or edits to objects, property values, and links that a user can take at once."
> Action types "capture data from operators in your organization or orchestrate decision-making processes," while functions provide "a way to author and evolve business logic with arbitrary complexity." (Ontology Overview)

**예시 — "Assign Employee" 액션 유형**:
- Employee 객체의 `role` 속성 변경
- 새 role 값을 파라미터로 입력받음
- 매니저 링크 자동 생성(Rule)
- 영향받는 매니저에게 알림 발송(Side Effect)
- 권한 있는 HR 담당자로 실행 제한(Validation)
- 결과: HR이 단일 트랜잭션으로 "Melissa Chang"을 "Product Manager"로 재배정하고 모든 변경이 writeback에 기록됨

---

## 5. 레이어 간 통합 패턴

세 계층은 다음과 같이 협력한다(시나리오 기반):

1. **모델링(Object + Link)**: 먼저 도메인의 명사를 Object Type으로, 그 관계를 Link Type으로 정의하여 의미 그래프(semantic graph)를 구성한다. 데이터소스가 이를 실체화한다.
2. **변경(Action)**: 사용자가 Action을 실행하면 → 대상 Object의 Property Value가 변경되고 → Rule에 따라 Link가 생성/수정되어 그래프가 갱신되며 → Side Effect(알림·웹훅·빌드)가 트리거된다.
3. **거버넌스**: Action의 Validation/Submission Criteria와 동적 보안(dynamic security)이 변경 전 과정에 걸쳐 권한·규정 준수를 보장한다.
4. **로직 확장(Function)**: Function이 Object 속성을 읽어 계산·검증 로직을 제공하고 Action과 애플리케이션 전반에 재사용된다.
5. **소비(애플리케이션)**: 갱신된 온톨로지는 Object View, Object Explorer, Workshop 애플리케이션 등 사용자 대상 분석·운영 도구를 통해 즉시 반영된다.

**핵심 흐름**: 의미 계층(Object/Link)이 "현실의 디지털 트윈"을 구성하고, 동적 계층(Action/Function)이 거버넌스 하에서 그 트윈을 변화시키며, 모든 변경이 writeback으로 일관되게 영속화된다.

---

## 6. 출처 목록

| # | 제목 | URL | 비고 |
|---|------|-----|------|
| 1 | Overview • Ontology • Palantir | https://www.palantir.com/docs/foundry/ontology/overview | 1차 (공식). semantic/kinetic 요소, 통합 아키텍처 |
| 2 | Core concepts • Palantir | https://www.palantir.com/docs/foundry/ontology/core-concepts | 1차 (공식). Object/Property/Link/Action 정의 |
| 3 | Action types • Overview • Palantir | https://www.palantir.com/docs/foundry/action-types/overview | 1차 (공식). Action 구성요소·예시 |
| 4 | Object and link types • Link types • Overview • Palantir | https://www.palantir.com/docs/foundry/object-link-types/link-types-overview | 1차 (공식). Link Type 정의·카디널리티·백킹 |
| 5 | Object and link types • Type reference • Palantir | https://www.palantir.com/docs/foundry/object-link-types/type-reference | 1차 (공식). 타입 레퍼런스 |
| 6 | The Ontology system • Palantir (Architecture center) | https://www.palantir.com/docs/foundry/architecture-center/ontology-system | 1차 (공식). 시스템 아키텍처 |

**주요 인용 발췌**:
- (출처 2) "An object type is the schema definition of a real-world entity or event."
- (출처 2) "A property is the schema definition of a characteristic of a real-world entity or event."
- (출처 4) "A link type is the schema definition of a relationship between two object types. A link refers to a single instance of that relationship between two objects in the same Ontology."
- (출처 4) "In the case of link types where object types are related with a many-to-many cardinality, datasources back the link types themselves."
- (출처 3) "An action type is the definition of a set of changes or edits to objects, property values, and links that a user can take at once."
- (출처 1) Action types "capture data from operators in your organization or orchestrate decision-making processes"; functions provide "a way to author and evolve business logic with arbitrary complexity."

**상충/주의 사항**:
- 일부 2차 자료(예: Medium "Semantic, Kinetic, and Dynamic Layers")는 온톨로지를 "Semantic / Kinetic / Dynamic"의 3계층으로 명명한다. 공식 문서는 semantic·kinetic 요소를 명시하나 "Object/Link/Action Layer"라는 명칭은 사용하지 않으므로, 본 문서의 3-layer 구분은 구현 구성요소(Object Type / Link Type / Action Type) 기준의 개념적 재구성임을 밝힌다. 2차 자료는 교차 검증 참고용으로만 활용했다.
