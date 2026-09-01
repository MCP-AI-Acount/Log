# Next Session Prime

> **목적**: 다음 세션 시작 시 자동 로드 — 맥락 복원용. 수동 편집 금지(자동 갱신).

## 직전 세션 인계 항목

- [M1 evidence] 아카이브 24h 레이스 — closed_at 기록 후 24h 내에 같은 날 후속 세션이 시작하면 미아카이브 저널이 라이브 슬롯에서 밀려나 _archive/journals/ 에 영영 도달하지 못함(git 에만 잔존). 단일 슬롯 원장 계열 — fatum-LT-approval-ledger-single-slot-overwrite 와 동일 병.
- [M1 evidence] 0_Config/_archive/ 는 config_registry 가 에이전트 쓰기를 차단하는데, 그 디렉터리로 옮겨줄 기계 경로가 무엇인지 미특정 — 이동자 부재면 아카이브는 선언만 있고 집행이 없는 상태(전 세션 회고의 '선언과 집행이 따로 자람' 재발).
- [신규 결함 · T3 가드 오탐] cc-pre-t3-delegation-guard.sh:355-358 — 인라인 인터프리터(python3 -c·히어독) 명령 안의 '따옴표 친 / 포함 문자열' 을 전부 경로 후보로 수집한 뒤 normalize() 가 상대경로를 **레포 루트 기준으로 resolve** 한다. 그 결과 스크래치패드 전용 스크립트 안의 상대 심링크 타겟('_journals/scope-a.json')이 레포 파일로 둔갑해 차단됨. is_allowlisted 는 /private/tmp/ 를 정상 허용하므로 절대경로만 썼다면 통과했을 것 — 즉 allowlist 가 아니라 상대경로 재해석이 원인. 이미 사망선고된 fatum-LT-gitqueue-touched-paths-scratchpad-leak(rel_repo_path 컨테인먼트 결함)과 **동일 계열 재발**: 상대경로를 컨테인먼트 검사 없이 레포 경로로 승격. 추가로 is_variable_path 가 $ 포함 경로를 전부 보수적 차단해 셸 변수 쓰는 스크래치패드 작업도 막힘.
- [세션 블로커] T3 §8 전면위임 가드가 CC 직접 코드 Write 를 차단. 유효 grant 없음 — user_explicit_approval.json 은 automation/t3_approval_record.py scope·2026-08-05T14:47:43Z·ttl 60m(만료), t3_direct_grant.json 은 2026-07-27·ttl 30m(만료). 셀프승인은 가드가 명시 차단(SELF_APPROVAL_BLOCK_MSG)이라 AI 가 원장을 쓸 수 없음 — 유저 채팅 발화 → UserPromptSubmit 훅 기록이 유일한 정당 경로. 해소 선택지: (a) cc_bridge push 로 Cursor 위임 (b) 유저 L1 명시 승인.

## 최근 세션 인텐트 체인 (최신 → 과거)

- **2026-08-06 [2026-08-05-harness-root-repair]**: 이월된 하네스 근본 수리 5건(M1 저널 스코프화 · M2 데몬 이중기동 · M3 verify 린트 · M4 게이트 코퍼스 · M5 승인기록 누락) 처리

## 상호 연결 메모

_GitHub Actions가 세션 간 공통 키워드·피벗을 추출해 여기에 추가 예정._

_generated: 2026-09-01 18:36 UTC_
