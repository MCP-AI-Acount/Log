---
tags: [incident, harness, investigation-closed]
ts: 2026-07-04T13:20:00Z
error_key: blackpaper-hook-smoke-test-state-pollution
status: investigation-closed
---
<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/2026-07-04-blackpaper-hook-smoke-pollution.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# blackpaper 훅 검증 스모크 — 프로덕션 상태 오염 (확정·정정 완료)

## 관측된 현상

`cc_bridge` executor 태스크 `20260704-220109-52382`("blackpaper 실집행 강제", `.claude/hooks/cc-error-blackpaper.sh` 3-mode 구현) 완료 직후, 무관한 별도 CC 세션의 Stop 훅이 다음 이유로 차단됨:

```
[블랙/화이트페이퍼 BLOCK — N5 집행경로] trigger=priming-text
keys=user-correction-틀렸어왜그랬어-2026-07-04
```

## 확인된 사실

- `0_Config/cc_error_log.jsonl`에 동일 `error_key`로 13:17:57~13:18:39(42초) 사이 attempt 1~4(scope L1→L4) 연속 append
- `desc` 필드가 "틀렸어 왜 그랬어" — 저장소 전체(`grep -r`, .py/.sh/.md)에 이 문자열이 코드·테스트 픽스처로 존재하지 않음
- `0_Config/o_format_state.json`의 `blackpaperSession`에 `ccWriteMtime`만 있고 `ccWriteTs`/`ccWritePath`가 없는 비정상 상태(정상 흐름이면 `_mark_cc_write`가 셋을 항상 함께 씀)
- 타임스탬프가 task `20260704-220109-52382` 완료 시각과 근접

## 추정 (미확정)

executor가 `.claude/hooks/cc-error-blackpaper.sh`를 검증하며 `CC_BLACKPAPER_MODE=user-prompt`로 합성 프롬프트를 **프로덕션 `ROOT` 경로**(`ROOT` 환경변수 미override)에 대고 직접 실행 → 실제 운영 로그·상태 파일에 가짜 correction이 기록되고, 그 잔여 트리거가 이후 무관한 세션까지 오탐 차단.

**미확정 이유**: executor의 정확한 실행 커맨드·컨텍스트를 직접 확인하지 못함 — 정황 증거(문자열 부재+시각 근접+상태 비정상)만으로 CC가 스스로 결론 내려 로그를 무효화하는 것은 auto-mode 분류기가 차단(Logging/Audit Tampering 사유). Cursor 디베이트로 재확인 필요.

## 확정 (2026-07-04 Devil 판정)

Angel 조사 + `o_format_state.json` `recentShellCommands`(L306–315) 교차 확인으로 **확정**. task 52382 executor가 프로덕션 ROOT에서 `"틀렸어 왜 그랬어"` user-prompt ≥4회 + Stop 스모크 전 `ccWriteTs`/`ccWritePath` 수동 pop.

**정정**: VOID append (attempt 1~4 보존) · resolution closed · `blackpaperSession` reset · `tests/lib/hook_sandbox.sh` + `tests/test_cc_error_blackpaper.sh` · `smoke_hooks.sh` stateful hook sandbox 격리.

## 요청 (Cursor 위임 예정)

1. `cc-error-blackpaper.sh` 검증에 실제로 어떤 커맨드가 쓰였는지 확인(쉘 히스토리·result.md 상세본)
2. 근본 원인 확정 시 재발 방지: 훅 자체 스모크 테스트는 격리된 `ROOT`(임시 디렉터리)로 실행하도록 테스트 구조 수정
3. 확정 후 오염된 4건(`cc_error_log.jsonl`/`cc_resolution_log.jsonl`/`o_format_state.json`)의 정정 여부·방법 결론
4. 비투(F1~F4) 재검증

## 관련

- 유사 과거 패턴: git_queue foreign lease overlap 사고(`154bf7b`) — 공유 mutable state를 여러 세션이 동시에 건드릴 때 발생하는 동일 계열 구조적 위험
