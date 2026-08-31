# 노트북LM 대체 파이프라인 (MCP 기반 RAG, OR 5단계 게이트 배선)

> 노트북LM은 개인용 공식 API/MCP가 없음(엔터프라이즈 GCP API만 존재). 커뮤니티 `notebooklm-mcp`류는 스텔스 브라우저 자동화로 로그인 세션을 스크래핑하는 방식이라 ToS/보안 리스크 있어 채택하지 않음. 대신 **Obsidian MCP + `collaboration/openrouter_client.py`(OR 5단계 게이트)**로 동일한 "내 문서 근거 기반 Q&A/요약" 기능을 대체 구현.

## 정정 이력 (2026-07-06)

1차 버전은 `mcp__openrouter__chat_completion`(범용 MCP 커넥터)로 모델을 직접 지정해 호출 — 유저가 "그건 내가 세팅한 오픈라우터 5단계 맞지?"라고 확인 요청, 실제로는 **아니었음**(별도 경로, org 예산·curfew 미적용). 유저 선택("5단계 게이트로 재배선")에 따라 `collaboration/openrouter_client.py`의 `chat_with_gate()`로 전환. 재배선 직후 실제 curfew(299s)에 걸려 게이트가 살아있음을 확인.

## 구조적 이슈 발견 및 수정 (2026-07-07, ntd-099)

**문제**: `chat_with_gate()`는 free 시도가 429(HTTP)를 맞으면 즉시 `ops_auth_gate.record_openrouter_429()`로 전역 curfew(300s)를 세팅하는데, 같은 호출 안에서 이어지는 paid 폴백까지 이 curfew에 막혔음 — `openrouter_calls_allowed(tier='paid')`가 `or_mission_mode()`(카드뉴스 등 특정 파이프라인 env로만 활성화)가 꺼져 있으면 paid 요청도 즉시 차단. 무료 모델의 전세계적 혼잡(계정 자체 문제 아님, `/auth/key` 확인 결과 하드 리밋 없음) 한 번으로 같은 호출의 유료 폴백까지 연쇄 차단되는 구조 — `chat_with_gate`를 쓰는 모든 파이프라인(`topic_news_daily_card.py`, `interview_llm_client.py`, `parallel_universe_pipeline.py`, `cerberus_llm_heads.py`)에 영향 가능했음.

**수정(결합안)**: `ops_auth_gate.openrouter_calls_allowed()`에 `allow_override: bool = False` 파라미터 신설 — 우회 조건이 `or_mission_mode() OR allow_override`로 확장됨. `openrouter_client.py`의 `_post_json → chat_with_usage → chat_with_gate` 체인에 `allow_paid_override: bool = False`를 관통시킴. 기존 스케줄 파이프라인(카드뉴스 등)의 env 기반 자동 통과는 그대로 유지(회귀 없음, 기본값 False), ad-hoc 호출자는 `chat_with_gate(..., allow_paid_override=True)`로 명시 옵트인해 free 혼잡 curfew 중에도 paid 폴백을 받을 수 있음. 1안(paid 항상 예외)은 "ad-hoc 스크립트가 조용히 자동으로 유료 전환되는 것을 막는다"는 원래 브레이크 의도를 없애버려 기각. 테스트 `tests/test_openrouter_bottleneck.py`에 회귀+신규 4케이스 추가, 전체 PASS.

**향후 이 파이프라인에서 congestion 시**: 아래 예시처럼 `allow_paid_override=True`를 넘기면 됨 — 더 이상 `CARD_OPS_MISSION=1` 같은 무관한 env 우회가 필요 없음.

## 트리거

사용자가 CC에게 "노트북LM 대신 OO 폴더/노트 요약·분석해줘" 또는 "이 문서들 근거로 질문 답해줘" 라고 요청.

## 파이프라인

1. **수집**: `mcp__obsidian__list_notes` / `read_note`로 대상 폴더·파일 원문 확보
2. **합성**: Bash로 `collaboration/openrouter_client.py`의 `chat_with_gate()` 호출 (OR 5단계 — 무료 3회 → 유료 2회, org 예산·curfew·healthy_free_pool 로테이션 전부 적용):
   ```python
   import sys
   sys.path.insert(0, "collaboration")
   from openrouter_client import chat_with_gate

   messages = [
       {"role": "system", "content": "제공된 소스 문서에만 근거해서 답하고, 각 주장 끝에 [출처: <파일경로>] 인용을 달아라. 소스에 없는 내용은 추측하지 말고 '소스에 없음'이라고 밝혀라."},
       {"role": "user", "content": "=== 소스 1: <path> ===\n<본문>\n\n질문: ..."},
   ]
   text, source = chat_with_gate(
       messages, slot="notebooklm_alt", temperature=0.3, max_tokens=2048,
       allow_paid_override=True,  # free 혼잡 curfew 중에도 paid 폴백 허용 (2026-07-07 신설)
   )
   # source 예: "openrouter:nvidia/nemotron-3-ultra-550b-a55b:free:free" 또는 ":paid" 폴백
   ```
   - `slot="notebooklm_alt"`로 고정 — `or_free_tier_budget.py`가 슬롯 단위로 무료 소진 추적
   - `allow_paid_override=True` 덕에 free 429 curfew가 떠도 paid 폴백은 정상 진행됨 (2026-07-07 이전엔 5분간 완전 차단됐음)
3. **저장**: 결과를 `NotebookLM-Alt/Insights/<날짜>-<주제>.md`에 `write_note`로 저장, frontmatter에 `source` 라벨(무료/유료 실제 사용 모델) 기록

## 한계

- 오디오/비디오 개요 생성 기능 없음 (노트북LM 고유 기능, 대체 안 함)
- 소스 총량이 모델 컨텍스트 한도 넘으면 폴더를 나눠서 여러 번 호출 필요
- **org 공유 curfew/예산** 적용 대상 — 다른 자동화 파이프라인(카드뉴스 등)과 같은 한도 공유. free curfew는 여전히 free 재시도 자체는 막음(D-24 취지 유지), paid 폴백만 `allow_paid_override`로 우회 가능

## 실행 기록

- 2026-07-06 1차: `mcp__openrouter__chat_completion` + `google/gemini-3.5-flash`(유료, $0.0085) — 정상 작동하나 게이트 미적용 경로였음
- 2026-07-06 2차: 같은 MCP 경로 + `nvidia/nemotron-3-ultra-550b-a55b:free`(무료, $0) — 유저 "오픈라우터 자체 기능 써서 무료로" 지시 반영, 여전히 게이트 미적용
- 2026-07-06 3차(확정): 유저 확인 결과 실제 OR 5단계 게이트가 아님이 드러나 `collaboration/openrouter_client.py chat_with_gate(slot="notebooklm_alt")`로 재배선. 즉시 curfew(299s) 발생 확인 — 게이트 정상 작동 실증. 다음 세션에서 curfew 해제 후 전체 데모 재실행 필요(NTD 등재)
- 2026-07-07(ntd-099 완료): curfew 해제 후 재시도했으나 2회 더 429 재발(D-24 3회 한도 근접) — ps aux/list_sessions로 세션 경합 조사했으나 무관 확인, 근본원인은 free→paid 자기차단 구조(상단 참조). `CARD_OPS_MISSION=1` 1회 우회로 paid 폴백(`~anthropic/claude-fable-latest:paid`) 성공, Harness/Rules/00-Index.md+RULE_TREE.md 종합 요약 생성 → [[../../NotebookLM-Alt/Insights/2026-07-07-rule-docs-summary]]
- 2026-07-07(구조 수정): 위 §구조적 이슈 발견 및 수정 — `allow_paid_override` 파라미터 신설로 근본 해결, 향후 이 파이프라인은 `CARD_OPS_MISSION` env 우회 불필요
- 데모 산출물(1~2차 결과) → [[../../NotebookLM-Alt/Insights/2026-07-06-rule-tree-synthesis]] (레거시 경로 산출물 — 내용은 유효하나 파이프라인 경로는 3차 기준으로 교체됨)
