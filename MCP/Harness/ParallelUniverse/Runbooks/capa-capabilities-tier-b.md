<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/ParallelUniverse/Runbooks/capa-capabilities-tier-b.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# capa — Centralized Configuration (Tier B 참고)

> **미적합 사유**: K-0-C 단일 Rule Gate — `capabilities.yaml`을 Core에 편입 금지.

## MCP-Auto에서 쓸 수 있는 부분

- MCP 서버 **descriptor 선행** (agent-tooling K-7)
- `CallMcpTool` 전 스키마 읽기

## 쓰지 않는 부분

- 레포 밖 전역 capability registry
- capa 이벤트 버스 → Cursor hooks 대체
