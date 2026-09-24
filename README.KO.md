# 안녕하세요, 문경태입니다 👋

[🇺🇸 English](README.md)&nbsp;|&nbsp;🇰🇷 한국어&nbsp;|&nbsp;[🇯🇵 日本語](README.JP.md)

**게임 클라이언트 개발자** — C++ / C# · Unity · Unreal Engine 5 + 실무 경력 (Unity 4.5년 + Unreal Engine 5 경험)

직접 만들어보면서 엔진을 익히는 게임 클라이언트 개발자입니다. Unreal Engine 4/5와 Unity로 잘 알려진 캐릭터·게임을 재현해보고, ImGui 같은 툴을 엔진에 직접 통합해보며, 매일 알고리즘 문제를 풀면서 기본기도 놓지 않고 있습니다. 특히 **툰(셀 셰이딩) 렌더링**에 관심이 많고, 이론만 파기보다는 직접 부딪혀 문제를 해결하며 성장하는 편입니다 — 이 꾸준한 문제 해결 습관이 개발자로서 제 가장 큰 강점이라고 생각합니다.

---

## 대표 코드

GitHub 커밋 히스토리만으로는 다 드러나지 않는, 제가 자신 있게 소개할 수 있는 코드 3가지를 설계 의도와 함께 정리했습니다. 나머지 아카이브는 아래 연락처 항목에 링크되어 있습니다.

### 🎮 [Misaka-](https://github.com/zpfhfh0124/Misaka-) — 커스텀 2D 게임 엔진 (Win32/GDI, C++)
상용 엔진 없이 Win32/GDI 기반으로 처음부터 직접 만든 약 6,000줄 규모의 2D 엔진입니다.
- 템플릿 기반의 제네릭 싱글톤 베이스 클래스(`singletonBase<T>`)를 만들어 scene/sound/image/effect/key/time/camera 7개 서브시스템 매니저가 공통으로 상속 — 매니저마다 보일러플레이트 코드가 중복되지 않도록 설계
- init/update/render/release를 가상 함수로 갖는 다형적 엔티티 베이스 클래스(`gameNode`)를 축으로 boss/enemy/player/stage 계층 구조를 구성
- 방향 × 상태 enum 조합으로 동작하는 보스 AI — 플레이어와의 거리에 따라 행동을 전환
- **기술 스택:** C++, Win32 API, GDI

<details>
<summary>singletonBase.h — 모든 매니저가 상속하는 재사용 가능한 싱글톤</summary>

```cpp
template <typename T>
class singletonBase
{
protected:
    static T* singleton;
    singletonBase() {}
    ~singletonBase() {}
public:
    static T* getSingleton();
    void releaseSingleton();
};

template <typename T>
T* singletonBase<T>::singleton = 0;

template<typename T>
inline T* singletonBase<T>::getSingleton()
{
    if (!singleton) singleton = new T;
    return singleton;
}

template<typename T>
inline void singletonBase<T>::releaseSingleton()
{
    if (singleton)
    {
        delete singleton;
        singleton = nullptr;
    }
}

// 사용 예: class sceneManager : public singletonBase<sceneManager> { ... };
```
</details>

### 🃏 [PriConeTCG](https://github.com/zpfhfh0124/PriConeTCG) — 카드 배틀 게임 (Unity)
프린세스 커넥트! 스타일의 카드 배틀 프로토타입으로, 깔끔한 데이터/이벤트 구조 설계에 초점을 맞췄습니다.
- 카드 데이터를 ScriptableObject로 모델링하고, 카드 등장 확률에 가중치 기반 랜덤 추첨 알고리즘을 적용
- 코루틴과 정적 이벤트(`Action`)로 턴 진행을 구동해 턴 매니저와 카드/UI 로직을 분리 — 파괴 시 이벤트 구독 해제까지 꼼꼼히 처리
- 원의 방정식과 `Slerp`/`Lerp`를 활용해 손패를 자연스러운 부채꼴로 배치하는 연산 로직 구현
- **기술 스택:** Unity, C#

<details>
<summary>CardManager.cs — ScriptableObject 아이템 테이블에서 가중치 기반 랜덤 추첨</summary>

```csharp
public Item PopRandomItem() // 카드 한 장 뽑기
{
    if (_itemBuffer == null || _itemBuffer.Count == 0) SetupItemBuffer();

    Item draw_item = new Item();

    // 가중치 기반 랜덤 추첨: 전체 아이템의 prevalence(등장 가중치) 합계 범위 내에서
    // 임의의 위치를 뽑고, 그 위치가 어느 아이템의 구간에 속하는지로 결과를 결정
    int sum_prev = SumPrevalenceItems(_itemBuffer);
    int cur_prev = 0;
    float pop = Random.Range(0, sum_prev);

    foreach (var item in _itemBuffer)
    {
        if (pop >= cur_prev && pop < cur_prev + item.prevalence)
        {
            draw_item = item;
            break;
        }
        else cur_prev += item.prevalence;
    }

    return draw_item;
}
```
</details>

### 🛠️ [UE4_ImGui_Ex](https://github.com/zpfhfh0124/UE4_ImGui_Ex) — 엔진 툴링
Unreal Engine 4에 ImGui(즉시 모드 UI 라이브러리)를 통합하고, UMG 위젯과 연결해 시간 표시/이미지 프리뷰/컬러 피커/텍스트 입력 같은 디버그 패널을 런타임에 토글할 수 있도록 구현했습니다.
- **기술 스택:** Unreal Engine 4, C++, ImGui

<details>
<summary>UIWidgetMain.cpp — UMG에서 디버그용 액터를 지연 탐색 후 캐싱하는 처리</summary>

```cpp
void UUIWidgetMain::OnClickTimeWindowBtn()
{
    SetImGuiTestActor();
    if (ImGuiTest == nullptr) return;
    ImGuiTest->ImGui_Show_NowTime();
}

// ImGui 창을 열기 전(버튼 클릭 시 등) 반드시 호출되어야 함
void UUIWidgetMain::SetImGuiTestActor()
{
    if (ImGuiTest == nullptr)
    {
        for (auto* currActor : TActorRange<AImGuiTest>(GetWorld()))
        {
            ImGuiTest = currActor;
        }
    }
}
```
</details>

---

## 기술 스택

**언어:** C++, C#, C
**엔진:** Unity(4.5년), Unreal Engine 5, Unreal Engine 4
**관심 분야:** 게임플레이/캐릭터 시스템, 툰(셀 셰이딩) 렌더링, 엔진 툴링(ImGui), 알고리즘 문제 풀이

<!-- TODO: paiza 랭크 취득 시 배지/링크 추가 -->

## 연락처

- 포트폴리오 (PDF): [한국어](https://canva.link/0n2c330t6z45v85) · [日本語](https://canva.link/5oaydl44ak1jmwy)
- 이메일: [yuki79000@gmail.com](mailto:yuki79000@gmail.com)
