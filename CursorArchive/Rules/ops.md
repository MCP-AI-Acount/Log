# 층 체계: D.(Detail/절차층 = D·NTD 운영) · M.(Manual/행동층 = M-4 등)

# cc_rules/ops.md — 운영: NTD·블랙페이퍼·관습헌법·Bridge·루프

> 로드 시점: NTD 작업, 오류 등록, cc_bridge 위임, 루프 제어, 세션 관리.

---

## NTD 운영 (`cc_ntd.json`)

**등록 단위 원칙 (1 NTD = 1 독립 작업)**:
- 별도 수행·검증·완료 판정 가능 작업 = 반드시 별도 ID
- detail 내 (1)~(N) 나열 = 분리 의무 회피 판정 → 등록 시점 STOP 후 분리
- 필수 필드: `id` / `title` / `category` / `owner` / `acceptance`

**티어 필드**:
- `tier: 1` — 맥락 불필요·단일 실행으로 완료
- `tier: 2` — AI 답변·대화 필수인 새 주제 (1턴 이상 대화 = 일반 작업, tier 2로 등록)
- `tier: 3` — 장기 플랜·메모 **(LT)**

**LT (tier: 3) 등록 제약**:
- 유저가 명시적으로 "장기"·"LT"·"나중에" 등 장기 성격을 언급한 경우 → 유저 승인 후 등록
- CC가 장기 판단 시 → 반드시 유저 승인 취득 후 등록. 임의 LT 등록 금지
- 1턴 이상 대화가 필요한 작업이더라도 LT가 아니면 tier 2 일반 작업으로 등록

**LT (tier: 3) 착수 제약**:
- 세션 시작 시 자동 착수 금지 — tier 1·2 pending이 존재하면 LT는 대기
- 착수 가능 조건: ① 유저가 직접 LT 작업을 언급·요청 OR ② tier 1·2 NTD 전부 소진(할 일 없음) 후 CC가 "LT [id] 진행할까요?" 제안 → 유저 승인
- 유저 승인 없이 LT 착수 절대 금지

**배치 처리**: 동일 카테고리·파일·변경 레이어 항목은 묶어서 한 턴에 처리. O1에 `묶음 처리: [id 목록]` 명시.

**NTD 프로세스 발동 기준**:
- 유저가 "ntd" 직접 언급 시 → 즉시 발동
- 세션 첫 메시지의 명령 구체성이 낮을 때 → NTD 확인 후 pending 목록 제시
  - 판단 기준: "이 메시지가 무엇을 해야 하는지 명확히 지시하는가?" NO → NTD 발동
  - 구체적 명령 O (발동 안 함): "ntd 규칙 읽고 f4 수정하자", "파일 X 고쳐줘"
  - 구체적 명령 X (발동): "다음 작업?", "뭐 해?", "규칙 얘기 좀 하자", 짧은 인사 등

**세션 내 실행 전략 (유저 "남은 작업 하자" 발화 시)**:
1. **의존도 기반 묶음**: 서로 의존하거나 같은 레이어에 속하는 작업을 하나의 묶음으로 편성
2. **컨텍스트 판단**: 묶음 전체가 현 세션 컨텍스트 내에서 완료 가능하면 그 자리에서 진행
3. **초과 예상 묶음**: 컨텍스트 초과가 예상되면 → 유저에게 언급 후 해당 묶음은 이번 세션에서 착수하지 않음
   - 새 세션을 열어 해당 묶음만 집중 처리
4. **병렬 위임 옵션**: 의존도 높은 묶음이 컨텍스트 초과 예상 시 → Cursor에게 병렬 분담 위임, CC가 의존도 순서대로 결과 수신·조립해 완료 처리
5. **LT는 위 전략 대상 아님** — LT 착수 제약 별도 적용

**하위 항목 분화**: 미완결·세션 종료·추가 분화 시 기존 항목 수정 금지 — 하위 항목(ntd-028-1) append. `parent` 필드에 원본 id 기재.

**세션 운영 사이클**:
- 시작: `pending` 확인 → 이번 세션 우선순위 설정
- 작업 중 이월: `deferred`에 추가
- 종료: `in_progress` → `pending` 또는 완료 처리

---

## 블랙/화이트페이퍼 운영

**블랙페이퍼** (`cc_error_log.jsonl`) — 동일 오류 재발 시:
```jsonl
{"ts":"ISO8601","error_key":"string","desc":"string","attempt":1,"scope":"L1~L5","consecutive_fail_count":0,"error_class":"expected|unexpected|unknown"}
```
- `attempt`: L1→L5 scope 확장 (D-18). 동일 error_key 이전 시도 확인 후 다음 시도 — 조회 없이 새 시도 금지.
- `consecutive_fail_count`: D-24 Purgatory-lite 카운터. 동일 접근 연속 실패 횟수. 탈출(PASS) 시 0 리셋.
- `error_class`: 훅 자동 등록분은 `"unknown"` 기본값. CC self-report 시 등록 즉시 분류 필수.
  - `"expected"` — 알려진 구조적 한계·deferred NTD 기인. 해결은 structural fix(NTD) 우선.
  - `"unexpected"` — 적용 가능한 규칙이 있었음에도 위반. 해결은 L-scope 확장 + 원인 분석 우선.
  - `"unknown"` — 분류 근거 불충분. 화이트페이퍼 closed 전 재분류 시도 필수.

**화이트페이퍼** (`cc_resolution_log.jsonl`) — 블랙페이퍼 등록 즉시부터 단계별 append:
```jsonl
{"ts":"ISO8601","error_key":"string","status":"open|partial|closed","resolution":"string","resolution_class":"structural|behavioral|unknown","ref_error_class":"expected|unexpected|unknown","verified":false}
```

**status 흐름**:
- `"open"` — 블랙페이퍼 등록과 동시에 훅이 자동 생성. `resolution: "등록됨, 조치 미정"`, `resolution_class: "unknown"`
- `"partial"` — 동일 세션 내 행동 교정 완료 시 CC가 append. `resolution_class: "behavioral"`
- `"closed"` — 훅·NTD 구현 등 구조 수정 완료 + 재발 없음 확인 시 append. `verified: true`

**업데이트 규칙**:
- 상태 변경 = 덮어쓰기 금지. 신규 행 append, 동일 `error_key` 최신 행이 현재 상태.
- 세션 종료 전: 미처리 `open` 항목에 최소 `partial` + 조치 사유 기재 후 마감.
- `expected` 오류를 `behavioral`만으로 `closed` 전환 시 → `note: "structural fix 미완, 재발 위험"` 필수.

**open 장기 방치 에스컬레이션 (신규)**:
- 세션 시작 훅(`cc-session-start-check.sh`)이 `cc_resolution_log.jsonl`에서 `status: "open"` 항목의 `ts` 와 현재 시각을 비교.
- **3일 이상** `open` 유지된 항목 존재 시 → O3에 `[화이트페이퍼 경고] open {count}건 {N}일 초과 — structural fix 또는 closed 판정 필요` 자동 삽입.
- 세션 내 처리 없이 종료 시 → `cc_ntd.json`에 `category: "debt"` 항목 자동 등록 (id: `ntd-wp-{error_key}`).

**D-26 자동 수정장치 참조**:
- 블랙페이퍼 훅이 `open` 화이트페이퍼 동시 생성 (→ `cc-error-blackpaper.sh`).
- Stop 훅이 세션 종료 시 `open` 잔여 항목 수 카운트 → 1개 이상이면 경고 + `o_format_state.json pendingWhitepaperCount` 세팅.

**바벨 과오 원장** (`0_Config/babel_fault_ledger.jsonl`) — 유저 교정·거부 신호 감지 시 직접 기록(`+5`). 오류 재발(D-18) 시 `+1`.

**자기교정 로그** (`cc_self_correction_log.jsonl`) — 유저 교정 전 내가 먼저 발견·선언할 때:
```jsonl
{"ts":"ISO8601","xenia_type":"A-T1|A-T2|A-T3|A-T4","stated":"이전 발언","actual":"실제 사실","correction":"수정 내용","points":1,"confirmed":false}
```
- **A-T1** 도구 역충돌: 내 발언 ↔ Read/Bash 결과 모순 → 즉시 `points:1`
- **A-T2** 전제 붕괴: 내 가정이 후속 확인에서 틀렸을 때 → 즉시 `points:1`
- **A-T3** 완료 후 의도 불일치: Write/실행 결과가 의도와 다를 때 → 즉시 `points:1`
- **A-T4** 내부 모순 감지: 세션 내 발언 충돌 → `confirmed:false` 기록 → 유저 확인 후 `confirmed:true` + `points:3`. 미확인 시 `points:0`.

선언 형식 (응답 첫 줄): `[제니아-A / T{n}] 이전에 "X"라고 했으나 실제는 "Y". 원인: ___. 수정:`

> **소비 경로 한계 (N5)**: `cc_self_correction_log.jsonl`은 기록 MUST이나 이 로그를 자동 감지·차단하는 집행 경로 없음. AI 자기집행에 의존 — 외부 소비자(refresh_cadence 집계·훅 감지) 미구현 상태. 향후 개선 대상(ntd-052 deferred).

---

## 관습 헌법 (`cc_customary.json`)

유저가 5회 이상 같은 방향을 명시하면 `registry`에 등재.  
등재분은 K-2 위반 override 가능 (effective_count ≥ 5, 3 이하 제외).

---

## cc_bridge · cc_relay 운영

**왕복 1명령 (권장):**
```bash
bash EXE/cc_judge.sh run "작업 내용" --persona executor --timeout 600
bash EXE/cc_judge.sh classify "작업"          # route + ambiguity JSON만
bash EXE/cc_judge.sh run "…" --auto-cloud     # Cloud Agent 승인 생략
bash EXE/cc_judge.sh run --plan 0_Config/delegation_inbox/plan.json
```

모호 시 exit 2 + `0_Config/cc_relay_state.json` `pendingUserQuestion` — 유저 답 후 재실행.

**수동 2단계 (디버그):**
```bash
bash EXE/cc_bridge.sh push "작업" --mode execute --persona executor
bash EXE/cc_bridge.sh open-session
bash EXE/cc_judge.sh watch --persona executor
bash EXE/cc_bridge.sh pull
bash EXE/cc_bridge.sh status
bash EXE/cc_bridge.sh reset
```

**완료 신호:** Cursor 응답 말미 `[CC-BRIDGE-FINAL]` → `outbox_{persona}.json` status=`completed`

> ⚠️ 구버전 호환: Cursor가 `ready`만 쓰는 경우도 있음. outbox task_id가 현재 태스크와 일치하고 status가 `completed` **또는** `ready`(단, task_id 일치 시)이면 완료로 간주.

**위임 직후 수신 확인 + 완료 대기 (MUST):**

`cc_bridge push` 실행 후 **같은 턴 안에** 아래를 순서대로 수행한다.

```python
# 1. 수신 확인 (push 직후 — inbox marker 생성 여부)
#    cc_bridge.sh가 "[CC-Bridge] ✅ 수신 확인" 출력하면 OK
#    없으면 "수신 실패" 즉시 유저 보고

# 2. Monitor로 outbox 완료 대기 (자동)
python3 - <<'EOF'
import json, time
OUTBOX = "0_Config/cc_bridge/outbox_{persona}.json"
TARGET  = "{task_id}"   # push 반환값
prev = ""
while True:
    try:
        d = json.load(open(OUTBOX))
        cur = d.get("task_id","") + "|" + d.get("status","")
        if cur != prev:
            print(cur, flush=True)
            prev = cur
        if d.get("task_id") == TARGET and d.get("status") in ("completed","failed","ready"):
            break
    except:
        pass
    time.sleep(5)
EOF
```

Monitor 도구를 사용할 때는 위 python3 스크립트를 command로 전달하고  
`timeout_ms=1800000, persistent=False`로 설정한다.

**완료 알림 수신 후 즉시:**
1. outbox result 읽기 → 유저에게 결과 요약 보고
2. Unity/앱 등 확인이 필요한 작업이면 → 화면 Reload 후 스크린샷 확인
3. 다음 위임 태스크가 있으면 → 동일 절차로 연속 위임

**pull 후 검수 (MUST):** outbox 수신 직후 Cursor 출력에서 아래 확인:
1. **k2.10 A형**: 발견·판단변경이 있는데 `[제니아-A]` 없이 넘어갔는가?
2. **k2.10 B형**: 유저 전제 오류가 보이는데 `[제니아-B]` 없이 수행했는가?
3. 위반 감지 시 → 유저에게 "Cursor 제니아 위반 — [내용]" 보고 후 중계.

---

## CC 세션 루프 직접 제어

> Cursor stop-hook 미가동(CC 단독 세션) 시만 Judge가 직접 제어. hook 가동 중이면 hook 우선.

| 루프 | 시작 | 종료 | 상태 |
|------|------|------|------|
| **시지프스** | `bash EXE/run_sisyphus_loop.sh arm --s1-from-ntd` | `bash EXE/run_sisyphus_loop.sh disarm --report` | `bash EXE/run_sisyphus_loop.sh status` |
| **켈베로스** | Write 직후: `python3 collaboration/cerberus.py post-edit --file X --change-class C1` | `all_pass` 자동 | `python3 collaboration/cerberus.py status` |
| **삼사라** | 아래 §삼사라 실행 프로토콜 참조 | PRD PASS → `bash EXE/run_samsara_loop.sh nirvana` | `bash EXE/run_samsara_loop.sh status` |

### 삼사라 실행 프로토콜

삼사라 시작 시 아래 두 명령을 **항상 함께** 실행한다. (순서 고정)

```bash
# 1. 삼사라 루프 시작
bash EXE/run_samsara_loop.sh start --task "작업 설명" --kind feature

# 2. Slack 진행상황 watcher 백그라운드 시작 (동시)
bash EXE/run_samsara_slack_notify.sh watch \
  --project "프로젝트명" \
  --task "작업 설명" &
```

**`--project` 네이밍 규칙**: 프로젝트 단위로 고정 (`Unity` / `스케쥴러` / `어플` 등).  
같은 프로젝트의 작업은 동일 Slack 스레드에 누적된다.

**Slack 메시지 타임라인** (watcher 자동 처리):

| 시점 | 메시지 |
|------|--------|
| arm 직후 | `📋 [프로젝트] 작업명` — 프로젝트 스레드 생성 (없으면 신규) |
| 즉시 | `🔍 [CC Watcher] {task} 대기 중` |
| 30초마다 | `⏳ 대기 중... (N초 경과)` |
| Cursor 시작 감지 | `🔄 Cursor 작업 시작됨` |
| 완료 | `✅ Cursor executor 완료` + 결과 요약 + macOS 알림 |
| 실패·오류 | `❌ 실패` + 오류 내용 |

**수동 명령:**

```bash
bash EXE/run_samsara_slack_notify.sh notify --project "Unity" --msg "메시지"
bash EXE/run_samsara_slack_notify.sh list          # 활성 스레드 목록
bash EXE/run_samsara_slack_notify.sh kill --project "Unity"
```

**발동 기준**:

| 상황 | 행동 |
|------|------|
| 세션 시작 + S1 미션 있음 | 시지프스 arm |
| 코드 Write 완료 (기능·수정) | 켈베로스 post-edit (C0/C1/C2+ 등급 명시) |
| 삼사라 Nirvana 후 큐 항목 있음 | **O2 확인 필수** — 자동 시작 금지 (T-V5) |
| **프로젝트·대규모 수정 착수** | 삼사라 자동 arm (→ 아래 §삼사라 자동 발동 기준) |

### 삼사라 자동 발동 기준 (Auto-arm)

유저 선언 없이도 아래 **모두** 충족 시 Judge가 즉시 `run_samsara_loop.sh start` 실행:

| 조건 | 판정 기준 |
|------|---------|
| NTD에 `feature` 또는 `improve` kind 항목 존재 | `cc_ntd.json` pending 확인 |
| 변경 대상 파일 3개 이상 OR 신규 모듈·디렉토리 생성 | Write 계획 확정 시점 |
| 기존 삼사라 미션 활성 없음 | `run_samsara_loop.sh status` → inactive |

> **단, 유저가 "삼사라 없이"·"직접 실행"을 명시하면 arm 금지.**  
> 발동 시 `[삼사라 auto-arm] task: ... kind: ...` 1줄 보고 필수.

> **집행 경로 현황 (N5)**: cc-session-start-check.sh가 NTD category=infra/feature + pending≥3 조합 시 auto-arm 경고 주입. run_samsara_loop.sh status 확인(inactive)은 AI 자기집행. 완전 자동화 미구현.

---

## 새 세션(Agent) 사용 기준

| 상황 | 처리 |
|------|------|
| 독립적 파일 생성·탐색 (컨텍스트 불필요) | Agent 위임 |
| 대형 코드베이스 탐색 | Agent Explore |
| C2+ 토론 (Angel/Devil 물리 분리) | cc_bridge Cursor 세션 |
| 컨텍스트 압축 임계 초과 | NTD 저장 → 새 세션 |
| 판단·통합 필요 | 직접 처리 |

---

## 컨텍스트 임계 대응

컨텍스트가 길어져 압축이 심해지면:
1. `cc_ntd.json`에 현재 상태·pending 전부 기록
2. Cursor에 판단 위임 (`cc_bridge push --mode review`)
3. 결과 수신 후 새 세션에서 NTD 기반으로 재개

---

## D.D-21 CC. 갱신체계 (Refresh Cadence)

> Cursor `cursorrules_Detail_Ops.mdc §D-21`의 CC 이식판.  
> Cursor 전용 요소(Realm.md 미러·harness_sync) 제거 — 스크립트 직접 실행으로 대체.

**스크립트:**
```bash
python3 automation/refresh_cadence.py panel   # 현재 retention 현황 확인
python3 automation/refresh_cadence.py prune   # 만료 로그 정리
bash EXE/run_or_weekly_router.sh              # 주간 앵커 (일요일 0시 KST)
```

**발동 기준 (MUST):**

| 시점 | 행동 |
|------|------|
| **신규 로그·문서 생성 직후** | `refresh_cadence.py panel`로 해당 항목 retention 등록 확인 — D-21 `retention 동시 등록` |
| **retention 적당한 규칙 없음** | 유저에게 보존 기간 1회 질문 후 등록. AI 추측 retention 금지 (M-0-A) |
| **prune/truncate 자동화 변경** | 적용 전·후 유저에게 알림. 침묵 자동 삭제 금지 |
| **주간 앵커 (일요일)** | `run_or_weekly_router.sh` — `refresh_cadence.py prune` 포함 |

**CC vs Cursor 차이:**

| Cursor | CC |
|--------|----|
| `Realm.md ## CREATE_ONLY_ARTIFACT_ROWS` 미러 | 없음 — `refresh_cadence.py panel`로 직접 확인 |
| harness_sync 연동 | 없음 |
| 주간 OR 라우터 자동 트리거 | 수동 또는 cron 별도 설정 |

---

## 평행우주 (Parallel Universe) CC 운영

> Cursor `.cursor/skills/parallel-universe/SKILL.md`의 CC 이식판.  
> Cursor 페르소나(`평행우주` 선언)·태초의 목소리 분리 개념 제거 — Bash 직접 실행.

**실행:**
```bash
bash EXE/run_parallel_universe_scan.sh scan              # GitHub 레포 스캔 + D-10 선별 (네트워크)
bash EXE/run_parallel_universe_scan.sh report            # 현재 state 확인용 (네트워크 없음)
bash EXE/run_parallel_universe_scan.sh weekly wake       # 주간 게이트 (이미 실행 시 skip)
python automation/parallel_universe_integrate.py apply   # 스킬·철학 로컬 병합
python automation/parallel_universe_integrate.py archive-rules  # 헌법만 Obsidian
```
> `scan --dry-run` = **네트워크 호출 없음** (state 요약만 출력). 실제 스캔 확인은 `report`.

**발동 기준:**

| 시점 | 행동 |
|------|------|
| **주간 앵커 (일요일)** | `run_or_weekly_router.sh` — `weekly wake` 내장 (skip gate 자동) |
| 외부 harness 레포 스킬 수집 필요 시 | `scan` → D-10 선별 → 유저 승인 → `apply` |
| 통합 후 | `refresh_cadence.py panel` 연동 |

**제약 (MUST):**
- `.mdc` 자동 Write **금지** — norm-refine + 유저 confirm 후 Write
- 외부 패턴이 로컬 skill과 유사 → **로컬 편입·확장** (병렬 skill 복제 금지)
- `0_Config/parallel_universe_integration/평행우주_미적합.md` — 통합 전 반드시 대조

**정본 산출물:**
- `0_Config/parallel_universe_integration/평행우주_통합매니페스트.json`
- `0_Config/parallel_universe_integration/평행우주_미적합.md`

**CC vs Cursor 차이:**

| Cursor | CC |
|--------|----|
| `[Parallel_Universe / PU]` 페르소나 선언 | 없음 — Bash 직접 실행 |
| OpenRouter LLM 스킬 추출 (Cursor 선, OR 백업) | OR 직접 호출 또는 Cursor 위임 |
| Pipeline D 자동 랭킹 | 스크립트 동일 사용 |

---

## 우주상수 (Cosmic Constants) CC 운영

> `automation/layer0_catalogs.py COSMIC_CONSTANTS_CATALOG`의 CC 접근 경로.  
> Python dict는 세션 중 직접 import 불가 — JSON 스냅샷으로 참조.

**스냅샷:**
```bash
python3 automation/layer0_catalogs.py export-snapshot
# → 0_Config/cc_cosmic_constants_snapshot.json 갱신
```
> 주간 라우터(`run_or_weekly_router.sh`)가 자동 갱신. 별도 수동 실행 불필요.

**CC에서 참조 시점:**

| 시점 | 참조 항목 |
|------|----------|
| 표본 크기·비율 결정 | `numericTiers.tier1.prime` — 프라임 사다리 (3,5,7,11…) |
| 엔트로피 임계 · Angel Interview | `numericTiers.tier1.naturalConstant` — e, φ |
| 토큰 경제 상한 계산 | `economyDynamicCeiling` — `automation/token_economy.py` 위임 |
| Power of Two 배치·배치 | `numericTiers.tier1.powerOfTwo` |

**정본:** `automation/layer0_catalogs.py COSMIC_CONSTANTS_CATALOG`  
**CC 접근 경로:** `0_Config/cc_cosmic_constants_snapshot.json`  
**수치 정의 복제 금지** — skill·command에서 숫자 재정의 시 PV_MISPLACED_CONCEPT


---

## 상태 파일 생명주기 (바벨 방지)

> 근거: ntd-LT-019 D-10 선택 A (33.1점). 바벨 패턴(무통제 적층) 방지를 위한 분기 감사 절차.

**감사 스크립트:**
```bash
python3 automation/state_file_audit.py           # dry-run: 후보 목록 출력 + 저장
python3 automation/state_file_audit.py --verbose  # 전체 파일별 판정 상세 출력
python3 automation/state_file_audit.py --execute  # 유저 확인 후 실제 rm 실행
```

**출력물:** `0_Config/audit_dead_candidates.json` — 사망선고 후보 목록 (덮어쓰기)

**발동 기준 (MUST):**

| 시점 | 행동 |
|------|------|
| **분기(3개월)마다 1회** | `state_file_audit.py` dry-run 실행 → 후보 목록 유저에게 보고 |
| **사망선고 후보 확인 후** | 유저 명시 승인 → `--execute` 실행. 승인 없이 자동 삭제 절대 금지 |
| **신규 .json 파일 생성 시** | `refresh_cadence.py panel`로 retention 등록 확인 (D-21 연동) |

**사망선고 기준 (AND 조건):**
1. `git log --all --diff-filter=AM -- <파일>` 결과 90일 이상 수정 없음
2. 코드베이스(`.py` / `.sh` / `.md` / `.json` / `.mdc`) 어디에서도 파일명 참조 없음

**보호 파일 (삭제 금지):**
- `cc_customary.json` — 관습헌법 원본
- `cc_ntd.json` — NTD 운영 원본
- `artifact_registry.json` — named artifact 레지스트리
- `cc_rules/` 디렉토리 전체

**제약 (MUST):**
- 스크립트 자동 rm 금지 — 유저 확인 단계(`--execute`) 반드시 거쳐야 함
- 후보 목록만으로 삭제 간주 금지 — dry-run 출력 ≠ 삭제 완료
- 삭제 후 git commit에 `[바벨방지] state_file_audit rm N개` 메시지 포함

> **집행 경로 현황**: `run_or_weekly_router.sh`가 매주 일요일 `state_file_audit.py --quarterly-check` 자동 호출 — 분기 미경과 시 skip, 경과 시 감사 시작. **분기 발동 자동화 구현 완료.**
> 추가 포인터: `bash EXE/run_or_weekly_router.sh` — 분기 체크 내장.

---

## §집행-이론-한계 — M-0·M-0-B Hook 불가 이유 (LT-018)

> 근거: ntd-LT-018. M-0·M-0-B는 **행동 결과** 및 **사고 시작점**을 제어하는 원칙으로, 구조적으로 Hook 집행 불가.

### 왜 Hook으로 집행할 수 없는가

| 원칙 | Hook 불가 이유 |
|------|--------------|
| **M-0** (쉬운 경로의 역설) | 행동 직전 1초 판단 — "이 선택이 나중에 더 많은 일을 만드는가?" 판단은 **맥락 전체**를 참조해야 함. 정규식·패턴 매칭 불가. LLM judge 투입 시 매 Write마다 호출 = 비용·지연 비율 역전 |
| **M-0-B** (원인 우선 진단) | 사고 시작점 — "빠른 경로가 먼저 떠오르면 재설정" 지시는 **추론 과정 내부**에 위치. 출력 패턴으로 감지 불가. 생성 완료 후 Stop 훅에서 regex를 써도 증상(결과)만 잡을 수 있고 원인(사고 시작점)은 이미 지나간 상태 |

### 대체 집행 경로 (현재 작동 중)

| 경로 | 작동 방식 |
|------|----------|
| **ARP (cc-pre-arp.sh)** | 모든 Write/Edit/Bash 직전 M-0-A·k2 리마인더 주입 → 사고 방향 교정 |
| **D-18 재발 감지 (cc-post-tool-error.sh)** | 동일 error_key 재발 시 L1→L5 scope 확장 → 누적 오류로 M-0 위반 사후 추적 |
| **바벨 Fault Ledger** | `babel_fault_ledger.jsonl` 누적 → 만성 과오 패턴 응축·시각화. M-0 패턴 위반 시 Realm.md 갱신 |
| **cc_self_correction_log.jsonl** | CC 자기교정 루프 기록 → 세션 간 M-0 패턴 추적 (N5 한계: 자동 감지자 미구현) |

### 집행 강화 방향 (미구현 · 착수 조건 충족 시)

1. **Stop 훅 LLM judge**: 응답 전체를 대상으로 "쉬운 경로 선택 여부" semantic 평가 → 비용 대비 효과 불확실, 오탐 고위험
2. **Refresh Cadence 통합**: 주간 라우터가 `cc_self_correction_log.jsonl` 집계 → M-0 위반 패턴 리포트. 현재 미구현(ntd별도)
3. **유저 교정 자동 D-18 등록**: M-0 관련 유저 교정 발생 시 error_key `m0-easy-path` 자동 등록 → 현재 [자기집행]

> **집행 경로 현황 (N5)**: M-0·M-0-B는 Hook 집행 불가 구조적 이유 확인됨(2026-06-29). 대체 경로 3축 운영 중. LLM judge 도입은 비용·오탐 분석 후 별도 NTD.

---

## §K6A0-훅 — 훅 완전처리 항목 (CLAUDE.md 표 별도 관리)

> CLAUDE.md K-6-A-0 표에서 분리된 항목. 훅이 완전 집행하므로 표 상시 메모리 불필요.  
> 집행 확인 필요 시 이 섹션 참조.

| 트리거 | MUST | 집행 훅 |
|--------|------|---------|
| **관습·override** | `cc_customary.json` 등재분 → K-2 위반 override | cc-session-start-check (customary 전문 주입) → `ops.md §관습헌법` |
| **A9** 범용·Scope Bleed | 미실행 auth·배포·로컬 fix paste 금지 | ①cc-pre-mdc-debate ②cc-stop-ogate ③cc-pre-arp — 3개 흡수 |
| **D-21** 갱신·로그 | 신규 로그·문서 = retention 동시 등록 · AI 추측 retention 금지 | cc-post-d21 → `ops.md §D-21 CC` |
| **설명·초보자 뉘앙스** | 내부 코드(K·D·Tier·k2) 그대로 금지 — 일상어 풀어쓰기 | cc-stop-b9-codename → `behavior.md §B9` |
| **룰트리** · `.mdc` 수정 | K-0-C-A 슬롯 배치 · flat 루트 금지 · LG-T1 체크 | cc-pre-mdc-debate (C2+에 흡수) → `rule_change.md §K-8-B` |
| **삼사라 auto-arm** | NTD feature/improve + 파일 3개+ + 미션 inactive → 자동 arm | cc-session-start-check → `ops.md §삼사라 자동 발동 기준` |
| **Trinity 가시성** · 삼사라 arm 시 | `run_samsara_loop.sh start` + `run_samsara_slack_notify.sh watch` | cc-session-start-check → `ops.md §삼사라 실행 프로토콜` |
| **D-4** Relay·persona | Judge = 유일 채널 · Angel/Devil = cc_bridge 물리 분리 | cc-stop-outbox-verify → `ops.md §cc_bridge` |
| **세션 상태** | 훅 자동 주입 — NTD·customary 별도 Read 불필요 | cc-session-init |
| **세션 저널 Write** | 첫 턴 `cc_session_journal.json` 작성 MUST — 미작성 시 cc-session-init 경고 주입. 3단 마무리 시 `rm` 의무 | cc-session-init |
| **NTD orchestrator** | pending 확인 → 명시 계속 시 O1 즉시 · defer 금지 | cc-session-start-check → `ops.md §NTD` |
| **M-4 수신 확인** | cc_bridge push 직후 `active_sessions/{persona}` 파일 확인 | cc-post-bridge-verify → `principles.md §M-4` |

---

## §저널-스키마 — cc_session_journal.json 템플릿

> 세션 시작 첫 턴에 Write. 3단 마무리 시 `rm 0_Config/cc_session_journal.json` 삭제.

```json
{
  "session": "YYYY-MM-DD-sessionN",
  "intent": "이 세션의 핵심 목표 한 줄",
  "missions": [
    { "id": "M1", "title": "미션 제목", "status": "active", "progress": "" }
  ],
  "pivots": [],
  "open": [],
  "keywords": {}
}
```

**status 값**: `active` · `done` · `pending`  
**세션 중 갱신**: missions.progress·pivots·open 업데이트  
**삭제 시점**: 3단 마무리(git push + 옵시디언 동기화) 직후 `rm`
