---
name: app-serving-build
description: Agentic HS 추론 엔진, FastAPI 서빙서버, Streamlit 사용자/관리자 데모 화면과 로깅·시각화를 구현하는 방법. 추론 엔진(Claude API tool use), 서빙 API, 데모 화면, 분류 로깅·시각화 구축이 필요할 때 반드시 이 스킬을 사용한다. 서빙/화면/엔진 구현 보완/수정/재실행 요청에도 사용.
---

# Agentic Engine, Serving & Demo Build (Stage 5)

## 언제 사용하는가
설계와 온톨로지를 기반으로 추론 엔진·서빙서버·데모 화면을 구현할 때. `app-engineer`의 주 스킬.

## 핵심 원칙
- **포그라운드에서만 실행**: 코드 파일을 쓰므로 백그라운드 금지.
- **키 없이도 데모 가능**: 실 추론은 Claude API tool use(프롬프트 캐싱)로 하되, `ANTHROPIC_API_KEY` 부재 시 결정적 mock 추론기로 폴백해 데모/QA가 항상 실행되게 한다.

## 워크플로우

### 1. 추론 엔진 (`mvp/engine/`)
- `design_agentic.md`의 flow 구현: HS2→4→6→10 cycle, 단위별 후보군 도출(온톨로지 `get_children`+LLM 대조), 추가질문 생성, GRI 적용, 종료 조건.
- 각 단위에서 `design_serving.md`의 JSON 로그 스키마대로 근거·증적·확신도를 기록한다.
- Claude API tool use로 온톨로지 함수를 도구로 노출. mock 폴백 경로 포함.

### 2. 서빙서버 (`mvp/server/`)
- FastAPI로 `POST /classify`, `POST /answer`, `GET /logs/{session}`, `GET /ontology/{code}` 구현.
- 요청/응답이 설계 스키마와 일치하도록 pydantic 모델 정의.

### 3. 데모 화면 (`mvp/app/`)
- **사용자(Streamlit)**: HS2류 선택+질문 → cycle 진행/추가질문 응답 → 최종 HS10 + 단위별 근거.
- **관리자(Streamlit)**: 추론 트리, 단위별 확신도 차트, 후보군 변화, 질문-판별 추적 시각화(로그 소비).

### 4. 실행 진입점
`mvp/README.md`에 설치/실행법을 적고, 가능한 한 단일 명령(`streamlit run ...`, `uvicorn ...`)으로 띄운다.

### 5. 자가 검수
데모 시나리오(HS2류 질문→HS10+근거)를 1회 실행해 cycle·근거·시각화가 동작하는지 스스로 확인한 뒤 qa-gate에 인계.

### 6. 출력
코드 + `_workspace/build_app.md`(추론 구조, 프레임워크, 실행법, 데모 재현 절차, 자가검수 결과).

## 품질 기준
- 키 유무와 무관하게 데모가 실행됨(mock 폴백)
- FastAPI 응답 shape ↔ Streamlit 소비 코드 일치
- 관리자 화면이 근거/확신도를 시각적으로 보여줌

## 흔한 실수
- API 키를 필수로 만들어 데모가 안 돌아감
- 로그 필드와 시각화가 읽는 필드 불일치
- 추론 종료 조건 누락으로 무한 cycle
