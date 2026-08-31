# 기술 보고서 — SNS 2단계 게시 + 정시 스케줄 (2026-06-11)

> **정본 코드**: `collaboration/sns_instagram_schedule.py`, `social_daily_post.py`, `daily_ops_bridge.py`  
> **TRAP**: `Realm.md` T012  
> **운영 로그**: `06_Operations_Log.md` (2026-06-11 16:04 KST)

---

## 1. 문제 요약 (2026-06-11 장애)

| 증상 | 원인 |
|------|------|
| catchup SNS 404 | `daily_ops_bridge`에 `/v1/sns-queue`, `/v1/enqueue-sns`, `/v1/publish-pending-sns` 미구현 |
| Instagram `Application request limit reached` | 날씨·경제·외신을 **동시에** FB+IG+Threads 3플랫폼 연속 호출 (파이프당 API 2회) |
| Slack `not_in_channel` | `slack_post` 실패 시 `SystemExit` → 카드 생성 후 SNS 단계까지 못 감 |
| 외신/경제 shell trap | `CARD_JSON`/`BASE_JSON` unbound variable |

**오늘(06-11) 복구**: 수동 catchup으로 FB+Threads ✅, IG ⚠️ rate limit. ops state `ok` (IG는 큐 패턴 도입 전).

---

## 2. 확정 패턴 — 2단계 SNS 게시

```
[카드 완료] → wait-publish (정시까지 대기) → Phase1: FB + Threads 즉시
                                              → Phase2: Instagram 큐 enqueue (scheduled_at)
[5분마다] run_instagram_queue_drain.sh → POST /v1/publish-due-instagram
[n8n 종료 시] Wait IG slot → Drain Instagram queue (워크플로 내 1회)
```

### Phase 1 — 즉시 (`defer_instagram: true`)

- **플랫폼**: Facebook, Threads
- **진입점**: `:8795/v1/publish` (`social_post_bridge.py` → `publish_all`)
- **플래그**: `defer_instagram: true`, `pipeline: <키>`

### Phase 2 — Instagram 지연

- **모듈**: `sns_instagram_schedule.py`
- **큐 파일**: `~/.instagram-sns-queue.json`
- **상태**: `~/.instagram-sns-state.json` (`last_published_at`, min gap)
- **최소 간격**: 900초 (15분) — `INSTAGRAM_SNS_MIN_GAP_SEC`
- **끄기**: `SOCIAL_DEFER_INSTAGRAM=0`

### Drain API

| Method | Path | 용도 |
|--------|------|------|
| GET | `/v1/instagram-queue` | 큐 스냅샷 |
| POST | `/v1/publish-due-instagram` | `scheduled_at ≤ now` 항목 게시 |
| POST | `/v1/publish-pending-sns` | 일간 SNS 큐 + **IG drain 연동** |

---

## 3. 정시 스케줄 (KST) — 확인 완료

> **wait-publish** (`daily_ops_bridge` `/v1/wait-publish`): `preferred` 시각 전이면 **실제 sleep**. 이미 지났으면 즉시 게시(`late: true`).

### 일간 (n8n + `card_news_settings.py`)

| 파이프 | cron 트리거 | wait-publish 목표 | FB/Threads | Instagram 큐 오프셋 |
|--------|-------------|-------------------|------------|---------------------|
| weather | 07:50 매일 | **08:00** | 08:00 | +0분 → ~08:00 |
| economy | 07:50 화–토 | **08:00** | 08:00 | +15분 → ~08:15 |
| foreign_news | 07:50 월–금 | **08:30** (`hour:8,minute:30`) | 08:30 | +30분 → ~09:00 |

- 외신: `wait-base-pipelines`로 날씨·경제 완료 후 카드 생성 → 08:30 wait 노드
- n8n JSON: `daily-*-canva.json` — `defer_instagram: true` + Drain 노드

### 주간 알림 (`weekly_inform_settings.py` + `wi_*.json`)

| 항목 | 값 |
|------|-----|
| cron 트리거 | 요일별 **08:40** |
| wait-publish | **09:00** (`publish_hour:9, publish_minute:0`) |
| FB/Threads | 09:00 |
| IG 큐 | +5분 (`INSTAGRAM_WEEKLY_DELAY_SEC=300`) → ~09:05 |
| n8n drain wait | 5분 (`_IG_RETRY_WEEKLY_MS`) 후 `publish-due-instagram` |

### 스케줄 검증 수식 (예시)

```
07:55 + wait(08:00) = 300초 대기 ✓
08:10 + wait(08:30) = 1200초 대기 ✓
08:45 + wait(09:00) = 900초 대기 ✓ (주간)
09:05 + wait(08:00) = 0초, late=true ✓ (지연 시 즉시)
```

---

## 4. 인프라 체크리스트

```bash
# 브릿지
bash EXE/run_daily_ops_bridge.sh restart   # :8796
bash EXE/run_social_post_bridge.sh restart # :8795

# launchd (instagram 5분 drain + 07:40 bridges)
bash EXE/install_mac_failover.sh install

# n8n VM 반영
bash EXE/sync_n8n.sh

# 수동 IG drain
bash EXE/run_instagram_queue_drain.sh

# 큐/상태 확인
curl -s http://127.0.0.1:8796/v1/instagram-queue | python3 -m json.tool
curl -s http://127.0.0.1:8796/v1/state | python3 -m json.tool
```

---

## 5. 재발 시 빠른 진단

1. **SNS 404** → `daily_ops_bridge` 기동 여부 (`:8796/health`)
2. **IG rate limit** → `~/.instagram-sns-queue.json` 항목·`scheduled_at` 확인 → drain
3. **정시 이탈** → n8n 실행 시각·`wait_sec` in workflow output·`DAILY_PUBLISH_MAX_WAIT_SEC` (기본 7200)
4. **FB만 되고 IG 안 됨** → `defer_instagram` 누락? drain plist 등록?
5. **Slack만 실패** → 파이프라인은 `|| true` warn-only — SNS와 분리됨

---

## 6. 관련 파일

| 파일 | 역할 |
|------|------|
| `collaboration/sns_instagram_schedule.py` | IG 오프셋·큐·reserve·publish_due |
| `collaboration/social_daily_post.py` | publish_all / publish_instagram_only |
| `collaboration/daily_sns_queue.py` | 일간 SNS 큐 + defer 기본값 |
| `collaboration/daily_ops_bridge.py` | wait-publish sleep, IG/SNS API |
| `n8n/build_daily_canva_workflows.py` | daily JSON 생성 + add_ig_retry |
| `n8n/build_weekly_inform_workflows.py` | wi_* JSON, 09:00 wait |
| `EXE/run_instagram_queue_drain.sh` | 5분 주기 drain |
| `EXE/com.mcp-auto.instagram-drain.plist` | launchd |

---

*작성: 2026-06-11 16:04 KST · harness verify PASS · chat eab98e9f*
