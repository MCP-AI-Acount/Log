# Trinity 공통 작업 디스패치 방식

> 프로젝트 무관 공통 적용 | 작성: 2026-06-28
> 정본 규칙: `0_Config/cc_rules/principles.md §M-4-B`

---

## 핵심 원칙

멀티파일 작업은 의존성 분석 후 병렬/직렬 그룹으로 나눠 Cursor 오케스트레이터에 위임.
**Judge는 발송만 하지 않는다 — 착수 확인 · 완료 감지 · 결합 · 유저 알림까지 책임진다.**

---

## Judge 오케스트레이션 의무 (M-4-B)

| 단계 | 내용 | 알림 |
|------|------|------|
| **① 착수 확인** | executor 실제 작업 시작 감지 (수신 확인과 별개) | 유저에게 착수 알림 + 터미널 링크 |
| **② 그룹 완료 감지** | GROUP-A outbox 완료 → GROUP-B 자동 발송 | 그룹 전환 알림 |
| **③ 전체 완료 + 결합** | 모든 그룹 완료 → 결과 결합 → 최종 보고 | 완료 보고 |

### 현재 유저 알림 방식 (임시 — 맥 온라인 시 터미널)
```bash
# outbox 실시간 모니터링
watch -n 3 cat 0_Config/cc_bridge/outbox_executor.json

# 또는 cc_bridge 전체 상태
bash EXE/cc_bridge.sh status --all
```
> 장기: 대시보드 통합 예정 (NTD ntd-LT-015)

---

## 디스패치 판단 흐름

```
작업 목록 확정
  │
  ▼
의존성 분석
  ├─ 전부 독립적 → 병렬 일괄 전송 (오케스트레이터 불필요)
  └─ 의존 관계 있음 → 그룹 분리
       ├─ Group A: 독립 작업 → 병렬 동시 전송
       └─ Group B: A 완료 후 → 직렬 순차 전송
            └─ Judge가 A 완료 감지 후 B 자동 발송
```

---

## 그룹 분류 기준

| 조건 | 분류 |
|------|------|
| 다른 파일 참조 없음, 출력이 다른 태스크 입력 아님 | 병렬 가능 |
| 다른 태스크가 먼저 수정한 파일을 읽어야 함 | 직렬 필요 |
| 같은 파일을 동시에 수정 | 직렬 필요 |

---

## cc_bridge 전송 포맷

### 병렬 그룹
```
executor: [PARALLEL-GROUP-A]
- Task1: <설명> | file: <경로>
- Task2: <설명> | file: <경로>
완료 후 Judge가 GROUP-B 자동 발송
```

### 직렬 그룹 (A 완료 후 Judge 발송)
```
executor: [SERIAL-GROUP-B] depends-on: GROUP-A
- Task4: <설명> | file: <경로>
- Task5: <설명> | file: <경로>
```

---

## 적용 예시 — OR Slack Layer 0 (2026-06-28)

**GROUP-A 병렬:** T1(or_router.json) + T2(trinity archive) + T3(repair_log schema)
→ Judge가 outbox 완료 감지 후 GROUP-B 발송

**GROUP-B 직렬:** T4(ask_llm 교체) → T5(A/B/C 분류 로직)

---

## 수리 감사 분리 원칙

- **수리 실행**: Cursor
- **수리 결과 감사**: Claude Judge (이해충돌 방지)
- 동일 주체 실행+채점 금지
