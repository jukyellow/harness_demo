---
name: data-pipeline-build
description: 특정 HS2류 데이터를 수집·정제하고 그래프 온톨로지를 Python으로 구축하는 방법. 데이터 수집(관세포털 크롤링), 정제, 그래프DB(networkx/Neo4j) 적재, 온톨로지 빌드가 필요할 때 반드시 이 스킬을 사용한다. MVP 데이터/온톨로지 구축 보완/수정/재실행 요청에도 사용.
---

# Data & Ontology Build (Stage 5)

## 언제 사용하는가
설계 산출물을 기반으로 선정 HS2류의 데이터를 수집·정제하고 그래프 온톨로지를 구축할 때. `data-engineer`의 주 스킬.

## 핵심 원칙
- **포그라운드에서만 실행**: 코드/데이터 파일을 쓰므로 백그라운드 서브 에이전트로 실행하면 쓰기가 거부된다. 반드시 포그라운드.
- **실행 가능 우선**: 외부 DB 서버 없이도 돌아가도록 `networkx`를 기본 그래프 엔진으로 쓴다. Neo4j는 선택적 운영 경로로 Cypher export만 추가 제공.

## 워크플로우

### 1. HS2류 선정 확인
`design_ontology.md`의 MVP 시드 범위에서 대상 HS2류를 확인한다.

### 2. 수집 (`mvp/data/raw/`)
- 관세법령정보포털 상세페이지를 httpx + BeautifulSoup로 수집(rate-limit 준수, User-Agent 명시).
- 접근 차단/구조 불확실 시: 1회 재시도 → `analysis.md` 기반 샘플 데이터로 폴백하고 출처/가정을 README에 명시.

### 3. 정제 (`mvp/data/clean/`)
- 결측·중복 제거, 코드 형식 정규화, 계층 무결성(모든 하위 코드가 상위 parent를 가짐) 검증.
- 정제 규칙과 통계를 로그로 남긴다.

### 4. 온톨로지 구축 (`mvp/ontology/`)
- `design_ontology.md`의 노드/관계/Action 스키마대로 networkx 그래프를 적재한다.
- 그래프 접근 인터페이스 함수 제공: `get_node(code)`, `get_children(code)`, `get_rate(code)`, `search(text)` 등 — app-engineer가 호출.
- Neo4j용 `load.cypher`를 함께 생성(운영 경로).
- `mvp/requirements.txt`에 의존성 고정.

### 5. 자가 검수 (Gate 사전점검)
온톨로지 통계(노드/엣지 수), 계층 무결성, 인터페이스 함수 동작을 스스로 확인한 뒤 qa-gate에 인계한다.

### 6. 출력
코드/데이터 + `_workspace/build_data.md`(빌드 로그: 대상 HS2류, 수집 범위/방법, 정제 규칙, 그래프 통계, 실행 방법, 자가검수 결과).

## 품질 기준
- `python -m mvp.ontology.build` 류의 단일 명령으로 그래프가 재현됨
- 스키마(레이블·관계)가 설계와 일치
- 접근 실패 시 폴백·가정이 명시됨

## 흔한 실수
- 외부 DB 서버를 전제해 QA가 실행 불가
- 계층 무결성 검증 누락(고아 노드)
- 스키마 임의 변경 후 미기록
