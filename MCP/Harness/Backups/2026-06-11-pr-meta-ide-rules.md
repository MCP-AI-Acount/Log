<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Backups/2026-06-11-pr-meta-ide-rules.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# Harness 백업 — 2026-06-11 (PR1 · IDE rules · PR 메타 운용)

## 현재 버전

| 항목 | 값 |
|------|-----|
| **BUILD_ID** | `20260611222059` |
| 대시보드 표기 | **현재 버전 - 20260611222059** |

## 이번 완료 범위

| 항목 | 내용 |
|------|------|
| **PR1** | 맥락 → 문자 → AI 추론 3축 (Essence §0-A-PR) |
| **PR 메타 지시 읽기 (운용)** | 부정·한정 / 목적어·장소 / 시점·순서 / 층 구분 — PR1 보조 절, 신규 PR 코드 없음 |
| **IDE User rules** | 개요 상단 노출 · 한글 미러 (`Main Rules/ide_user_rules_communication.md`) |
| **분리** | PR = `.mdc` · IDE 1-12 = 커뮤니케이션 · `readingPriority` UI 제거 |
| **대시보드** | 단어 정의 탭 PR 메타 운용 4행 · Rule Gate initials `PR 메타 운용 vs B8` |

## 정본 경로

| 층 | 파일 |
|----|------|
| PR1 + 운용 | `.cursor/rules/cursorrules_Essence.mdc` §0-A-PR |
| K-6-A 요약 | `.cursor/rules/cursorrules_Core.mdc` |
| 표·미러 | `automation/harness_reference_tables.py` |
| IDE 미러 | `Main Rules/ide_user_rules_communication.md` |

## verify

```bash
python automation/harness_sync.py verify   # PASS (2026-06-11)
```

## 배포

```bash
bash EXE/sync_dashboard.sh      # 스냅샷 → Cloud Run (DASHBOARD_WEBHOOK_TOKEN)
bash EXE/deploy_dashboard.sh    # BUILD_ID·html/js 반영
```
