<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Runbooks/canva-oauth-revoked.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# canva-revoked

- pattern: `revoked|Token lineage|canva.*401`
- step: `canva-export`
- fix: BEFORE OAuth: curl Cloud Run /health+/v1/warm (single owner=broker). revoked=false → DO NOT re-OAuth (used twice/lineage). Mac/VM :8792 MUST set CANVA_BROKER_URL relay — never dual refresh. Only revoked/lineage → bash EXE/setup_canva_connect_oauth.sh start→finish → GCS push → warm once → keepalive.

> auto-generated from 0_Config/daily_pipeline_runbook.json
