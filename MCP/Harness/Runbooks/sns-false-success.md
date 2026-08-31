<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Runbooks/sns-false-success.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# SNS 거짓성공 (sns-false-success)

> 정본: `0_Config/daily_pipeline_runbook.json` id=`sns-false-success` · TRAP `T-sns-false-success`  
> 사건: 2026-07-16 — 유저 「카드뉴스 안 올라옴」 vs 로그 `Lucifer SNS ok`

## 한 줄

**카드 완료·상태 ok·로그 「SNS ok」≠ 실게시.** `sns_status=ok` + `public_url`(또는 `sns_results`) 있을 때만 「올라갔다」고 말·기록한다.

## 증상

| 보이는 것 | 실제 |
|-----------|------|
| `pipeline_ops_log` `Lucifer SNS ok` | 피드 없음 |
| `8796` `status=ok` | `sns_status` 빈값 또는 `manual-republish-meta-ok` |
| marionette/catchup skip | 재게시 안 함 |
| 저녁 catchup `no today PNG` | 로컬 Canva PNG는 있음 (GCS만 봄) |

## 근본 원인 (4갈래)

1. **Lucifer claim 거절 → ok:True** — `claim held` 등도 `sns.ok=True` 스킵으로 위장 (수정: 멱등 `sns=ok`만 스킵, 그 외 FAIL; 성공 훅 금지)
2. **`status=ok` 단독 = 완료** — `claim_publish`·`session_ok`·`all_done`이 sns 미검증 (수정: `sns_status=ok` 필수)
3. **mark-done 위조** — `sns_status=ok`를 `public_url`/`sns_results` 없이 기록 (수정: 증거 없으면 `card_ready` 강등; `manual-republish*` 거부)
4. **저녁 catchup GCS-only** — 로컬 PNG 무시 (수정: local→GCS 폴백)

## 진단 (다음에 이 순서만)

```bash
# 1) 상태 증거
curl -sf http://127.0.0.1:8796/v1/state | python3 -c "
import sys,json
st=json.load(sys.stdin).get('state') or {}
for k,v in sorted(st.items()):
  if isinstance(v,dict):
    print(k, v.get('status'), v.get('sns_status'), (v.get('public_url') or '')[:60], v.get('sns_note'))
"

# 2) 로그 vs 실게시
grep 'Lucifer SNS ok' ~/MCP-Auto/0_Config/pipeline_ops_log.jsonl | tail
# Mac 미러 sns registry 는 고아일 수 있음 — VM GCS/피드가 정본

# 3) 복구 (실게시)
cd ~/MCP-Auto
set -a; source ~/n8n-secrets.env; set +a
export DAILY_SNS_APPROVED=1 DAILY_SNS_FORCE=1 DAILY_SNS_DRY_RUN=0 SOCIAL_PUBLISH_SOURCE=pipeline
# 오전
python3 -c "import os,sys; sys.path.insert(0,'collaboration'); from daily_pipeline_runner import run_pipeline
for p in ['weather','economy','foreign_news','history_today']:
  print(p, run_pipeline(p, source='pipeline').get('ok'))"
# 저녁 (로컬 PNG 있으면)
bash EXE/run_evening_topic_sns_catchup.sh
```

## 코드 가드 (어디를 고쳤는지)

| 파일 | 가드 |
|------|------|
| `collaboration/daily_ops_bridge.py` | `claim_publish`: ok+sns_ok 만 deny · `mark_done`: ok/sns_ok 증거 필수 · `session_ok`/`all_done`: sns 필수 |
| `collaboration/daily_pipeline_runner.py` | claim deny ≠ 성공 · `_ops_mark_sns` · 성공 훅은 `results`/`public_url` 있을 때만 |
| `collaboration/evening_topic_sns_catchup.py` | GCS 없으면 로컬 Canva PNG → gsutil → publish |
| `collaboration/marionette.py` + `0_Config/marionette_ops_gates.json` | **기본 고정**: `_pipeline_done` 은 sns+증거 필수 · catchup `DAILY_SNS_FORCE=1` · 자주실패 슬롯 grace↑ (foreign 40·evening 45·netflix 45) |

## 마리오네트 (자동 복구)

```bash
# post_grace / wake / manual — 미게시·거짓성공 슬롯을 FORCE SNS catchup
bash EXE/run_marionette.sh run --trigger manual
# VM cron 재설치(슬롯 grace 변경 반영)
bash EXE/install_marionette.sh vm-cron
```

## 금지 (에이전트)

- `mark-done` 에 `sns_note=manual-republish-meta-ok` 로 상태만 올리기
- `status=ok`만 보고 「게시 완료」단정 (M-0-A)
- SNS 스킵을 `Lucifer SNS ok` / pipeline-success 로 남기기

## 관련

- `card_ops_policy.json` `publish_quality`
- D-18 key: `sns-false-success-report`
- TRAP: `T-sns-false-success`
