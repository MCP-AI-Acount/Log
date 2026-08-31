<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Runbooks/n8n-bridge-host.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# n8n-bridge-host

- pattern: `172\.17\.0\.1|N8N_DAILY_BRIDGE_HOST`
- step: `n8n-http`
- fix: 맥 native n8n: N8N_DAILY_BRIDGE_HOST=127.0.0.1 + bash EXE/sync_local_daily.sh

> auto-generated from 0_Config/daily_pipeline_runbook.json
