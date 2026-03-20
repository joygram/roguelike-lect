# UI 이벤트 연결 방식 비교

**원칙** ([심화 — 이벤트와 UI](심화-게임이벤트와-UI분리.md)): UI는 **`_UI/`**, **HP 깎기·사망 판정**은 **`_Framework/`**. UI는 **이벤트 구독** 또는 **팀이 정한 한 줄 API**만 호출.

---

## 이 강의에서 끝나면

- **패턴 A(이벤트)**: 기반에서 `PlayerHpChanged` 발행 → UI가 구독해 바 갱신.
- **패턴 B(직접 참조)**: 플레이어 스크립트가 `HpBarUI` 참조를 가지고 `UpdateHp` 호출 — **작은 프로토타입용**.
- **재시작**: UI 버튼 → **`SceneFlow` 또는 기반 씬 로드**만 ([README #scene-flow](00-강좌-규약-및-참고.md#scene-flow)).

---

## 1. 팀이 정할 것 (코드 치기 전)

| 항목 | 예시 |
|------|------|
| 이벤트 버스 이름 | 정적 클래스 `GameEvents` |
| HP 변경 시그니처 | `Action<int current, int max> PlayerHpChanged` |
| 게임 오버 | `Action GameOver` |
| 재시작 | `SceneFlow.LoadGame()` 같은 **기반** 메서드 |

[04 F-16](04-요구사항-및-시스템설계.md) 등 팀 표와 맞출 것.

---

## 2. 패턴 A — 이벤트로 HP → UI

### 기반 (`_Framework/`) — 발행

플레이어 HP가 바뀔 때 **한 곳**에서만:

```csharp
// 예: PlayerHealth.cs (개념)
void ApplyDamage(int amount)
{
    currentHp = Mathf.Max(0, currentHp - amount);
    GameEvents.RaisePlayerHpChanged(currentHp, maxHp);  // 팀 시그니처에 맞출 것
    if (currentHp <= 0)
        GameEvents.RaiseGameOver();
}
```

### UI (`_UI/`) — 구독

```csharp
// HpBarUI.cs — Fill Image에 연결
void OnEnable()
{
    GameEvents.PlayerHpChanged += OnHpChanged;
}
void OnDisable()
{
    GameEvents.PlayerHpChanged -= OnHpChanged;
}
void OnHpChanged(int cur, int max)
{
    if (fillImage != null && max > 0)
        fillImage.fillAmount = (float)cur / max;
}
```

| 단계 | 할 일 | 확인 |
|:----:|------|------|
| **1** | 기반에 **발행** 한 줄 넣었는지. | 피격 시 로그 또는 브레이크포인트로 이벤트 호출 확인. |
| **2** | UI 스크립트 **OnEnable/OnDisable**에서 구독 해제. | 씬 전환 시 **중복 구독** 없음. |
| **3** | Play → 피격. | HP 바 줄어듦. |

---

## 3. 패턴 B — 직접 참조 (간단 루프만)

| 단계 | 할 일 | 확인 |
|:----:|------|------|
| **1** | `HpBarUI`를 씬에 배치. | |
| **2** | 플레이어(또는 매니저)에 `public HpBarUI hpBar;` 드래그 연결. | |
| **3** | HP 바뀔 때 `hpBar.UpdateHp(current, max)`. | 동작은 하지만 씬마다 연결 관리 필요 → **이벤트 쪽이 확장에 유리**. |

---

## 4. 게임 오버 패널 — 이벤트로 켜기

| 단계 | 할 일 | 확인 |
|:----:|------|------|
| **1** | `GameOverPanelListener.cs` (이름 예시)에서 `GameOver` 구독. | |
| **2** | 콜백에서 `panel.SetActive(true)`. | 사망 시 패널 표시. |
| **3** | **재시작 버튼** OnClick → **Inspector**에서 `SceneFlow`의 public 메서드만 연결. UI 스크립트에 `using UnityEngine.SceneManagement`로 직접 LoadScene 해도 되나, **씬 목록·흐름은 기반**에 모아 두는 편이 좋음. | 버튼 누르면 같은 스테이지 또는 메뉴로 이동. |

---

## 5. 금지·허용 (다시 한 번)

| 금지 | 허용 |
|------|------|
| UI에서 `currentHp -= damage` | UI에서 `fillAmount`·`text`만 변경 |
| UI에서 스테이지 클리어 판정 | 기반에서 클리어 → 이벤트 → UI 표시 |
| `_Content` 프리팹 `Find`로 HUD 내용 채우기 | `GameSession` 읽기 + 이벤트 |

---

## 6. 통합 예시(선택)

전체가 한 덩어리로 이어진 예는 [26 Phase 4](26-CSharp-참고구현-해설.md). **위 절만으로도 구현 가능** — 26은 **원할 때** 흐름만 읽기. 복붙 아님.

**다음**: 항목별로 [36 체력바](36-UI-체력바-구현-강의.md) → [37 게임오버·재시작](37-UI-게임오버-재시작-구현-강의.md) 순으로 진행.
