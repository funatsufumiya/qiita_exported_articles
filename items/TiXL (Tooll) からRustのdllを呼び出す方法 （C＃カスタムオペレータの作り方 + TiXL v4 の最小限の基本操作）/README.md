:::note warn
この記事は、執筆時点でまだプレビュー版のTiXL (v4)を取り扱った記事になります。

今後の安定版では仕様などが変更される可能性がありますので、あくまで参考程度にし、都度公式ドキュメントやソースコードをよく確認されることをおすすめします。
:::


:::note info
【追記】2025/07/26にTiXL v4がついに[正式リリース](https://github.com/tixl3d/tixl/releases)になったので、下記内容も再検証を行い、インストールパスの修正や、注意点などを追記しています。流れ自体に変更はありません。
:::

## まずTiXL (Tooll) とは

[TiXL](https://www.tixl.app/)というVJツールキットがある。バージョン3まではToollと呼ばれていて、執筆時点で[プレビュー版のバージョン4](https://github.com/tixl3d/tixl/releases)からTiXLと呼ばれるようになった。

[![SnapCrab_NoName_2025-5-20_9-58-31_No-00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/45d555f0-bea9-4fb2-b931-4e0a08f6e566.png)](https://www.tixl.app/)

TiXL (Tooll) の特徴として、[TouchDesigner](https://derivative.ca/)のようなノードベースのリアルタイムグラフィックスが扱える。

![SnapCrab_NoName_2025-5-19_17-52-30_No-00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/51975b88-b1ef-4fe9-b1c4-2e7d93064beb.png)

内部構造的にはどちらかといえばUnityの[VFX Graph](https://unity.com/ja/features/visual-effect-graph)に近くて、コンピュートシェーダーを基本としたGPU駆動のグラフィックス操作に特徴がある。特にv4 (TiXL) からはSDF (Signed Distance Field) 周りのオペレータなどが拡充されたり、操作性や全体構造などが改善された。

![スクリーンショット (15).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/aff34ac0-db12-4195-801f-c5c4bb85399e.png)

## 本記事の概要

![スクリーンショット (5).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/f206e7a2-633c-4a63-9bb0-d969a3ff1102.png)

本記事では、プレビュー版のTiXL v4（執筆時点でv4.0.2 Preview 3）において、Rustのdllを自作のC#オペレータ（ノード）から呼び出す方法を紹介する。

:::note warn
TiXL (v4) も基本的にはTooll3と同じなので、極力両方のバージョンを意識しながら解説していくものの、**今後のバージョンアップ等によってフォルダ構造の変更の可能性がある点に注意**。
:::

## TiXL (v4) で最初のプロジェクトを作る方法

執筆時点で TiXL (v4) の情報が皆無なので、最初の基本操作だけ紹介しておきたい。（**Tooll3の場合は全く手順が違うので飛ばしてほしい。**）

TiXL (v4) を起動すると、Project Hubというものが最初に起動する。その横にある「＋」(プラスマーク) から、新たにプロジェクトを作る。

![スクリーンショット 2025-05-20 122304.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/1abe31cd-322a-4ddf-be5a-f6508c8e8d21.png)

Project Hubから出来上がったプロジェクトを開くと、Exampleなどの雛形が生成されている。[^1]

![スクリーンショット 2025-05-20 122448.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/60196bcd-23b4-45b6-819e-c031e10f3a53.png)

ここで、「TiXL > New Operator...」を使って新しいシーン（オペレータ）を作る。

![スクリーンショット (6).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/442fa962-b7b2-424d-9b88-5b8004dac62c.png)

![スクリーンショット 2025-05-20 122549.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/b26bc6a7-9dec-4ff8-a8bb-afec93a5686e.png)

作られたオペレータは先程のExampleの雛形に混じって置かれているので、ダブルクリックすると開ける。

![スクリーンショット (7).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/3a794dec-d3c8-47aa-8757-09281e23e1c3.png)

![スクリーンショット (8).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/d553b4eb-ac9f-4dd0-a433-8de627cd918a.png)

あとの基本操作はTooll3と変わらないが、MagGraphと呼ばれる、磁石のマグネットのようにノードがくっつく新しい操作スタイルなどの若干の変更点はある（無効化もできる）。これについては当記事の範疇を超えるので、割愛する。

（新規ノードを作るときはTabキー、削除するときはバックスペースを2回押し、上の階層に戻るときは背景をダブルクリック、くらい覚えておけば大丈夫なはず。）

## TiXL (Tooll3) でカスタムオペレータ (C#) を作る方法

これについては以下動画が詳しい。

<iframe width="560" height="315" src="https://www.youtube.com/embed/fMxJLrElHaA?si=1zi3pYZ_u1r7AvfC" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


TiXL (Tooll) では、ノード（オペレータ）を右クリックすると「Symbol definition... > Duplicate as new type...」という、既存のノードを複製して独自のカスタムノードを作る機能がある。

![スクリーンショット (9).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/dc0c6024-5a77-4830-81f0-03395a03fcd4.png)

ここで一点注意として、どんなオペレータでも複製できるのだけれど、ノードベースなのですべてのオペレータがC#で作られているとは限らない。どのオペレータもインターフェイスとしてのC#は必ず生成されるが、中身がほとんどないものはノードベースで構築されていて、C#以外のメタファイル（`.t3`や`.t3ui`）などに結線の情報が記述されている。

## Moduloを複製してカスタムオペレータを作る

まずここでは、「Modulo」というオペレータを複製する。名前はRustInteropTestとしておく。

![スクリーンショット (9).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/dc0c6024-5a77-4830-81f0-03395a03fcd4.png)

![スクリーンショット 2025-05-20 104635.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/bab4a622-e845-4c6d-b7ea-132c7ca6d102.png)

複製元のModuloの元コードは以下のようになる。（複製によって`namespace`とクラス名は変更される。） **※ コードの開き方はすぐ後の項で後述します。**

https://github.com/tixl3d/tixl/blob/813a220493bacc5469a581891feb3c5a04473ccc/Operators/Lib/numbers/float/basic/Modulo.cs

以下に念の為全文を掲載しておく。

```cs:Modulo.cs
namespace Lib.numbers.@float.basic;

[Guid("5202d3f6-c970-4006-933d-3c60d6c202dc")]
internal sealed class Modulo : Instance<Modulo>
{
    [Output(Guid = "4e4ebbcf-6b12-4ce7-9bec-78cd9049e239")]
    public readonly Slot<float> Result = new();

    public Modulo()
    {
        Result.UpdateAction += Update;
    }

    private void Update(EvaluationContext context)
    {
        var v = Value.GetValue(context);
        var mod2 = ModuloValue.GetValue(context);

        if (mod2 != 0)
        {
            Result.Value = v - mod2 * (float)Math.Floor(v/mod2);
        }
        else
        {
            T3.Core.Logging.Log.Debug("Modulo caused division by zero", this);
            Result.Value = 0;
        }
    }
        
    [Input(Guid = "8a401e5d-295d-4403-a3af-1d6b91ce3dba")]
    public readonly InputSlot<float> Value = new();

    [Input(Guid = "62a8185f-32c0-41d2-b8be-d8c1d7178c00")]
    public readonly InputSlot<float> ModuloValue = new();
}
```

ここで注意として、各Guidは一意になっていないといけない[^2]。Powershellの`New-Guid`コマンドからもGuidの新規生成は可能で、自分の場合はVSCodeのGuid生成の拡張機能を使っている。（ショートカットキーなど好みがあると思うし、複数種あるので好きなものを選んでほしい。）

ここで生成したGuidは、先ほどちらっと触れた、`.t3` や `.t3ui` などのメタデータの中で利用される。本来はそちらの記述も変更したり、エディタから生成されるものからコピーしたりする必要があるが、これについては当記事では割愛する。

## C#のコードを開く方法

順番が少し前後して申し訳ないが、オペレータを複製して**一度プロジェクトを保存する**と、プロジェクトフォルダに `.cs` が生成される。（**→ 一度プロジェクトを保存するのが必要。** 保存していないとファイルも生成されない。）

これを開くには、素朴に`.cs`をVSCodeなどから開いて編集しても良い。この場合も`.cs`は自動的に再ビルドされる。

ただそれだと構文解析などがあまり効かなくて不便なので、TiXL (v4) の場合は、一度プロジェクトを一つ開いたあと、「TiXL > Open Project in... > Development IDE」からVisual Studioのプロジェクトを開くと良い。[^b] （※ Tooll 3 の場合は、脚注参照。）

![スクリーンショット 2025-05-20 105819.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/dd5f205e-104d-4645-b3c1-a5e5f1023745.png)

![スクリーンショット 2025-05-20 114904.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/27544299-a14a-4884-8a67-323c0cb84e89.png)

注意として、Visual Studioで開く場合も、ビルドはファイルの変更を検知して基本的に自動で行われるので、Visual Studio側から再ビルドを行う必要はない。ただ、稀にTiXL自体が起動しなくなるようなdllを生成してしまうことがあるので、その場合はプロジェクトフォルダの`bin\Debug` 以下を一度削除して再起動する必要が出てくる。（ブラウザでいうところのキャッシュを消すような感じ。）[^c] (※ Tooll 3 の場合は、脚注参照。)

## TiXL (v4) と Tooll3 のフォルダ構造の違い

ここで一点、バージョンごとにフォルダ構造の違いに触れておきたい。

**Tooll3までは、すべてのプロジェクトが一箇所で一元管理されていた。** インストールフォルダそのものがプロジェクトフォルダとなっているので、ある意味わかりやすいが、ファイルがごちゃごちゃしやすい。

TiXL (v4) からは、ユーザのプロジェクトは基本的に `Documents\TiXL` 以下に配置されるようになった。**インストールフォルダ** (例えば `C:\TiXL\TiXL-v4.0.2`や`C:\Program Files\TiXL\TiXL 4.0.5.2`) **とは分離されている** ので注意が必要。

## カスタムオペレータを作っていく

さて、ここからは好きにコードをいじれば良いのだけれど、先に全文を掲載する。（namespaceなどは各人で読み替えてほしい。）

※ 前述のように、Guidは一意になっていないといけないので、自分で自作するときは、PowershellのNew-GuidコマンドなどからGuidを適宜生成することを推奨。これを自動化するアドオン等については割愛。

:::note warn
**【追記】4.0.5正式版で確認したところ、TiXLの起動中に編集してしまうと、.csファイルごと消えてしまう現象が確認されており、編集時はTiXLを閉じることを強く推奨。** [^a] （※ 既に編集済みでこの現象によってファイルが消えてしまった場合は、脚注を参照のこと！）
:::

```cs:RustInteropTest.cs
using System.Runtime.InteropServices;
namespace fu.test.test2;


[Guid("4334b296-8559-4707-a35b-99ccb08215a7")]
internal sealed class RustInteropTest : Instance<RustInteropTest>
{
    [DllImport("rust_interop_lib.dll", CallingConvention = CallingConvention.Cdecl)]
    private static extern float add(float a, float b);

    [Output(Guid = "8fd92c8e-d043-4a6e-97b7-594830f59e2c")]
    public readonly Slot<float> Result = new();

    [Output(Guid = "7b1b40b9-9450-43d0-a8e6-1d05ac5ff313")]
    public readonly Slot<string> OutText = new();

    public RustInteropTest()
    {
        OutText.UpdateAction += Update;
        Result.UpdateAction += Update;
    }

    private void Update(EvaluationContext context)
    {
        var a = ValueA.GetValue(context);
        var b = ValueB.GetValue(context);
        var v = 0.0f;

        try {
            v = add(a, b);
            Result.Value = v;
        }
        catch (Exception e) {
            T3.Core.Logging.Log.Debug("RustInteropTest caused exception (maybe dll not found): {0}", e);
            Result.Value = 0.0f;
        }

        OutText.Value = $"{a} + {b} -> {v}";
    }
        
    [Input(Guid = "61814eeb-54eb-4f0e-a896-1292698e3c4e")]
    public readonly InputSlot<float> ValueA = new();

    [Input(Guid = "ca343900-a386-43b8-91b7-366cd9d72379")]
    public readonly InputSlot<float> ValueB = new();
}
```

このコードでは、出力が2つあって、`Result` (`float`) に 入力された `ValueA` (`float`) と `ValueB` (`float`) の足し算の結果を返す。

そして、デバッグ用に `OutText` (`string`) に、 `a + b -> c` のテキストを返す。（冒頭のスクリーンショット ↓ ではこちらを使って結果を表示している。）

![スクリーンショット (5).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/f206e7a2-633c-4a63-9bb0-d969a3ff1102.png)

## Rust側のdllを準備する

先程のコードのDLL呼び出し部分を見ておきたい。

```cs
[DllImport("rust_interop_lib.dll", CallingConvention = CallingConvention.Cdecl)]
private static extern float add(float a, float b);
```

ごくシンプルに、`rust_interop_lib.dll` というものから、`float add(float a, float b)` に該当する関数を呼び出すというものになっている。（呼び出し時にdll関係のエラーが出る可能性があるので、try-catchで囲んで呼び出している点にも注意。）

となるとあとは単純にこの名前のdllを作って、TiXL (Tooll) が実行時にdllを確認できるパスにそのdllを配置すれば良いことになる。

雛形は以下に作った。

https://github.com/funatsufumiya/rust_interop_lib

上記をユーザのプロジェクトフォルダにクローン (`git clone`) して、`cd rust_interop_lib; cargo build --release` すれば、`target\release\rust_interop_lib.dll` にdllが生成される。

あとは、TiXL (v4) の場合は (ユーザプロジェクトフォルダの) `dependencies` 直下に（`PlaceNativeDllDependenciesHere.txt`というファイルがあるフォルダの直下に） dllを置いて、冒頭のスクリーンショット ↓ のように正しい結果が出れば成功となる。

**もし、入力のValueAとValueBを0以外にしているにも関わらず計算結果が表示されなかったり、`-> 0` のように結果がおかしい場合はdll読み込みに失敗している**。

（読み込まれない場合は、`bin\Debug` を一度削除して再起動するなどして、再ビルドを促すと正しくdllがコピーされ、読み込みが成功する。）

![スクリーンショット (5).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/f206e7a2-633c-4a63-9bb0-d969a3ff1102.png)

## 補足: TiXL (v4) でのdll配置場所

:::note warn
【追記】dll配置場所について、記事公開当初誤った記載をしていたので、前項の内容も含めて正しい記載に直しました。以下内容についても、追記後の内容となっています。
:::

記事公開当初、誤った配置場所にdllを置いていたり、再ビルドしていなかったりしたため、一度置いたdllが消えてしまったり、読み込まれなかったりのトラブルに見舞われていた。

**再ビルドしなくても必ずdllを読み込ませるための回避策は一応あって**、TiXL自体のインストールフォルダの `runtimes\win-x64\native` にdllを置くという方法がある。具体的には、`C:\TiXL\TiXL-v4.0.2\runtimes\win-x64\native` や `C:\Program Files\TiXL\TiXL 4.0.5.2\runtimes\win-x64\native`以下にdllを置けば、C#オペレータの再ビルドに関係なくdllが読み込まれる。

これで一応解決なのだけれど、これだと先に説明した、TiXL (v4) からユーザのプロジェクトフォルダが分離された意味があまりなくなってしまうので、ユーザプロジェクトフォルダの dependenciesフォルダに基本的にはユーザのdllは置き、必要に応じて再ビルド（`bin\Debug`を消して再起動するなど）させるようにしてほしい。（dependencies以下に置いたdllは、再ビルド時にdllは必ず実行パスにコピーされる。）

## まとめ

これで好きな（Rustなどの）dllを呼び出すことが可能になった。例えばRustのDeep Learningフレームワークである[burn](https://burn.dev/)なども使えるようになるし、Rust以外にもC/C++やPyPyなどのdllも読み込めるので、できることの幅がぐんと広がる。

なお、dllを毎回作るのはちょっと面倒だなぁと思う人は、[Chataigne](https://benjamin.kuperberg.fr/chataigne/en) (※フランス語で栗の意味) などのOSC通信によっていろんなやり取りを可能にする方法をまず検討すると良いかもしれない。これだとTiXL (Tooll) と別のソフトをいくつか立ち上げて通信することにはなるものの、OSCを介在して手っ取り早くいろんなやり取り（例えばArduinoとやり取りしたり、センサーやDAWとやり取りしたり）が可能になる。

個人的に、dllを呼び出すよりもC#で直接書いたほうがいいケースのほうが多いとは思うので、nugetなどを使って外部ライブラリとやり取りする方法なども調べていきたい。（基本的には通常のC#プロジェクトと同じ流れになるはず。 **【追記】** `.csproj` があるフォルダで、`dotnet add package NumSharp` などすれば普通に使えた。Visual Studioの補完もきちんと効く。）

[^1]: Exampleなどの雛形を消したい場合には、マウスで全選択して消せば良いのだけれど、そのとき全てのノードがプレビューされようとして重いので、Output Viewの画面をなにか一つにPin留めをしておくとスムーズに消せる。（Pinについては、F12キーのFocus Modeに入ったときに勝手にPin留めされたりとハマりがちなポイントなので、覚えておくと良いと思う。） ![スクリーンショット 2025-05-20 123847a.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/623754a0-63a3-4b37-893e-1242ef457d60.png)

[^2]: Guidが一意になっていない場合（他と重複がある場合）は、Visual StudioやTiXL (Tooll3) の起動時に警告やエラーが出る。このとき最悪の場合はTiXL (Tooll) 自体が起動できなくなってしまうので、特に TiXL (v4) では、[本文中で解説した](#cのコードを開く方法)、中間生成結果のdllを消して再ビルドを促すような処置が必要になる。

[^a]: なぜ消えてしまうかについては、実はプロジェクトフォルダのLib以下に、元々のModuloのnamespaceと対応するフォルダ名の中に`.cs`/`.t3`/`.t3ui`が生成されている場合があり、**namespaceをシステム側が何らかの要因で誤認するため**だと思われる。これら3つのファイルに関して、フォルダを正しく移動させ直せば、直せると思われるが、現時点では未検証というか、個々の事象に対応できないので、このバグに見舞われた場合は、今後のリリースでの修正を待つか、個々人で丁寧にファイルを確認・移動・修正する必要がある。（筆者の場合は面倒なので別名で作り直したりしている。）

[^b]: Tooll3でもVisual Studioを使いたい場合は、公式ドキュメントの [Installing with development environment](https://github.com/tixl3d/tixl/wiki/help.Installation#installing-with-development-environment) を参考にしてほしい。ただし、TiXL v4の公開によってWikiの内容が変更になる可能性があるので、リンク切れに注意。現時点で既にgit cloneのURLはTiXL v4を指してしまっていたりするので、読み替えが必要。

[^c]: Tooll3の場合は、dllの配置先はインストールフォルダ直下だが、各オペレータに関する中間ファイルは明示的には生成されないようなので、自作オペレータの再ビルドをかけたい場合は、単にTooll3を再起動すれば良いはず。
