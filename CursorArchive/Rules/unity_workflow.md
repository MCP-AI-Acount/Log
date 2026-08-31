# 유니티 변환 워크플로우 (Unity Transform Workflow)

> **로드 시점**: Unity 프로젝트 UI 작업 발생 시 lazy 로드  
> **관할 파일**: `Assets/Editor/BrightThemeApplier.cs` · `Assets/Editor/apply_bright_theme_scene.py`  
> **적용 진입점**: Tools > Apply Bright Theme (Unity Editor 메뉴)

---

## 1. 전문

Unity 프로젝트의 UI를 코드로 일괄 변환할 때 적용하는 원칙과 절차.  
MCP/컴퓨터 유즈 없이 Editor 스크립트만으로 씬 전체를 처리하는 것을 목표로 한다.  
- Judge(CC)가 Editor 스크립트를 작성·수정하고, 실행은 유저가 Unity Editor에서 직접 트리거.  
- 시각 결과물은 유저가 스크린샷으로 피드백하면 CC가 코드 보정.

---

## 2. 공통 규칙

### A. 진입 전 체크
1. Unity Editor가 열린 상태인지 확인 (컴퓨터 유즈 대신 유저에게 구두 확인)
2. Editor 스크립트는 반드시 `Assets/Editor/` 에 위치 (런타임 빌드 오염 방지)
3. 씬 수정 후 반드시 `Cmd+S` 저장 요청
4. `FindObjectsByType<T>(FindObjectsInactive.Include, FindObjectsSortMode.None)` — 비활성 오브젝트 포함 필수

### B. 적용 범위 원칙
- **Image 컴포넌트**: 커스텀 스프라이트(아이콘)는 건드리지 않음 — `IsBuiltinUISpriteOrNull()` 로 필터
- **TextMeshProUGUI**: 특별 지시 없으면 색 변경 금지 ("글씨 색은 그냥 놔둬" 원칙)
- **TMP_Dropdown / Dropdown**: 반드시 버튼 분류보다 먼저 체크 (ButtonOK 오분류 방지)
- **Undo 등록**: 모든 `SetDirty` 직전 `Undo.RecordObject` 필수

### C. 실패 대응
- Unity Editor가 막혀있으면 Python 오프라인 패처(`apply_bright_theme_scene.py`)로 fallback
- `.unity` 파일 직접 YAML 수정 후 Unity에서 reimport 확인 필수
- "Build asset version error" 발생 시 Unity Editor에서 직접 씬 열고 재적용

---

## 3. 프로젝트별 세부 작업방식

### 3-1. Insurance_2D 프로젝트

**경로**: `/Users/Windows/Documents/Task/Unity/Insurance_2D 복사본 오전 10.22.08/`

#### 씬 구성
| 씬 | 용도 |
|----|------|
| `Assets/#Make/00_/Scenes/1_App.unity` | 메인 앱 화면 |
| `Assets/#Make/00_/Scenes/2_Call.unity` | 통화/수신 화면 |

#### 하이라키 구조 (1_App)
```
Canvas
├── Register       → 피치/코랄 계열 (Palette[0])
├── Main           → 스카이블루 계열 (Palette[1])
│   ├── Calendar
│   ├── Note
│   ├── PopUp
│   ├── Option
│   └── Customer
├── Menu           → 라벤더 계열 (Palette[2])
├── CommonPopUp    → 민트/틸 계열 (Palette[3])
└── Test           → 앰버/골드 계열 (Palette[4])
```

#### 색 배분 규칙 (BrightThemeApplier.cs)

**Zone 결정 방식**: 키워드가 아닌 **Canvas 직계 자식의 sibling index % 5**  
→ 같은 부모(패널) 아래 자식들은 무조건 같은 팔레트 계열

**팔레트 구조** (각 패널마다):
| Role | 설명 |
|------|------|
| BG | 패널 전체 배경 — 가장 밝은 파스텔 |
| Header | 상단 헤더 바 — 채도 높임 |
| Card | 카드/아이템 — 흰색 유지 |
| List | 리스트 영역 — BG보다 살짝 밝게 |
| Input | 입력창 배경 |
| **ButtonOK** | 확인/저장/등록 — **딥 한 색 (대비)** |
| **ButtonCancel** | 취소/닫기 — **뮤트 한 색** |
| **ButtonAlt** | 삭제/초기화 — **더 다크한 색** |
| Scroll | 스크롤바 트랙 |
| ScrollHandle | 스크롤바 핸들 |

**3-버튼 분류 기준**:
- `ButtonOK`: Button 컴포넌트 + 이름에 ok/confirm/save/add/register/등록/저장/확인
- `ButtonCancel`: Button + cancel/close/exit/back/취소/닫기
- `ButtonAlt`: Button + delete/remove/reset/clear/삭제/초기화

#### 버튼 모서리
- `AssetDatabase.GetBuiltinExtraResource<Sprite>("UI/Skin/Background.psd")` — Unity 내장 9-slice 둥근 스프라이트
- `Image.Type.Sliced` + `fillCenter = true` + `pixelsPerUnitMultiplier = 1f`

#### 드롭다운 규칙
- 폰트 크기: 부모 계층에서 첫 번째 TMP 글씨 크기 × 0.8 (없으면 fallback 28f)
- RectTransform: `anchorMin=(0,0)`, `anchorMax=(1,1)`, `offset=(0,0)` — 부모에 100% stretch
- **반드시 버튼 분류보다 먼저 체크** (TMP_Dropdown/Dropdown 컴포넌트 직접 확인)

#### 글씨 색
- **변경하지 않음** — ApplyTMP 호출 제거, 유저 원본 유지

#### 피해야 할 패턴
- `GetZone()` 키워드 기반 분류: 오브젝트 이름 바뀌면 깨짐 → sibling index 기반으로 전환됨
- 고정 폰트 사이즈(24f): 해상도·폰트 크기 다른 씬에서 깨짐 → 부모 상대값으로 전환됨
- `screen_default` 핑크(#FCE4EC) / 붉은 버튼(#E91E63): 파스텔 계열 위반 → 전면 교체됨
