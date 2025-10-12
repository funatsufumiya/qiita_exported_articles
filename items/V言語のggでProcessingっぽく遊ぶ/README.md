:::note
当記事はGIFアニメーションを多く含みます。自動再生されないときは画像をタップしてください。
:::

![画面収録 2025-09-10 9.34.21.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/5a57e865-cdd8-4aa5-a540-bb27096caa4a.gif)

（↑GIFアニメ）

## V言語について

2021年頃に一度話題になった[V言語](https://vlang.io/)ですが、その時点ではちょっと[誇大広告気味だった](https://zenn.dev/zakuro9715/articles/vlang-from-contributor-perspective)のもあってか、その後下火になりあまり話題にならなくなりました。

2025年になって改めてウォッチしてみると、言語としての安定度も高まってきており [^1]、[GCがデフォルトでオンになっていたり](https://docs.vlang.io/memory-management.html) [^2]、当時OpenGLだけ標準でサポートされていたのが、Metal, D3D11, WebGL2などに対応できる[Sokol](https://github.com/floooh/sokol)ベースの[gg](https://modules.vlang.io/gg.html)というフレームワークができたり、それをベースにした[新たなGUIフレームワーク](https://github.com/vlang/gui)ができたりと、気づかぬうちにいろいろ進化しています。


## gg について

![画面収録 2025-09-10 10.40.52.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/7ec11566-24b0-4fdf-a7db-c4cddd9023b8.gif)

（↑ [examples/gg/stars.v](https://github.com/vlang/v/blob/bae7684276f5e56a9293a38bb286df3644157109/examples/gg/stars.v)、GIFアニメ）

[gg](https://modules.vlang.io/gg.html)は、[Sokol](https://github.com/floooh/sokol)ベースの描画フレームワークで、基本的に2D用ですが、[Sokol GL (sgl)](https://modules.vlang.io/sokol.sgl.html) というOpenGL互換（OpenGL風）レイヤーの上に構築されているので、3Dにも対応できます。基本的にSokolにできることはggでもできます。

なんとなく[p5.js](https://p5js.org/) ([Processing](https://processing.org/)、Proce55ing) に似たAPIになっていて、それが今回の記事の動機になっています。

V言語は[Nim](https://nim-lang.org/)と同じくC言語へのトランスパイラ [^a] で、爆速かつ省メモリなので、ggと合わせて、Processingや[openFrameworks](https://openframeworks.cc/ja/) に代わる（あるいは[LÖVE 2D](https://love2d.org/)の代替など）、新たな存在となる可能性もあるように私は感じています。

## 開発環境について

V言語のインストール自体は簡単で、

```bash
$ git clone --depth=1 https://github.com/vlang/v
$ cd v
$ make
$ ./v symlink
```

ですぐに完了します。makeとありますが、makeは内包されているので何も他にインストールする必要はありません。gccなどの別のコンパイラも基本的には要りません。TCC（Tiny C Compiler）というコンパイラが自動で内包されます。 [^1]

あとは `v` と打つと、インタプリタ（対話環境）で遊べます。

![スクリーンショット 2025-09-10 9.12.05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/69194246-1067-4a15-ac57-37672d15b6d8.png)

`v run hello.v` でファイルを実行することもできます。

IDEがほしいときは、VSCodeに[v-analyzer](https://marketplace.visualstudio.com/items?itemName=VOSCA.vscode-v-analyzer)をインストールすれば、もうバッチリです。

:::note warn
【追記】v-analyzerの正しいインストール方法を誤って記載していたので、詳しくは下記を参照してください。
:::

https://qiita.com/funatsufumiya/items/4f2824d304f006d3a366

![スクリーンショット 2025-09-10 9.18.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/ae3da200-7cf0-4d87-985e-74b2d31c93d3.png)

~~v-analyzerのバイナリのインストールが途中で促されますが、環境によっては失敗してしまうこともあるので、その場合は[v-analyzer/releases](https://github.com/v-analyzer/v-analyzer/releases)からダウンロードして解凍してPATHに配置すればOKです。~~ (先の追記の通り、正しいインストール手順に従ってください。)

（v-analyzer、たまにおかしな現象に見舞われるのですが、そういうときはv-analyzerやVSCodeの再起動をすれば直ります。脚注 [^4] も参考にしてください。）

## Hello World

百聞は一見に如かずなので、Hello Worldを書いてみます。

![スクリーンショット 2025-09-10 9.26.30.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/34d7d941-7e6c-45ef-a624-a172840ab9ed.png)

:::note warn
現時点でV言語はQiitaではシンタックスハイライトされないようです。
:::

![スクリーンショット 2025-09-10 11.28.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/32c3f31a-653e-45f1-80a0-20c4b7f8c416.png)

```v
import gg

const win_width = 600
const win_height = 300

struct App {
mut:
	gg    &gg.Context = unsafe { nil }
}

fn main() {
	mut app := &App{}
	app.gg = gg.new_context(
		bg_color:      gg.white
		width:         win_width
		height:        win_height
		create_window: true
		window_title:  'Hello'
		frame_fn:      frame
		user_data:     app
	)
	app.gg.run()
}

fn frame(mut app App) {
	app.gg.begin()
	app.draw()
	app.gg.end()
}

fn (mut app App) draw() {
	g := app.gg
	g.draw_text_def(10, 10, 'hello world!')
	g.draw_rect_filled(30, 30, 40, 40, gg.blue)
	g.draw_rect_empty(25, 25, 50, 50, gg.black)
}
```

`v run hello.v` で実行することができます。

基本的には最後の `draw()` の中身だけ見ればOKです。

![スクリーンショット 2025-09-10 11.29.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/d775c599-0f39-49a6-8efe-3adc14833c0c.png)

```v
fn (mut app App) draw() {
	g := app.gg
	g.draw_text_def(10, 10, 'hello world!')
	g.draw_rect_filled(30, 30, 40, 40, gg.blue)
	g.draw_rect_empty(25, 25, 50, 50, gg.black)
}
```

ggでは `draw_xxx_filled` で塗りつぶし図形の描画、`draw_xxx_empty` で枠線だけの図形描画になるようです。

draw系関数は他に、ellipse (楕円) / circle (円) / bezier (ベジェ曲線) など豊富に種類があり、楽しそうです。

## マウスに追従させてみる

![画面収録 2025-09-10 9.34.21.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/5a57e865-cdd8-4aa5-a540-bb27096caa4a.gif)

（↑GIFアニメ）

![スクリーンショット 2025-09-10 11.36.10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/12cdf825-584a-4488-bb5b-4a8b71dc9889.png)

```v
fn (app &App) draw() {
	g := app.gg

	x := g.mouse_pos_x
	y := g.mouse_pos_y

	g.draw_text_def(x - 10, y - 20, 'hello world!')
	g.draw_rect_filled(x + 10, y + 10, 40, 40, gg.blue)
	g.draw_rect_empty(x + 5, y + 5, 50, 50, gg.black)
}
```

ノリはProcessingとそっくりですね。

## キーボードを使う

![画面収録 2025-09-10 11.00.12.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/733ded53-570d-4d9c-8e61-500adec1be22.gif)

（↑GIFアニメ）

![スクリーンショット 2025-09-10 11.33.23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/053484d1-ef45-4d9f-aae0-7fff77a26cb0.png)

```v
fn (mut app App) draw() {
	mut g := app.gg
	
	if g.is_key_down(gg.KeyCode.space) {
		g.draw_text_def(10, 110, 'space key press')
		g.draw_rect_filled(10, 10, 100, 100, gg.blue)
	}else{
		g.draw_text_def(10, 110, 'space key release')
		g.draw_rect_empty(10, 10, 100, 100, gg.black)
	}
}
```

こちらもなんとなくProcessingと似ていますね。~~`fn key_pressed()` などのイベント関数はありませんが、必要なら作れそうな雰囲気です~~ (**【追記】** ドキュメントを読んでいると、コールバックはちゃんとありました: [参考](https://modules.vlang.io/gg.html#Config)。`frame_fn` と同じように `keyup_fn` などを設定できます)。

## 回転させてみる

![画面収録 2025-09-10 9.45.55.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/dde94ecd-d2c1-4653-9cd3-240834c6f642.gif)

（↑GIFアニメ）

ggだけでは回転できないので、sgl (Sokol GL) を呼び出します。コードの最上部に `import sokol.sgl` が要ります。

![スクリーンショット 2025-09-10 11.34.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/13860733-8272-445b-b655-30600b9d3d42.png)

```v
fn (mut app App) draw() {
	g := app.gg
    
	r := g.mouse_pos_x
	sgl.translate(200, 200, 0)
	sgl.rotate(sgl.rad(r), 0, 0, 1)
	g.draw_text_def(0, 0, 'hello world!')
}
```

[sglの関数一覧](https://modules.vlang.io/sokol.sgl.html)を見てみるとわかるのですが、できることは基本的にOpenGLと同じなので、例えば `glPushMatrix()` は `push_matrix` であったりと、読み替えができる作りになっているようです。

## まとめ

今回はこの辺で終わりたいと思いますが、[sglのドキュメント](https://modules.vlang.io/sokol.sgl.html)や、[ggのドキュメント](https://modules.vlang.io/gg.html) をざっと眺めてみると、どんなことができるかなんとなくわかるかと思います。

[ggのexamples](https://github.com/vlang/v/tree/master/examples/gg) （V言語インストール時のフォルダに `v/examples/gg` に入っている）には、3Dの例などもあるので、参考にされてみると良いかと思います。（ `v run xxx.v` で実行可能。）

なお、V言語自体の情報は後述のように2021年に流行ったときとだいぶ変わっているので、[英語の公式ドキュメント](https://docs.vlang.io/introduction.html)を参考にされることをおすすめします。

## 補足: V言語の当初との違いについて

補足になりますが、V言語では[グローバル変数はデフォルトでは無効になっている](https://docs.vlang.io/global-variables.html)ことなどもあり、2021年当初は[V言語にはクロージャがない](https://zenn.dev/zakuro9715/articles/vlang-no-closure)というのが懸念となっていたりもしていました。

しかし現時点では[クロージャは既に実装されています](https://docs.vlang.io/functions-2.html#closures)。

![スクリーンショット 2025-09-10 10.37.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/14e2c69a-bcee-4011-ba97-da4de8aa044b.png)

```v
my_int := 1
my_closure := fn [my_int] () {
    println(my_int)
}
my_closure() // prints 1
```

このように、2021年頃にはあった様々な懸念事項の多くが、2025年現在では解消されているので、当時微妙だなと感じた方はまた改めてウォッチされてみると楽しいのではないかと思います。

[^1]: V言語のデフォルト（TCC）では、サードパーティのライブラリを使うときに、C言語のヘッダインクルード時（`#include "xxx.h"`）に正しくパースされないというエラー報告がとても多い印象で、実際にそれで困ることもあるのですが、V言語というよりTCC（Tiny C Compiler）の問題なので、`v -cc clang` や `v -cc gcc` などしてCコンパイラを切り替えれば（`v run`の代わりに`v -cc clang run`など）、比較的安定して使えます。

[^2]: GCは重くなるなどいろんな懸念がある方もいるかと思いますが、[開発者本人も当初はGCに懐疑的だった様子](https://github.com/vlang/v/discussions/17419)で、アンチGCだったらしいのですが、実際に導入してみると実用上全く問題なかったとのことで、導入が決まったようです。（GCはオフにすることもできますし、9割くらいを静的解析で自動開放できる `-autofree` もあります。）

[^4]: v-analyzerは、現時点では、`module main` など、main関数を持つvファイルが**1つのフォルダに複数**あると、アルファベット順で最初のファイル以外はどうやらパースがおかしくなってしまうようです。（サンプルでは複数の.vファイルがそのままフォルダの中にあることも多いのですが…。）

[^a]: Nim言語とV言語の違う点はいろいろありますが、個人的にV言語の出力するC言語のほうが自然で読みやすく、C言語側でトラブルがあった際にデバッグがしやすい印象です。NimのようにC++連携は強くないのですが、`v translate`や`v translate wrapper`で自動的にC言語のラッパーが書けるのは強みで、うまく使い分けていくと良いのかなと思います。
