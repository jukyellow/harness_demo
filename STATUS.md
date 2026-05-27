# HS 온톨로지 실험실 — 진행 상황 (STATUS)

> 다른 터미널/세션에서 이어받기용 핸드오프 문서. 최종 갱신: 2026-05-28

## 현재 상태
- **하네스 구축: 완료·검증됨** (에이전트 10 · 스킬 11 · 오케스트레이터 · CLAUDE.md)
- **실험실 실행(Stage 1~6): 미시작** — 산출물 없음(`_workspace/` 비어있음, 신규 기준)
- **다음 할 일: Stage 1(구축 리서치)부터 실행**

## 이어받는 방법 (다른 터미널)
1. **반드시 같은 프로젝트 폴더에서 열기**: `C:\02_repository\harness_demo` (그래야 `.claude/` 하네스 공유)
2. "HS 온톨로지 실험실 이어서 진행" 또는 "Stage N 진행" 등으로 요청 → CLAUDE.md 트리거가 `hscode-ontology-orchestrator` 스킬로 라우팅
3. 오케스트레이터 **Phase 0**가 `_workspace/`·`mvp/`·`docs/`·`reports/` 파일 존재로 진행 단계를 자동 판별하여 이어감
4. 세밀한 진행 추적은 Stage 3 산출물 `_workspace/wbs.md`의 **완료여부 체크**가 담당

## 6단계 계획과 산출물 (단계 번호 = 오케스트레이터 기준)
| 단계 | 담당 에이전트 | 산출물 | 상태 |
|------|--------------|--------|------|
| ① 구축 리서치 | palantir-researcher, hscode-analyst | `_workspace/research_tech.md`, `research_domain.md` | ☐ |
| ② 분석 | hscode-analyst | `_workspace/analysis.md` | ☐ |
| ③ 요구사항 정의 | requirements-analyst | `_workspace/requirements.md`, `wbs.md` | ☐ |
| ④ 설계 | ontology-architect, agentic-flow-architect, serving-ux-architect | `_workspace/design_ontology.md`, `design_agentic.md`, `design_serving.md` | ☐ |
| ⑤ MVP 구축 | data-engineer, app-engineer, qa-gate | `mvp/`, `_workspace/build_*.md`, `qa_gate.md` | ☐ |
| ⑥ 최종보고서 | lab-documenter | `docs/stage{1-5}_*.md`, `reports/final_report.md` | ☐ |

> 각 단계 완료 직후 `lab-documenter`가 `docs/stage{N}_*.md` 작성.

## 확정된 결정 (재구성 시 사용자 승인)
- MVP 성격: **동작하는 코드 MVP**
- ③ 요구사항 단계에서 **claude가 후보군 제시 → 사용자가 `AskUserQuestion`으로 결정**하는 흐름 유지
- ④ 설계는 **3명 아키텍트 분담**(온톨로지 / Agentic flow / 서빙·UX)

## 기본 기술 결정 (③ 요구사항 단계에서 사용자 최종 확정 가능)
- 그래프 엔진: `networkx`(실행 보장) + Neo4j Cypher export(선택)
- Agentic: Claude API tool use(프롬프트 캐싱) · 키 부재 시 mock 폴백 / 대안 LangGraph
- 서빙·UI: FastAPI + Streamlit / 수집: httpx + BeautifulSoup
- ③에서 결정할 항목: MVP 대상 HS2류, 스택 최종 확정, 평가 기준 등

## 실행 규칙 (중요)
- 파일을 쓰는 에이전트는 **포그라운드**로만 실행(백그라운드 쓰기 거부됨). 병렬은 한 메시지 다중 포그라운드 호출.
- 모든 Agent 호출에 `model: "opus"`.

## 참조
- 워크플로우 전체: `.claude/skills/hscode-ontology-orchestrator/SKILL.md`
- 하네스 포인터·변경이력: `CLAUDE.md`
- 요구사항/범위 원본: `README.md`
