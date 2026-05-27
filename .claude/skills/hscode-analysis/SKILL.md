---
name: hscode-analysis
description: HS코드(국제통일상품분류체계, Harmonized System)의 계층 구조, 분류 규칙, 한국 HSK 확장, WCO 표준을 분석하는 방법. HS코드, HSK, 관세분류, WCO, 품목분류, 관세청 키워드가 등장하거나 HS코드 도메인 분석이 필요할 때 반드시 이 스킬을 사용한다.
---

# HS Code Domain Analysis

## 언제 사용하는가
HS코드 체계를 온톨로지 적용 대상으로 분석해야 할 때. `hscode-analyst` 에이전트의 주 스킬.

## 워크플로우

### 1. 1차 소스 우선 (WebSearch + WebFetch)
- 한국 관세청: `customs.go.kr`
- 관세법령정보포털: `unipass.customs.go.kr`
- WCO(세계관세기구): `wcoomd.org` (HS Nomenclature)
- 한국 HSK 자료: 관세청 HSK 통합검색

검색 쿼리 예:
- `HS code structure section chapter heading subheading`
- `HSK 10자리 한국 관세청 분류`
- `WCO General Rules for the Interpretation HS`

### 2. 계층 구조 정리

```markdown
## HS코드 계층 구조

| 계층 | 자릿수 | 단위명 | 개수(기준) | 의미 |
|------|--------|--------|-----------|------|
| 부 | - | Section | 21 | 산업 대분류 |
| 류 | 2자리 | Chapter | 96 | 품목 군 |
| 호 | 4자리 | Heading | ~1,200 | 품목 |
| 소호 | 6자리 | Subheading | ~5,300 | 세부 품목 (WCO 국제표준) |
| HSK | 10자리 | - | ~12,000 | 한국 통계세분류 |
```

### 3. 핵심 엔티티 도출
HS코드 도메인에서 등장하는 개체를 식별한다. 최소 다음을 포함:
- **품목(Commodity)**: 분류 대상 상품
- **분류항목(HS Code Item)**: HS 코드 자체 (부/류/호/소호/HSK)
- **분류규칙(GRI)**: 일반 해석 규칙 6개
- **관세율(Tariff Rate)**: 코드별 세율
- **원산지(Origin)**: FTA 관련
- **사전심사(Advance Ruling)**: 분류 결정
- **분류분쟁(Classification Dispute)**: 이의신청, 행정심판

각 엔티티의 속성과 관계(예: 품목 → 분류항목 → 관세율)를 표/목록으로 정리한다.

### 4. 운영 프로세스 정리
- 신규 품목의 분류 결정 흐름
- 사전심사 신청 → 회신 → 효력
- 분쟁 처리 흐름 (이의신청, 심사청구, 행정소송)

### 5. 출력
결과는 `_workspace/02_hscode_analysis.md`에 저장한다.

## 품질 기준
- WCO 표준(6자리)과 한국 HSK 확장(10자리)을 분리 명시
- GRI 1~6 모두 정리 (온톨로지 Action Layer 매핑의 기반)
- 핵심 엔티티는 속성과 관계를 명시 (예: 품목.속성=[명칭, 재질, 용도], 품목→[분류된다]→분류항목)

## 흔한 실수
- HS와 HSK를 혼용 — 6자리는 국제표준, 10자리는 한국 확장
- 분류규칙을 단순 "참조 문서"로 처리 — GRI는 실제 의사결정 절차이므로 Action Layer 매핑의 핵심 입력
