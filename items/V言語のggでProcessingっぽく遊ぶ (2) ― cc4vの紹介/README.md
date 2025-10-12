https://qiita.com/funatsufumiya/items/ec1b72aae4deb845d929

上記記事の続きです。

## ggをさらにProcessingっぽくしてみる

[前の記事](https://qiita.com/funatsufumiya/items/ec1b72aae4deb845d929)で、[gg](https://modules.vlang.io/gg.html) がなんとなく[Processing](https://processing.org/)っぽく使えると書きました。

似ているなら寄せてしまえばいいんじゃないかと思い、[openFrameworks](https://openframeworks.cc/)と[Ebitengine](https://ebitengine.org/)のソースを参考にしつつ、ProcessingのAPI設計に寄せたAPIを作ってみました。

https://github.com/cc4v/cc4v

## Hello World (Processing風)

[cc (cc4v)](https://github.com/cc4v/cc4v) はまだまだ開発中なので、今後APIが変わっていく可能性がありますが、Processing風の使い方と、openFrameworks風の使い方の二種類を用意しています。

どんなAPIがあるかは [docs/coverage.md](https://github.com/cc4v/cc4v/blob/main/docs/coverage.md) を見てもらうとわかりますが、Processing風のHello Worldは以下になります。

![スクリーンショット 2025-10-04 12.06.31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/f6e33f7e-8cc8-455a-acaf-70d2ebbc3448.png)

https://github.com/cc4v/cc4v-examples/blob/86841c7801312f7d637ee82cd0bc6e914bc89ce0/hello_world/main.v#L5-L13

ggに比べて、だいぶ簡単になったことがわかるんじゃないかなと思います。（参考: ggの[minimal example](https://github.com/vlang/v/blob/396ef5c9aa11ae0c6595af6c0172c1c060e09b8a/examples/gg/minimal.v)）

（※ draw関数がなぜ `draw()` ではなく `draw(_ voidptr)` なのかは、記事を最後まで読めばわかります。）

さらに、円を表示するサンプルは次のようになります。

![スクリーンショット 2025-10-04 12.06.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/e0a9424b-47c1-44b5-9b4a-0e10bf493cc3.png)

https://github.com/cc4v/cc4v-examples/blob/86841c7801312f7d637ee82cd0bc6e914bc89ce0/tests/circle/main.v#L5-L22

cc (cc4v) では、すべてを `cc.xxx()` とシンプルに書けるようにちょっとしたトリックを使っていて、隠された動作が極力ないようにする（= 可読性を最優先する）V言語の哲学とは若干背反してしまっているのですが、gg の覚えにくい部分を極力排除するには仕方ないかなと思っています。

:::note
ちなみに、cc (cc4v) は、openFrameworks（およびEbitengine）のソースコードを基本的に参考にしていて、Processingは[API Reference](https://processing.org/reference/)にできるだけ互換性を保つようにしているだけで、`fill()` `no_fill()` などの哲学はopenFrameworksに寄っています。

※ MPL-2.0ライセンスで公開しているのは、Processingコミュニティへの敬意です。コードベースはあくまでopenFrameworks / Ebitengine 準拠です。
:::

## openFrameworks風に使う

先程はProcessing風のAPIでしたが、一応openFrameworks風のものも用意しているのでご紹介します。

![スクリーンショット 2025-10-04 12.10.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/080a51c1-c7ac-4d9c-a11a-a571e292d44c.png)

https://github.com/cc4v/cc4v-examples/blob/86841c7801312f7d637ee82cd0bc6e914bc89ce0/hello_world3/main.v#L5-L55

このサンプルでは、毎フレームごとに `count` が追加されていくものになっています。

実は現時点でまだすべての関数 (例えば `windowResized`) を実装しているわけではないのですが、 `on_event(event &gg.Event)` が存在するので、[gg の examples](https://github.com/vlang/v/tree/master/examples/gg) を参考に好きなイベントをトリガーすることができますし、cc (cc4v) は gg のラッパーでしかないので、以下のように gg を直接呼び出すこともできます。

![スクリーンショット 2025-10-04 13.11.39f.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/fa965718-7694-42d5-9673-a6ce2c250c0d.png)

https://github.com/cc4v/cc4v-examples/blob/86841c7801312f7d637ee82cd0bc6e914bc89ce0/gg/main.v#L5-L15

## Ebitengine風の just_pressed

cc (cc4v) では、[Ebitengine](https://ebitengine.org/)のような[IsKeyJustPressed](https://pkg.go.dev/github.com/hajimehoshi/ebiten/v2/inpututil#IsKeyJustPressed) 風のAPIも利便性のために用意しています。

cc4vでもProcessingやopenFrameworksのように `key_pressed` や `mouse_pressed` などのコールバックを定義できるようにしているものの、V言語はGo言語とよく似ていて、コールバックが多いとコードが多くなる傾向があって、個人的にはこちらのほうが簡単です。

![スクリーンショット 2025-10-04 12.18.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/91b01c28-ba37-4e2a-b999-fc375418abf1.png)

https://github.com/cc4v/cc4v-examples/blob/86841c7801312f7d637ee82cd0bc6e914bc89ce0/tests/key_just_pressed/main.v#L5-L24

これにより、記述量を減らしながらやりたいことをスクラッチでサッと書くということができるかなと思います。

## グローバル変数っぽいものの扱い

最後に、V言語は実はデフォルトではグローバル変数が使えなくて、[`-enable-globals` オプションを有効にすることで使えるようになる](https://docs.vlang.io/global-variables.html)のですが、ユーザが自由に選べるように、グローバル変数を使わなくても良いようにする工夫をしています。[^1]

https://github.com/cc4v/cc4v-examples/blob/86841c7801312f7d637ee82cd0bc6e914bc89ce0/tests/image/main.v#L5-L27

ちなみにこれはProcessing風APIだけで、openFrameworks風APIの場合は、openFrameworksでもそうであるように、Appに直接変数を定義すればOKです。

※ 冒頭部にちょっと書いていた、draw関数がなぜ `draw()` ではなく `draw(_ voidptr)` （`voidptr`は汎用ポインタ）だったのかはこのためで、V言語ではポインタの型（参照型）は関数呼び出し時にキャストされるという性質を使っています。

この書き方が面倒に感じる場合は、オプションでグローバル変数を有効化して使っていただいても構いません。

## まとめ

今回はだいぶ早足でしたが、[gg](https://modules.vlang.io/gg.html) をよりProcessingやopenFrameworksっぽく使う方法をご紹介しました。

V言語はC言語にトランスパイルされるので、openFrameworks並に高速で動作し、C/C++のAPIも簡単にラップまたは変換することができ、それでありながらProcessingのようにメモリ管理などをあまり気にすることなく、やりたいことを書くことに専念することができます。[^2]

個人的には夢のような言語だなと感じているのですが、まだアーリーステージであることから、ggやV言語の仕様も年々変更されていく可能性があり、安定していないので、あくまでホビーユースで楽しんでもらえるといいかなと思います。

ちなみに [cc (cc4v)](https://github.com/cc4v/cc4v) は MPL-2.0 ライセンスで公開しています（exampleの利用等には一部例外規定を設けています）。PR (Pull Request) などは随時歓迎しています。

今後、3Dやシェーダーなどに関しても便利な関数を追加していければと思っていますが、時間があるときに少しずつ拡充しているのであまり期待しないでもらえればとは思います。

gg 自体、ラップせずとも十分にコードが書きやすいAPI設計だと感じているので、ライセンス等気にせず好きに書きたい方は直接 gg を使うのも楽しいと思います。

[^1]: お気づきの方もいるかもしれませんが、cc 自体、内部でグローバル変数の "ようなもの" を使っているおかげで簡便に書けるようにしていて、グローバル変数オプション (`-enable-globals`) を使わなくても良いトリックを使っているので通常時でも問題なく利用できます。（トリックと言っても、言語仕様に即しているので心配はありませんが、今後のV言語の動向には注意が必要かもしれません。）

[^2]: 余談として、[Nelua](https://nelua.io/) など、最近この系統の言語が増えている気がする。若干乱立気味な感は否めないけれど、いずれはどれかに人気が集まるのだろうか…？
