---
name: hscode-analysis
description: HS코드 도메인지식과 관세법령정보포털 HS 2~10단위 데이터를 분석하는 방법. HS체계/GRI/HSK, 포털 상세페이지 수집 정보·데이터 relation·수집방안·엔진선정·파이프라인 분석이 필요할 때 반드시 이 스킬을 사용한다. 분석 보완/업데이트/재실행 요청에도 사용.
---

# Stage 1~2: HS Domain & Data Analysis

## 언제 사용하는가
- Stage 1: HS 도메인지식 정리
- Stage 2: 관세법령정보포털 HS 데이터 분석 + 수집/엔진/파이프라인 검토
`hscode-analyst`의 주 스킬.

## 워크플로우

### A. 도메인지식 (Stage 1)
- 계층: 부(21) → 류(2자리) → 호(4자리) → 소호(6자리, WCO 국제표준) → HSK(10자리, 한국 확장). 각 계층 의미 정리. WCO와 HSK를 분리 명시.
- GRI 1~6(일반 해석 규칙) 정리 — Action 설계의 핵심.
- 운영 프로세스: 분류 결정, 사전심사, 분쟁.
- 1차 소스: customs.go.kr, unipass.customs.go.kr, wcoomd.org.
- 출력: `_workspace/research_domain.md`

### B. 포털 데이터 분석 (Stage 2)
관세법령정보포털(관세율표>한국2026)의 HS 2~10단위 상세페이지를 분석한다.

1. **페이지 구조·수집 필드**: 상세페이지에서 얻을 수 있는 필드 식별 — 코드, 품명(국/영문), 단위, 기본/협정 세율, 신고요령, 주(註)·규정, 분류사례, 변경이력 등. 페이지 URL 패턴·계층 탐색 방식도 기록.
2. **데이터 범위/수집방안/활용방안**: MVP 시드로 적절한 범위, 수집 방법(httpx+BeautifulSoup 등, rate-limit 준수), 활용처(온톨로지 노드/추론 근거).
3. **데이터 relation 표**:
   | relation | from → to | 의미 |
   |----------|-----------|------|
   | parent_of | HSCodeItem → HSCodeItem | 계층 |
   | has_rate | HSCodeItem → TariffRate | 세율 |
   | governed_by | HSCodeItem → Rule/Note | 주·규정 적용 |
   | has_ruling | HSCodeItem → AdvanceRuling | 분류사례 |
4. **엔진/기술 선정 후보**: 수집·저장(그래프DB/관계형)·서빙·추론 엔진 후보를 비교하고 권장안 제시(최종 확정은 설계).
5. **파이프라인 검토**: 수집 → 정제 → 온톨로지 → 서빙 → 추론 전체 흐름의 단계·산출물·리스크를 검토.
6. 출력: `_workspace/analysis.md`

## 품질 기준
- WCO(6자리)와 HSK(10자리)를 분리 명시, GRI 1~6 모두 정리
- 실제 페이지에서 수집 가능한 필드가 구체적으로 식별됨(추측이면 가정 명시)
- relation 표가 이후 온톨로지 설계에 바로 입력 가능
- 엔진/기술 후보가 비교표로 정리되고 권장안 제시

## 흔한 실수
- HS와 HSK 혼용 — 6자리는 국제표준, 10자리는 한국 확장
- 일반론만 적고 실제 포털 페이지 구조를 분석하지 않음
- relation을 빠뜨려 온톨로지 설계가 트리 구조만 남음
- 접근 불가 시 가정을 명시하지 않음
