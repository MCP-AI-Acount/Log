<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Tools/00-Index.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# Harness Tools — 초안·승격

> **정본**은 항상 레포 `.cursor/` 입니다. Obsidian(본 미러)은 **초안·메모·아카이브**만.

## 워크플로

1. 반복해서 잘 쓰인 패턴을 `staging/`에 마크다운으로 적는다.
2. frontmatter에 `status: ready` 를 넣는다.
3. Cursor에서 `/tool-promote` 또는 `python automation/tool_promote.py promote <파일>` 실행.
4. `python automation/harness_sync.py verify` 로 검증.
5. 승격된 원본은 `archive/`로 이동된다.

## frontmatter (필수)

```yaml
---
tool_type: skill    # skill | command | agent
tool_id: my-tool    # skill=폴더명, command/agent=파일 stem
status: draft       # draft | ready (ready만 승격)
description: 한 줄 설명
---
```

템플릿: `staging/_template-skill.md` · `_template-command.md` · `_template-agent.md`

## 사용 빈도 (자동)

서브에이전트 종료 시 `0_Config/tool_usage.jsonl`에 기록된다.

```bash
python automation/tool_usage_log.py report --days 7
```

자주 쓰이는 도구 → staging 초안 작성 · 덜 쓰이는 도구 → archive 검토 후 삭제(수동).

## 금지

- Obsidian만 수정하고 `.cursor/` 미승격 — Cursor는 vault를 읽지 않음
- `TOOL_ROUTING` 표를 Obsidian에 복제 — 정본은 `harness_reference_tables.py`
