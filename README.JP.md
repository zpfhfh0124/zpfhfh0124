# Hi, I'm Kyeongtae Moon (문경태 / 文 景泰) 👋

[🇺🇸 English](README.md)&nbsp;|&nbsp;🇯🇵 日本語

**ゲームクライアントエンジニア** — C++ / C# · Unity · Unreal Engine 5
実務経験5年以上(Unity:4.5年、加えてUnreal Engine 5の経験あり)

ゲームクライアントエンジニアとして、実際に手を動かしてエンジンを学んでいます。Unreal Engine 4/5やUnityで既存の有名タイトルやキャラクターの再現に取り組み、ImGuiのようなツールのエンジン統合も行いながら、日々のアルゴリズム問題演習で基礎力も磨いています。特に**トゥーン(セルシェーディング)レンダリング**に関心があり、理論だけでなく実際に手を動かして問題を解決し続けることで成長するタイプです — この地道な問題解決の習慣が、エンジニアとしての一番の強みだと考えています。

---

## サンプルコード

GitHubのコミット履歴だけに頼るのではなく、自分が良く書けたと思うコードを3つ、設計意図のメモと一緒に紹介します。残りのアーカイブは下記のお問い合わせ欄からご覧いただけます。

### 🎮 [Misaka-](https://github.com/zpfhfh0124/Misaka-) — 自作2Dゲームエンジン (Win32/GDI, C++)
市販のゲームエンジンを使わず、Win32/GDIベースでゼロから作った約6,000行の2Dエンジンです。
- ジェネリック(テンプレート)なシングルトン基底クラス(`singletonBase<T>`)を作成し、scene / sound / image / effect / key / time / camera の7つのサブシステムマネージャーがこれを継承 — マネージャーごとにボイラープレートコードを重複させない設計
- init/update/render/releaseを仮想関数として持つポリモーフィックなエンティティ基底クラス(`gameNode`)を軸に、boss/enemy/player/stageの階層構造を構築
- 方向 × 状態のenumで管理するボスAI。プレイヤーとの距離に応じて行動を切り替えるロジックを実装

**技術スタック:** C++, Win32 API, GDI

<details>
<summary>singletonBase.h — すべてのマネージャーが継承する再利用可能なシングルトン</summary>

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

// 使用例: class sceneManager : public singletonBase<sceneManager> { ... };
```
</details>

### 🃏 [PriConeTCG](https://github.com/zpfhfh0124/PriConeTCG) — カードバトルゲーム (Unity)
「プリンセスコネクト!」風のカードバトルプロトタイプで、データ設計とイベント設計のクリーンさを重視して実装しました。
- カードデータをScriptableObjectとしてモデル化し、出現率(重み)に基づくランダム抽選アルゴリズムを実装
- ターンの進行はコルーチンと静的イベント(`Action`)で駆動し、ターン管理とカード/UIロジックを疎結合に — 破棄時のイベント解除もきちんと処理
- 円の方程式と`Slerp`/`Lerp`を使い、手札を自然な扇形に配置する計算ロジックを実装

**技術スタック:** Unity, C#

<details>
<summary>CardManager.cs — ScriptableObjectのアイテムテーブルから重み付きランダム抽選</summary>

```csharp
public Item PopRandomItem() // カードを1枚引く
{
    if (_itemBuffer == null || _itemBuffer.Count == 0) SetupItemBuffer();

    Item draw_item = new Item();

    // 重み付きランダム抽選: 全アイテムのprevalence(出現重み)の合計範囲内から
    // ランダムな位置を選び、その位置がどのアイテムの区間に入るかで抽選結果を決定
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

### 🛠️ [UE4_ImGui_Ex](https://github.com/zpfhfh0124/UE4_ImGui_Ex) — エンジンツーリング
Unreal Engine 4にImGui(immediate-mode UIライブラリ)を統合し、UMGウィジェットから時刻表示・画像プレビュー・カラーピッカー・テキスト入力などのデバッグパネルを実行時にトグルできるようにしました。

**技術スタック:** Unreal Engine 4, C++, ImGui

<details>
<summary>UIWidgetMain.cpp — UMGからデバッグ用アクターを遅延取得してキャッシュする処理</summary>

```cpp
void UUIWidgetMain::OnClickTimeWindowBtn()
{
    SetImGuiTestActor();
    if (ImGuiTest == nullptr) return;
    ImGuiTest->ImGui_Show_NowTime();
}

// ImGuiウィンドウを開く前(ボタンクリック時など)に必ず呼び出す必要がある
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

## 技術スタック

**言語:** C++, C#, C
**エンジン:** Unity(4.5年), Unreal Engine 5, Unreal Engine 4
**注力分野:** ゲームプレイ/キャラクターシステム、トゥーン(セルシェーディング)レンダリング、エンジンツーリング(ImGui)、アルゴリズム問題演習

<!-- TODO: paiza랭크取得時にバッジ/リンクを追加 -->

## お問い合わせ

- ポートフォリオ (PDF): [한국어 / 韓国語](https://drive.google.com/file/d/1Q8_2gciXw_UbDPHGQjs476y7HYCNIKex/view?usp=sharing) · [日本語](https://drive.google.com/file/d/1eFbF7LYRC4iME7W3O7-ij-fg2vke-lZ3/view?usp=sharing)
- メール: [yuki79000@gmail.com](mailto:yuki79000@gmail.com)
