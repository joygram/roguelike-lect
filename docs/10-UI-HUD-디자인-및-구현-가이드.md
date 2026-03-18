# UI·HUD 디자인 및 필요 UI 목록·구현

로그라이크 게임에 필요한 **UI·HUD 디자인**, **필요 UI 목록**, **가장 쉬운 구현 방법**을 정리한 문서입니다.

**사용 지식·영역** ([01 §1.1](01-요구분석.md#11-이-프로젝트에서-사용하는-개발-영역지식-공통-기준)): **UI·사용자 경험**(HUD 레이아웃, HP·점수 표시, 게임 오버·재시작 버튼, 사용자 피드백) · **데이터 설계**(표시할 수치 소스와 UI 연동) · **소프트웨어 설계**(UIManager·GameManager와의 이벤트 흐름).

**구현 = `_UI/` 폴더** ([README 세 폴더](../README.md#folder-layers)): Canvas·HUD·패널은 **`_UI/`**, 체력·재화 등은 **`_Framework/` 이벤트·세션**만 구독. **`_Content/` 프리팹을 Find로 찾아 표시하지 않습니다.**

**전체 흐름**: 루트 README **[네 축](../README.md#dev-flow-axes)** 에서 **UI 구현**이 모이는 구간(Phase 4)의 실무 본문입니다. **메뉴·재시작 버튼**은 [README `#scene-flow`](../README.md#scene-flow)대로 **`_UI/` → 기반 `SceneFlow`만** 호출합니다. 앞선 Phase에서 **코어·컨텐츠**가 돌아가야 HP·게임오버 이벤트가 의미 있습니다.

## Phase 4 — 이 문서가 담당하는 단계 목표·에셋

| | |
|--|--|
| **단계 목표** | HP 표시, 게임 오버 패널, 재시작, (선)증강 선택 UI |
| **필수 에셋** | Canvas, Canvas Scaler, HP Fill Image, 게임오버 Panel+Button |
| **대표 스크립트** | HpBarUI, GameOverAndRestart, (선)증강 UI — [25](25-참조게임-UI-프리팹-디자인-레이아웃.md) |
| **진행 뼈대** | [07 Phase 4](07-단계별-개발-가이드.md) |
| **코드 복사 부록** | [26 Phase 4](26-CSharp-코드복사-부록.md) |

<a id="ui-folder-check"></a>

## `_UI/` 폴더 체크리스트 (Phase 4·G10)

[README `#folder-depth`](../README.md#folder-depth) · [07 Phase별 세 폴더 점검](07-단계별-개발-가이드.md)과 짝입니다.

| # | 항목 | 완료 조건 |
|:--:|------|-----------|
| 1 | **HUD** | `PlayerHpChanged`(또는 팀 F-16 채널) 구독 → 바/텍스트만 갱신 |
| 2 | **재화** | `CurrencyChanged` 구독 |
| 3 | **게임 오버** | `GameOver` 구독 → 재시작은 **`SceneFlow`** 등 기반 API만 호출 |
| 4 | **증강 선택** | `GameSession`에서 3개 표시 + 선택 시 세션 반영 + `OnAugmentAdded` — [25](25-참조게임-UI-프리팹-디자인-레이아웃.md) |
| 5 | **일시정지·설정** | timeScale·볼륨 (게임 규칙 분기 없음) |

**금지**: `currentHp -= damage`, 스테이지 클리어 판정, `_Content/` 프리팹 `Find`. **허용**: `GameSession` 읽기 + 이벤트 구독 + 버튼 → 기반 씬 전환.

---

# Part 1. HUD 디자인 (화면 구성)

## 1.1 HUD가 하는 일

- **HUD(Heads-Up Display)**: 플레이 중 항상 보이는 정보 (체력, 점수, 레벨 등).
- **팝업/패널**: 특정 상황에만 뜨는 화면 (게임 오버, 일시정지, 인벤토리 등).

아래는 **가장 단순한 배치 예시**입니다. 해상도나 팀 디자인에 맞게 위치만 바꿔 쓰면 됩니다.

---

## 1.2 기본 HUD 레이아웃 (추천)

```
┌─────────────────────────────────────────────────┐
│ [HP 바 ████████░░]  Lv.3    [점수: 1200]        │  ← 상단: 체력·레벨·점수
│                                                 │
│                                                 │
│                   게임 화면                      │
│                  (맵·캐릭터)                     │
│                                                 │
│                                                 │
│                                    [일시정지]   │  ← (선택) 우하단 버튼
└─────────────────────────────────────────────────┘
```

### 배치 요약

| 영역 | 배치 요소 | 비고 |
|------|-----------|------|
| **좌상단** | HP 바, HP 숫자 | 플레이어가 가장 자주 확인 |
| **중상단 또는 좌상단 옆** | 레벨(Lv.N) | 레벨 시스템 있을 때 |
| **우상단** | 점수·코인 | 점수 시스템 있을 때 |
| **중앙** | 게임 월드(카메라 뷰) | UI가 가리지 않도록 여백 유지 |
| **우하단** | (선택) 일시정지·설정 버튼 | 클릭 영역만 넉넉히 |
| **전체 화면** | 게임 오버·타이틀 등 패널 | 필요할 때만 활성화 |

---

## 1.3 해상도·안전 영역

- **기본 해상도**: 1920×1080 또는 1280×720 등 팀에서 하나 정하기.
- **Canvas Scaler**: **Scale With Screen Size** + Reference Resolution 동일하게 + Match 0.5~1 (가로·세로 비율에 따라 UI 비율 유지).
- **안전 영역**: 중요한 버튼·텍스트는 화면 가장자리에서 안쪽으로 5~10% 정도 띄우면, 다양한 화면에서 잘리지 않음.

---

# Part 2. 필요 UI 목록

## 2.1 필수 UI (반드시 있으면 좋은 것)

| 번호 | UI 요소 | 표시 내용 | 비고 |
|:----:|---------|-----------|------|
| 1 | **HP 바(또는 HP 숫자)** | 현재 HP / 최대 HP | 플레이어 생존 직결 |
| 2 | **게임 오버 패널** | "게임 오버" 문구 + 재시작 버튼 | 플레이어 사망 시 표시 |
| 3 | **(선택) 재시작 버튼** | 클릭 시 같은 씬 다시 로드 | 게임 오버 패널 안에 포함 가능 |

HP만 숫자로 "100/100"처럼 써도 되고, **바(이미지 fillAmount)** 로 넣어도 됨.

---

## 2.2 권장 UI (있으면 플레이가 편한 것)

| 번호 | UI 요소 | 표시 내용 | 비고 |
|:----:|---------|-----------|------|
| 4 | **레벨** | "Lv.3" 등 | 레벨 성장 있을 때 |
| 5 | **경험치 바** | 현재 EXP / 다음 레벨 필요 EXP | 레벨업 진행도 |
| 6 | **점수·코인** | "1200" 등 | 점수 시스템 있을 때 |
| 7 | **일시정지 버튼** | 클릭 시 일시정지 | (선택) |
| 8 | **일시정지 패널** | "일시정지" 문구 + 재개 버튼 | (선택) |

---

## 2.3 선택 UI (여유 있을 때)

| 번호 | UI 요소 | 표시 내용 | 비고 |
|:----:|---------|-----------|------|
| 9 | 미니맵 | 던전 일부 또는 전체 구조 | 구현 난이도 높음 |
| 10 | 스킬 쿨타임 표시 | 스킬 아이콘 + 쿨 남은 시간 | 스킬 여러 개일 때 |
| 11 | 인벤토리·아이템 창 | 획득 아이템 목록 | 아이템 많을 때 |
| 12 | 타이틀 화면 | 게임명 + "시작하기" 버튼 | 메인 메뉴용 |

---

## 2.4 UI 목록 요약표 (체크용)

| 구분 | UI 요소 | 필수/선택 | 구현 순서 제안 |
|------|---------|-----------|----------------|
| HUD | HP 바(또는 HP 숫자) | 필수 | 1 |
| HUD | 레벨 텍스트 | 권장 | 2 |
| HUD | 경험치 바 | 권장 | 3 |
| HUD | 점수·코인 텍스트 | 권장 | 4 |
| 패널 | 게임 오버 (문구 + 재시작) | 필수 | 5 |
| 패널 | 일시정지 | 선택 | 6 |
| 기타 | 타이틀·메인 메뉴 | 선택 | 7 |

---

# Part 3. 구현 방법 (가장 쉬운 방안)

## 3.1 구현 방식 비교

| 방식 | 난이도 | 장점 | 단점 | 추천 |
|------|--------|------|------|:----:|
| **Unity uGUI (Canvas + Image/Text)** | ★★☆ | 에디터에서 배치, 스크립트로 값만 갱신 | 초기 설정 필요 | **가장 추천** |
| **UI 툴킷** | ★★★ | 고성능, 복잡한 UI에 유리 | 학습·마크업 필요 | 대규모 시 |

**가장 쉬운 구현**: **Canvas + Image + Text** 조합. 체력 바는 **Image (Image Type: Filled)** 로 처리하면 코드 양이 적습니다.

---

## 3.2 씬·캔버스 구성

1. **Canvas** 1개 (Render Mode: Screen Space - Overlay).
2. **Canvas Scaler**: UI Scale Mode = **Scale With Screen Size**, Reference Resolution 1920×1080, Match = 0.5.
3. HUD용 **빈 오브젝트** 하나 (예: `HUD`)를 Canvas 자식으로 두고, 그 안에 HP 바·레벨·점수 등을 배치하면 정리하기 좋습니다.
4. 게임 오버용 **패널**은 Canvas 자식으로 별도 오브젝트 (예: `GameOverPanel`). 기본은 비활성(SetActive(false)), 게임 오버 시만 SetActive(true).

---

## 3.3 요소별 구현 방법

### (1) HP 바

- **구성**: 부모에 배경용 Image(검은색 또는 어두운 색), 자식에 채우기용 Image(빨간색/초록색).
- **채우기 Image**: Image Type = **Filled**, Fill Method = **Horizontal**, Fill Origin = **Left**. Fill Amount = 0~1 (현재 HP / 최대 HP).
- **스크립트에서 갱신** (가장 쉬운 방법):

```csharp
// HpBarUI.cs — HP 바가 붙은 Image 오브젝트에 연결 (또는 별도 오브젝트에서 참조)
using UnityEngine;
using UnityEngine.UI;

public class HpBarUI : MonoBehaviour
{
    public Image fillImage;  // Filled 타입인 Image
    public void UpdateHp(float current, float max)
    {
        if (fillImage != null && max > 0)
            fillImage.fillAmount = current / max;
    }
}
```

- 플레이어 HP가 바뀔 때마다 `FindObjectOfType<HpBarUI>()?.UpdateHp(currentHp, maxHp)` 처럼 호출하면 됩니다.

### (2) HP 숫자 (대체 또는 보조)

- **Text (TextMeshPro 또는 Legacy Text)**: 내용을 `"50/100"` 형태로 갱신.
- **스크립트 예시**:

```csharp
public Text hpText;  // 또는 TMP_Text
public void SetHp(int current, int max) { hpText.text = $"{current}/{max}"; }
```

### (3) 레벨 텍스트

- **Text** 하나. 예: `"Lv.3"`.
- **스크립트**: 레벨 업 시 또는 매 프레임(필요 시) `levelText.text = "Lv." + level;`

### (4) 경험치 바

- HP 바와 동일하게 **Image Filled** 사용. Fill Amount = 현재 EXP / 다음 레벨 필요 EXP.
- 레벨업 시 구간이 바뀌므로 “현재 레벨 기준 남은 EXP / 필요 EXP”로 계산해 넣으면 됨.

### (5) 점수·코인

- **Text** 하나. 예: `"Score: 1200"`. 점수 변수 변경 시마다 `scoreText.text = "Score: " + score;`

### (6) 게임 오버 패널

- **패널**: 빈 오브젝트 + Image(반투명 검정 배경) + 자식으로 Text("게임 오버") + Button(재시작).
- **버튼 클릭 시**: `SceneManager.LoadScene(SceneManager.GetActiveScene().name);` 로 현재 씬 다시 로드.
- **표시 시점**: 플레이어 사망 시 GameManager 등에서 `gameOverPanel.SetActive(true);`

### (7) 재시작 버튼

- 게임 오버 패널 안에 Button 배치. On Click에 스크립트 메서드 연결.
- **재시작 메서드 예시**:

```csharp
public void OnRestartClicked()
{
    SceneManager.LoadScene(SceneManager.GetActiveScene().name);
}
```

---

## 3.4 UI 갱신 흐름 (가장 쉬운 패턴)

- **옵션 A**: 플레이어·게임 매니저가 **직접** UI 컴포넌트 참조를 가지고, HP/점수/레벨 변경 시 해당 UI의 `UpdateHp`, `SetScore` 등을 호출.
- **옵션 B**: **이벤트** 사용. 플레이어 HP 변경 시 이벤트 발행 → UI 스크립트가 구독해 갱신. (구현이 조금 더 많아지지만, 나중에 확장하기 좋음.)

초등학생·단순 프로젝트는 **옵션 A**만으로도 충분합니다.

---

## 3.5 구현 단계 요약

| 단계 | 할 일 |
|------|--------|
| 1 | Canvas + Canvas Scaler 설정, HUD용 빈 오브젝트 생성 |
| 2 | HP 바: Image 2개(배경 + Filled), HpBarUI 스크립트로 fillAmount 갱신 |
| 3 | (선택) 레벨·경험치·점수 Text/바 추가, 해당 값 바뀔 때 텍스트·fill 갱신 |
| 4 | 게임 오버 패널(Image+Text+Button) 만들고 기본 비활성, 사망 시 SetActive(true) |
| 5 | 재시작 버튼 On Click에 씬 리로드 연결 |

---

# Part 4. 필요 에셋·리소스 (UI)

| 용도 | 필요한 것 | 비고 |
|------|-----------|------|
| HP 바 배경 | Image 1개 (색만 있어도 됨) | Unity 기본 UI Image로 색만 지정 가능 |
| HP 바 채우기 | Image 1개 (Filled) | 색만 지정해도 됨 |
| 버튼 | Button 기본 스타일 | Unity 기본 사용 가능 |
| 폰트 | 기본 폰트 또는 프로젝트 폰트 1개 | Text에 지정 |
| (선택) 아이콘 | 스킬·아이템 아이콘 | 나중에 추가 가능 |

**가장 쉬운 시작**: 이미지 에셋 없이 **색상만 다른 Image + Text**로만 구성해도 HUD·게임 오버까지 구현 가능합니다.

---

# Part 5. 한눈에 보기

| 항목 | 내용 |
|------|------|
| **HUD 배치** | 좌상단: HP(·레벨), 우상단: 점수, 중앙: 게임 화면, 필요 시 우하단: 일시정지 |
| **필수 UI** | HP 표시(바 또는 숫자), 게임 오버 패널 + 재시작 버튼 |
| **권장 UI** | 레벨, 경험치 바, 점수, (선택) 일시정지 |
| **구현** | Canvas + Image(Filled) + Text + Button, 스크립트에서 fillAmount·text·SetActive만 갱신 |
| **갱신** | 플레이어/GameManager가 UI 컴포넌트 참조해 HP·점수·레벨 변경 시 직접 갱신 |

이 문서만 따라도 **필요한 UI·HUD 디자인**과 **목록**, **가장 쉬운 구현 방법**까지 한 번에 정리할 수 있습니다.

<a id="step-gate"></a>

## 단계 연결 · 준비 확인 (이 문서)

**연동 Phase**: [07](07-단계별-개발-가이드.md) **Phase 4**. **시작 전**: ☐ Phase 3 완료·HUD 배치 합의·[README 세 폴더](../README.md#folder-layers) — UI는 이벤트 구독만·게임오버 씬 로드 방식(07 Phase 4 표).
