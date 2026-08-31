<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Rules/ide_user_rules_communication.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# IDE User rules — 커뮤니케이션 (안내 미러)

> **정본**: `cursorrules_Core.mdc` K-6-A · `cursorrules_Detail.mdc` **D-14**.  
> **① 1-12** 문체·코드 인용만 IDE. **① 1-3~11** git·배포·verify → `cursorrules_Technique_AAOn.mdc · AAOff.mdc`. **②** B4·D-16·K-2 = MUST.

## Cursor IDE에 이미 있을 수 있는 것 (본 문서에 없음)

Cursor Settings → User rules에 **직접** 적혀 있을 수 있는 항목 (레포 미러 아님):

- git commit / PR / `gh` — **① 1-3** (`cursorrules_Technique_AAOn.mdc · AAOff.mdc`). **② B4** 턴 commit·push MUST
- 코드 인용 형식 (`startLine:endLine:filepath`) — **IDE 전용**
- 문체·비율·완결 문장 (blog-post 톤 등)

→ **아래 「활성 룰」만** 레포 정본. IDE에 위 항목이 있으면 **삭제하지 말고 유지**하고, slim + 활성 룰을 **추가**. **mcp-auto-core·cardnews-length·풀 하네스 커널 블록은 삭제** — `.cursor/rules/` + slim만.

## 삭제 대상 (IDE User rules에서 제거)

- `mcp-auto-core` 요약·우선순위 블록 → `cursorrules_Core.mdc` K-0
- `cardnews-length` 문안 규칙 → `topic-news-pipeline` skill · `card_news_settings.py`
- Core/Main/Sub **풀 복제** → `.cursor/user-rules-slim.txt` 포인터만

## 활성 룰 (대시보드·IDE 공통 미러)

> **PR(지시 해석)** = Essence **PR1** · `.cursor/rules/` 정본 — **본 절에 넣지 않음**.  
> **본 절** = Cursor **① 1-12** 커뮤니케이션만 (대화 맥락·답 순서 등).  
> 대시보드 개요 **「IDE User rules」** 섹션과 동기화.

### 대화 맥락·의도 (1-12 · 커뮤니케이션)

Cursor 영문 룰 *Reason about conversation history…* 의 **한글 정본**. **지시 범위 해석(PR)** 과 별계열.

- 모든 유저 메시지는 **대화 전체 맥락** 속에서 읽는다. 직전 턴·논의 주제를 물려 받는다.
- 예: edge case를 논의한 뒤 「이거 어떻게 돌아가?」→ **그 edge case 맥락**의 동작 설명이 자연스러운 후보. 전체 개요가 맞는지 **모호하면** 1문 확인.
- 대화 궤적에서 **암묵 목표·제약**을 파악한다. 범위가 이번 턴 문자와 어긋나면 **Essence PR1**·Rule 1에 맡긴다 (본 절이 PR을 대체하지 않음).
- 작업 중 메시지는 **기본적으로 진행 중 작업의 보정(steering)** 으로 본다. **완전히 새 방향**일 때만 작업 전환. 단, 이번 턴 **「멈춰」「답만」「찾기만」** 등 명시가 있으면 Rule 1.

### 복합 메시지 — 답 먼저 (D-14 요약)

한 메시지에 질문(특히 네/아니오·「짧게」)과 구현·점검·배포 등 작업이 같이 있으면, **도구 호출·배포·탐색 전에** 질문에 **1~2문장(또는 네/아니오)만** 먼저 쓴다.

- 유저가 별도 형식(`Q:`/`TASK:`)을 쓰지 않아도 적용
- 「답만」「작업 금지」 등 **그 턴 명시 지시**가 있으면 K-0 Rule 1이 우선

### 응답 형식 (O0~O4 · Essence §0-A-O)

라벨은 코드 옆 **짧은 요약**을 반드시 병기. **이모지+볼드 생략 금지**:

| 코드 | 채팅 표기 |
|------|-----------|
| O0 · 이해 | `💬 **O0 · 이해**` (의도 한 줄도 볼드) |
| O1 · 실행 | `✅ **O1 · 실행**` (+ **O1-운영** 3단 한 줄) |
| O2 · 나머지 | `📋 **O2 · 나머지**` |
| O3 · 사용자 | `👤 **O3 · 사용자**` |
| O4 · 세션 | `🚪 **O4 · 세션** — 진행\|경고\|마무리\|완료` |

- **O0 · 이해** (맨 앞 · 매 턴): `1.` `2.` … 인덱싱 + 항목마다 **AI 이해(볼드 한 줄)** → 본문

작업·파일 변경 턴 — 말mi **O1→O2→O4** (O3=유저 전용 신호 시):

- **O1 · 실행**: AI가 이번 턴에 **완료한** 것 bullet + O1-운영
- **O2 · 나머지**: AI가 **아직 안 한** 것 bullet
- **O3 · 사용자**: **유저가 직접** 해야 하는 것 bullet
- **O4 · 세션**: 세션 판정 (O4-C)

Talk 모드·작업 없음·「고지 생략」이면 O1~O4 생략. O0은 순수 인사·빈 메시지만 생략.

## slim 붙여넣기 블록

`bash EXE/apply_user_rules_slim.sh` → `.cursor/user-rules-slim.txt` (하네스 포인터 — D-13/D-14는 Core K-6-A 참조)

## 관련

- Q&A 아카이브: Obsidian `Harness/Rules/2026-06-07-answer-first-user-rules.md`
- 대시보드: 개요 → **IDE User rules (커뮤니케이션)**
