<!-- LEGACY_UNIVERSAL: Three_Chronicle_Temporal_Temporal_Realm_World_.md -->
<!-- HARNESS_STAMP: role=Angel_Chronicle_Chronicler stage=Temporal ts=2026-08-24 22:20:11 KST -->

# Realm — 기술 맥락·운영 기록 (DOC_TECH_CONTEXT)

> 자동 갱신: 2026-08-24 22:20:11 KST | `python automation/harness_sync.py all`

## 상태 플래그

- harness_ok: `False`
- cursor_rules: `18`
- commands: `30`
- skills: `13`
- agents: `2`
- rules_total_lines: `4094`

## 최적화 제안
- alwaysApply 565줄 — agent-tooling 슬림화·Tier B 지연 로드 (목표 ≤150)
- cursor rules 총 라인이 400 초과 — file-specific rule로 분리 검토
- rules 파일 수 과다 — agent-tooling 외 glob rule로 분리 검토

## 상세

전체 스냅샷은 `tests/result.json`, Obsidian `Harness/CI/` 참조.

## CARD_OPS_POLICY

> 정본: `0_Config/card_ops_policy.json` · 버전 `2026-07-29.6` · 상태 `active-refinement`

주간·일일알림 통합 이후 첫 안정 운영 전. 레거시 플로우 전체 롤백 금지 — 현행 방식 보완·버전업. 2026-07-29.5: economy/weather 필드 실패는 스냅샷 재수집 후 재검증 · weekly_inform은 bridge 전체 재실행만 허용. 2026-07-29.6: catchup never-ran carve-out · n8n bridge host/errorWorkflow · missed-pipeline Slack — 포인터만(아래 publish_workflow.regression_guards).

### 브리프 체인

- 1. OpenRouter 무료 (or_free)
- 2. OpenRouter 유료 (or_paid)
- 3. AI Studio (stage2) (gemini_studio)

- 품질 실패: 기사 유지 → 해당 기사(부분)만 다음 AI · 상위 run_pre_publish 재시도≤2 후 슬랙
- 게시: 4칸 중 1칸 실패 시 카드 전체 미게시

### NewsAPI

- base=global_soft_max / n (균등 cap) · max=3× · RSS=일일 한도 소진 시에만 (기사 수집 실패 ≠ RSS) · counter=newsapi_budget.py 단일 (legacy newsapi_quota_*.json shim)

### Rate limit 우선순위

- 순서: mission_complete > token_economy > deadline_compliance
- per-minute 429: {"if_deadline_lt_wait_sec": "paid_openrouter", "if_no_surplus_and_pipeline_used_gte_base": "paid_openrouter", "else": "sleep_or_defer_parallel", "defer_note": "surplus 이월·base 미만 사용 중이면 sleep/defer 유지. 그 외 유료 OR 즉시.", "surplus_note": "앞 스케줄이 base 미만으로 끝나면 surplus 이월 → 뒤 파이프라인 시도 횟수 증가(현행 유지)"}

### revisionProtocol (운영 실패 시)

- 전략: `in_place_refinement` · legacy 전면 롤백: `False`

1. 0. D-18 register-error-fix — 1차 원인·해결안·logSnippet (수정 전)
2. 0.5 Gate0 ops_auth_preflight — 실패 로그 tail 스캔 → 토큰·권한·oauth 우선 검사 · blocks_catchup FAIL 시 recovery/catchup 중단
3. 0.55 ensure_daily_bridges → Gate0 순서 (catchup) — bridge health 선행
4. 0.7 D-19-A sns_closure — catchup/recovery·세션 종료 전 run_daily_sns_closure --assert
5. 0.6 D-21 갱신체계 — 신규 로그·문서면 retention 등록(refresh_cadence) · 규칙 없으면 유저 질문 · 자동 prune 전후 알림
6. 1. pipeline_ops_log + 홈page 실패 로그에 failureClass·원인 기록
7. 2. policy_version patch (0_Config/card_ops_policy.json) — 실패 규칙만 수정
8. 3. harness_sync all → Realm·Obsidian·홈page 스냅샷 갱신
9. 4. register-ops로 06_Operations_Log에 결론 1행
10. 5. 연속 실패 시 TRAP_REGISTRY 후보 — register-trap 검토


## CUSTOMARY_REGISTRY

관습 헌법 (Core K-0-D). K-2 금지규칙 위반 override만 · effective≥7 등재 · effective≤3 제외.

| rule_ref | intent | override_count | elevated_at | note |
|----------|--------|----------------|-------------|------|
| Core K-2.3 | [O-gate VIOLATION] 직전 응답에 xenia-a-self-admission-undeclared 누락. Write/Edit/Bash 포함 턴에는 반드시 포함해야 함. 지금 바로 O패널 전체를 다시 출력하라: ✅ **O1**(실측 한 줄 + | 7 | 2026-08-24 19:32:47 KST | auto-lifecycle-refresh |
| Core K-2.1 | 여기서 구름 크기만 키우라는게 그렇게 어려웠냐...... 만들진 말고 대답만 해봐 | 7 | 2026-08-24 19:32:47 KST | auto-lifecycle-refresh |

## TRAP_REGISTRY

연옥·아마겟돈 탈출 시 등록. 서술형 금지 — 테이블만.

| id | bug_pattern | trigger_condition | confirmed_fix |
|----|-------------|-------------------|---------------|
| T-session-face-soft-hard-collapse | soft OFF interpreted as hard OFF | session_face UF/IN | soft_ne_hard + assert |
| T-session-face-downshift | AI downshifts SF-T severity for convenience | session_face route | assert_no_downshift |
| T-catchup-never-ran-deadlock | or_stages_completed_missing on never-ran state entry=None | homunculus_or_stages.gate_catchup_pipelines · rule_ref=card_ops_policy.publish_workflow.regression_guards.catchup_never_ran | entry is None → catchup_allowed carve-out · collaboration/homunculus_or_stages.py#gate_catchup_pipelines |
| T-n8n-bridge-host-loopback | N8N_DAILY_BRIDGE_HOST=127.0.0.1 or missing → ECONNREFUSED from n8n container | recovery_preflight.check_n8n_bridge_host · EXE/sync_n8n.sh · rule_ref=T-n8n-bridge-host-loopback | require 172.17.0.1 or non-loopback · FAIL if loopback/missing · skip(ok) if docker unavailable |
| T-slack-failure-alert-silent | report_failure TypeError on unknown kwargs OR missed-pipeline notify path absent | daily_failure_report.record_ops · homunculus.notify_missed_pipelines · rule_ref=card_ops_policy.publish_workflow.slack.missed_pipelines | match record_ops signature · notify_missed_pipelines bundles Slack · no silent swallow |
| T-ops-log-test-pollution | pytest writes live 0_Config/pipeline_ops_log.jsonl | HARNESS_OPS_STATE_DIR Ether · tests autouse fixture · rule_ref=K-3 Ether | tests must redirect STATE_DIR/tmp · never mutate live pipeline_ops_log |

## JUDGMENT_LOG

담합 방지 — Devil_Judgment_Sentinel이 FAIL/PASS 선언 시 독립 증빙과 함께 등록.
동일 verdict 연속 3회 이상이면 Judge 소환 트리거.

| timestamp | role | verdict | k2_code | evidence_summary |
|-----------|------|---------|---------|-----------------|
| 2026-08-24 01:06:40 KST | surface-decision | reject | - | 확장 package.json 실측: description='GUI for Grok Build CLI' · b |

## BABEL_FAULT_LEDGER

자가 강화 과오 피드백 (Essence §6 바벨 · Detail D-18-D). babel_ledger.py 소유.
재발 +1 · 수정지시 +5 · 5배수=강화+Slack · 10배수=1문장 응축·기존삭제(강도 누적).

| key | 누적 | 비율% | 응축 | 현재 강화/응축 문장 |
|-----|------|-------|------|---------------------|
| stop-or-defer-when-told-continue | 690 | 22.7 | 69 | [바벨 응축69·강도누적] 'stop-or-defer-when-told-continue'는 누적 690회 반복된 만성 과오 — 최우선 차단. |
| d18-skip-register-on-user-correction | 401 | 13.2 | 40 | [바벨 응축40·강도누적] 'd18-skip-register-on-user-correction'는 누적 401회 반복된 만성 과오 — 최우선 차단. |
| uncategorized-correction | 375 | 12.3 | 37 | [바벨 응축37·강도누적] 'uncategorized-correction'는 누적 375회 반복된 만성 과오 — 최우선 차단. |
| lethe-stenosis | 165 | 5.4 | 16 | [바벨 응축16·강도누적] '레테 협착 — 집행 코드 생존·발화 0'는 누적 165회 반복된 만성 과오 — 최우선 차단. |
| sige-orphan | 154 | 5.1 | 15 | [바벨 응축15·강도누적] '시게 잔존 — 선언만 있고 기록 코드 부재'는 누적 154회 반복된 만성 과오 — 최우선 차단. |
| next-session-missions-stringified-carryover-20260723 | 61 | 2.0 | 6 | [바벨 응축6·강도누적] 'next-session-missions-stringified-carryover-20260723'는 누적 61회 반복된 만성 과오 — 최우선 차단. |
| epoch-git-reconfirm-blind | 48 | 1.6 | 4 | [바벨 응축4·강도누적] 'epoch-git-reconfirm-blind'는 누적 48회 반복된 만성 과오 — 최우선 차단. |
| lethe-erosion | 47 | 1.5 | 4 | [바벨 응축4·강도누적] '시게 침식 — 선언만 남고 집행 표지 0'는 누적 47회 반복된 만성 과오 — 최우선 차단. |

