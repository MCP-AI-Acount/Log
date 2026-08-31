# "명시적(明示的)" 정의 — 원본 소재 추적 (2026-07-06)

## 배경
세션 중 spawn_task 승인 관련 트리니티 매뉴얼 §5-A 문구 작업 중, "명시적" 정의의 정확한 과거 워딩을 찾아야 했음. 1차로 Haiku 서브에이전트가 `0_Config/cc_rules/tier-a/principles.md` §C.K-0-A (커밋 `7c520fd`, 2026-07-04)에서 "명시·override 용어" 표를 찾아냄.

## 유저 지적
"니가 찾은게 7월 4일꺼면 제대로 된건 아닌거 같은데. 그걸 가져온 출처에 오히려 원본이 있을거 같아."

## 확인 결과 — 유저 지적이 맞음
principles.md 표 자체에 출처 각주가 있었음: `"(Cursor cursorrules_Detail_Ops.mdc §D-12-A 대칭)"`.

실제로 `.cursor/rules/canon-lazy/cursorrules_Detail_Ops.mdc` §D-12-A ("명시 · 인식" 계층, "단문 동의(Compact Assent)" 섹션)에 훨씬 더 상세한 원본이 존재:

- **명시**: 이번 턴 유저 메시지에 동사형 지시 또는 범위 잡힌 Y/N. AI 추론 목표·암묵 배치 아님 → Rule 1
- **암묵**: 동사형 지시 없으나 맥락상 분명 → Rule 1 아님 → 상위 원칙 충돌 시 B5
- **인식 명시**: 명시 + 현재(t) 또는 직전 유저 메시지(t-1)에 해당 rule_ref·문제와 1차·직접 관련 언급. **AI 턴은 t/t-1에 포함 안 함**
- **비인식 명시**: 명시이나 인식 명시 아님 — 그 외 전부
- **단문 동의(Compact Assent)**: 「그래 진행해」「ㅇㅇ」「ㄱㄱ」「ok」「진행」 등 — 전면 동의로 해석하지 않음. override_count는 **단문 동의만으로는 +0**. 인식 명시 여부는 변경 없음(t/t-1에 rule_ref 직접 없으면 비인식 명시)
- **override ≠ 인식 명시** 명확히 구분: override는 "확인 후 성문을 넘는 실행", 인식 명시는 "대화에 문제가 직접 나옴"

## git 근거 (원본 최초 도입 시점)
```
git log --follow -S"단문 동의" -- .cursor/rules/canon-lazy/cursorrules_Detail_Ops.mdc
→ e610fc4  2026-06-11  fix(harness): K-0 rank 4 implicit intent, P5, legacy rule purge
```
즉 원본은 **2026-06-11**, principles.md 요약본(7/4, `7c520fd`)보다 약 3주 이상 앞섬.

## 결론
- 정본(canon) = `.cursor/rules/canon-lazy/cursorrules_Detail_Ops.mdc` §D-12-A (2026-06-11~)
- `principles.md` §C.K-0-A 표(7/4)는 CC 적응 요약본 — 대칭이라 명시했지만 "단문 동의" 세부 조항(AI 턴 제외, override≠인식명시 구분 등)은 요약 과정에서 축약됨
- 이후 "명시적" 관련 판단이 필요하면 principles.md 요약이 아니라 **cursorrules_Detail_Ops.mdc §D-12-A 원본**을 먼저 확인할 것

## 처리
유저 결정: 이번 트리니티 §5-A 작업은 일단 지금까지 찾은 내용(7/4 요약본 기준, 문구는 다듬어서)으로 진행. 원본이 더 정확하다는 사실만 기록해두고 추후 참고.
