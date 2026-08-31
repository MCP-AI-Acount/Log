<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Backups/2026-06-13-kernel-slim-rule-gate.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# Harness 백업 — 2026-06-13 (C 커널 슬림 · Rule Gate · Devil rename)

## 세션 요약

| 항목 | 내용 |
|------|------|
| 전략 | **대안 C** — 피라미드 유지 · 중복 제거 (~400 tok/턴 절감) |
| Core K-6-A | MUST 4행 표 → `RULE_CANON_REGISTRY`·AAOn 포인터 + 교차참조 표만 |
| AAOn | O·B 중복 표 삭제 → Essence §0-A-O · 1.5-2-B/D |
| agent-tooling | Tier A → AAOn 1.5-2/3 위임 |
| Detail D-1 | `Core > agent-tooling > Main > Sub > Detail` |
| Essence | O 권장/MUST 브릿지 · A3 계층 표기 |
| token_economy | `트레이드\s*오프` → analysis kind |
| Realm | `Devil_Sentinel` → `Devil_Judgment_Sentinel` (JUDGMENT_LOG) |
| lint | `harness_sync.py` DEPRECATED_TERMS 자기참조 제외 |

## 정본 위치 (런타임)

| 계층 | 경로 |
|------|------|
| 헌법 문장 | `.cursor/rules/*.mdc` |
| IDE slim | `.cursor/user-rules-slim.txt` |
| 표·인덱스 | `automation/harness_reference_tables.py` → `RULE_CANON_REGISTRY` |
| Obsidian Canon 미러 | `Harness/Rules/Canon/*.mdc` (본 세션 스냅샷) |

## alwaysApply (2026-06-13)

| 파일 | 역할 |
|------|------|
| cursorrules_Core.mdc | K-0~K-6-A |
| agent-tooling.mdc | K-7 라우팅 |
| cursorrules_Technique_AAOn.mdc | 1.5-x MUST |

## 복구

```bash
git checkout HEAD -- .cursor/rules/ Realm.md automation/token_economy.py automation/harness_sync.py
python3 automation/harness_sync.py all
python3 automation/dashboard_sync.py --dry-run --out web-browser/data/harness_snapshot.json
```
