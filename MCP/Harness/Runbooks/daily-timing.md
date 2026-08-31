<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Runbooks/daily-timing.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# slow-recovery

- pattern: `retry|timeout|cold-start|ensure_daily|wait-base`
- step: `timing`
- fix: 브릿지 이미 up이면 fast-path(즉시). recovery: bash EXE/run_daily_recovery.sh --skip-foreign (날씨+경제만). BRIDGE_READY_TIMEOUT_SEC=45, DAILY_SKIP_PREFLIGHT=1 after ensure.

> auto-generated from 0_Config/daily_pipeline_runbook.json
