# Harness/Realm.md

> 자동 갱신: 2026-06-07 14:48:52 KST | `python automation/harness_sync.py all`

## 상태 플래그

- harness_ok: `True`
- cursor_rules: `7`
- commands: `5`
- skills: `3`
- agents: `2`
- rules_total_lines: `1291`

## 최적화 제안
- cursor rules 총 라인이 400 초과 — file-specific rule로 분리 검토
- rules 파일 수 과다 — agent-tooling 외 glob rule로 분리 검토

## 상세

전체 스냅샷은 `tests/result.json`, Obsidian `Harness/CI/` 참조.

## TRAP_REGISTRY

연옥·아마겟돈 탈출 시 등록. 서술형 금지 — 테이블만.

| id | bug_pattern | trigger_condition | confirmed_fix |
|----|-------------|-------------------|---------------|

## JUDGMENT_LOG

담합 방지 — Devil_Sentinel이 FAIL/PASS 선언 시 독립 증빙과 함께 등록.
동일 verdict 연속 3회 이상이면 Judge 소환 트리거.

| timestamp | role | verdict | k2_code | evidence_summary |
|-----------|------|---------|---------|-----------------|
| 2026-06-06 01:26:17 KST | Devil_Sentinel | PASS | - | 테스트 통과 확인 |
