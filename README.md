# Hi, I'm Kyeongtae Moon 👋

🇺🇸 English&nbsp;|&nbsp;[🇯🇵 日本語](README.JP.md)

**Game Client Developer** — C++ / C# · Unity · Unreal Engine 5
5+ years of experience (Unity: 4.5 years, plus Unreal Engine 5 experience)

I'm a game client developer who learns engines by building with them — recreating known characters/games in Unreal Engine 4/5 and Unity, and integrating tooling like ImGui, while also keeping algorithm fundamentals sharp through daily problem solving. I'm especially interested in **toon/cel-shaded rendering**, and I grow by consistently working through problems hands-on rather than just studying theory — that steady, problem-driven habit is my biggest strength as a developer.

---

## Sample Code

Rather than relying on GitHub commit history alone, here are three pieces of code I'm proud of, with notes on the design decisions behind each — the rest of my curated archive is linked under Get in Touch below.

### 🎮 [Misaka-](https://github.com/zpfhfh0124/Misaka-) — Custom 2D Game Engine (Win32/GDI, C++)
A ~6,000-line 2D engine built from scratch, without a commercial engine underneath.
- A generic, template-based singleton base class (`singletonBase<T>`) reused across 7 subsystem managers (scene, sound, image, effect, key, time, camera) — no duplicated boilerplate per manager
- Polymorphic entity base (`gameNode`) with virtual init/update/render/release, driving a boss/enemy/player/stage hierarchy
- Direction × state enum-driven boss AI with distance-based behavior switching
- **Stack:** C++, Win32 API, GDI

<details>
<summary>singletonBase.h — the reusable singleton every manager inherits from</summary>

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

// usage: class sceneManager : public singletonBase<sceneManager> { ... };
```
</details>

### 🃏 [PriConeTCG](https://github.com/zpfhfh0124/PriConeTCG) — Card Battle Game (Unity)
A Princess Connect!-style card battle prototype focused on clean data/event architecture.
- Card data modeled as ScriptableObjects, with a weighted-random draw algorithm for card drop rates
- Turn flow driven by coroutines + a static event (`Action`), decoupling the turn manager from card/UI logic, including proper unsubscription on destroy
- Hand-layout math using the circle equation and `Slerp`/`Lerp` to fan cards out realistically
- **Stack:** Unity, C#

<details>
<summary>CardManager.cs — weighted-random draw from the ScriptableObject item table</summary>

```csharp
public Item PopRandomItem() // draw a card
{
    if (_itemBuffer == null || _itemBuffer.Count == 0) SetupItemBuffer();

    Item draw_item = new Item();

    // Weighted random draw: pick a random position within the sum of every
    // item's prevalence (drop weight), then find which item's range it falls into
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

### 🛠️ [UE4_ImGui_Ex](https://github.com/zpfhfh0124/UE4_ImGui_Ex) — Engine Tooling
Integrated the ImGui immediate-mode UI library into Unreal Engine 4, wired into a UMG widget so debug panels (time, image preview, color picker, text input) can be toggled at runtime.
- **Stack:** Unreal Engine 4, C++, ImGui

<details>
<summary>UIWidgetMain.cpp — lazily resolving and caching the debug actor from UMG</summary>

```cpp
void UUIWidgetMain::OnClickTimeWindowBtn()
{
    SetImGuiTestActor();
    if (ImGuiTest == nullptr) return;
    ImGuiTest->ImGui_Show_NowTime();
}

// Must be called before opening any ImGui window (e.g. from a button click)
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

## Tech Stack

**Languages:** C++, C#, C
**Engines:** Unity (4.5 yrs), Unreal Engine 5, Unreal Engine 4
**Focus areas:** Gameplay/character systems, toon/cel-shaded rendering, engine tooling (ImGui), algorithm problem solving

<!-- TODO: paiza 랭크 취득 시 배지/링크 추가 -->

## Get in Touch

- Portfolio (PDF): [한국어](https://drive.google.com/file/d/1Q8_2gciXw_UbDPHGQjs476y7HYCNIKex/view?usp=sharing) · [日本語](https://drive.google.com/file/d/1tIWVTJJ0urTxC1sxyQ6Bpj-LIRsWApbl/view?usp=sharing)
- Email: [yuki79000@gmail.com](mailto:yuki79000@gmail.com)
