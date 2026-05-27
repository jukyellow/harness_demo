---
name: hscode-ontology-orchestrator
description: HS 온톨로지 실험실의 6단계(구축 리서치 / 분석 / 요구사항 정의 / 설계 / MVP 구축 / 최종보고서)를 조율한다. HS 온톨로지, 팔란티어 3-layer, Agentic HS 추론, HS2~10단위 분류, 관세포털 데이터, 요구사항정의/WBS/작업지시서, 그래프DB, 서빙/데모, 검수Gate, 평가, MVP 구축 키워드가 등장하거나, 실험실/단계 작업 시작 요청, 또는 다시 실행/재실행/업데이트/수정/보완/특정 단계만 다시/이전 결과 기반 개선 요청이 있을 때 반드시 이 스킬을 사용한다.
---

# HS Ontology Lab Orchestrator (6단계)

## 언제 사용하는가
- HS 온톨로지 실험실 전체 또는 특정 단계 작업 시작
- 특정 단계/산출물의 부분 재실행, 사용자 피드백 기반 개선

## 팀 구성 (단계별 활성화)
| 단계 | 에이전트 | 스킬 | 산출물 |
|------|---------|------|--------|
| ① 리서치 | `palantir-researcher` | palantir-research | `_workspace/research_tech.md` |
| ① 리서치 | `hscode-analyst` | hscode-analysis | `_workspace/research_domain.md` |
| ② 분석 | `hscode-analyst` | hscode-analysis | `_workspace/analysis.md` |
| ③ 요구사항 | `requirements-analyst` | requirements-definition | `_workspace/requirements.md`, `_workspace/wbs.md` |
| ④ 설계 | `ontology-architect` | ontology-mapping | `_workspace/design_ontology.md` |
| ④ 설계 | `agentic-flow-architect` | agentic-flow-design | `_workspace/design_agentic.md` |
| ④ 설계 | `serving-ux-architect` | serving-ux-design | `_workspace/design_serving.md` |
| ⑤ MVP | `data-engineer` | data-pipeline-build | `mvp/data`, `mvp/ontology`, `_workspace/build_data.md` |
| ⑤ MVP | `app-engineer` | app-serving-build | `mvp/engine`, `mvp/server`, `mvp/app`, `_workspace/build_app.md` |
| ⑤ MVP | `qa-gate` | qa-gate | `_workspace/qa_gate.md` |
| ⑥ 보고서 | `lab-documenter` | lab-documentation | `docs/stage{1-5}_*.md`, `reports/final_report.md` |

> 산출물 파일명은 단계 순서가 바뀌어도 안정적이도록 의미 기반(번호 없음)으로 둔다. 단계 번호는 이 표가 기준.

## 실행 모드: 포그라운드 서브 에이전트
- 모든 에이전트는 `Agent` 도구로 **포그라운드** 호출(`run_in_background` 금지). 이유: 이 환경에서 백그라운드 서브 에이전트는 파일 쓰기가 거부된다.
- **병렬화**: 같은 단계의 독립 에이전트는 한 메시지에 여러 포그라운드 `Agent` 호출을 함께 넣어 동시 실행(쓰기 가능 + 병렬).
- 모든 호출에 `model: "opus"` 명시. 각 프롬프트는 "agents/{name}.md를 읽고 {skill} 워크플로우대로 {산출물}을 작성하라" 형태.

## 기본 기술 결정 (조정 가능 — CLAUDE.md에 기록, 요구사항 단계에서 사용자 확정)
- 그래프 엔진: `networkx`(실행 보장) + Neo4j Cypher export(선택)
- Agentic 프레임워크: Claude API tool use(프롬프트 캐싱). 키 부재 시 mock 폴백. 대안 LangGraph
- 서빙/UI: FastAPI + Streamlit / 수집: httpx + BeautifulSoup

## 워크플로우

### Phase 0: 컨텍스트 확인
1. `_workspace/`, `mvp/`, `docs/`, `reports/` 존재 여부 확인
2. 실행 모드 판별:
   - 산출물 없음 → **초기 실행** (Stage 1부터)
   - 산출물 있음 + 특정 단계/부분 수정 → **부분 재실행** (아래 매트릭스)
   - 산출물 있음 + 전체 재시작/새 입력 → **새 실행** (`_workspace/`→`_workspace_prev/`, `mvp/`→`mvp_prev/` 이동 후 Stage 1)
3. 판별 결과를 사용자에게 1줄로 알린다.

### Phase 1: 범위 확인 (초기 실행 시, 명시돼 있으면 생략)
어디까지 진행할지(리서치만/요구사항까지/설계까지/MVP까지/보고서까지)를 확인한다. 그 외 세부 결정은 **Stage 3(요구사항)에서 후보군으로 사용자에게 묻는다** — 여기서 과도하게 묻지 않는다.

### Stage 1 — 구축 리서치 (병렬, 포그라운드)
한 메시지에 두 호출: `palantir-researcher`→`research_tech.md`, `hscode-analyst`(도메인)→`research_domain.md`. 완료 후 `lab-documenter`로 `docs/stage1_research.md`.

### Stage 2 — 분석 (포그라운드)
`hscode-analyst`(포털 데이터 분석)→`analysis.md`. 완료 후 `docs/stage2_analysis.md`.

### Stage 3 — 요구사항 정의 (사용자 결정 루프, 핵심)
1. `requirements-analyst` 호출 → `requirements.md` 초안(기능/비기능 요구사항 + **사용자 결정 필요 항목 + 후보군**) 작성
2. **메인 에이전트가 결정 필요 항목을 `AskUserQuestion`으로 사용자에게 제시**한다(후보군 = 옵션). MVP 대상 HS2류, 기술 스택 확정, 평가 기준 등.
3. 사용자 결정을 받아 `requirements-analyst`를 재호출(또는 결정 전달) → `requirements.md` 확정 + `wbs.md`(WBS + 단계별 작업지시서) 작성
4. `lab-documenter`로 `docs/stage3_requirements.md`.
> 이후 Stage 4~5는 `wbs.md`의 작업지시서·완료Gate를 실행 기준으로 삼는다. 각 에이전트 프롬프트에 해당 작업지시서 경로를 전달한다.

### Stage 4 — 설계 (병렬 후 정합, 포그라운드)
1. `ontology-architect` 먼저 → `design_ontology.md`(스키마가 둘의 입력)
2. 이어서 `agentic-flow-architect` + `serving-ux-architect`를 한 메시지에 병렬 호출
3. 세 산출물 인터페이스 정합 확인(온톨로지 함수 ↔ 추론 호출 ↔ 로그 스키마 ↔ 시각화 필드). 불일치 시 해당 아키텍트 1회 재호출
4. `qa-gate`로 설계 정합성 검토(선택) → `docs/stage4_design.md`

### Stage 5 — MVP 구축 (생성→검증→수정 루프, 포그라운드 순차)
파이프라인: 수집→정제→온톨로지→서빙→화면, 각 단계 검수Gate(=wbs 작업지시서 완료Gate).
1. `data-engineer` → 수집/정제/온톨로지(`mvp/`)
2. `qa-gate` → 수집/정제/온톨로지 Gate. FAIL이면 data-engineer 재호출(최대 2회)
3. `app-engineer` → 추론 엔진/서빙/화면
4. `qa-gate` → 서빙/화면/추론 Gate + 데모 실행 검증. FAIL이면 app-engineer 재호출(최대 2회)
5. 전체 PASS → `docs/stage5_mvp.md`

### Stage 6 — 최종보고서 (포그라운드)
`lab-documenter` → `docs/stage*.md`와 `_workspace/`, `mvp/`를 통합해 `reports/final_report.md`.

### 마무리: 검증 및 피드백
최종 산출물 경로 안내 + 피드백 요청("팀 구성/워크플로우에 바꾸고 싶은 점이 있나요?").

## 데이터 전달 프로토콜
- **파일 기반**: `_workspace/`(중간), `mvp/`(코드), `docs/`(단계 문서), `reports/`(최종) — 모두 보존
- **반환값 기반**: 각 포그라운드 에이전트가 산출 경로/요약을 메인에 반환
- **사용자 결정**: Stage 3에서 AskUserQuestion으로 수집 → requirements.md에 기록 → 후속 단계 프롬프트로 전파

## 에러 핸들링
- 에이전트 1회 실패 → 1회 재시도. 재실패 시 산출물 없이 진행하고 문서에 "후속 보완 필요" 표기
- 웹/포털 접근 실패 → 에이전트가 샘플 폴백 + 가정 명시
- API 키 부재 → app-engineer/qa-gate가 mock 경로, "실 추론 미검증" 표기
- 검수Gate 반복 FAIL(2회) → 한계로 기록하고 진행, 보고서에 명시
- 상충 정보/요구사항 → 삭제 금지, 양측 병기하거나 후보군으로 사용자 결정에 위임

## 부분 재실행 매트릭스
| 요청 패턴 | 재실행 대상 |
|----------|------------|
| "리서치 보완" | palantir-researcher / hscode-analyst(도메인) |
| "분석 보완/포털 재분석" | hscode-analyst(Stage2) → 요구사항 영향 시 requirements-analyst |
| "요구사항 수정/재정의, 결정 변경" | requirements-analyst(+AskUserQuestion) → WBS 갱신 → 영향 단계 반영 |
| "WBS/작업지시서만 수정" | requirements-analyst |
| "온톨로지 설계 수정" | ontology-architect → agentic/serving 정합 확인 |
| "추론 flow/평가/Gate 수정" | agentic-flow-architect → app/qa 반영 |
| "서빙/화면/시각화 수정" | serving-ux-architect → app-engineer 반영 |
| "MVP 데이터/온톨로지 수정" | data-engineer → qa-gate 재검수 |
| "추론엔진/서빙/화면 수정" | app-engineer → qa-gate 재검수 |
| "검수만 다시" | qa-gate |
| "보고서/문서 수정" | lab-documenter |
| "전체 다시" | `_workspace/`,`mvp/` 백업 후 Stage 1부터 |

## 테스트 시나리오

### 정상 흐름
1. "HS 온톨로지 실험실 전체를 진행해줘" → Phase 0 산출물 없음 → 초기 실행
2. Stage 1 병렬 리서치 → 문서화
3. Stage 2 분석 → 문서화
4. Stage 3 요구사항 초안 → **AskUserQuestion으로 HS2류·스택 등 결정** → 확정 requirements.md + wbs.md → 문서화
5. Stage 4 설계 3종 + 정합 → 문서화
6. Stage 5 data-engineer→qa(PASS)→app-engineer→qa(데모 PASS) → 문서화
7. Stage 6 `reports/final_report.md` → 경로 안내 + 피드백 요청

### 에러 흐름
1. Stage 5 포털 접근 실패 → data-engineer 샘플 폴백 + 가정 명시
2. qa-gate가 서빙 응답 shape 불일치 FAIL → app-engineer 재호출 → 재검수 PASS
3. API 키 부재 → mock 데모 검증, 보고서에 "실 추론 미검증"

### 후속 흐름
1. "MVP 대상 류를 09류로 바꾸고 다시" → Phase 0 산출물 존재 + 결정 변경 → 부분 재실행
2. requirements-analyst로 결정 갱신 → wbs 갱신 → Stage 4~5 영향 부분 재실행 → 문서/보고서 갱신
