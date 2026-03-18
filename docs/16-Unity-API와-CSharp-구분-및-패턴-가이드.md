# Unity API와 C# 함수 구분 및 패턴

코드를 읽을 때 **Unity가 제공하는 것(Unity API)** · **C# 언어/라이브러리** · **우리가 직접 구현하는 스크립트·함수**를 구분할 수 있도록 하고,  
자주 나오는 **패턴**을 이해하기 쉽게 정리한 문서입니다.  
구현 문서(08~15)의 코드 설명에서 **[Unity]** / **[C#]** / **[우리 구현]** 표기를 일관되게 사용합니다.

**영어 용어 한글·발음·정의**: 아래 나오는 영어 단어의 **한글 표기**, **발음**(한글로 읽는 법), **정의**가 필요하면 [22번 문서](22-용어-발음-정의-및-심화학습.md)를 보세요.  
**어려운 개념**(ScriptableObject, MonoBehaviour, API, deltaTime, 쿨다운 등)은 같은 문서의 **심화 학습** 섹션에 자세히 나와 있습니다.

---

# Part 1. Unity API vs C# vs 우리 구현 구분하기

## 1.1 한 줄로 구분하는 방법

| 구분 | 의미 | 예시 |
|------|------|------|
| **[Unity]** | 유니티 엔진이 제공하는 기능. **게임 오브젝트·씬·물리·입력** 등과 관련됨. | `Transform`(트랜스폼), `GameObject`(게임오브젝트), `Instantiate`(인스턴시에이트), `MonoBehaviour`(모노비헤이비어) — 발음·정의: 22번 문서 |
| **[C#]** | C#(씨샵) 언어 또는 .NET이 제공하는 기능. **일반적인 프로그래밍**(숫자, 문자열, 리스트 등)에 쓰임. | `int`(인트), `string`(스트링), `List<>`(리스트), `void`(보이드), `class`(클래스) — 발음·정의: 22번 문서 |
| **[우리 구현]** | **우리가 직접 작성**하는 스크립트·클래스·함수(메서드). 프로젝트 안에 만드는 코드. | `PlayerStats`, `PlayerHealth`, `MakeSingleRoom()`, `DungeonGenerator` |

- **쉽게 기억**: “게임 화면·오브젝트·에디터”와 직접 관련되면 보통 **[Unity]**,  
  “숫자·글자·목록·조건문·반복문”처럼 일반 코드라면 **[C#]**, **우리가 만드는 스크립트/함수 이름**이면 **[우리 구현]** 입니다.

---

## 1.2 코드에서 출처 확인하는 방법

- **이름 앞에 `UnityEngine.`** 이 붙거나, `using UnityEngine;` 가 있으면 → **[Unity]**  
  예: `UnityEngine.GameObject`, `UnityEngine.Transform`
- **스크립트에서 `using UnityEngine;`** 를 쓰면 `GameObject`, `Transform`만 써도 Unity API입니다.
- **`System`** 이나 아무 네임스페이스 없이 쓰는 기본 타입 → **[C#]**  
  예: `int`, `float`, `string`, `List<int>`, `void`, `bool`
- **우리 프로젝트 스크립트에 정의한 클래스·메서드 이름** → **[우리 구현]**  
  예: `PlayerStats`, `PlayerHealth`, `DungeonGenerator`, `MakeSingleRoom()` — 문서나 Assets/Scripts 안에 우리가 작성한 코드.

---

# Part 2. 자주 쓰는 Unity API 정리

아래는 로그라이크 프로젝트에서 자주 나오는 **Unity API**입니다. “무엇을 할 때 쓰는지”만 보면 됩니다.

| 표기 | API (이름) | 용도 (쉬운 설명) |
|------|------------|------------------|
| [Unity] | `GameObject` | 씬에 있는 “오브젝트” 하나를 가리킴. 플레이어, 적, 벽 모두 GameObject. |
| [Unity] | `Transform` | 오브젝트의 **위치·회전·크기**. `transform.position`, `transform.rotation` |
| [Unity] | `transform.position` | 현재 오브젝트의 좌표 (Vector3). 이동할 때 바꿔 줌. |
| [Unity] | `Instantiate(프리팹, 위치, 회전)` | 프리팹을 **복사해서** 씬에 하나 만들어 넣음. 이펙트·몬스터 생성 시 사용. |
| [Unity] | `Destroy(오브젝트)` | 오브젝트를 **씬에서 제거**. 몬스터 사망, 이펙트 끝날 때 사용. |
| [Unity] | `GetComponent<T>()` | 이 오브젝트에 붙어 있는 **컴포넌트(스크립트 등)** 를 가져옴. |
| [Unity] | `Time.deltaTime` | **지난 프레임까지 걸린 시간**(초). 이동할 때 “초당 속도”로 쓰기 좋음. |
| [Unity] | `Time.time` | **게임 시작 후 흐른 시간**(초). 쿨타임 비교할 때 사용. |
| [Unity] | `Input.GetKeyDown(KeyCode.X)` | **해당 키를 이번 프레임에 눌렀는지** true/false. |
| [Unity] | `Input.GetAxis("Horizontal")` | 좌우 방향키/스틱 -1~1 값. 이동 입력에 사용. |
| [Unity] | `Vector3` | x, y, z 좌표 하나. 위치·방향을 나타낼 때 사용. |
| [Unity] | `Quaternion.identity` | “회전 없음”. Instantiate 할 때 회전 그대로 쓸 때 많이 씀. |
| [Unity] | `CompareTag("Player")` | 이 오브젝트의 태그가 "Player"인지 확인. |
| [Unity] | `SendMessage("메서드이름", 인자)` | 이 오브젝트의 스크립트에 있는 **메서드를 이름으로** 호출. (간단할 때만) |
| [Unity] | `ScriptableObject` | **에셋으로 저장되는 데이터**의 부모 클래스. SkillData, EnemyStatsData 등. |
| [Unity] | `MonoBehaviour` | **씬에 붙는 스크립트**의 부모 클래스. Start, Update 사용 가능. |
| [Unity] | `void Start()` | 게임이 시작되고 이 오브젝트가 켜졌을 때 **한 번** 실행. |
| [Unity] | `void Update()` | **매 프레임** 한 번씩 실행. 입력·이동 처리할 때 사용. |
| [Unity] | `void LateUpdate()` | Update가 **다 끝난 뒤** 한 번 실행. 카메라 따라가기 등에 사용. |
| [Unity] | `OnTriggerEnter(Collider other)` | **충돌체(Trigger)** 안에 다른 것이 들어왔을 때 한 번 호출. |
| [Unity] | `OnCollisionEnter(Collision other)` | **충돌체**가 다른 것과 부딪혔을 때 한 번 호출. |
| [Unity] | `SceneManager.LoadScene(이름)` | **다른 씬**을 불러옴. 재시작·다음 층 이동 시 사용. |

→ 위 표에서 **[Unity]** 는 모두 **Unity 엔진**이 제공하는 API입니다.

---

# Part 3. 자주 쓰는 C# 기능 정리

아래는 **C#** 쪽 기능입니다. Unity 없이도 C# 프로그램에서 쓰는 것들입니다.

| 표기 | 기능 (이름) | 용도 (쉬운 설명) |
|------|-------------|------------------|
| [C#] | `int`, `float`, `bool` | **숫자·참/거짓** 타입. HP, 속도, 쿨타임 등. |
| [C#] | `string` | **글자 열**. "Player", "Slime" 같은 문자열. |
| [C#] | `if (조건) { }` | **조건문**. “조건이 참일 때만” 안쪽 코드 실행. |
| [C#] | `for`, `foreach` | **반복문**. 여러 번 같은 코드 실행. |
| [C#] | `void 메서드이름()` | **메서드(함수)**. 한 번 부르면 정해진 코드가 실행됨. |
| [C#] | `return 값` | 메서드에서 **결과값을 돌려주고** 끝냄. |
| [C#] | `new List<>()`, `new int[]` | **목록·배열** 만들기. 스킬 목록, 그리드 등. |
| [C#] | `list.Count`, `array.Length` | 목록/배열 **길이(개수)**. |
| [C#] | `class 클래스이름` | **클래스** 정의. 스크립트·데이터 형식이 클래스. |
| [C#] | `public`, `private` | **밖에서 보일지(public)** 안에서만 쓸지(private) 지정. |
| [C#] | `?.` (null 조건) | 앞이 null이면 아무것도 안 하고, 아니면 뒤를 실행. `obj?.Do()` |
| [C#] | `$"{변수}"` | **문자열 안에 변수** 넣기. 예: `$"HP: {hp}"` |

→ 위는 **C# 언어**가 제공하는 문법·타입입니다. **[Unity]** 표시가 없으면 C# 쪽으로 보면 됩니다.

---

# Part 4. 자주 나오는 패턴 — 이해하기 쉽게

코드에서 **같은 형태**가 반복되면 “패턴”이라고 부릅니다.  
아래는 로그라이크 개발에서 자주 나오는 패턴을 **쉽게** 설명한 것입니다.

---

## 패턴 1: GetComponent — “이 오브젝트에 붙은 걸 가져오기”

**어떤 패턴인가**  
다른 컴포넌트(스크립트, Rigidbody 등)를 **이 오브젝트에서** 찾아서 쓰는 방식.

```csharp
// [Unity] GetComponent — 이 오브젝트에 붙은 "PlayerHealth" 스크립트를 가져옴
PlayerHealth health = GetComponent<PlayerHealth>();
if (health != null)
    health.TakeDamage(10);
```

**쉽게 이해**  
- “이 오브젝트에 **PlayerHealth** 스크립트가 붙어 있나? 붙어 있으면 그걸 **health**에 넣고, **TakeDamage**를 부른다.”  
- **[Unity]** API: `GetComponent<T>()`  
- **[C#]** : `if`, `!= null`, 메서드 호출.

**언제 쓰나**  
- 같은 GameObject에 붙은 **다른 스크립트**의 메서드를 부를 때.  
- 예: 플레이어 오브젝트에 `PlayerCombat`와 `PlayerHealth`가 같이 붙어 있을 때, Combat에서 Health를 찾아서 `TakeDamage` 호출.

---

## 패턴 2: Instantiate / Destroy — “하나 만들기, 하나 지우기”

**어떤 패턴인가**  
프리팹을 **복사해서 씬에 하나 만들고**, 나중에 **그걸 지우는** 방식.

```csharp
// [Unity] Instantiate — hitEffectPrefab을 hitPosition 위치에 하나 생성
GameObject effect = Instantiate(hitEffectPrefab, hitPosition, Quaternion.identity);
// 나중에 이펙트는 SimpleEffectLife 스크립트가 Destroy로 지움
```

**쉽게 이해**  
- **Instantiate** = “이 프리팹을 **따로 하나** 만들어서 씬에 넣어 줘.”  
- **Destroy** = “이 오브젝트를 **씬에서 없애 줘.**”  
- 둘 다 **[Unity]** API.

**언제 쓰나**  
- 이펙트 생성, 몬스터 스폰, 총알 생성 등 “**뭔가를 새로 만들 때**” → Instantiate.  
- 몬스터 사망, 이펙트 끝, “**더 이상 안 보이게 할 때**” → Destroy.

---

## 패턴 3: 매 프레임 이동 (Update + deltaTime)

**어떤 패턴인가**  
**매 프레임**마다 조금씩 움직이게 하는 방식. `Update()` 안에서 `Time.deltaTime`을 곱해 주면 “초당 속도”가 됨.

```csharp
// [Unity] Update — 매 프레임 실행
// [Unity] Time.deltaTime — 지난 프레임까지 걸린 시간(초)
// [C#] float, *=
void Update()
{
    float move = moveSpeed * Time.deltaTime;  // 1초에 moveSpeed만큼 이동
    transform.position += new Vector3(move, 0, 0);
}
```

**쉽게 이해**  
- **Update** = “매번 화면이 그려질 때마다 한 번 실행.”  
- **deltaTime** = “이전 화면부터 지금까지 걸린 시간(초).”  
- `속도 * deltaTime` = “이번 프레임 동안 이만큼만 움직여라” → 컴퓨터가 느리든 빠르든 **이동 속도가 비슷**해짐.  
- **[Unity]** : `Update()`, `Time.deltaTime`, `transform.position`  
- **[C#]** : `float`, `*=`, `+=`

---

## 패턴 4: 쿨타임 (Time.time으로 “다음에 쓸 수 있는 시간” 저장)

**어떤 패턴인가**  
“**다음에 이걸 쓸 수 있는 시간**”을 변수에 넣어 두고, 지금 시간이 그보다 크면 사용 가능.

```csharp
// [Unity] Time.time — 게임 시작 후 흐른 시간(초)
// [C#] float, if
float nextAttackTime;

void Update()
{
    if (Input.GetKeyDown(KeyCode.X) && Time.time >= nextAttackTime)
    {
        nextAttackTime = Time.time + 1f;  // 1초 뒤에 다시 사용 가능
        DoAttack();
    }
}
```

**쉽게 이해**  
- **nextAttackTime** = “이 시간이 되기 전에는 공격 못 함.”  
- **Time.time >= nextAttackTime** = “지금 시간이 됐으니 사용 가능.”  
- 사용한 뒤 **nextAttackTime = Time.time + 1f** = “1초 뒤에 다시 쓸 수 있게 해라.”  
- **[Unity]** : `Time.time`, `Input.GetKeyDown`  
- **[C#]** : `float`, `if`, `>=`

---

## 패턴 5: 싱글톤 (Instance) — “딱 하나만 있는 매니저”

**어떤 패턴인가**  
**그런 스크립트가 씬에 하나만** 있다고 가정하고, `Instance`로 그걸 찾아 쓰는 방식.

```csharp
// [C#] static — “클래스에 하나만 있는” 변수
public static SFXPlayer Instance;

// [Unity] Awake — Start보다 먼저, 한 번 실행
void Awake()
{
    Instance = this;  // “내가 그 하나다”라고 등록
}

// 다른 스크립트에서:
// [Unity] SFXPlayer는 MonoBehaviour
SFXPlayer.Instance?.Play(clip);  // [C#] ?. — null이 아니면 Play 호출
```

**쉽게 이해**  
- **Instance** = “SFXPlayer가 씬에 **한 개**만 있으니까, 그걸 **Instance**로 부르자.”  
- **Awake**에서 **Instance = this** = “이 오브젝트가 그 하나다.”  
- 다른 스크립트는 **SFXPlayer.Instance**만 부르면 **항상 같은 SFXPlayer**를 가리킴.  
- **[Unity]** : `Awake()`, `MonoBehaviour`  
- **[C#]** : `static`, `?.`

**언제 쓰나**  
- 사운드 재생, 게임 매니저, UI 매니저처럼 “**전체에서 하나만** 있으면 되는 것”을 쉽게 참조할 때.

---

## 패턴 6: ScriptableObject — “데이터만 있는 에셋”

**어떤 패턴인가**  
**코드(클래스)** 는 “어떤 항목이 있는지”만 정해 주고, **실제 숫자·참조**는 **에셋 파일**에 넣어 두는 방식.

```csharp
// [Unity] ScriptableObject — 에셋으로 저장되는 데이터
// [Unity] CreateAssetMenu — Project 창에서 우클릭 → Create로 에셋 만들 수 있게 함
[CreateAssetMenu(fileName = "Skill", menuName = "Roguelike/Skill Data")]
public class SkillData : ScriptableObject
{
    // [C#] public 필드 — Inspector와 에셋에 값이 저장됨
    public int damage = 10;
    public float cooldown = 0.5f;
    public GameObject effectPrefab;  // [Unity] 참조
}
```

**쉽게 이해**  
- **SkillData** = “스킬 하나의 **설명서** (이름, 데미지, 쿨, 이펙트).”  
- **ScriptableObject** = “이 설명서를 **프로젝트에 에셋으로 저장**할 수 있게 해 주는 부모.”  
- 게임로직은 **SkillData 타입 변수**만 가지고, Inspector에서 **에셋을 연결**.  
- **[Unity]** : `ScriptableObject`, `CreateAssetMenu`, `GameObject`  
- **[C#]** : `class`, `public`, `int`, `float`

**언제 쓰나**  
- 스킬, 몬스터 스탯, 던전 설정처럼 “**여러 종류가 있고, 숫자만 다르게**” 쓸 때.  
- 코드는 한 번만 쓰고, **에셋을 여러 개** 만들어서 확장.

---

## 패턴 7: 배열/리스트로 “여러 개” 다루기

**어떤 패턴인가**  
스킬·몬스터 종류처럼 **여러 개**를 **한 번에** 다룰 때, 배열이나 리스트에 넣고 **반복문**으로 돌림.

```csharp
// [C#] 배열 — skills[0], skills[1], ...
public SkillData[] skills;
// [C#] for — 0번부터 끝까지
for (int i = 0; i < skills.Length; i++)
{
    if (Input.GetKeyDown(skillKeys[i]))
        TryUseSkill(i);  // [C#] 메서드에 인덱스 전달
}
```

**쉽게 이해**  
- **skills** = “스킬 0번, 1번, 2번 …”  
- **for** = “i가 0, 1, 2, … 일 때마다 **skillKeys[i]** 로 키를 확인하고, 맞으면 **TryUseSkill(i)** 호출.”  
- **[Unity]** : `Input.GetKeyDown` (그리고 SkillData가 Unity 타입을 참조할 수 있음)  
- **[C#]** : `[]`, `for`, `Length`, `int i`

**언제 쓰나**  
- 스킬 목록, 몬스터 종류 목록, 이펙트 종류처럼 “**같은 타입이 여러 개**”일 때.

---

# Part 5. 코드 읽을 때 구분·패턴 보는 순서

1. **이름이 뭔가**  
   - `Transform`, `GameObject`, `Instantiate`, `Time` → 보통 **[Unity]**  
   - `int`, `float`, `List`, `if`, `for` → **[C#]**
2. **어디서 나오는가**  
   - `transform.`, `GetComponent<>`, `Instantiate`, `Destroy` → **[Unity]**  
   - 변수 선언, `if/for`, `return`, `new List<>` → **[C#]**
3. **패턴인가**  
   - “한 번만 있는 걸 Instance로 참조” → 싱글톤  
   - “프리팹을 복사해서 씬에 넣기” → Instantiate  
   - “매 프레임 조금씩 이동” → Update + deltaTime  
   - “N초 뒤에 다시 사용” → Time.time + 쿨  
   - “데이터만 에셋으로” → ScriptableObject  
   - “여러 개를 배열로 + for” → 배열/리스트 패턴

---

# Part 6. 한눈에 보기

| 항목 | 내용 |
|------|------|
| **Unity vs C#** | 게임 오브젝트·씬·입력·시간 관련 → Unity API. 숫자·문자·조건·반복·목록 → C#. |
| **표기** | [Unity], [C#] 표로 구분. 코드 읽을 때 “이건 Unity, 이건 C#”으로 보면 됨. |
| **패턴** | GetComponent, Instantiate/Destroy, Update+deltaTime, 쿨타임(Time.time), 싱글톤(Instance), ScriptableObject, 배열+for — 각각 “무엇을 위한 패턴인지”만 이해하면 됨. |
| **이해하기 쉽게** | 각 패턴마다 “어떤 패턴인가” → “쉽게 이해” → “언제 쓰나” 순서로 설명. |

이 문서를 보면 **Unity API와 C#을 구분**하고, **자주 나오는 패턴**을 이해하기 쉬운 흐름으로 따라갈 수 있습니다.

<a id="step-gate"></a>

## 단계 연결 · 준비 확인 (이 문서)

**연동**: [07](07-단계별-개발-가이드.md) **Phase 1~3** 코딩 중 막힐 때. **순서 게이트 없음** — 필요 시 열어 참고.
