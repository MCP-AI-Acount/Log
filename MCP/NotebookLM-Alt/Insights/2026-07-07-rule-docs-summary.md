---
date: 2026-07-07
sources:
  - Harness/Rules/00-Index.md
  - Harness/Rules/RULE_TREE.md
pipeline: notebooklm-alt-rag (OR 5단계 게이트)
source_label: "openrouter:~anthropic/claude-fable-latest:paid"
gate_note: "free tier 429(전세계 무료모델 혼잡)로 자체 curfew 발동 → CARD_OPS_MISSION=1 임시 우회로 paid 폴백 강제 성공. 상세: ntd-099."
---

# 하네스 규칙 문서 구조 요약 (00-Index + RULE_TREE 종합)

이 하네스의 규칙 문서 구조는 `.cursor/rules/` 디렉터리 아래의 "룰트리(Rule Tree)"를 중심으로 조직되어 있다. 룰트리는 15개의 `.mdc` 파일로 구성된 "헌법 계층"으로, 3-a 우선순위와 로드축을 따르며, git worktree나 `.cursor/skills/` 스킬트리와는 별개의 개념이다 [출처: Harness/Rules/RULE_TREE.md]. 물리적 배치는 4개 계층으로 나뉜다: ① `tier-a/`는 alwaysApply(Coding Harness 항시 적용)로 Core, agent-tooling, Technique_AAOn, 그리고 Claude Code Judge 레이어(K-8)인 CCLayer를 포함하고, ② `technique-lazy/`는 git/deploy/MCP/n8n 관련 HOW 지연 로드용(Technique_AAOff), ③ `canon-lazy/`는 Main/Sub와 Detail 계열(토픽 라우터인 Detail.mdc, Tier A HOW인 Detail_Coding, Tier B HOW인 Detail_Ops) 및 Essence 계열(조리 핵심/확장), ④ `reference-lazy/`는 명명·Config·Day1 관련(Value, Plan, Delivery_Audit)을 담는다 [출처: Harness/Rules/RULE_TREE.md].

로드 방식의 핵심 원칙은 "D-ID·§절은 토픽 식별자일 뿐, 번호·파일 나열 순서가 로드·적용 순서가 아니다"라는 점이다. 로드는 토픽 트리거로 이루어지는데, 예를 들어 D-10·§6·A 격자 토픽은 Essence.mdc를, ARP·F-gate·충돌·출력 관련은 Detail_Coding.mdc를, Relay·배포·Cerberus·규칙개정은 Detail_Ops.mdc를 트리거한다 [출처: Harness/Rules/RULE_TREE.md]. 충돌 시에는 3-a가 적용되며, 신규·교체·이동 시에는 Core K-0-C-A, Detail D-7-B, PV `ruleTree`를 반드시 따라야 한다. 금지 사항으로는 `.mdc` 파일의 루트 flat 배치(orphan), D 번호 순 로드 가정, 룰트리 밖 신규 `.mdc` 배치, 실제 `git mv` 없는 포인터-only 이동, 표를 User rules·skill에 복제(K-0-C)하는 것이 명시되어 있다. 운용은 `automation/rule_paths.py`, `rule_reloc.py` 등 코드 앵커와 `rule_deletion_guard_baseline.json`(LG-T2 앵커) 같은 도구로 뒷받침된다 [출처: Harness/Rules/RULE_TREE.md].

룰트리 외에, `Harness/Rules/` 폴더에는 하네스·User rules·커뮤니케이션 관련 사용자 질문과 정리 답변을 모은 Q&A 아카이브가 별도로 존재하며, 이는 대시보드의 Obsidian 아카이브 링크와 1:1로 대응된다. 현재 아카이브에는 "복합 메시지 — 답 먼저 vs User rules", "IDE User rules — 커뮤니케이션 정본", "백업 2026-06-11 — PR1 · IDE · PR 메타 운용" 세 개의 노트가 등재되어 있다 [출처: Harness/Rules/00-Index.md]. 다만 두 문서를 종합해도 룰트리와 Q&A 아카이브 사이의 우선순위 관계나 각 `.mdc` 파일의 세부 내용(K-0~K-8, D-ID 각 조항의 실제 규정 등)은 소스에 없어 확인할 수 없다 [출처: Harness/Rules/00-Index.md; Harness/Rules/RULE_TREE.md].
