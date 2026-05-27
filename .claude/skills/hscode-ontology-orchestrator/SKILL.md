---
name: hscode-ontology-orchestrator
description: 팔란티어 3-layer 온톨로지를 HS코드에 적용하는 리서치/설계/보고서 작성 워크플로우를 조율한다. 팔란티어 온톨로지, HS코드, HSK, 온톨로지 적용, 3-layer, Object/Link/Action, 관세 분류 데이터 모델 키워드가 등장하거나, 보고서 다시 실행/재실행/업데이트/수정/보완 요청, 특정 섹션만 다시/이전 결과 기반 개선 요청이 있을 때 반드시 이 스킬을 사용한다.
---

# HS Code Ontology Orchestrator

## 언제 사용하는가
- HS코드에 팔란티어 3-layer 온톨로지를 적용하는 리서치/보고서 작업 시작
- 기존 보고서의 일부 섹션 또는 전체 재실행
- 사용자 피드백 기반 개선 요청

## 팀 구성
| 에이전트 | 역할 | 산출물 |
|---------|------|--------|
| `palantir-researcher` | 팔란티어 3-layer 리서치 | `_workspace/01_palantir_research.md` |
| `hscode-analyst` | HS코드 도메인 분석 | `_workspace/02_hscode_analysis.md` |
| `ontology-architect` | 매핑 설계 (복수 대안) | `_workspace/03_ontology_design.md` |
| `report-writer` | 최종 보고서 작성 | `reports/hscode-ontology-report.md` |

## 실행 모드: 하이브리드
- Phase 2 (리서치): **서브 에이전트 패턴** — 두 리서처를 `run_in_background: true`로 병렬 실행
- Phase 3~4 (설계·작성): **에이전트 팀 패턴** — TeamCreate로 architect + writer 팀 구성, SendMessage로 협업
- Phase 1 (컨텍스트 확인): 메인 에이전트 단독

## 워크플로우

### Phase 0: 컨텍스트 확인
1. `_workspace/` 존재 여부 확인
2. `reports/hscode-ontology-report.md` 존재 여부 확인
3. 사용자 요청 분석:
   - `_workspace/` 없음 → **초기 실행** (Phase 2부터)
   - `_workspace/` 있음 + 부분 수정 요청 ("X 섹션만 다시", "Y 보완") → **부분 재실행** (해당 에이전트만 호출)
   - `_workspace/` 있음 + 새 입력/대규모 변경 요청 → **새 실행** (기존 `_workspace/`를 `_workspace_prev/`로 이동 후 Phase 2부터)

판별 결과를 사용자에게 1줄로 알린다 (예: "초기 실행으로 진행합니다.").

### Phase 1: 사용자 요구 명세 (필요 시)
초기 실행이면 다음을 사용자에게 확인한다 (이미 충분히 명시되어 있으면 건너뜀):
- 보고서 분량 (간략판 5p / 표준 10~15p / 상세 20p+)
- 한국어 vs 한영 병기 비중
- 권고안 강도 (대안 비교만 / 권고 1개 / 단계적 로드맵 포함)

### Phase 2: 병렬 리서치 (서브 에이전트)
두 리서처를 동시 호출한다.

```
Agent(subagent_type="general-purpose", model="opus",
      run_in_background=true,
      description="Palantir 3-layer ontology research",
      prompt="너는 palantir-researcher 에이전트다. .claude/agents/palantir-researcher.md를 먼저 읽고, .claude/skills/palantir-research/SKILL.md의 워크플로우를 따라 _workspace/01_palantir_research.md를 생성하라. 한국어로 작성.")

Agent(subagent_type="general-purpose", model="opus",
      run_in_background=true,
      description="HS code domain analysis",
      prompt="너는 hscode-analyst 에이전트다. .claude/agents/hscode-analyst.md를 먼저 읽고, .claude/skills/hscode-analysis/SKILL.md의 워크플로우를 따라 _workspace/02_hscode_analysis.md를 생성하라. 한국어로 작성.")
```

두 에이전트 완료 후 산출물 파일 존재 및 필수 섹션을 확인한다.

### Phase 3: 매핑 설계 (서브 에이전트)
Phase 2의 두 산출물이 모두 존재하면 ontology-architect를 호출한다.

```
Agent(subagent_type="general-purpose", model="opus",
      description="Ontology mapping design",
      prompt="너는 ontology-architect 에이전트다. .claude/agents/ontology-architect.md를 먼저 읽고, .claude/skills/ontology-mapping/SKILL.md의 워크플로우를 따라 _workspace/01_palantir_research.md와 _workspace/02_hscode_analysis.md를 입력으로 _workspace/03_ontology_design.md를 생성하라.")
```

### Phase 4: 보고서 작성 (서브 에이전트)
Phase 3 산출물 확인 후 report-writer 호출.

```
Agent(subagent_type="general-purpose", model="opus",
      description="Final report writing",
      prompt="너는 report-writer 에이전트다. .claude/agents/report-writer.md를 먼저 읽고, .claude/skills/report-writing/SKILL.md의 워크플로우를 따라 _workspace/의 세 파일을 통합하여 reports/hscode-ontology-report.md를 작성하라.")
```

### Phase 5: 최종 검증 및 피드백 요청
1. 최종 보고서의 필수 섹션(Executive Summary, 적용 대안 비교, 권고안, 부록) 존재 확인
2. 사용자에게 보고서 경로 안내 및 피드백 요청:
   > "보고서가 `reports/hscode-ontology-report.md`에 생성되었습니다. 결과를 검토하고 개선할 부분이 있으면 알려주세요."

## 데이터 전달 프로토콜
- **파일 기반**: `_workspace/` (중간 산출물, 보존), `reports/` (최종)
- **반환값 기반**: 각 서브 에이전트는 산출 파일 경로와 요약을 메인에 반환
- **태스크 기반**: 필요 시 TaskCreate로 Phase별 진행 상황 추적

## 에러 핸들링
- **에이전트 1회 실패** → 동일 프롬프트로 1회 재시도. 재실패 시 해당 산출물 없이 후속 단계 진행하고 보고서에 "후속 보완 필요" 표기
- **웹 접근 실패** → 리서치 에이전트가 자체 처리 (출처 미확보 표기)
- **입력 누락** → 후속 에이전트가 부분 입력으로 진행, 보고서에 누락 명시
- **상충 정보** → 삭제하지 않고 양측 병기, 부록에 비교

## 부분 재실행 매트릭스
| 사용자 요청 패턴 | 재실행 대상 |
|----------------|------------|
| "팔란티어 리서치 보완" | `palantir-researcher`만 → `ontology-architect` 재호출 → `report-writer` 재호출 |
| "HS 분석 보완" | `hscode-analyst`만 → `ontology-architect` 재호출 → `report-writer` 재호출 |
| "대안 추가/수정" | `ontology-architect`만 → `report-writer` 재호출 |
| "보고서 문체/구조 수정" | `report-writer`만 |
| "전체 다시" | `_workspace/` → `_workspace_prev/` 이동 후 Phase 2부터 |

## 테스트 시나리오

### 정상 흐름
1. 입력: "팔란티어 온톨로지를 HS코드에 적용하는 보고서 작성해줘"
2. Phase 0: `_workspace/` 없음 → 초기 실행
3. Phase 2: 두 리서처 병렬 완료 → 산출물 2개 생성 확인
4. Phase 3: 매핑 설계 산출물 생성 (대안 ≥ 2개)
5. Phase 4: `reports/hscode-ontology-report.md` 생성
6. Phase 5: 사용자에게 경로 안내 + 피드백 요청

### 에러 흐름
1. 입력: 동일
2. Phase 2: `palantir-researcher` 1회 실패 → 재시도
3. 재시도도 실패 → 산출물 없이 Phase 3 진행
4. `ontology-architect`가 입력 누락을 보고서에 "팔란티어 측 추가 리서치 필요"로 표기
5. 사용자에게 후속 보완 필요 안내

### 후속 흐름
1. 입력: "대안 C(하이브리드)도 추가해줘"
2. Phase 0: `_workspace/` 존재 + 부분 수정 요청 → 부분 재실행
3. `ontology-architect`만 재호출 (대안 C 추가)
4. `report-writer` 재호출 (적용 대안 비교 표 갱신)
5. 사용자에게 변경 사항 안내
