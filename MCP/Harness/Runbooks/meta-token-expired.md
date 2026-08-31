<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Runbooks/meta-token-expired.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# meta-token-expired

- pattern: `Session expired|EAAL|OAuthException|error.*190|invalid.*token|Graph.*102`
- step: `sns-publish`
- fix: |
  1. `bash EXE/run_meta_sns_recover.sh auto` (Explorer 열림)
  2. Graph API Explorer → Generate Access Token → EAAL 복사
  3. `bash EXE/run_meta_sns_recover.sh finish-token '<EAAL...>'`
  4. `bash EXE/run_meta_sns_recover.sh sync`
- trap: `T-meta-token-expired` (Realm TRAP_REGISTRY)
- never: VM ssh로 토큰 교환 · Access Token Debugger만 안내

> synced from 0_Config/daily_pipeline_runbook.json — 2026-06-14
