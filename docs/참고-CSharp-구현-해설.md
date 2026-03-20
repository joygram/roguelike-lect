# 26. C# 참고 구현·설계 해설 (Phase 0~5)

## 이 문서는 «복사해서 붙여넣기»용이 아닙니다

**여기 있는 긴 C#은 “가져다 붙이라”는 뜻이 아닙니다.** 목적은 다음 세 가지입니다.

1. **통합 예시(선택)**: [08·09·10…](08-데이터테이블-및-맵구현.md)로 **이미 따라 만든 뒤**, “한 프로젝트에 모아 둔 형태가 어떻게 이어지나”만 **읽어 보고 싶을 때** 연다.  
2. **확장 연습**: 아래 **Phase별 확장 표**처럼, **한 군데만 고쳐서** 동작이 어떻게 바뀌는지 실험한다.

**막힐 때 다른 문서와 줄 단위로 맞춰 볼 필요는 없습니다.** **정본은 항상 해당 [07-Phase-N](07-Phase-0.md) + 주제 문서** ([07 지도](07-단계별-개발.md?id=impl-map)) — **완료 조건·「이 단계에서 확인」**을 다시 밟으면 된다. **16·22**는 API·용어 막힐 때. 이 파일(26)은 **위가 안 될 때가 아니라**, 통째로 읽고 싶거나 확장 실험할 **선택**이다.

---

## 이 파일을 따로 둔 이유

| 질문 | 답 |
|------|-----|
| **왜 있나?** | Phase마다 파일이 흩어져 있으면 **전체 연결**을 한눈에 보기 어렵다. 여기는 **참고용 한 덩어리 구현**이다. |
| **정본은?** | **[07-Phase-N](07-Phase-0.md)** + **08·09·10…** + [지도](07-단계별-개발.md?id=impl-map) — 목표·이론·**단계별 입력·테스트**·확장 아이디어. |
| **읽는 순서** | 주제 문서로 **구현·완료** → (원할 때만) 26에서 **흐름만** 읽거나 확장 표로 실험. |
| **26 없이 할 수 있나?** | **가능하다.** 07·08·09… 만으로 완주하는 것이 기본 루트다. |

---

## Phase별 — 여기서 본 뒤 이렇게 확장해 보라

| Phase | 참고 구현에서 볼 것 | 스스로 확장 예 (한 가지만) |
|:-----:|---------------------|---------------------------|
| **0** | 씬 전환·폴더에 붙는 스크립트 흐름 | 메뉴에 **씬 하나** 더 넣고 `SceneFlow`에 분기 추가 |
| **1** | `PlayerController2D`·카메라 분리 | `moveSpeed`를 **SO**로 빼고 Inspector에서만 조절 |
| **2** | Tilemap·SO 로드 | **스테이지 ID** 하나 더 두고 씬 전환 시 다른 맵 로드 |
| **3** | HP·데미지·이벤트 발행 | **무적 시간** 0.5초 넣어서 연속 피격 방지 |
| **4** | UI 구독·게임오버 | **HP 숫자 텍스트**를 바 옆에 추가해 같은 이벤트로 갱신 |
| **5** | Audio·빌드 연동 | **볼륨 슬라이더** 하나로 BGM만 믹스 |

확장은 **전부 붙여넣은 뒤**가 아니라, **내 코드 한 줄을 고쳐서** Play로 확인하는 식으로 진행하는 것이 목표다.

**정본이 있는 문서 (목표·에셋·단계별 녹임)**

| Phase | 목표·에셋·절차가 **본문에** 있는 문서 |
|:-----:|----------------------------------------|
| 0 | [02](02-준비사항-체크리스트.md) §4, [07 Phase 0](07-Phase-0.md) |
| 1 | [07 Phase 1](07-Phase-1.md) |
| 2 맵 | [README #map-asset-pipeline](00-강좌-규약-및-참고.md#map-asset-pipeline) → [08](08-데이터테이블-및-맵구현.md), [11](11-맵이벤트-구성-스크롤-리소스.md), [28](28-기획서-맵구성-에셋-구현-연동.md), [07 Phase 2](07-Phase-2.md) |
| 3 전투 | [09](09-이펙트-몬스터-스킬-레벨.md), [14](14-전투-확장-스킬-데이터.md), [15](15-이펙트-구성-및-스킬-설계-구현.md), [30](30-보스-AI-설계-및-구현.md), [07 Phase 3](07-Phase-3.md) |
| 4 UI | [10 UI 허브](10-UI-HUD-디자인-및-구현.md)·[33~37 강의](33-UI-에셋-제작-강의.md), [25](25-참조게임-UI-프리팹-디자인-레이아웃.md), [07 Phase 4](07-Phase-4.md) |
| 5 | [12](12-사운드-이펙트-연결방안.md), [19](19-분야별-분리-통합-및-제품출시.md), [07 Phase 5](07-Phase-5.md) |

**README 순서**: 10~15단계에서 **07 + 위 표 문서**를 본 뒤, 코드가 필요할 때만 아래 **Phase 0~5** 섹션을 펼칩니다.

코드 주석: **[Unity API]** · **[우리 구현]** · **[직렬화]**. 용어는 [16](16-Unity-API와-CSharp-구분-및-패턴.md), [22](22-용어-발음-정의-및-심화학습.md).

**팀·Git**: [27](27-데이터-코드-에셋-병렬개발-및-커밋.md). **Phase 열기 전**: 해당 **[07-Phase-N](07-Phase-0.md)** 의 **시작 전/완료 후** 표를 먼저 확인.

**폴더**: [README 세 폴더](00-강좌-규약-및-참고.md#folder-layers) — `_Framework`·`_Content`·`_UI`.

---

# Part 0. 문서 구성·분류 (전체)

| 분류 | 번호·문서 | 용도 |
|------|-----------|------|
| **범위·환경** | 01 요구분석, 02 준비사항 | 무엇을 만들지·Unity 설치·폴더 |
| **기획·데이터** | [게임기획서초안](03-게임기획서-초안.md), [29 확인 체크리스트](29-게임기획서-확인-체크리스트.md), 24 | 수치·변수명 1:1 |
| **요구·역할** | 03~06 | GDD 리뷰, F-xx, 에셋 목록, 분담 |
| **구현 본선** | **07** + **08·09·10·11·12·28·25·14·15·30** 등 | Phase별 목표·에셋·절차 |
| **C# 참고 구현·해설** | **26 (본 문서)** | 통합 예시 **읽기·확장 실험** (선택·복붙 아님) |
| **맵·전투·UI 보강** | 08, 09, 10, 11, 12, 14, 15, 25 | 타일맵·적·HUD·로그라이크 UI 등 |
| **통합·출시** | 18, 19 | 연결 점검·빌드·체크리스트 |
| **병렬·Git** | 27 | 데이터/코드/에셋 분리·커밋·머지 |
| **세 폴더 규약** | README `#folder-depth` · 07 · 04 · 10 · 02 (+06·24·28) | 기반·컨텐츠·UI |
| **맵·장애물** | README `#map-asset-pipeline` · 28 | 기획서 테마·컨베이어·유압 등 |
| **보스** | 30 | 보스 AI·페이즈·탄막 |
| **참고** | 13, 16, 17, 21~23, 22 | 직무 연결, API 구분, 기획 보강, 용어 |

---

# Part 1. 직렬화·Inspector — 왜 값이 에디터에 보이나

| 표기 | 의미 | 예시 |
|------|------|------|
| **[직렬화] `SerializeField`** | `private` 필드도 **Inspector에 노출**. 씬·프리팹 저장 시 값이 같이 저장됨. | `[SerializeField] float moveSpeed;` |
| **[직렬화] public 필드** | 기본적으로 Inspector에 보임(직렬화 대상). 팀 규칙상 민감한 참조만 `SerializeField`+private 권장. | `public Transform target;` |
| **[직렬화] ScriptableObject** | **Project 창 에셋 파일**로 저장. 여러 오브젝트가 같은 데이터를 참조. | PlayerStatsData.asset |
| **[Unity API] `CreateAssetMenu`** | 우클릭 메뉴에서 SO 에셋을 만드는 항목 추가. | `[CreateAssetMenu(...)]` |

**단계 이해**: 코드에 숫자를 박아 넣지 않고 Inspector/SO에 두면 **기획 수정 시 코드 재컴파일 없이** 숫자만 바꿀 수 있습니다.

## MonoBehaviour 호출 순서 (자주 쓰는 것만)

| [Unity API] | 호출 시점 |
|-------------|-----------|
| `Awake` | 오브젝트 생성 직후, 비활성이어도 1회. 컴포넌트 `GetComponent` 캐시에 적합. |
| `OnEnable` | 활성화될 때마다. |
| `Start` | **첫 프레임 전**, 활성 오브젝트만 1회. 다른 오브젝트 참조 초기화에 적합. |
| `Update` | 매 프레임. 입력·이동 논리. |
| `FixedUpdate` | 물리 주기(고정). `Rigidbody`에 힘을 줄 때 여기서 쓰는 편이 안정적일 수 있음. |
| `LateUpdate` | 모든 `Update` 끝난 뒤. 카메라 추적에 자주 사용. |

## 저장·불러오기 (특징 요약)

| | 용도 |
|---|------|
| **[Unity API] `PlayerPrefs`** | 볼륨·최고 점수 등 **간단한 숫자/문자열** 저장. `SetInt` / `GetInt`. |
| **[Unity API] `JsonUtility.ToJson` / `FromJson`** | 클래스 인스턴스 ↔ JSON 문자열. **필드에 `[System.Serializable]`** 필요. ScriptableObject와 달리 **런타임 객체** 직렬화에 사용. |
| **[직렬화] `[System.Serializable]`** | **JsonUtility**가 필드를 보려면 붙임. public 필드 또는 `[SerializeField] private`. |

---

# Phase 0 — 프로젝트·씬·폴더

## 에셋·에디터 단계

1. Unity Hub → **2D** 템플릿으로 프로젝트 생성.
2. `Assets` 아래 폴더 생성: `Scenes`, `Scripts`, `Prefabs`, `Data`, `Sprites`, `Audio` (팀 합의대로).
3. **File → Save As** 로 `Assets/Scenes/Game.unity` 저장.
4. Hierarchy에 빈 씬이면 **Main Camera**만 두고 저장.

## 코드

이 Phase는 필수 스크립트 없음. (버전 관리 시 `.gitignore`는 02 문서 참고.)

---

# Phase 1 — 플레이어 이동·점프·카메라

## 에셋·에디터 단계

1. **Player** 빈 GameObject 생성.
2. **Sprite Renderer** 추가 → 임시로 `Sprites`에 넣은 PNG 할당(없으면 Unity 기본 스프라이트).
3. **Rigidbody 2D** 추가: Body Type **Dynamic**, Gravity Scale **3** 정도(팀 조정).
4. **Box Collider 2D** 추가: 플레이어 몸 크기에 맞게.
5. **Layer** `Player` 생성 후 Player에 할당(선택).
6. **Ground** 레이어 생성. 바닥/플랫폼 오브젝트에 Ground 레이어 할당.
7. `Scripts/PlayerController2D.cs` 생성 후 아래 코드 붙여넣기 → Player에 컴포넌트로 추가.
8. Inspector에서 **Move Speed**, **Jump Force**, **Ground Layer**를 Ground로 지정.
9. **Jump 입력**: Edit → Project Settings → **Input Manager** → **Jump** 가 있고 Positive Button이 `space` 인지 확인(없으면 Horizontal 옆 복사해 Jump 추가).
10. 바닥용 빈 오브젝트 + **Box Collider 2D** 배치(플레이어가 서도록).
11. **Main Camera**에 `SimpleCameraFollow2D.cs` 추가 → **Target**에 Player Transform 드래그.
12. Player를 **Prefabs** 폴더로 드래그해 프리팹화.

## 코드 1-1: `PlayerController2D.cs`

```csharp
using UnityEngine;

/// <summary>
/// 2D 플랫폼 이동·점프. [Unity API] 물리·입력, [직렬화] Inspector 수치.
/// </summary>
[RequireComponent(typeof(Rigidbody2D))]
public class PlayerController2D : MonoBehaviour
{
    // [직렬화] private이어도 Inspector에 노출. 기획서 수치(이동10 등)를 여기서 조정.
    [SerializeField] private float moveSpeed = 10f;
    [SerializeField] private float jumpForce = 14f;
    [SerializeField] private LayerMask groundLayer; // Ground 레이어만 체크

    [Header("착지 판정 (발 박스)")]
    [SerializeField] private Vector2 groundCheckOffset = new Vector2(0f, -0.55f);
    [SerializeField] private Vector2 groundCheckSize = new Vector2(0.4f, 0.08f);

    // [Unity API] 붙어 있는 Rigidbody2D 캐시
    private Rigidbody2D _rb;

    private void Awake()
    {
        _rb = GetComponent<Rigidbody2D>(); // [Unity API] 같은 오브젝트의 컴포넌트 가져오기
    }

    private void Update()
    {
        float inputX = Input.GetAxisRaw("Horizontal"); // [Unity API] 입력
        // Rigidbody2D: Unity 6에서는 linearVelocity만 있을 수 있음 — 오류 나면 velocity → linearVelocity
        Vector2 v = _rb.velocity;
        v.x = inputX * moveSpeed;
        _rb.velocity = v;

        if (Input.GetButtonDown("Jump") && IsGrounded())
        {
            v.y = jumpForce;
            _rb.velocity = v;
        }
    }

    /// <summary>[우리 구현] 발 박스와 Ground 레이어 겹침 검사.</summary>
    private bool IsGrounded()
    {
        Vector2 pos = (Vector2)transform.position + groundCheckOffset;
        // [Unity API] OverlapBox(중심, 크기, 각도, 레이어마스크)
        return Physics2D.OverlapBox(pos, groundCheckSize, 0f, groundLayer) != null;
    }

    private void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.green;
        Vector2 pos = (Vector2)transform.position + groundCheckOffset;
        Gizmos.DrawWireCube(pos, groundCheckSize);
    }
}
```

## 코드 1-2: `SimpleCameraFollow2D.cs`

```csharp
using UnityEngine;

/// <summary>[우리 구현] 타깃 위치를 부드럽게 따라감. [Unity API] Transform, LateUpdate</summary>
public class SimpleCameraFollow2D : MonoBehaviour
{
    [SerializeField] private Transform target; // [직렬화] Player를 드래그
    [SerializeField] private Vector3 offset = new Vector3(0f, 0f, -10f);
    [SerializeField] private float smooth = 5f;

    private void LateUpdate()
    {
        if (target == null) return;
        // [Unity API] Vector3.Lerp — 선형 보간, deltaTime으로 프레임 독립
        Vector3 desired = target.position + offset;
        transform.position = Vector3.Lerp(transform.position, desired, smooth * Time.deltaTime);
    }
}
```

## 코드 1-3 (선택): `PlayerStatsData.cs` — ScriptableObject

```csharp
using UnityEngine;

// [Unity API] CreateAssetMenu — Project 우클릭 → Create → Game → Player Stats
[CreateAssetMenu(fileName = "PlayerStats", menuName = "Game/Player Stats")]
public class PlayerStatsData : ScriptableObject
{
    public int maxHp = 50;
    public float moveSpeed = 10f;
    public float jumpForce = 14f;
    public int attackPower = 5;
    public float attackCooldown = 1f;
}
```

**에디터**: `Assets/Data`에 **PlayerStats** 에셋 생성 → `PlayerController2D`에서 SO를 읽어 `moveSpeed`를 덮어쓰려면(선택) `Start()`에서 `stats.moveSpeed`를 필드에 대입.

---

# Phase 2 — 바닥·플랫폼

**맵·기획서 장애물 흐름**: [README #map-asset-pipeline](00-강좌-규약-및-참고.md#map-asset-pipeline) (에셋·구현·연동 정본). 프리팹·전체 C#은 [28](28-기획서-맵구성-에셋-구현-연동.md).

## 에셋·에디터 단계

1. **Tilemap** 사용 시: GameObject → 2D Object → **Tilemap**. Grid + Tilemap 생성.
2. **Tile Palette**에 바닥 타일 드래그 후 타일 찍기.
3. **Tilemap Collider 2D** + 필요 시 **Composite Collider 2D**(합쳐진 충돌).
4. 타일맵 대신 **스프라이트 + Box Collider 2D** 여러 개로 발판만 만들어도 됨.
5. 발판 레이어를 **Ground**로 통일(Phase 1 `groundLayer`와 일치).

## 코드

맵만 있으면 추가 스크립트 필수 아님. 스테이지별 스폰 좌표를 쓰려면 `LevelSettingsData`(ScriptableObject)에 `playerSpawn` Vector2를 두고 읽는 **[우리 구현]** 스크립트를 Phase 2~3에 추가하면 됩니다.

---

# Phase 3 — 체력·적·데미지·사망

## 에셋·에디터 단계

1. **Enemy** 프리팹: Sprite + Rigidbody2D + Box Collider 2D.
2. Enemy 태그 `Enemy`, Player 태그 `Player` 설정.
3. Player에 `PlayerHealth.cs`, Enemy에 `EnemyHealth.cs` 추가.
4. Player 자식으로 **공격 판정용** 빈 오브젝트 + **Box Collider 2D** → **Is Trigger** 체크, `PlayerAttackHitbox.cs` 부착.

## 코드 3-1: `PlayerHealth.cs`

```csharp
using UnityEngine;
using UnityEngine.Events;

public class PlayerHealth : MonoBehaviour
{
    [SerializeField] private int maxHp = 50; // [직렬화] 또는 PlayerStatsData 참조
    [SerializeField] private UnityEvent onDeath; // [Unity API] Inspector에서 이벤트 연결

    private int _currentHp;

    private void Start()
    {
        _currentHp = maxHp; // [우리 구현] 런타임 체력
    }

    public void TakeDamage(int amount)
    {
        _currentHp -= amount;
        if (_currentHp <= 0)
        {
            _currentHp = 0;
            onDeath?.Invoke(); // 게임 오버 UI 등 연결
        }
    }

    public int CurrentHp => _currentHp;
    public int MaxHp => maxHp;
}
```

## 코드 3-2: `EnemyHealth.cs`

```csharp
using UnityEngine;

public class EnemyHealth : MonoBehaviour
{
    [SerializeField] private int maxHp = 15;
    private int _hp;

    private void Start() { _hp = maxHp; }

    public void TakeDamage(int dmg)
    {
        _hp -= dmg;
        if (_hp <= 0)
            Destroy(gameObject); // [Unity API] 씬에서 제거
    }
}
```

## 코드 3-3: `PlayerAttackHitbox.cs` — 트리거로 적에게 데미지

```csharp
using System.Collections;
using UnityEngine;

public class PlayerAttackHitbox : MonoBehaviour
{
    [SerializeField] private int damage = 5;
    [SerializeField] private KeyCode attackKey = KeyCode.J; // 또는 Mouse0

    private Collider2D _col;

    private void Awake()
    {
        _col = GetComponent<Collider2D>();
        _col.enabled = false; // 평소 끔
    }

    private void Update()
    {
        if (Input.GetKeyDown(attackKey))
            StartCoroutine(AttackRoutine()); // [C#] 코루틴은 using System.Collections 필요
    }

    private IEnumerator AttackRoutine()
    {
        _col.enabled = true;
        yield return new WaitForSeconds(0.15f); // [Unity API]
        _col.enabled = false;
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        // [Unity API] 트리거 충돌 시
        if (!other.CompareTag("Enemy")) return;
        var eh = other.GetComponent<EnemyHealth>();
        if (eh != null) eh.TakeDamage(damage);
    }
}
```

## 코드 3-4 (선택): `EnemyChase2D.cs` — 플레이어 쪽으로 이동

```csharp
using UnityEngine;

public class EnemyChase2D : MonoBehaviour
{
    [SerializeField] private float moveSpeed = 4f;
    [SerializeField] private Transform player; // [직렬화] 비우면 태그 Player 자동 검색

    private void Start()
    {
        if (player == null && GameObject.FindGameObjectWithTag("Player") != null)
            player = GameObject.FindGameObjectWithTag("Player").transform; // [Unity API]
    }

    private void Update()
    {
        if (player == null) return;
        float dx = Mathf.Sign(player.position.x - transform.position.x); // [C#] 방향만
        transform.position += Vector3.right * dx * moveSpeed * Time.deltaTime; // [Unity API] deltaTime
    }
}
```

**물리 기반 이동**을 쓰려면 Enemy에 Rigidbody2D를 두고 `MovePosition`으로 바꿀 수 있음(09번 문서 참고).

## 코드 3-5: 적이 플레이어에게 닿으면 데미지

Enemy에 붙일 `EnemyContactDamage.cs`:

```csharp
using UnityEngine;

public class EnemyContactDamage : MonoBehaviour
{
    [SerializeField] private int damage = 10;
    [SerializeField] private float cooldown = 1f;
    private float _nextHitTime;

    private void OnCollisionEnter2D(Collision2D collision)
    {
        if (!collision.collider.CompareTag("Player")) return;
        if (Time.time < _nextHitTime) return;
        var ph = collision.collider.GetComponent<PlayerHealth>();
        if (ph != null) ph.TakeDamage(damage);
        _nextHitTime = Time.time + cooldown;
    }
}
```

---

# Phase 4 — UI·게임 오버·재시작

## 에셋·에디터 단계

1. **GameObject → UI → Canvas**. Canvas Scaler: **Scale With Screen Size**, 1920×1080.
2. Canvas 자식 **Image** (HP 바 배경) + 자식 **Image** (Fill, Type **Filled**, Horizontal).
3. **Text — TextMeshPro** import 안내 나오면 Import.
4. **Panel** 하나 전체 화면 반투명 → 자식 Text "게임 오버", Button "재시작". 기본 **비활성**.
5. `PlayerHealth`의 **On Death ()** 에서 `GameOverUI.Show()` 같은 식으로 연결하거나, 아래 스크립트로 자동 연결.

## 코드 4-1: `HpBarUI.cs` — fillAmount

```csharp
using UnityEngine;
using UnityEngine.UI;

public class HpBarUI : MonoBehaviour
{
    [SerializeField] private Image fillImage; // Filled 타입 Image
    [SerializeField] private PlayerHealth playerHealth; // [직렬화] Player 드래그

    private void Update()
    {
        if (playerHealth == null || fillImage == null) return;
        float t = playerHealth.MaxHp > 0 ? (float)playerHealth.CurrentHp / playerHealth.MaxHp : 0f;
        fillImage.fillAmount = t; // [Unity API] Image.fillAmount 0~1
    }
}
```

## 코드 4-2: `GameOverAndRestart.cs`

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameOverAndRestart : MonoBehaviour
{
    [SerializeField] private GameObject gameOverPanel; // [직렬화] 패널 오브젝트

    public void ShowGameOver()
    {
        gameOverPanel.SetActive(true); // [Unity API]
        Time.timeScale = 0f; // [Unity API] 일시정지
    }

    public void Restart()
    {
        Time.timeScale = 1f;
        SceneManager.LoadScene(SceneManager.GetActiveScene().name); // [Unity API] 현재 씬 다시 로드
    }
}
```

**에디터**: `PlayerHealth` → **On Death** → `GameOverAndRestart.ShowGameOver` 드래그 연결.

## 로그라이크 UI (증강 3중 1택)

[25-참조게임-UI-프리팹-디자인-레이아웃.md](25-참조게임-UI-프리팹-디자인-레이아웃.md)의 프리팹 구조를 만든 뒤, 아래는 **최소 동작** 예시입니다.

### 코드 4-3: `AugmentData.cs` (ScriptableObject)

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "Augment", menuName = "Game/Augment")]
public class AugmentData : ScriptableObject
{
    public string augmentId;
    public string augmentName;
    [TextArea] public string description;
    public Sprite icon;
}
```

### 코드 4-4: `AugmentPickUI.cs` — 3장 중 1장 선택

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class AugmentPickUI : MonoBehaviour
{
    [SerializeField] private GameObject panel;
    [SerializeField] private AugmentData[] pool; // [직렬화] 가능한 증강 전부
    [SerializeField] private Image[] cardIcons;
    [SerializeField] private TMP_Text[] cardNames;
    [SerializeField] private Button[] cardButtons;

    private AugmentData[] _choices = new AugmentData[3];

    public void OpenRandomThree()
    {
        panel.SetActive(true);
        Time.timeScale = 0f;
        for (int i = 0; i < 3; i++)
        {
            _choices[i] = pool[Random.Range(0, pool.Length)]; // [C#] 중복 허용. 비허용이면 로직 추가
            cardIcons[i].sprite = _choices[i].icon;
            cardNames[i].text = _choices[i].augmentName;
            int idx = i;
            cardButtons[i].onClick.RemoveAllListeners();
            cardButtons[i].onClick.AddListener(() => Pick(idx)); // [Unity API] UI Button
        }
    }

    private void Pick(int index)
    {
        // [우리 구현] RunState에 저장 등 — 여기서는 로그만
        Debug.Log("선택: " + _choices[index].augmentName);
        panel.SetActive(false);
        Time.timeScale = 1f;
    }
}
```

스테이지 클리어 시 `AugmentPickUI.OpenRandomThree()`를 호출하는 **[우리 구현]** 트리거(골인 충돌 등)를 씬에 두면 됩니다.

---

# Phase 5 — 사운드·빌드

## 에셋·에디터 단계

1. `Audio` 폴더에 **BGM·효과음** WAV/OGG 넣기.
2. Player 또는 전용 오브젝트에 **Audio Source** 추가. **Play On Awake**로 BGM 재생 가능.
3. 공격·피격 시 `AudioSource.PlayOneShot(clip)` 호출하는 한 줄을 기존 스크립트에 추가.

## 코드 5-1: 효과음 한 번 재생

```csharp
// [Unity API] AudioSource.PlayOneShot — 겹쳐서 짧게 재생하기 좋음
public AudioSource sfxSource;
public AudioClip hitClip;

public void PlayHit() => sfxSource.PlayOneShot(hitClip);
```

## 빌드

**File → Build Settings** → 씬 추가 → **PC** 타깃 → Build. 상세 체크는 [19-분야별-분리-통합-및-제품출시.md](19-분야별-분리-통합-및-제품출시.md).

---

# 요약 체크리스트

| Phase | 에셋 핵심 | 코드 스크립트 |
|-------|-----------|----------------|
| 0 | 폴더, Game 씬 | — |
| 1 | Rigidbody2D, Collider, Ground 레이어 | PlayerController2D, SimpleCameraFollow2D |
| 2 | 타일맵 또는 발판 콜라이더 | (선택) LevelSettingsData |
| 3 | Enemy 프리팹, 태그 | PlayerHealth, EnemyHealth, PlayerAttackHitbox, EnemyChase2D(선택), EnemyContactDamage |
| 4 | Canvas, HP 바, 게임오버 패널 | HpBarUI, GameOverAndRestart, (선택) AugmentPickUI |
| 5 | AudioClip, Audio Source | PlayOneShot 호출, 빌드 |

이 문서의 코드는 **교육용 최소 예시**입니다. 24번 문서의 영문 필드명·기획 수치에 맞춰 숫자·이름을 바꿔 쓰면 기획서와 일치시킬 수 있습니다.

<a id="step-gate"></a>

## 단계 연결 · 준비 확인 (이 문서)

**연동**: **동일 Phase**의 [07-Phase-N](07-Phase-0.md)를 펼칠 때 사용. **시작 전**: 해당 Phase **시작 전 준비**·**이전 Phase 완료 후** 먼저 확인.
