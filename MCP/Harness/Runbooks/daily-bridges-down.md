<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Runbooks/daily-bridges-down.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# daily-bridges-down — 일간 브릿지 미기동 RCA (2026-06-12)

> 기술저장소 정본 · Obsidian `Harness/Runbooks/` 미러  
> 레포: `0_Config/daily_pipeline_runbook.json` · `EXE/ensure_daily_bridges.sh`

## 증상

- n8n 07:50 cron은 돌았으나 날씨·경제·외신 카드가 Slack/SNS에 없음
- `~/.daily-ops-state.json` 해당일 `weather/economy/foreign_news: null`
- `~/ensure-daily-bridges.log`에 `[fail] ensure_daily_bridges timeout`
- `launchctl list` → `com.mcp-auto.daily-bridges` exit **1**

## 근본 원인 (3층)

### 1. 맥 슬립 → 브릿지 프로세스 소멸

일간 브릿지(8791~8799)는 `nohup python …` 데몬. **KeepAlive 없음**.  
맥 슬립/재부팅 후 전부 DOWN. VM(`mcp-auto-worker`)은 TERMINATED면 맥 local만 담당.

### 2. 아침 preflight 실패 (`ensure_daily_bridges`)

| 문제 | 내용 |
|------|------|
| **`restart` 연쇄 kill** | `_start_if_down`이 `restart` → 진행 중 Canva/sketch 요청 끊김 |
| **8791 폴백 누락** | port dedup으로 `~/run_gemini…` 실패 시 repo 스크립트 미시도 |
| **Canva cold-start** | 8792 토큰 refresh 60~90s+ · 구 180s 타임아웃 부족 |
| **스케줄 레이스** | 슬립 해제 시 launchd 지연 실행 → **07:50 n8n과 동시** |
| **웨이크 훅 오류** | `notify_wake_to_local.sh`가 VM용 `vm_boot_services`(restart storm) 호출 |

### 3. 2차: 만평 브릿지 restart 버그

`run_foreign_news_daily_card.sh`가 sketch bridge **restart** → 외신 `incomplete slots`.  
→ `start`만 사용 + 만평 순차(1 worker) + 2차 재시도.

## 적용 fix (2026-06-12)

| 파일 | 변경 |
|------|------|
| `EXE/ensure_daily_bridges.sh` | `start` 우선 · ops/sketch/canva 선행 · timeout 300s · 8791/8792 동시 대기 · 폴백 재시도 |
| `EXE/notify_wake_to_local.sh` | `vm_boot_services` → **`ensure_daily_bridges`** |
| `EXE/com.mcp-auto.daily-bridges.plist` | **07:30 + 07:45** (n8n 07:50 전 2회) |
| `EXE/run_foreign_news_daily_card.sh` | sketch cold-start `start` only |
| `collaboration/card_sketch_engine.py` | 만평 1 worker + missing 2nd pass |

**검증:** cold-start(8791·8792 stop) 후 `ensure_daily_bridges` → **exit 0**, 전 포트 ok (~75s).

## 운영 체크리스트

```bash
bash EXE/install_mac_failover.sh install   # plist 갱신 후 1회
bash EXE/ensure_daily_bridges.sh
bash EXE/run_daily_recovery.sh             # 놓친 카드
launchctl list | grep mcp-auto
tail -30 ~/ensure-daily-bridges.log
```

## TRAP

`Realm.md` `## TRAP_REGISTRY` — **T-daily-bridges-sleep-restart**

- pattern: `879[1-8]|bridge.*fail|ensure_daily.*timeout|restart.*sketch`
- fix: `ensure start-first 300s; wake→ensure; launchd 07:30+07:45; foreign sketch start-only`
