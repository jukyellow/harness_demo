# 프로젝트: HS 온톨로지 실험실

## 하네스: HS 온톨로지 실험실 (6단계)

**목표:** HS 온톨로지를 ① 구축 리서치 → ② 분석 → ③ 요구사항 정의 → ④ 설계 → ⑤ MVP 구축 → ⑥ 최종보고서 순으로 진행하여, Agentic HS 추론(HS2→HS10 cycle, 근거 제시) 데모 MVP와 산출물을 만든다.

**트리거:** HS 온톨로지, 팔란티어 3-layer, Agentic HS 추론, HS2~10단위 분류, 관세포털 데이터, 요구사항정의/WBS, 그래프DB, 서빙/데모, 검수Gate, 평가, MVP 구축 관련 작업이나 특정 단계 시작/재실행/수정/보완 요청 시 `hscode-ontology-orchestrator` 스킬을 사용하라. 단순 질문은 직접 응답 가능.

**기본 기술 결정 (요구사항 단계에서 사용자 확정 가능):**
- 그래프 엔진: `networkx`(실행 보장) + Neo4j Cypher export(선택)
- Agentic 프레임워크: Claude API tool use(프롬프트 캐싱), 키 부재 시 mock 폴백 / 대안 LangGraph
- 서빙·UI: FastAPI + Streamlit / 데이터 수집: httpx + BeautifulSoup

**실행 모드:** 포그라운드 서브 에이전트(백그라운드 금지 — 쓰기 거부됨). 같은 단계의 독립 에이전트는 한 메시지 다중 포그라운드 호출로 병렬. 전 호출 `model: "opus"`.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-05-27 | 초기 구성 (에이전트 4 + 스킬 4 + 오케스트레이터 1) | 전체 | 초기 구축 |
| 2026-05-28 | "HS 온톨로지 실험실"로 전면 재구성 — 단일 리서치+보고서 → 6단계 라이프사이클. 에이전트 10종(requirements-analyst·agentic-flow-architect·serving-ux-architect·data-engineer·app-engineer·qa-gate·lab-documenter 신규/전환, report-writer 제거), 스킬 11종, 오케스트레이터 6단계화 | 전체 | README.md 프로젝트 성격 변경 (요구사항 단계 포함 6단계, Agentic 추론 MVP) |
