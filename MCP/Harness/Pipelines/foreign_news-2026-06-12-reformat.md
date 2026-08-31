<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Pipelines/foreign_news-2026-06-12-reformat.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# 외신 카드 — 2026-06-12 형식 수정 재게시

> `Harness/Pipelines/` · 기술 저장소 미러

## 요약

- **작업**: 기존 스냅샷(`foreign-news-daily-20260612.json`) 유지, Canva 본문 `\n\n` → `\n` (불릿 빈 줄 제거)
- **방법**: `--canva-only` → SNS 재게시
- **결과**: SNS 성공 (Facebook · Instagram(queue) · Threads)

## 산출물 (로컬)

| 파일 | 설명 |
|------|------|
| `temp/foreign-news-daily-20260612.json` | 원본 스냅샷 (미변경) |
| `temp/foreign-news-daily-20260612-canva-fields.json` | 수정된 Canva 필드 (CONTENTS 3줄) |
| `temp/foreign-news-daily-20260612-canva.png` | Canva PNG |
| `temp/foreign-news-daily-20260612-reformat-publish.json` | SNS 게시 결과 |

## SNS

- as-of: 2026.06.12.(금) 오전 7시 기준
- public URL: https://storage.googleapis.com/mcp-auto-assets-project-6868f681-721e-4d33-b59/social/foreign_news/1781224291.png

## 코드 변경

- `card_news_settings.format_canva_contents_text` — 단일 줄바꿈
- `pipeline_staging_backup.py` — SNS 실패 시 GCS `pipeline-staging/{pipeline}/` (성공 시 삭제)

## Runbook

- [[Harness/Runbooks/pipeline-staging-gcs]]
