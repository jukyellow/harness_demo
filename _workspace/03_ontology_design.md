# HS코드 온톨로지 적용 설계안 — 팔란티어 3-layer 매핑

> 작성: ontology-architect 에이전트 | 산출 스킬: ontology-mapping | 작성일: 2026-05-27
> 입력: `01_palantir_research.md`(팔란티어 3-layer 리서치), `02_hscode_analysis.md`(HS코드 도메인 분석)
> 목적: 팔란티어 Object/Link/Action 3-layer 패턴을 HS코드 도메인에 적용하는 **복수의 설계 대안**을 제시·비교하고 권고안을 도출한다.

---

## 0. 입력 검증 결과

| 검증 항목 | 결과 |
|-----------|------|
| 팔란티어 3-layer 정의가 명확히 정리되어 있는가 | ✅ Object Type / Link Type / Action Type + Property / Rule / Side Effect / Function까지 정의됨 (`01` §2~4) |
| HS코드 핵심 엔티티 목록이 존재하는가 | ✅ 품목·분류항목·GRI·관세율·사전심사·분쟁 등 10개 엔티티 + 관계 12종 + 프로세스 3종 정의됨 (`02` §4~6) |
| 용어 충돌 여부 | ⚠️ 팔란티어 공식 문서는 "semantic/kinetic" 용어를 쓰고 "Object/Link/Action **Layer**"는 개념적 재구성임(`01` §1 주석). 본 설계는 구현 단위인 **Object Type / Link Type / Action Type** 기준으로 매핑하며, "Layer"는 개념 분류로만 사용한다. |

→ 양쪽 입력 모두 설계에 충분. 보강 요청 없이 진행.

---

## 1. 매핑 전략 개요 (전제·원칙)

### 1.1 매핑 원칙 (skill 기준)

| 레이어 | 매핑 기준 | HS코드 적용 |
|--------|-----------|-------------|
| **Object** | "존재"하는 개체(명사) | 품목, 분류항목(HSCodeItem), 관세율, 사전심사 등 정태적 실체 |
| **Link** | 객체 간 "관계"(관계명) | 분류결과(classified_as), 계층(parent_of), 세율보유, 사전심사 확정 등 |
| **Action** | 시스템이 "수행"하는 절차(행위/규칙) | GRI 1~6 분류결정, 사전심사 신청/회신, 불복 처리, HSK 세분 |

### 1.2 핵심 설계 전제 (`02` §7 유의점 반영)

1. **HS(6자리 국제표준) vs HSK(10자리 한국 확장)** → 동일 클래스의 레벨 차이로 보되 `isInternational` 속성으로 구분. (단, 대안별로 단일/분리 모델링이 갈림 — §2/§3 참조)
2. **분류 계층(부→류→호→소호→HSK)** → 반드시 self-referential **Link(parent_of)** 로 표현. Object만으로 매핑하면 트리 구조가 손실되므로 금지.
3. **GRI는 단순 참조 문서가 아니라 결정 절차(알고리즘)** → Action Type으로 모델링하여 분류 결정의 자동화·추적을 가능하게 한다. (대안별로 Action 단일 vs Action+RuleObject로 갈림)
4. **부 주/류 주는 법적 효력을 갖는 분류 근거** → 단순 메타데이터가 아닌 Object + Link로 모델링.

### 1.3 대안 차별 축

본 설계는 다음 3개 축으로 두 대안(+하이브리드)을 구분한다.

| 축 | 대안 A(보수적) | 대안 B(확장적) |
|----|---------------|----------------|
| **엔티티 범위** | 핵심 4객체(품목·분류항목·관세율·부주류주)만 | 사전심사·분쟁·원산지·신고·해설서까지 전체 |
| **분류항목 세분도** | `HSCodeItem` 단일 클래스 + `level` 속성 | 부/류/호/소호/HSK 클래스 분리(상속) |
| **GRI 모델링** | `ClassifyCommodity` Action 단일(GRI는 Rule/분기 내장) | Action + `GRIRule` Object로 통칙을 1급 객체로 명시 |

---

## 2. 대안 A: 보수적 매핑 (Minimal Core)

**전략 요약**: 분류의 핵심 경로 `품목 → 분류항목 → 관세율`만 모델링하고, 분류항목은 단일 클래스 + level 속성으로 처리하며, GRI는 단일 Action에 내장하여 빠르게 운영 가능한 MVP를 구성한다.

### Object Types

| 이름 | 속성 | 예시 인스턴스 |
|------|------|---------------|
| **Commodity** | name, material, purpose, function, completeness(완성/미완성), form, essentialCharacter | "리튬이온 전동 자전거" |
| **HSCodeItem** | code, level(section/chapter/heading/subheading/hsk), digitCount, description, parentCode, **isInternational**, edition(HS2022) | code="8711.60", level="subheading", isInternational=true |
| **TariffRate** | code, rateType(기본/WTO양허/FTA협정/잠정), rateValue, validFrom, validTo | code="8711.60", rateType="기본", rateValue="8%" |
| **ClassificationNote** | scope, definition, inclusionRule, exclusionRule, noteType(section/chapter) | "제87류 주: 철도용 차량 제외" |

> 매핑 근거: 위 4개는 `02` §4에서 ★(핵심)로 표시된 객체 중 분류·과세에 직접 필요한 최소 집합. 부 주/류 주(ClassificationNote)는 통칙 1의 법적 근거이므로 메타데이터가 아닌 Object로 승격(전제 4).

### Link Types

| 이름 | from → to | 카디널리티 | 의미 |
|------|-----------|-----------|------|
| **classified_as** | Commodity → HSCodeItem | N:1 | GRI 적용 결과 확정 분류 |
| **parent_of** | HSCodeItem → HSCodeItem | 1:N (self-link) | 부→류→호→소호→HSK 계층 |
| **has_tariff** | HSCodeItem → TariffRate | 1:N | 코드별 다중 세율(기본/양허/FTA) |
| **applies_note** | ClassificationNote → HSCodeItem | N:M | 부/류 주가 적용되는 분류항목 |

> 매핑 근거: `parent_of` self-link로 트리 구조 보존(전제 2). 다중 세율은 1:N. `applies_note`는 한 주가 여러 호에 적용되고 한 호에 여러 주가 적용되므로 N:M → 팔란티어 규칙상 Link Type 자체를 데이터소스로 백킹(`01` §3).

### Action Types

| 이름 | 입력(Parameter) | 출력 | 절차(Rule/Validation/Side Effect) |
|------|----------------|------|-----------------------------------|
| **ClassifyCommodity** | Commodity(명칭·재질·용도·완성도) | `classified_as` Link 생성 | **Rule**: GRI 1→6 순차 분기(1=호용어+부/류주, 2=미완성/혼합, 3a/b/c, 4=유사, 5=용기, 6=소호)를 단일 Action의 조건분기로 내장. **Validation**: 분류 담당자 권한. **Side Effect**: 확정 시 `has_tariff` 경로로 세율 조회 트리거 |
| **DetermineHSK** | 확정된 HSCodeItem(6자리) | HSK HSCodeItem(10자리) Object + `parent_of` Link | 한국 7~10자리 세분 결정, isInternational=false로 생성 |
| **UpdateTariffRate** | HSCodeItem, 신규 rate | TariffRate Object 갱신 | 세율 개정 반영, validFrom/To 관리 |

> 매핑 근거: GRI는 "입력=품목 특성, 출력=HS코드"인 결정 함수(`02` §3 시사점) → Action으로 매핑(전제 3). 보수적 대안에서는 GRI 6단계를 별도 Object 없이 `ClassifyCommodity`의 분기 규칙(Rule)으로 내장하여 단순화한다.

### 트레이드오프
- **장점**: 객체·링크 수가 적어 빠른 MVP 구축·운영 부담 최소. 핵심 경로(분류→과세)가 직관적. 단일 클래스라 코드 노드 쿼리가 단순.
- **단점**: GRI 분기가 Action 내부에 숨어 분류 근거의 추적·감사·재사용이 어려움. 사전심사·분쟁·원산지 미포함으로 법적 효력·FTA 시나리오 처리 불가. level이 속성이라 레벨별 제약(예: 소호 주 vs 류 주) 강제가 약함.
- **적합 시나리오**: 사내 통관 보조 도구, 분류 결과 조회·세율 확인 중심의 초기 PoC.

---

## 3. 대안 B: 확장적 매핑 (Full Governance)

**전략 요약**: 사전심사·분쟁·원산지·신고까지 전체 엔티티를 포함하고, 분류항목을 부/류/호/소호/HSK 클래스로 분리하며, GRI 통칙을 1급 `GRIRule` Object로 명시화하여 분류 결정의 법적 추적성·거버넌스·FTA까지 지원하는 운영 온톨로지를 구성한다.

### Object Types

| 이름 | 속성 | 예시 인스턴스 |
|------|------|---------------|
| **Commodity** | name, material, purpose, function, completeness, form, essentialCharacter | "리튬이온 전동 자전거" |
| **HSCodeNode**(추상) | code, description, parentCode, edition | — (아래로 상속) |
| ├ **Section** | sectionNo, title | "제17부 수송기기" |
| ├ **Chapter** | chapterNo(1~97, 77유보) | "제87류 일반차량" |
| ├ **Heading** | code(4자리) | "8711 모터사이클" |
| ├ **Subheading** | code(6자리), isInternational=true | "8711.60" |
| └ **HSKItem** | code(10자리), isInternational=false, statSuffix | "8711.60-0000" |
| **GRIRule** | ruleNo(1~6), name, applyLevel(호/소호), applyOrder, branchCondition | GRI 3(a) "최협 특정 우선" |
| **ClassificationNote** | scope, definition, inclusion, exclusion, noteType, legalBinding=true | 제87류 주 |
| **ExplanatoryNote** | targetHeading, body, legalBinding=false | 8711 HSEN |
| **TariffRate** | code, rateType, rateValue, validFrom/To | 기본 8% |
| **Origin** | country, agreement, originCriteria(세번변경/부가가치) | 한-EU FTA |
| **AdvanceRuling** | applicant, item, rulingCode, rulingDate, validUntil, reexamined | 사전심사 회신 |
| **ClassificationDispute** | disputedCode, appealStage(이의/심사/심판/소송), agency, result | 불복 건 |
| **Declaration** | declNo, item, appliedCode, declarant | 수출입 신고 |

> 매핑 근거: `02` §4 전체 엔티티 채택. 분류항목을 추상 `HSCodeNode` + 5개 구상 클래스로 분리(차별 축: 세분도)하여 레벨별 속성·제약(예: Subheading만 isInternational=true, Chapter 77 유보)을 클래스 수준에서 강제. **GRIRule을 Object로 승격**하여 통칙을 데이터로 조회·인용·버전관리 가능하게 함(차별 축: GRI 모델링).

### Link Types

| 이름 | from → to | 카디널리티 | 의미 |
|------|-----------|-----------|------|
| **classified_as** | Commodity → HSCodeNode | N:1 | 분류 결과 |
| **parent_of** | HSCodeNode → HSCodeNode | 1:N (self-link) | 부→류→호→소호 계층 |
| **extends_intl** | HSKItem → Subheading | N:1 | HSK 10자리가 HS 6자리를 확장 |
| **applies_note** | ClassificationNote → HSCodeNode | N:M | 부/류 주 법적 적용 |
| **explains** | ExplanatoryNote → Heading | N:1 | 비구속 해설 |
| **governed_by** | classified_as(결정) → GRIRule | N:M | 어떤 통칙이 분류를 지배했는가(추적) |
| **has_tariff** | HSCodeNode → TariffRate | 1:N | 코드별 세율 |
| **has_origin** | Commodity → Origin | N:M | FTA 원산지 |
| **determines_fta_rate** | (Origin + HSCodeNode) → TariffRate | N:M | 원산지×코드 조합 → FTA 세율 |
| **confirms** | AdvanceRuling → HSCodeNode | N:1 | 법적 효력 코드 확정 |
| **disputes** | ClassificationDispute → HSCodeNode | N:1 | 쟁점 코드 |
| **applies_code** | Declaration → HSCodeNode | N:1 | 통관 적용 |

> 매핑 근거: `02` §5 관계 12종 전부 매핑. **`governed_by`** 가 핵심 추가 — 분류 결과 Link에 어떤 GRIRule이 적용됐는지 연결하여 결정 근거의 감사 추적을 제공(보수적 대안엔 없는 능력). `determines_fta_rate`는 Origin×HSCodeNode 다대다 → Link Type 자체 백킹(`01` §3).

### Action Types

| 이름 | 입력 | 출력 | 절차(Rule/Validation/Side Effect) |
|------|------|------|-----------------------------------|
| **ClassifyCommodity** | Commodity | `classified_as` + `governed_by` Link | GRIRule Object들을 applyOrder대로 조회·적용. 각 단계 결정 시 적용 통칙을 `governed_by`로 기록. **Function**: `evaluateGRI(commodity, candidateHeadings)` 본질적특성·최협특정 판정 로직 | 
| **DetermineHSK** | Subheading | HSKItem + `extends_intl` Link | 한국 7~10자리 세분 |
| **IssueAdvanceRuling** | Application(신청인·견본) | AdvanceRuling Object + `confirms` Link | **Validation**: 관세평가분류원 권한, 범칙/불복 진행중 신청 제한 검사. 처리기한 30일(신속 15일). **Side Effect**: 회신 시 신청인 알림(notification), 효력기간 타이머 |
| **RequestReexamination** | AdvanceRuling | 재심사 건 | **Validation**: 통지일 30일 이내 1회 한정 |
| **FileDispute** | Declaration/처분 | ClassificationDispute + `disputes` Link | 이의→심사/심판→소송 단계 전이. **Side Effect**: 단계별 처리기관 라우팅 |
| **AssessFtaRate** | Commodity + Origin | `determines_fta_rate` 경로 TariffRate | 원산지결정기준 검증 후 협정세율 산정 |

> 매핑 근거: `02` §6 프로세스 3종(분류·사전심사·분쟁) + FTA를 모두 Action으로 매핑. 사전심사의 권한·기한·신청제한·재심사 1회 한정 등 제약은 팔란티어 Action의 **Validation/Submission Criteria**(`01` §4)로 매핑. GRI 판정 로직은 **Function**으로 분리해 재사용성 확보.

### 트레이드오프
- **장점**: 분류 결정의 법적 근거(GRIRule)와 권한·기한 거버넌스가 1급으로 추적됨(감사·분쟁 대응 강력). 사전심사·FTA·분쟁 전 과정 커버. 레벨별 클래스 분리로 스키마 제약이 견고.
- **단점**: 객체 13종·링크 12종·액션 6종으로 모델·운영 복잡. 클래스 상속/추상 타입 관리 비용. 초기 데이터 적재(GRIRule·부주류주·HSEN) 부담 큼.
- **적합 시나리오**: 관세청/관세법인 수준의 정식 운영 시스템, 사전심사·분쟁 대응·FTA 원산지 통합 관리가 필요한 환경.

---

## 4. 대안 C: 하이브리드 (단계적 확장형) — 권고 베이스

**전략 요약**: A의 단일 `HSCodeItem`(level 속성) 모델은 유지해 운영 단순성을 확보하되, B의 핵심 거버넌스 자산인 **GRIRule Object + governed_by Link**와 **AdvanceRuling/confirms**만 선택적으로 도입한다. 분쟁·FTA는 후속 단계로 미룬다.

- **Object**: A의 4개(Commodity, HSCodeItem(level 속성형), TariffRate, ClassificationNote) + **GRIRule**, **AdvanceRuling**
- **Link**: A의 4개 + **governed_by**(분류→GRIRule), **confirms**(AdvanceRuling→HSCodeItem)
- **Action**: A의 3개 + **IssueAdvanceRuling**(권한·기한 Validation 포함)

> 근거: GRI 추적성과 사전심사 법적 효력은 HS 도메인의 차별적 가치이므로 조기에 확보하되, 클래스 분리·분쟁·FTA의 복잡성은 지연. A↔B 사이의 실용적 균형.

---

## 5. 대안별 검증 시나리오

### 시나리오 A — 신규 품목 "전기 자전거(리튬이온 모터)" 분류

| 단계 | 대안 A | 대안 B | 대안 C |
|------|--------|--------|--------|
| 품목 등록 | Commodity 인스턴스 생성 | 동일 | 동일 |
| GRI 적용 | `ClassifyCommodity` 내부 분기로 GRI 1(호용어+제87류 주)→경합 시 3(b) 본질적특성(모터=동력) | GRIRule Object를 순서대로 조회, 각 적용 통칙을 `governed_by`로 기록 | B와 동일(GRIRule+governed_by 사용) |
| 결과 | `classified_as` → 8711.60 | `classified_as` + `governed_by`(GRI1, GRI3b) | A의 단일 HSCodeItem + governed_by |
| **근거 추적** | ❌ Action 로그만, 통칙 추적 약함 | ✅ governed_by로 "왜 8711.60인가" 추적 | ✅ 추적 가능 |
| 세율 | `has_tariff`로 기본세율 조회 | 동일 + FTA 시 `determines_fta_rate` | A와 동일(FTA 미지원) |

→ **검증 결과**: 세 대안 모두 분류 자체는 동작. 결정 근거 추적이 필요하면 B/C가 우월.

### 시나리오 B — HS 개정(HS2022→차기판)으로 코드 매핑 변경

| 항목 | 대안 A | 대안 B | 대안 C |
|------|--------|--------|--------|
| 개정 표현 | HSCodeItem.edition 속성 + 신구 코드 신규 노드 | HSCodeNode.edition + 신구 노드, parent_of 재구성 | A와 동일 |
| 신구 코드 연결 | 별도 Link 없음(약점) → `parent_of`만으로는 버전 간 매핑 불가 | `parent_of`로 계층 재구성, 버전 전이용 Link 추가 필요(B도 명시적 supersedes Link 권장) | A 한계 동일 |

→ **검증 결과**: 두 대안 모두 버전 전이용 `supersedes`(구코드→신코드) Link가 명시되지 않음. **개선 권고**: 권고안에 `supersedes` Link Type 추가 필요(아래 §6 로드맵 반영).

### 시나리오 C — 사전심사 신청 → 결정 → 효력 적용

| 단계 | 대안 A | 대안 B | 대안 C |
|------|--------|--------|--------|
| 신청 | ❌ AdvanceRuling 객체 없음 → 처리 불가 | `IssueAdvanceRuling` Action, 신청제한(범칙/불복중) Validation | ✅ B와 동일 도입 |
| 회신 | ❌ | AdvanceRuling Object + `confirms` Link, 신청인 알림 Side Effect | ✅ 동일 |
| 재심사 | ❌ | `RequestReexamination`(30일·1회 한정 Validation) | △ C는 IssueAdvanceRuling만 도입(재심사는 후속) |

→ **검증 결과**: 사전심사 시나리오는 대안 A로는 불가, B는 완전 지원, C는 핵심(신청·회신)만 지원. **법적 효력 코드 확정이 핵심 요구라면 최소 C 이상 필요.**

---

## 6. 권고안 및 단계적 도입 로드맵

### 권고: **대안 C(하이브리드)를 MVP로 채택 → 대안 B로 점진 확장**

**근거**: 대안 A는 HS 도메인의 차별 가치(GRI 결정 근거 추적, 사전심사 법적 효력)를 놓쳐 단순 코드 조회기에 머문다. 대안 B는 완성도가 높으나 초기 구축·데이터 적재 부담이 크다. 대안 C는 GRIRule 추적성과 사전심사 효력이라는 두 핵심 자산을 조기에 확보하면서 분쟁·FTA·클래스 분리의 복잡성을 지연시켜 ROI가 가장 높다.

### 단계적 로드맵

| 단계 | 범위 | 추가 구성요소 | 비고 |
|------|------|--------------|------|
| **MVP (대안 C)** | 분류 + GRI 추적 + 사전심사 | Object 6 / Link 6 / Action 4 | 단일 HSCodeItem(level 속성), GRIRule+governed_by, AdvanceRuling+confirms |
| **확장 1** | 버전 관리 강화 | `supersedes` Link(구→신 코드), edition 정책 | 시나리오 B 검증서 도출된 보강 항목 |
| **확장 2** | FTA·원산지 | Origin Object, has_origin·determines_fta_rate Link, AssessFtaRate Action | 협정세율 자동 산정 |
| **확장 3 (대안 B 완성)** | 분쟁 + 클래스 분리 | ClassificationDispute, FileDispute Action, HSCodeNode 추상→5클래스 상속 분리, ExplanatoryNote | 정식 운영·감사 대응 수준 |

### 즉시 반영 권고 (검증서 도출)
- 모든 대안에 **`supersedes`(구코드→신코드) Link Type**를 추가하라 — HS 5~6년 주기 개정의 버전 전이를 표현하지 못하는 공통 결함(시나리오 B).

---

## 부록: 매핑 근거 요약 (원칙 일관성 체크)

| 매핑 결정 | 적용 원칙(§1) | 일관성 |
|-----------|--------------|--------|
| 분류 계층을 parent_of self-link로 | 전제 2 / skill "트리 손실 금지" | ✅ |
| GRI를 Action(+B/C에선 GRIRule Object)으로 | 전제 3 / skill "GRI는 Action" | ✅ |
| 부 주/류 주를 Object+Link로 | 전제 4 | ✅ |
| HSK/HS를 level·isInternational로 구분 | 전제 1 | ✅ |
| 사전심사 권한·기한을 Action Validation으로 | `01` §4 Validation/Submission Criteria | ✅ |
