<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `0_Config/cc_rules/RULE_TREE.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# CC 룰트리 (Claude Code Rule Tree) · `0_Config/cc_rules/`

> **CC 룰트리** = Claude Code 하네스 규칙 계층 매핑. Cursor `.mdc` 룰트리와 구별.  
> **지위 (2026-08-22 유저 지시 · 자립화)**: Claude Code = **보조 세션**(`automation/contracts/harness_platform_contract.json` `claude.role=auxiliary_review_or_subtask`). Trinity Judge live 연결은 **CG 게이트(Grok TUI)** 로 이관됐으므로(`naming_canon.json`), 본 트리는 Cursor 룰을 런타임에 읽지 않아도 **CC 세션의 0-hop 판정이 서는 자립 적응본**이다. Cursor `.mdc`는 **정의의 원산지**(출처 표기 `출처=`) — 해석이 갈리면 원문이 정의 정본이고, 원문 개정 시 본 트리도 같이 개정한다(F3).  
> **교차 문서**: `.cursor/rules/RULE_TREE.md`(Cursor 트리) · `0_Config/RULE_TREE_COMBINED.md`(두 트리 교차 · AI Write 불가 — 갱신은 `_proposals/` 초안 + 유저 적용).  
> **Write checklist**: 본 트리 수정 후 `python3 automation/norm_coherence_gate.py` · `python3 automation/check_n5.py --fail-only` · `python3 automation/rule_ref_integrity.py` PASS.

---

## 디렉터리 구조 (물리 배치)

```
0_Config/cc_rules/
├── tier-a/                — 세션 시작 항시 로드 · 위반 즉시 FAIL
│   ├── principles.md      — M-0 계열 전문(흡수) · K-0·K-0-A·K-0-B·K-0-E·K-0-G·K-0-F · Soul · K-8-D · §6 금지패턴
│   └── k2.md              — K-2 기만 금지 k2.1~k2.10 판정 기준 + §k2.5-④ 막힘 분기
├── lazy/                  — 트리거 발생 시 로드
│   ├── behavior.md        — 집행 현황 맵 · PR1·RC · A1~A11(채점 입력) · B1~B9 · D-2·D-8·D-11/14/16/17/18 · D-24·D-25·D-26·D-28·D-29
│   ├── ogate.md           — O0~O4 판정 · Orbit · O-Loop · O2-O3 분리 · O4 판정 (CC 정본)
│   ├── ogate_flow.md      — 회고 · O1-운영 · O2-S/SXN · 세션 이월
│   ├── ogate_git.md       — Gitter Claude adapter (trigger·mode mapping·rendering만)
│   ├── rule_change.md     — K-8-B · D-10(+S/B/C) · D-22·D-23 · M-R-0·M-R · F1~F5 · 바벨 · D-15
│   └── ops.md             — Fatum · 안키라 · 장부 라우팅 · Gate-prompt-author · 블랙/화이트페이퍼 · 관습 · 루프 제어 · D-21 · PU · 우주상수 · 상태 파일 생명주기 · K6A0-훅 · CC-hooks · 저널 스키마
├── _proposals/            — 초안 슬롯 (C2+ 성문·registry 등록 요청 · 멱등 적용기) — 유저 적용 전 단계
├── _audit/                — 은퇴 2026-08-21 · README만 (F1~F4 감사 리포트 · 규범 아님 · AL 정책 근거 유지). 본문=`_archive/harness_md_retired_20260821/cc_rules_audit/`
└── RULE_TREE.md           ← 이 파일
```

> `specialty-lazy/`(Slack OR · Unity)는 2026-08-21 은퇴 — 본문 `_archive/harness_md_retired_20260821/specialty-lazy/` · 런타임 Slack OR=`or_router.json`. 빈 슬롯 디렉터리는 2026-08-22 제거.

---

## 출처 맵 (흡수본 ↔ Cursor 원산지 · 정의 충돌 시 Cursor 원문 우선)

| CC 파일 · 절 | Cursor 원산지 | 흡수 형태 |
|---|---|---|
| `principles.md` §M.M-0~M-0-I · M-1~M-6 | `Manual.mdc` §M-* · `Core.mdc` §M.M-0-F/C/H/M-1/M-2 | **전문 적응**(2026-08-22) — 종전 포인터 |
| `principles.md` §C.K-0 · K-0-A · K-0-B · K-0-E | `Core.mdc` §K-0 · K-0-A(「CC 정본 이관」) · K-0-B · K-0-E | CC 적응판 |
| `principles.md` §C.K-0-G Tanist | `Core.mdc` §K-0-G | 표 ①~⑦ 흡수 |
| `principles.md` §K-8-D | `CCLayer.mdc` §K-8-D | 흡수 |
| `principles.md` §6 · §0-Aπ · §2-B · A-op · §0-A-auto | `Essence.mdc` §6·§0-Aπ·§2-B · `Essence_Body.mdc` §0-A*-op | 표 흡수 |
| `k2.md` | `Core.mdc` §K-2(「CC 정본 이관」) | 판정 기준 흡수 |
| `behavior.md` §A1~A11 · §B · §Subagent · §MCP · §일원화 | `Essence.mdc` §0-A2·§3 · `Essence_Body.mdc` §0-A·U5 · `agent-tooling.mdc` · `cursorrules_Technique_AAOff.mdc` 1.5-8 | 흡수 |
| `behavior.md` §D-2 · §D-8 · §D-11~D-18 | `Detail_Coding.mdc` · `Detail_Ops.mdc` | 흡수 |
| `behavior.md` §D-24·D-25·D-26·D-28·D-29 · §D-14-G 스텁 | — (**CC가 본문 소유** · `Detail.mdc` 라우터가 가리킴) | 발번 이원 |
| `rule_change.md` §D-10(+S/B/C) · §D-22·D-23 | `Detail_Coding.mdc` · `Value.mdc` §4-B | 흡수 |
| `rule_change.md` §K-8-B · F1~F5 · 바벨 · D-15 · M-R | `CCLayer.mdc` §K-8-B · `Detail_Ops.mdc` §D-7 · `Essence.mdc` §6 바벨 | CC 판본(바벨=CC 정본) |
| `ogate.md`·`ogate_flow.md` | `Essence_Body.mdc` §0-A-O · `cursorrules_Technique_AAOn.mdc` | **CC가 O 판정 정본** — 원문은 기본 문법 |
| `ogate_git.md` | `projects/gitter/CANON.md` | Claude trigger·mode mapping·rendering 포인터만 |
| `ops.md` | `Detail_Ops.mdc` D-18·D-21 · skill `parallel-universe` | 운영 HOW |

**CC 비흡수(Cursor 전용)**: Trinity-Core/Ops(Relay·T-V·Oath·§T-4a Solo spine) · Main §0-2~§1·§6 · Sub §4·§5·APPENDIX · Plan · Delivery_Audit · Value §0·§2·§4-B 절약 · Detail_Ops D-4*·D-7-B·D-13·D-19·D-20 · Core K-0-C·K-0-C-A·K-3·K-5·K-6·K-6-B · agent-tooling Tier A/B 표·Heimdall·K-7-A/B · AAOn/AAOff deploy 구간 · Manual M-4·M-6 모델 표면 — Git 도메인은 Gitter를 직접 로드한다.

**CC에서 은퇴(2026-08-22)**: `ops.md §cc_bridge · cc_relay 운영`(사망선고 스텁 · 앵커 보존) · M-4 위임수신 · Slack 타임라인·Unity 수동 명령 · 포인터-only 절(페르소나 코드맵·새 세션 사용 기준·컨텍스트 임계) · 빈 스텁·`.bak-midturn`.

---

## 로드 트리거

정본 = `CLAUDE.md` **lazy 로드 표** (여기 복제 금지 · N3). 세션 시작 항시 = `tier-a/principles.md` · `tier-a/k2.md`.

## Hook 집행 현황

정본 = `behavior.md §집행 현황 맵` (회귀 `tests/test_hook_fire_audit.py`·`tests/test_hook_fire_map_snapshot.py`). 본 파일은 포인터만.
