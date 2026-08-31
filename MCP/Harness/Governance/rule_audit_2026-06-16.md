---
author: cursor
authored_by_llm: true
---

# Harness 규칙 감사 · 확정 결정 (2026-06-16)

> 정본: `.cursor/rules/*.mdc` · slim: `.cursor/user-rules-slim.txt` · 표: `RULE_CANON_REGISTRY`

## 1. 감사 요약 (이전 턴)

| 테마 | 조치 |
|------|------|
| C1+ verify ↔ D-20 | **D-20 C→R** 정본 — C3만 verify |
| AAOff alwaysApply | character 표 분리 |
| D-1~D-20 · O0~O5 · N1~N5 | cross-ref 갱신 |
| command 중복 | skill 포인터로 경량화 |

## 2. 유저 확정 (B1~B5)

### B1 — B4-GIT 3단 (commit · push)

| 단계 | 트리거 | 행동 |
|------|--------|------|
| **A** | Samsara **Nirvana** 직후 | pre-push-guard → commit → push → D-16 · **질문 금지** · B6 보고 |
| **B** | 비-Samsara · **O5 세션 종료** · 추적 변경 있음 | A와 동일 (충돌·유실 방지) |
| **C** | 중간 턴 · WIP | commit/push **전 유저 확인** |

**판정 기준 (확정)**:
- A: `run_samsara_loop.sh nirvana` 또는 동등 종료 직후
- B: O5 작성·세션 종료 의도 **AND** `git status`에 추적 변경
- C: 위 미해당 — 기본 ask

성문: Essence B4 · Detail D-16 B4-GIT · AAOn 1.5-1

### B2 — D-20 엄격

- C1/C2 중간 턴: **R1/R2만** (`all`·전체 verify 금지)
- C3: R3 verify
- `all`: **세션 마무리**(O5)·C3 R3·명시 harness-sync만

### B3 — Essence「권장」vs AAOn MUST

- **트레이드오프**: Essence lazy load 시「권장」만 보고 O 생략 위험 vs 헤더에 MUST 박으면 Essence 토큰·강도 혼재
- **결정**: 현행 유지 — 강도 분리(V10) + AAOn·훅 집행. 별도 개정 불요

### B4 — command 경량 · skill 정본

- 자연어·K-7-B 훅 = 1차 · command = IDE 별칭(비상)
- 절차 정본 = `.cursor/skills/*/SKILL.md`
- 경량화: harness-sync · harness-verify · feature-clarify · topic-evening

### B5 — 헌법 개정 · 천사/악마

- **D-7 정본**: Angel 초안 → Devil 감리 → N1~N5 → 유저 승인
- **C3 Write 직후**: `harness_sentinel` Devil 1-call (기존)
- **대규모·아키 2+**: `/debate` 3모델 (선택·권장)
- **평가**: 소규모 cross-ref 패치 = sentinel 1-call 충분. 구조 개정 = debate 추가. 유저「괜찮음」= D-7 유지 + sentinel tiered

## 3. 미결·운영

- Cursor Settings git User rules vs B4-GIT: slim·② 우선 — IDE 전역 블록 1줄 보강은 선택
- constitution sentinel 휴리스틱 FAIL: verify PASS면 의도적 정합 패치로 무시 가능

---
tags: [harness, governance, rule-audit]
source: cursor session 2026-06-16
