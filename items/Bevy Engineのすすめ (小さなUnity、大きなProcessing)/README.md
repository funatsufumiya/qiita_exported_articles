(英語版も書きました: https://dev.to/funatsufumiya/bevy-engine-as-a-small-unity-big-processing-3edg)

![スクリーンショット 2024-01-22 17.35.32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/512755b8-198c-c19c-1e33-d38febb9cea7.png)
[^1]

## はじめに: クリエイティブ・コーディングとProcessing

ゲームエンジンってたくさんありますが、あなたは何を使ってますか？

私はいわゆるクリエイティブ・コーディングが好きで、思いついたことを形にするために、例えばUnityやUE5、[Touch Designer](https://derivative.ca/)や[Notch](https://www.notch.one/)、Reactなど、その都度プロトタイピングにちょうど良さそうなツールを選んでは、思いついたものを作っていたりします。

そんなときふと、クリエイティブ・コーディングの自分にとっての源流ってなんだろうと思うと、やはり[Processing](https://processing.org/)だろうなと思います。

Processingの何が良かったかと振り返ると、やはり気軽さでしょう。今でも[p5.js](https://p5js.org/)や[openFrameworks](https://openframeworks.cc/ja/)のような形でその哲学は引き継がれ続けていますが、例えばp5.jsは[オンラインエディタ](https://editor.p5js.org/)をブラウザで開くだけで気軽にコードが書けたりします [^2] 。

![スクリーンショット 2024-01-22 17.05.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/ae34f4ba-e027-863f-e851-349b076d6fe0.png)

当時から、Processingはエディタ一つ入れるだけで他に何もいらず、ただシンプルにコードを書くことを楽しむことができていました。

### 大規模ゲームエンジンとその狭間

ところで、クリエイティブ・コーディングというのは別に形態を問わない概念であって、私はBlenderやUnreal Engineなどに搭載されている、いわゆるビジュアルエディタも立派なクリエイティブ・コーディングだと思っています。

![image01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/de09eae3-2ea2-7c23-bed1-ead514624689.png) (図: Blenderのジオメトリノード [^3])

![material-editor-hero-image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/a36cbedc-01cd-cc2a-5988-4f3a0d439639.png) (図: Unreal Engine 5のマテリアルエディタ [^4])

実際、私自身もこうしたものをフル活用してアイデアを形にしていますし、ノーコードという考え方があるように、もう今はコードというのは一つの概念になった感すらあります。

Webブラウザなどもエンジンの一つですが、ゲームエンジンに限らずエンジンというのは、私たちの制作を支える屋台骨であり、なくてはならないものになっています。

しかしながらときどき、Processingのように、軽量で使い勝手の良いものに原点回帰したいなという想いと、Reactのようなリアクティブプログラミングといった、**近代的な概念と気軽さが融合したものが欲しい**、つまりUnityとProcessingの狭間にあるものが欲しいと、ふと思うことがあります。何十GBもあるエディタのダウンロードを待つ必要がなく、すぐに手を動かせるくらいのものが欲しいな、と [^5]。

## Bevy Engineにみる、UnityとProcessingのあいだ

そう思っていたとき、ふと出会ったのが[Bevy Engine](https://github.com/bevyengine/bevy)というものでした。

![スクリーンショット 2024-01-22 17.39.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/dcd305b8-cc3d-3154-fd65-096b934e158e.png) [^6]

Bevy Engineはオープンソースであり、まだ開発されてそんなに年数が経っていないのですが、まるで[openFrameworksのアドオン群](https://ofxaddons.com/)のように、多数の有志によるプラグイン群が存在しています。

ProcessingやopenFrameworksと似ているのは、そうしたプラグインシステムを**必要に応じて**追加していく形式をとっており、コアは小さく書かれているということです。

そして何より、Rust (cargo) 以外のツールは基本的に要りません。まず `$ cargo new bevy-hello` [^bash] で空のプロジェクトを作り、`$ cd bevy-hello` でプロジェクトフォルダに移動し、`$ cargo add bevy` でbevyをライブラリとして追加し、あとは `src/main.rs` に好きなExampleにあるコード (例えば以下) を貼り付けたりして書き、`$ cargo run` するだけで動きます。

```rust:hello-world
use bevy::prelude::*;

fn main() {
    App::new()
        // .add_plugins(DefaultPlugins) // ※ この行を含むとウィンドウが出ます
        .add_systems(Startup, hello_world)
        .run();
}

fn hello_world() {
    println!("hello world");
}
```

![スクリーンショット 2024-01-22 18.58.17a.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/1250ac7e-85ba-5d84-7c78-696611bc86d9.png)

初回の起動 (ビルド、コンパイル) には時間がかかりますが、2回目以降は驚くほど高速に再読み込みでき、必要に応じてアセットなどのホットリロードもできます。

## ECS (Entity Component System) を眺める

![IMG_0627.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/7f99290b-03ac-8dd9-c33b-a995ead50733.png)  (図: ECSの模試図 [^e])


### まずは "System"

さて、いわゆるHello Worldが書けたところで、せっかくなので少しだけ他のコードも眺めてみましょう。

```rust
// (前略)

fn main() {
    App::new()
        .add_plugins(DefaultPlugins) // これはおまじない
        .add_systems(Startup, setup)
        .add_systems(Update, (system, rotate_camera, update_config))
        .run();
}
```

これは[3D Gizmos](https://bevyengine.org/examples/3D%20Rendering/3d-gizmos/)というサンプルからとったコードの冒頭部ですが、`add_systems(Startup, ...)` と `add_systems(Update, ...)` という部分が、Processingでいうところの`setup` (一回だけ呼ばれる関数) と`draw` (更新時に毎回呼ばれる関数) を登録しています。

実際に登録されている関数を一つ見てみましょう。

```rust
fn system(mut gizmos: Gizmos, time: Res<Time>) {
    gizmos.cuboid(
        Transform::from_translation(Vec3::Y * 0.5).with_scale(Vec3::splat(1.)),
        Color::BLACK,
    );
    gizmos.rect(
        Vec3::new(time.elapsed_seconds().cos() * 2.5, 1., 0.),
        Quat::from_rotation_y(PI / 2.),
        Vec2::splat(2.),
        Color::GREEN,
    );
    
    // (後略)
}
```

ここが驚くべきポイントですが、引数には欲しいものを指す**型**を指定すると、自動でその値がバインドされるようになっています。まるで[DI (依存性注入)](https://qiita.com/okazuki/items/a0f2fb0a63ca88340ff6)とも呼べるトリックで、クールですね。

ちなみに`Startup`では基本的に何を行うかというと、Reactなどでいうところの構成要素の登録を行います。いわばHTMLを書いているようなものです。HTMLでいうJavaScriptにあたる、要素の更新は、`Update`に登録した関数群で行います。

この関数群を **System** と総称していますが、何か作業を行うものは基本的にすべてSystemに書きます [^7]。

ちなみにProcessingでいうところの `draw` と違うところは、**描画自体は基本的に自動で行われる**という点にあります [^d]。だから `draw` ではなく `update` であって、updateでは基本的に描画自体ではなくて値などのパラメータや、構成要素だけを変えていきます。ここはReactなどと似ているところです。

### すべてはEntity、属性はすべてComponent、データはすべてResource

そしてここがUnity (厳密には[Unity DOTS](https://unity.com/ja/dots)) と似ている部分ですが、すべての構成要素は `Entity` と呼ばれるものから派生しています。すべては `Entity` (Unityでいうところの `Game Object`) なので、統一的に扱いやすく、さらに、属性はすべて `Component` から派生しています。Unityでもコンポーネントは同じ概念で存在するので、読み替えがわかりやすいですね。

さらに、いわゆる変数にあたる保持しておきたいデータはすべて `Resource` になり、画像や3Dデータなどのアセットは `Asset` です。これもシンプルでわかりやすいです。

そしてこれらをどうやって `System` から取得するかといえば、先ほどと同じく、引数に**型**を宣言すると勝手に取ってきてくれます [^8]。不思議ですね。

```rust
fn check_zero_health(
    // `Health`と`Transform`Componentを持つEntityにアクセスする
    // `Health`は読み取り専用で取得し、`Transform`は変更可能(mut)で取得する
    // オプション: `Player`Componetが存在するなら取得する
    mut query: Query<(&Health, &mut Transform, Option<&Player>)>,
) {
    // 一致する全てのEntityを取得する
    for (health, mut transform, player) in query.iter_mut() {
        eprintln!("Entity at {} has {} HP.", transform.translation, health.hp);

        // HPが0なら、中央に移動
        if health.hp <= 0.0 {
            transform.translation = Vec3::ZERO;
        }

        if let Some(player) = player {
            // ここではEntityは`Player`
        }
    }
}

// ※コードの引用元 (コメント等の解説含む):
// https://xianliang2032.hatenablog.com/entry/2021/09/28/Rust_Bevy_Query%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6
```

今回は紹介に留めておきたいので、**ECS (Entity Component System)** と呼ばれているこのアーキテクチャの紹介は、これくらいにしておきますが、Unityなどと同様に、汎用的な機構を備えていることがわかります。

![UnityとBevyの用語対応表.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/0ee1352e-1d56-13c6-a837-c7437066c31c.png) (図: UnityとBevyの用語対応表)

## その他の特徴: クロスプラットフォーム、低レベル描画エンジンの抽象化などなど (割愛)

![スクリーンショット 2024-01-22 19.50.31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/f9645187-f1da-2efd-c816-287c87b6f0e1.png) [^g]

他にもBevy Engineの面白い特徴はいろいろあり、例えば`wgpu`という仕組みでDirectXやMetal、Vulkan、WebGLなどに一つのコードで自動でクロスプラットフォーム対応できたりしますが、こうしたことはUnity等ではある意味で当たり前のことなので今回は割愛します。

## 最後に: プラグインシステムでどんどん拡張 / 例えばInspectorやGizmo、パーティクル …etc

最後に、プラグインについてちょっと触れておきます。例えば欲が出てくると、Unityのようなエディタがやっぱりほしいな、って思ったりしますよね。こうしたことはだいたいプラグインでできます。

![0f18eacdfb26-20231117.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/69fa08c2-2a41-32da-a3d9-2ef330ed0b7a.png) [^9]

例えば、GPUパーティクルシステムが欲しいと思えば、[bevy_hanabi](https://github.com/djeedai/bevy_hanabi)が使えますし、[bevy-inspector-egui](https://github.com/jakobhellermann/bevy-inspector-egui)などを使うと、さっきより気軽なInspector (操作盤)を表示させたりできます。

![スクリーンショット 2024-01-22 18.28.57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/18ac4ff1-2426-256e-8454-7791a495287b.png)

挙げればきりがないですが、例えばHTMLでUIを書きたい！とか思うと、[belly](https://github.com/jkb0o/belly) というプラグインがそれを実現してくれたりします。

![スクリーンショット 2024-01-22 18.34.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/55fb0ed0-8381-4b43-e34d-6e3a916aeb52.png)


こういう気軽な拡張ができるのが、やはりどんなエンジンでも楽しい部分でもあり、そしてECSアーキテクチャのおかげで、他の `Entity` や `Component` などの構造物と並列して、違和感なく使えるのが嬉しいですね。

**追記:** Bevyのプラグイン一覧は https://bevyengine.org/assets/#assets や https://github.com/topics/bevy-plugin にリストアップされています。

## まとめ: クリエイティブ・コーディングの過去と未来を融合していくと…

さて、今回はUnityやUnreal Engineなどのゲームエンジンにちょっと疲れた人や、プログラミングの授業でProcessingの次に何をしようかと迷っている人などにとって、その間くらいにある「ちょうどいい存在」として、Bevy Engineを紹介してみました。

ちなみに実際に書いてみると、Rustが出すエラーのわかりにくさにきっとビックリすることとは思います 笑 [^10]。

…これは半分冗談で、慣れの部分が大きいですが、Bevy EngineはECSというコンポーネントシステムの恩恵で、Rustの難しい部分である所有権やライフタイムをそこまで意識しなくて済むように工夫されています。ですがきっと今後は、このBevy EngineのようなものがTypeScriptなどの一般的な言語からも利用できるようになって、こういう姿がきっと未来のクリエイティブ・コーディングなんだろうな、と思ったりします。

Bevyはまだ登場して年数が浅いので、例えばp5.jsなどのように気軽に作品をオンラインでシェアするようなプラットフォームとか、アドオンが一覧になったサイトとかは~~ありません~~ (**追記:** [Bevyプラグイン一覧](https://bevyengine.org/assets/#assets)は既に公式にありました！) が、既に[Example](https://bevyengine.org/examples/)はWebGLで対話的に見ることができますし、そういう未来もすぐそこに来ていると思います。

こういうところから少しずつ、クリエイティブ・コーディングの過去と未来のそれぞれの良さが、うまく融合していくといいですね。

<hr>

## 追記・補足など

- wgslというシェーダーだけを気軽に試したいとき (いわゆる[ShaderToy](https://www.shadertoy.com/)的なツール) には、[shadplay](https://github.com/alphastrata/shadplay) が便利そうです。ECSとか気にせずエフェクトだけ試したい場合はおすすめ [^dc]。

[^1]: https://bevyengine.org/examples/2D%20Rendering/bloom-2d/ より引用、再構成。

[^2]: 蛇足ですが、[つぶやきProcessing](https://zenn.dev/kurogitsune/books/d6165036240279) などの文化はワンライナーにも通じるものがあって、興味深いですね。

[^3]: https://forest.watch.impress.co.jp/docs/serial/blenderwthing/1373040.html より画像を引用。

[^4]: https://docs.unrealengine.com/5.0/ja/organizing-a-material-graph-in-unreal-engine/ より画像を引用。

[^5]: 容量について念の為フォローアップしておきますが、容量やスペックというのは相対的なものなので、シンプルさとは何か、の印象については常に変わっていくことと思います。実際、当時のProcessingも300MBくらいあって、容量は小さいとは言い難いものでした。そこでこの記事の文脈では、シンプルさというのは、プラグインシステムや抽象化などを使ってコアをシンプルに保っておくことと仮にしておきます（Unityなども既に充分シンプルであるとは思います）。

[^6]: https://bevyengine.org/examples/3D%20Rendering/3d-gizmos/ より引用、再構成。

[^bash]: 以下、`$` から始まるコマンドはターミナルに打ち込むものを指します。(`$`は省いてそれ以降を入力します。)

[^7]: ちなみにSystemには依存関係を指定したり、セットと呼ばれる組み合わせも作ることができますが、冗長なので今回は割愛します。

[^d]: 注意としては、ECSなどの高レベルAPIから描画エンジン自体(wgpu)の低レベルAPI、そしてその中間に位置する中レベルAPIがバランスよく提供されているので、描画をカスタムできないという意味ではありません。シェーダーパイプラインや、各種バッファーオブジェクトなどはちゃんと操作できるのでご安心を。

[^8]: https://xianliang2032.hatenablog.com/entry/2021/09/28/Rust_Bevy_Query%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6 よりコードを引用。

[^9]: https://zenn.dev/eloy/articles/ea11899ee3dbe4 より画像を引用。

[^10]: 記事中では割愛しましたが、実際にコードを書くときは[VSCodeにrust-analyzerを入れれば](https://qiita.com/simonritchie/items/a8bdcbc65a0ae1861e6b)完璧に補完が効くので、もう準備はバッチリです。Have a happy coding!

[^g]: https://bevy-cheatbook.github.io/gpu/intro.html より表を引用。

[^e]: https://blog.mozvr.com/introducing-ecsy/ より図を引用。

[^dc]: Bevy公式Discord[コミュニティ](https://bevyengine.org/community/)からいただいた情報です。コミュニティの皆さんの支援には本当に感謝します。
