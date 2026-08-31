<!-- MIRROR-GENERATED -->
> [!warning] 생성 파일 — 여기서 편집하지 마세요
> 이 문서는 `automation/obsidian_mirror/Harness/Rules/2026-06-07-answer-first-user-rules.md` 에서 자동 생성된 **사본**입니다.
> 여기 가한 수정은 다음 동기화(`obsidian_archive.regenerate_rules_mirror`)에
> 정본 내용으로 **덮어써져 사라집니다**. 고칠 곳은 위 정본 경로입니다.

# 복합 메시지 — 답 먼저 vs User rules

> 아카이브 · 2026-06-07 · MCP-Auto 하네스 운영 Q&A

## 사용자 질문 요약

1. **인터넷 끊기면 Cloud Run 대시보드는 계속 돌아가나?** → 네/아니오로 먼저 답하고, 그다음 작업하라는 의도였는데 에이전트가 작업부터 들어감.
2. **「답 먼저 + 작업」** 한 메시지에 넣으려면? 형식(`Q:`/`TASK:`)은 까먹기 쉬움.
3. **User rules 1~2문장** vs **헌법 `.mdc`** — 어디에 두는 게 맞나? 턴당 수십~100토큰은 큰가?
4. **기존 `.mdc` 형식 규칙**도 전부 낭비 아닌가?

---

## 빠른 답 (Y/N)

| 질문 | 답 |
|---|---|
| 맥/아이폰 오프라인인데 **사이트 서버**는 살아 있나? | **네** — Cloud Run은 GCP에서 호스팅. 마지막 sync 스냅샷 유지. |
| 오프라인에 **접속·sync** 되나? | **아니오** |
| 턴당 30~100토큰 규칙 추가는 **큰 비용**인가? | **아니오** — 항시 Core+agent-tooling 대비 압도적으로 작음 |
| 이 룰을 **헌법 `.mdc`**에 넣는 게 맞나? | **아니오** — Value §1 Rule Gate상 **IDE User rules**가 정본 |

---

## 추천 룰 (1문장 · 형식 강제 없음)

한 메시지에 질문(특히 네/아니오·「짧게」)과 구현·점검 등 작업이 같이 있으면, **도구 호출·배포·탐색 전에** 질문에 **1~2문장(또는 네/아니오)만** 먼저 쓴다.

**넣을 위치:** Cursor Settings → User rules (또는 `user-rules-slim.txt` 동일 1줄)

---

## 토큰 규모

| 층 | 규모 | 비고 |
|---|---|---|
| 추가 1~2문장 규칙 | ~30~100 토큰/턴 | 작음 |
| 항시 Core + agent-tooling | 수천 토큰급 | Tier A |
| Main+Sub+Detail+Value (K-5) | 더 큼 | 기획·연옥·채점 시 |
| 진짜 낭비 | Y/N 건너뛰고 도구·배포·장문부터 | 출력·작업 비용 |

---

## Rule Gate — 어디에 속하나

| 위치 | 역할 |
|---|---|
| `.cursor/rules/*.mdc` | 헌법 — 코딩·절차·Soul Contract·채점·**파일** 출력 |
| **IDE User rules** | 비헌법 — git·PR·**커뮤니케이션** + slim 포인터 |
| `user-rules-slim.txt` | User rules 붙여넣기용 |

Core **K-4:** 일반 에이전트 대화 → **Cursor User rules 커뮤니케이션** 위임.

---

## 비슷한 기존 규칙 (전수)

### 커뮤니케이션·응답 (가장 가까움)

| 규칙 | 위치 | 하는 일 | 이번 룰과 관계 |
|---|---|---|---|
| K-0 **Rule 1** (이번 턴 지시) | Core | 그 턴 명시 지시 최우선 | 「짧게 답해」**명시** 시만. 암묵 복합 메시지 갭 |
| K-4 → User rules | Core | 말투·구조 위임 | **이번 룰은 User rules** |
| Value §1 IDE User rules | Value | 커뮤니케이션 분류 | **정본 위치** |
| D-3-A 인용 vs D-3 출력 | Detail | User rules=인용, D-3=파일 납품 | **역할 분리 선례** |
| §4-B 「짧게/요약」 | Value | 유저가 **그 단어** 쓸 때 1턴 | 신호어 있을 때만 |
| §4-B 150/200줄 한도 | Value | 줄 수 절약 | 순서 아님 |
| D-10 판정 1) Yes/No | Detail | Y/N 턴 D-10 생략 | Y/N 인식만, **답 순서** 없음 |
| user-rules-slim Oracle/Temporal | slim | 세션 시작 1회 질문 | 다른 목적 |

**갭:** 「질문+작업 → **도구 전** 1~2문장」— **미정의**.

### 형식·출력 (다른 종류 — 헌법 낭비 논쟁과 별개)

| 규칙 | 위치 | 하는 일 |
|---|---|---|
| D-3 | Detail | 코드 **납품** 형식 |
| K-4 파이프라인 | Core | harness 산출물 JSON/줄글 |
| D-10 별점 표 | Detail | **아키 선택** 시만 |

→ 채팅 예의가 아니라 **코드·파이프라인·채점**. User rules에 넣을 종류 아님.

---

## 독립 1문장 넣을 때 충돌 검토

| 기존 | 충돌? |
|---|---|
| K-0 Rule 1 | 없음 |
| D-2 자율 실행 | 1~2문장 답은 지연 아님 |
| Value §4-B 절약 | **순응** (Y/N 먼저가 절약) |
| D-10 Yes/No | **보완** |

---

## 배치 선택지

| 선택 | 판단 |
|---|---|
| **IDE User rules 1문장** | **최선** |
| `user-rules-slim.txt` 동일 1줄 | 레포·apply 스크립트 동기화용 |
| Core K-6-A 1행 | 항시 보장 원할 때만 (역할상 User rules 쪽) |
| 새 `.mdc` / 헌법 장문 | **비추** — K-0-C·중복 |

---

## 관련 링크

- 레포: `.cursor/user-rules-slim.txt`
- 헌법: `cursorrules_Core.mdc` K-0·K-4, `cursorrules_Value.mdc` §1
- 대시보드: Cloud Run 하네스 대시보드 → Obsidian 아카이브
