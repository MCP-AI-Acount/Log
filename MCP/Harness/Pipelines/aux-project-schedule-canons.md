<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Pipelines/aux-project-schedule-canons.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# 보조 프로젝트 스케줄 정본 (Fatum)

> **Fatum id**: `volva-LT-aux-project-schedule-canons`  
> **레포 인덱스**: `0_Config/aux_project_schedule_index.json`  
> **Archetype 정책**: `0_Config/archetype_policy.json` (실행·수정 참조 · 구문서 처리)  
> **양식**: `0_Config/archetypes/Archetype_Template.md`  
> **대시보드**: https://web-browser-dashboard-k2lum4ai7q-du.a.run.app  
> **등록**: 2026-07-22 — 유저: 보조 프로젝트별 정본 MUST · Fatum 보관  
> **갱신**: 2026-07-28 — card-news **시각표 본문 삭제** → Archetype + schedule JSON 포인터만 (드리프트 방지)  
> **Fatum**: `volva-LT-aux-project-schedule-canons` (전 제품 Archetype + 방향 정렬) · `volva-LT-genesis-planning-to-archetype` · `volva-LT-classify-rules-for-canon`  
> **정책**: `0_Config/archetype_policy.json` (`on_planning_confirmed` · `project_direction`)

## 공리

각 제품 파이프라인은 **자기 generate_at 전 생성·게시 금지**.  
스케줄이 없으면 `status=na` + 이유 1줄. catchup/recovery도 동일.  
**시각은 이 MD에 복제하지 말 것** — Archetype A1 → 기계 JSON만.

## 제품 체크리스트

| product | status | Archetype | schedule / config |
|---------|--------|-----------|-------------------|
| card-news | **done** | `0_Config/archetypes/Archetype_CardNews.md` | `0_Config/card_pipeline_schedule.json` |
| youtube | **stub** | `0_Config/archetypes/Archetype_Youtube.md` | `0_Config/youtube_caption_digest.json` |
| gcs-icloud-relay | **stub** | `0_Config/archetypes/Archetype_GcsIcloudRelay.md` | `0_Config/gcs_icloud_relay.json` |
| slack-voice-calendar | pending | Template 복제 대기 | — |
| browser-agent | pending | Template 복제 대기 | — |
| project-maker | pending | Template 복제 대기 | — |
| swiss-offline-guide | pending | Template 복제 대기 | — |
| italy-offline-guide | pending | Template 복제 대기 | — |
| walking-offline-guide | pending | Template 복제 대기 | — |
| unity | pending | Template 복제 대기 | — |

## card-news (포인터만)

- 운영 맵: `Archetype_CardNews.md` (A0–B)  
- 시각 정본: `card_pipeline_schedule.json` (`pipelines` · `weekly_pipelines`)  
- 실행 읽기: Homunculus `load_archetype_then_schedule` · 점검: `0_Config/archetype_reports/card-news/`

## 착수

Fatum 자동 착수 금지. 유저가 `volva-LT-aux-project-schedule-canons` 지목·승인 시 pending 제품 → `Archetype_Template` 복제 + JSON/na.
