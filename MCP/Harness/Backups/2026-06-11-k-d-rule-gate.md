<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Backups/2026-06-11-k-d-rule-gate.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# Harness 백업 — 2026-06-11 (K vs D · Rule Gate 단일화)

## 기준 상태 (baseline)

| 항목 | 값 |
|------|-----|
| git HEAD | `cdf6c55` — Reindex Technical as tier 1.5-x |
| verify | `harness_sync.py verify` PASS (백업 시점) |
| Fate Mirror | tests/harness_core.py 스냅샷 기준 |

## 저장 상태 (이번 작업)

| 항목 | 값 |
|------|-----|
| BUILD_ID | `1.7.6-k-d-rule-gate` |
| 단어 정의 축 | AI 주체 4축 (인식·행동·출력·응답) — 정보 축 제거 |
| Rule Gate | `automation/harness_reference_tables.py` → **RULE_CANON_REGISTRY** 단일 표 게이트 |
| K vs D | **병합 안 함** — Core>K · K-6-A=D 요약만 |
| 헌법 갱신 | Core K-0-C · Value §1 · Detail D-1 |

## K vs D 요약 (정본: 대시보드 규칙 패널 `kdTaxonomy`)

- **K (Kernel)**: `cursorrules_Core.mdc` — WHAT/WHEN/MUST (우선순위·역할·기만·Tier)
- **D (Detail)**: `cursorrules_Detail.mdc` — HOW (절차·ARP·PC·배포·백업)
- **적용 순위**: K-0-C 게이트 → K-0-A/D 파이프라인 → 교차우선 1~5 → Core>…>Detail → K-6-A → D 전체

## 단일 게이트 (룰 유실 방지)

| 계층 | 정본 |
|------|------|
| 헌법 **문장** | `.cursor/rules/*.mdc` |
| 표·인덱스 | `RULE_CANON_REGISTRY` (Python) |
| 미러 | 대시보드 규칙·Value 탭 · 본 Obsidian 노트 |

**금지**: User rules·skill·Obsidian·EXE에 Rule Gate 표·§1-A 인덱스 **복제** — 포인터만.

## 복구

```bash
git checkout cdf6c55 -- .cursor/rules/ automation/harness_reference_tables.py web-browser/
python3 automation/harness_sync.py verify
bash EXE/deploy_dashboard.sh && bash EXE/sync_dashboard.sh
```
