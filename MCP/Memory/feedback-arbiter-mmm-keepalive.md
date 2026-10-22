---
name: feedback-arbiter-mmm-keepalive
description: Arbiter·Mmm이 죽으면 수동 nohup이 아니라 KeepAlive로 살려야 한다
type: feedback
topic: 호스트·도구·환경
hosts: [cursor, claude]
status: active
---

# Arbiter·Mmm KeepAlive

- **증상**: `:18791` Arbiter / Mmm serve·마스코트 수치·모션이 멈춤.
- **원인**: supervisor 없음. Cursor 셸 `&`/`nohup`만으로는 잡 종료·야간 킬 후 자동 복구 없음. Mmm `local.mmm.agent`가 launchd **disabled**면 SMAppService `enabled`여도 KeepAlive 무력.
- **본질 Fix**:
  - Arbiter: `~/arbiter/install_serve_launch_agent.sh install` → `com.user.arbiter-serve` KeepAlive. `retrain.restart_serve`는 launchd 있으면 `kickstart`.
  - Mmm: `Mmm --register-login` + `launchctl enable gui/$UID/local.mmm.agent` (필요 시 unix-guard도 enable).
- **금지**: Cursor 셸에서 `serve.py &`만으로 상주 가정. 복구만 하고 KeepAlive 안 심으면 같은 실패 반복(P01).
