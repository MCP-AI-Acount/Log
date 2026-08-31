<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Runbooks/pipeline-staging-gcs.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# Pipeline staging (GCS) — SNS 실패 시에만

> 기술저장소 정본 · Obsidian `Harness/Runbooks/` 미러

## 목적

카드(Canva)까지 완료된 뒤 **SNS만 실패**하면, NewsAPI·LLM·만평·Canva를 다시 돌리지 않고 **게시만 재시도**할 수 있게 한다.

## 규칙

| 상황 | GCS `pipeline-staging/{pipeline}/` |
|------|-------------------------------------|
| SNS **성공** | **삭제** (백업 없음) |
| SNS **실패** | manifest + card.png + snapshot 등 **1세트만** 저장 (이전 staging 삭제 후 교체) |

## 경로

- GCS: `gs://{N8N_GCS_BUCKET}/pipeline-staging/{pipeline}/`
- manifest: `manifest.json` — `publish_payload`, `png_url`, `uploaded` URIs
- 코드: `collaboration/pipeline_staging_backup.py`
- 연동: `social_daily_post.publish_all`, `daily_sns_queue.publish_pending`

## 재시도

1. manifest의 `publish_payload`로 `POST :8795/v1/publish`
2. 또는 `bash EXE/run_daily_catchup_sns.sh` (큐에 남아 있을 때)

## 비활성화

`PIPELINE_STAGING_DISABLE=1` 또는 `N8N_GCS_BUCKET` 미설정 시 no-op.
