## 概要

この記事では、[Bevy Engine](https://bevyengine.org/)を活用していく上で、特におさえておくべきポイントをまとめています。実際にはそこからさらに深堀りして調べていく必要があるかと思いますが、入門記事の次の段階として、マイグレーションガイドとしての一助になれば幸いです。（なお執筆時点のBevyの最新版は`v0.13.2`です。）

## この記事の対象読者

- Bevyのことを少し知っていて、実際に活用していきたいなと思っている人
- Rustにあまり馴染みがなくてもOK。一方でC++やJava、JavaScriptやPythonなど、多少プログラミング経験があると望ましい。
- ゲームプログラミング以外でもBevyの使い道はいろいろあると筆者は感じているので、ゲームプログラミングの知識は必須ではありません。
- ECS (Entity Component System) については今回は触れず、解説は他記事に譲ることにします（[→参考記事](https://qiita.com/aobat/items/262293651fbbd696c171)）。

## ポイント1: ハンドル (Handle)

Bevyには、ハンドル(Handle)という概念が頻繁に登場します。このハンドルは、画像や3Dモデルなど、実体は重たいものを、IDなどとして参照したり、軽量に扱うための役割を持ちます。

https://bevy-cheatbook.github.io/assets/handles.html

C++などの経験者は、ポインタや参照 (Reference) を利用すれば良いのではないかと思うことかと思います。実際Bevyでも、参照を利用することはありますが、Rustの所有権システムとの兼ね合いから、あまり参照を多用するとライフタイム表記などでコードが煩雑になる傾向にあり、ハンドルはそうした部分をうまく回避してくれます。

一方で、例えばマテリアル (色情報等) などの実体にアクセスしたいのに、どうしたらいいかわからない、ということも多く発生します。

ハンドルは必ず対応する[アセット](https://bevy-cheatbook.github.io/builtins.html#assets)が存在し、そのアセットから`.get()`または`.get_mut()`することで実体にアクセスすることができます。

https://github.com/bevyengine/bevy/discussions/8487#discussioncomment-5716877

https://stackoverflow.com/questions/75801912/how-to-modify-material-property-in-bevy

```rust
let material = materials.get_mut(my_handle).unwrap();
material.base_color = Color::RED;
```

逆にいえば、実体が不要な処理はハンドルだけで可能です。例えばある場所に画像を表示したい、フォントはこれにしたいなど、多くの処理がハンドルだけで可能です。

ちなみにアセットの読み込みは、`bevy_asset_loader`などのプラグインを活用するとさらに便利です。

https://github.com/NiklasEi/bevy_asset_loader

## ポイント2: クエリ (Query)

Bevyでは、クエリを書くことで欲しいエンティティ (ゲームオブジェクト) に簡単にアクセスすることができます。

https://bevy-cheatbook.github.io/programming/queries.html

例えば `MyComponent` というコンポーネントがついたエンティティのTransform (位置・回転・大きさ) にアクセスしたい場合は、システムの引数に次のように書きます。

```rust
mut transforms: Query<&mut Transform, With<MyComponent>>
```

もし、Transformを変更する必要がない（不変、`immutable`で良い）ならば、次のようにします。[^1]

```rust
transforms: Query<&Transform, With<MyComponent>>
```

さらに、`MyComponent2` もついた要素にアクセスしたいし、背景色にもアクセスしたいなら次のようにします。

```rust
query: Query<(&Transform, &BackgroundColor), (With<MyComponent>, With<MyComponent2>)>
```

さらにさらに、`MyComponent3` は持っていないという条件をつけたい場合は以下です。

```rust
query: Query<(&Transform, &BackgroundColor), (With<MyComponent>, With<MyComponent2>, Without<MyComponent3>)>
```

もうおわかりかと思いますが、左のタプル（カッコ、簡易構造体）にあるものはアクセスしたいもの、右のタプルにあるものは条件です。左のものに`&mut`をつけると可変 (`mutable`) にできます。

```rust
query: Query<(&欲しいものA, &欲しいものB, ...), (With<条件1>, With<条件2>, Without<条件3>, ...)>
```

ちなみに、Rustでは単一所有者の原則から、`Without`をうまく使わないと実行時エラー (= パニック) になってしまうことがあります（[→参考記事](https://zenn.dev/msakuta/articles/40c1ad41b1c62e#component-%E3%81%AE%E5%90%8C%E6%99%82%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9) [^a]）。同じ理由で似たクエリ文を複数の引数として分割するとエラーになってしまうことがありますので、タプルを活用してできるだけ同じ条件のものは同時に取得するのがポイントです。

## ポイント3: イベント (Event)

Bevyは、様々なことをイベントにより処理します。例えばマウスの位置を取得したい場合、ウィンドウにアクセスして位置を取得する方法と、`CursorMoved`などのイベントを受信して読み取る方法があります。

ちなみに他のフレームワークでよくある、イベントハンドラの登録などの作業は特に必要はありません。クエリ(Query)と同じく、引数に`EventReader`などを書けば自動的にバインドされる仕組みになっています。

https://bevy-cheatbook.github.io/programming/events.html

```rust
fn debug_levelups(
    mut ev_levelup: EventReader<LevelUpEvent>,
) {
    for ev in ev_levelup.read() {
        eprintln!("Entity {:?} leveled up!", ev.0);
    }
}
```

なお、`.read()` しているループはイベント待ちで処理がブロックされるとかそういうことはなく、イベントが何もないときは空の配列が渡されると思えば良いです。1フレームに複数のイベントが起きる場合があるのでイテレータになっています。なのでイベント処理以外 (`Update`など) と同じシステム上に書いて問題ありません。

ちなみに、非公式チートブックにも書いてあるように、カスタムイベントを登録するのも簡単です。

```rust
#[derive(Event)]
struct LevelUpEvent;

// (中略)

app.add_event::<LevelUpEvent>();
```

例えばサウンド再生など、複数のシステムをまたぐような処理は、イベント化してしまうと楽です [^2]。

## ポイント4: ローカル (Local) と リソース (Resource)

Bevyで、システム内だけでいわゆる`static`な変数がほしいと思うとき、`Local`を利用します。複数のシステムで共通の変数や定数がほしい場合は、`Resource`を利用します。

https://bevy-cheatbook.github.io/programming/local.html

https://bevy-cheatbook.github.io/programming/res.html

例えば、あるシステム内で処理するたびに前の値を残しておき、次の処理に使いたいようなときは `Local` が最適です。一方で例えばプレイヤーの状態など、ゲーム内で共通して使う情報は `Resource` が最適です。

```rust:Localの例
fn my_system (
    mut my_time: Local<f32>,
    time: Res<Time>,
) {
    // 毎回経過時間を記録しておく、など
    *my_time += time.delta_seconds();
}
```

```rust:Resourceの定義例
#[derive(Resource)]
struct GameProgress {
    game_completed: bool,
    secrets_unlocked: u32,
}

#[derive(Resource)]
struct StartingLevel(usize);
```

```rust:Resourceの使用例
fn my_system(
    mut goals: ResMut<GoalsReached>,
    other: Res<MyOtherResource>,
    mut fancy: Option<ResMut<MyFancyResource>>,
) {
    // 訳注: fancyが存在していたらifの中に入る
    if let Some(fancy) = &mut fancy {
        // TODO: do things with `fancy`
    }
    // TODO: do things with `goals` and `other`
}
```

ちなみになぜ`Local`という、他のプログラミング言語では馴染みのない型がBevyに準備されているかというと、一つはRustでは`static`な変数を作るのに苦労するという背景があります。また、Bevyではシステムは並列処理される可能性があるため、安易に`static`な変数を作ると`Mutex`などの排他処理にハマる可能性もあります。

無闇にグローバル変数を増やさないためにも、`Local`をうまく使っていくと、システム固有の変数管理が楽になるはずです。

## ポイント5: ステート (State)

Bevyにはステート (State) というものが準備されていて、特に状態に応じてシステムを切り替えたい場合に便利です。

https://bevy-cheatbook.github.io/programming/states.html

```rust
// メニューがメイン状態のときだけ実行されるシステム
app.add_systems(Update, my_system1.run_if(in_state(MenuState::MainMenu)));
// アプリがゲーム中のときだけ実行されるシステム
app.add_systems(Update, my_system2.run_if(in_state(AppState::InGame)));
```

リソース (Resource) だけでもいろんなことができますが、特にステートはシステムまわりで便利に活用でき、例えばあるステートのときだけ特定のシステムを走らせたいなどを容易に書くことができます。

実行スケジュールについては、ポイント8で詳しく解説します。

## ポイント6: タイマー (Timer)

時間経過による処理を扱いたい場合は多くあると思いますが、例えば経過時間などを知るための `Time` と並んで便利なのがタイマー (Timer) です。

https://bevy-cheatbook.github.io/fundamentals/time.html

```rust:タイマーの使用例
#[derive(Resource)]
struct BombsSpawnConfig {
    timer: Timer,
}

// ゲーム起動時のセットアップ
fn setup_bomb_spawning(
    mut commands: Commands,
) {
    commands.insert_resource(BombsSpawnConfig {
        // 10秒で繰り返すタイマーを設置
        timer: Timer::new(Duration::from_secs(10), TimerMode::Repeating),
    })
}

// 時限爆弾をスポーン(生成)させるシステム
fn spawn_bombs(
    mut commands: Commands,
    time: Res<Time>,
    mut config: ResMut<BombsSpawnConfig>,
) {
    // 10秒で繰り返すタイマーを進める (tick)
    config.timer.tick(time.delta());

    // もしタイマーが終わったら
    if config.timer.finished() {
        commands.spawn((
            FuseTime {
                // 5秒の時限爆弾タイマーをスポーン(生成)させる
                timer: Timer::new(Duration::from_secs(5), TimerMode::Once),
            },
            // ...
        ));
    }
}
```

注意点は、タイマーは`tick`させないと時間経過しない点です。

タイマーは指定時間のどれくらいが経ったかを比率で取得できたり (`fraction()`)、残り時間を取得したりが便利にできます。

タイマーをうまく活用すると、複数のシステムで時間共有したりできますが、`tick`やスポーン(生成)・デスポーン(削除)の管理が一つポイントになります。

## ポイント7: スポーン (spawn) と デスポーン (despawn)

Bevyには `Commands` というものがあり、`Commands`を使ってエンティティのスポーン(作成)とデスポーン(削除)を行います。

https://bevy-cheatbook.github.io/programming/commands.html

注意点は、コマンドの実処理は最大で1フレームの遅延が起こることです。コマンドにキューを登録して、その処理がすぐに行われるとは限りません。なので例えば、エンティティの存在を仮定して行うような処理は、スポーン処理とは別システムで行うなどする必要があります。

ちなみに、このような時間的な前後関係を考えていくと、Unityでいうところの[コルーチン](https://docs.unity3d.com/ja/2018.4/Manual/Coroutines.html)がほしくなることがあります。これに該当するのは`await`/`async`を使った非同期処理です。ゆくゆくは公式APIが便利になると思いますが、執筆時点では`bevy_flurx`や`bevy_tweening`などのアドオンを使うと便利です。

https://github.com/not-elm/bevy_flurx

https://github.com/djeedai/bevy_tweening

例えば`bevy_flurx`を使って、1秒後に `Hello` と出力するには次のようにします。

```rust
commands.spawn(Reactor::schedule(|task| async move {
    task.will(Update, {
        delay::time().with(Duration::from_secs_f32(1.0))
            .then(once::run(|| {
                println!("Hello");
            }))
    }).await;
}));
```

https://zenn.dev/elm/articles/34d89e52839715

他にも`Commands`によるスポーン・デスポーンを活用すると、アイディア次第で単一システムの枠を超えたいろんなことができます。特にエンティティ以外にも、リソースの追加や削除も`Commands`からできるので、可能性は無限大です。

## ポイント8: システムの実行スケジュール

Bevyのシステムは、基本的に `Startup` と `Update` の大きく2種類のスケジュールで実行することができ、`Startup`に初期セットアップ（スポーンなど）、`Update`に毎フレーム処理したい内容を登録します。

https://bevy-cheatbook.github.io/programming/schedules.html

```rust:システムの実行スケジュールの例
// 毎フレーム実行したい処理 (Update)
app.add_systems(Update, camera_movement);

// 起動時に実行したい処理 (Startup)
app.add_systems(Startup, setup_camera);
```

`PreUpdate`や`PostUpdate`などの細かな制御も可能ですが、実際にはそれよりも、前述のステートと連動した、`OnExit(State)`と`OnEnter(State)`のほうがよく利用されます。この2つは指定のステートに状態が切り替わったとき、状態が変化するときに呼ばれます。

例えばよく使うのは、アセットがすべて読み込まれたあとに何かを処理する場合で、[bevy_asset_loader](https://github.com/NiklasEi/bevy_asset_loader)などのプラグインを使うと、そのあたりの実行スケジュール・ステート管理が楽にできます。

```rust:bevy_asset_loaderを使った、アセット読み込みの例
app
    // ステートを登録
    .init_state::<MyStates>()
    .add_loading_state(
        LoadingState::new(MyStates::AssetLoading)
            // 読み込み終わったら AssetLoaded ステートにする
            .continue_to_state(MyStates::AssetLoaded)
            // オーディオ関係のアセットを読み込み
            .load_collection::<AudioAssets>(), 
    )
    // 読み込みが終わったタイミングで、BGMを再生するシステム
    .add_systems(OnEnter(MyStates::AssetLoaded), start_background_audio)
```

また、実行スケジュールとは書き方は異なるものの、エンティティやコンポーネントの追加や削除、変更のタイミングを検知して何かを実行することも可能で、`Added<T>`や`Changed<T>`をクエリ内に書くことで、イベントみたいにトリガーすることができます。

https://bevy-cheatbook.github.io/programming/change-detection.html

```rust:変更検知の例
fn debug_stats_change(
    query: Query<
        // コンポーネント
        (&Health, &PlayerXp),
        // フィルター
        (Without<Enemy>, Or<(Changed<Health>, Changed<PlayerXp>)>), 
    >,
) {
    for (health, xp) in query.iter() {
        // 変更時のみ実行される
        eprintln!(
            "hp: {}+{}, xp: {}",
            health.hp, health.extra, xp.0
        );
    }
}
```

## ポイント9: UIと2Dの違い

Bevyでは、UI処理と2D処理は分かれています。それぞれに似た[バンドル](https://bevy-cheatbook.github.io/programming/bundle.html)が準備されていて、例えばテキストはUIでは`TextBundle`、2Dでは`Text2dBundle`を使います。

UIと2Dの一番の違いは、Flexboxがレイアウトに使えるか否かですが、その他にも、2Dを前提にしたアドオン、UIを前提にしたアドオンなど、どちらを利用するかで使えるアドオンも変わってきたり、マテリアルの処理などに違いがあります。

Flexboxは非常に柔軟なレイアウトができる一方で、CSSなどと同じく難しい部分もあり、シンプルな2Dとどちらを使うべきかはケースバイケースです。基本的にはゲームオブジェクトは2D、メニューなどはUIと分けておくと良いかと思いますが、Bevyのバージョンを重ねるごとにUIもカメラドリブンになっているので、両者を混在させた使い方もできたりします。

## ポイント10: プラグイン (Plugin)

ふつうプラグインというと、外部のアドオンのようなものをイメージすると思いますが、BevyでのPluginは、システムやリソースの登録などをまとめるためにも使えます。（もちろん外部のプラグインを利用する場合にも使います。）

https://bevy-cheatbook.github.io/programming/plugins.html

なので、ある程度の機能別にプラグインにまとめるようにしておくと、関連するリソースやシステム登録を一箇所のソースにまとめることができるので、見た目上も綺麗になります。

```rust
struct MyPlugin;

impl Plugin for MyPlugin {
    fn build(&self, app: &mut App) {
        app.init_resource::<MyOtherResource>();
        app.add_event::<MyEvent>();
        app.add_systems(Startup, plugin_init);
        app.add_systems(Update, my_system);
    }
}
```

こうして作ったプラグインを、別のゲームなどで活用することも簡単にできますし、一つのアドオンとして分離したり、クレートとしてライブラリに公開することもできます。

プログラムをパーツに分けていく利点は再利用の面でとても大きいと感じていて、ECSの大きな恩恵はこの「モジュラー」な部分にあると思います。

## ポイント11: ギズモ (Gizmos) や egui などの即時描画

Bevyは基本的にはUnityなどと同じく、エンティティやコンポーネントなどの構造体を制御して描画をコントロールしていきますが、即時描画も可能です。その代表格がギズモ (Gizmos) と egui (immediate-mode GUI)です。

https://bevyengine.org/examples/2D%20Rendering/2d-gizmos/

https://bevyengine.org/examples/3D%20Rendering/3d-gizmos/

https://github.com/mvlabat/bevy_egui

特にデバッグなどの試行錯誤において、即時描画はとても便利です。C++やOpenGLでもImGuiという、eguiに似た即時描画できるGUIがよく使われていますが、eguiは同じように使えますし、他にも`bevy-debug-text-overlay`など、即時描画できるものをいくつか知っておくと、デバッグと試行錯誤が非常に楽になると思います。

https://github.com/nicopap/bevy-debug-text-overlay

ちなみに執筆時点ではまだ開発段階ですが、クリエイティブ・コーディング向けフレームワークである[nannou](https://nannou.cc/)がBevyプラグインとして生まれ変わると、さらなる即時描画APIの選択肢が増えることになるので、こちらも将来性が楽しみです。（Unity×Processing = Unicessingと似たノリの使い方ができるようになるのでは。）

https://github.com/nannou-org/nannou/issues/953


[^1]: 可変・不変・定数の違いはJavaScriptの`var`,`let`,`const`の違いと似ていると考えて差し支えありませんが、Rustの定数 (`const`) は実行時に変更できない点が違っていて、その意味ではJavaScriptの`const`は不変（イミュータブル）に近いです。また、Rustでは`mut`を多用すると所有権まわりのエラーに悩まされがちなのもあり、できるだけ`immutable` (不変) にすることが望まれます。この辺は[参照透過性](https://ja.wikipedia.org/wiki/%E5%8F%82%E7%85%A7%E9%80%8F%E9%81%8E%E6%80%A7)を重視する[関数型言語](https://ja.wikipedia.org/wiki/%E9%96%A2%E6%95%B0%E5%9E%8B%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0)らしい部分でもあります。

[^2]: [ワンショットシステム](https://github.com/bevyengine/bevy/blob/main/examples/ecs/one_shot_systems.rs)を使うというのも一つの手ですが、`SystemId` を管理しなければならなかったりするので、特定用途以外はイベントが簡単で便利だと思います。

[^a]: [[Rust] Bevyのはまりどころ - Componentの同時アクセス](https://zenn.dev/msakuta/articles/40c1ad41b1c62e#component-%E3%81%AE%E5%90%8C%E6%99%82%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9)
