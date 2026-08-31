---
source_notes:
  - Harness/Rules/00-Index.md
  - Harness/Rules/RULE_TREE.md
model: nvidia/nemotron-3-ultra-550b-a55b:free
cost: "$0"
generated: 2026-07-06
pipeline: "[[../../Harness/Workflows/notebooklm-alt-rag]]"
---

# 룰트리 핵심 설계원칙 + 신규 규칙 체크리스트 (데모 합성, 무료 모델)

## 1. 룰트리 구조의 핵심 설계 원칙 (3~5개)

| # | 설계 원칙 | 근거 |
|---|-----------|------|
| 1 | **계층적 로드 트리거(Load Trigger) 기반 구성** — 파일명·번호 순서가 아닌 특정 트리거(D-ID, PR, ARP, Relay 등)에 따라 해당 .mdc만 로드되도록 설계됨 | [출처: Harness/Rules/RULE_TREE.md] |
| 2 | **4단계 티어(tier) 분리** — `tier-a(alwaysApply)`, `technique-lazy`, `canon-lazy`, `reference-lazy` 4개 디렉터리로 적용 시점·빈도·우선순위를 물리적으로 격리 | [출처: Harness/Rules/RULE_TREE.md] |
| 3 | **헌법 계층(Constitutional Hierarchy) 준수** — 룰트리는 15개 .mdc 헌법 계층(3-a 우선순위 + 로드축)으로 정의되며, git worktree·스킬트리와 엄격히 구분됨 | [출처: Harness/Rules/RULE_TREE.md] |
| 4 | **자동화 스크립트·베이스라인으로 무결성 강제** — `rule_paths.py`, `split_canon_essence_detail.py`, `rule_reloc.py`, `harness_reference_tables.py`, `rule_deletion_guard_baseline.json` 등이 이동·분할·삭제 가드를 코드 레벨에서 보장 | [출처: Harness/Rules/RULE_TREE.md] |
| 5 | **포인터-only 이동 금지·실제 `git mv` 필수** — 규칙 이동 시 심볼릭 링크나 인덱스 갱신만으로는 불충분하며, 실제 파일 이동(git mv)과 D-7-B 절차를 거쳐야 함 | [출처: Harness/Rules/RULE_TREE.md] |

## 2. 신규·이동 규칙(.mdc) 추가 시 반드시 확인해야 할 체크리스트

| 단계 | 확인 항목 | 세부 확인 내용 | 위반 시 조치 |
|------|-----------|----------------|--------------|
| A. 위치·티어 결정 | 올바른 티어 디렉터리에 배치했는가? | 로드 트리거 표에 맞춰 티어 선택 | 잘못된 티어면 즉시 `git mv`로 이동 (D-7-B) |
| B. 로드 트리거 매핑 | 해당 규칙이 어떤 트리거에 의해 로드되어야 하는지 명시했는가? | Essence/Essence_Body/Detail_Coding/Detail_Ops/Detail/Delivery_Audit 중 어느 파일이 이 규칙을 호출하는지 문서화 | 트리거 불일치 시 로드 누락·중복 발생 → 트리거 표 갱신 필수 |
| C. 헌법 계층·우선순위 | 15개 헌법 계층 내 자신의 D-ID·§절 번호를 할당받았는가? | 번호·파일 나열 순서 ≠ 로드·적용 순서임을 인지, 상위 계층과 충돌 여부 검증 | 충돌 시 상위 계층(Core K-0-C-A) 승인 후 수정 |
| D. 자동화 가드 통과 | 관련 automation 스크립트 실행 시 에러·경고 0건인가? | rule_deletion_guard_baseline 위반, orphan 배치, D 번호 순 로드 가정 위반 여부 | 실패 시 커밋 거부 → 스크립트 수정 후 재실행 |
| E. 문서·인덱스 동기화 | `00-Index.md`/`RULE_TREE.md`에 변경 사항 반영했는가? | 신규 제목·링크 추가, 이동 시 기존 링크 갱신, 금지 사항 재확인 | 문서 불일치 시 리뷰 단계에서 반려 |
| F. Git 히스토리 보존 | `git mv`로 이동했는가? | D-7-B 절차 준수 확인 | `git mv` 아닌 경우 히스토리 손실 → 되돌리고 재실행 |

> 요약: 티어·트리거·헌법 계층 3축으로 위치 결정 → 자동화 스크립트 전수 통과 → 문서·인덱스 동기화 → `git mv`로 히스토리 보존.
