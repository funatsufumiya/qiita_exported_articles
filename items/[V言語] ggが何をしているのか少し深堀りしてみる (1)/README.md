:::note
執筆時点でV言語 0.4.12 です。今後の変更に注意してください。
:::

先日以下のような記事を書いたのですが、今回はもう少し深く中身を見ていこうと思います。

https://qiita.com/funatsufumiya/items/ec1b72aae4deb845d929

たぶん今後も似た記事を書きそうなので「(1)」とつけてますが、予定は未定です。

## 余談: v-analyzer について

ちなみに以下の通りに v-analyzer を入れると、定義ジャンプなどが効くようになるのでコードリーディングに便利です。

https://qiita.com/funatsufumiya/items/4f2824d304f006d3a366

ただし、V言語のexampleは、.v ファイルが並んでいることが多いのですが、v-analyzer は現時点では `module main` を含むファイルが複数あると静的構文解析がエラーするので、フォルダに1つになるように移動したりコピーしたりして工夫する必要があります。

## draw_rect_filled が内部で何をしているのか

単純な四角形だけを描くexampleを準備してみました。

![スクリーンショット 2025-09-24 5.08.52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/e0dd1aea-48be-4a80-9752-8cec0c0ae01a.png)

https://github.com/funatsufumiya/v-lab/blob/c579453b0ca0732a462c1fc1057588d07abf5d64/rectangles/main.v

ポイントは以下の箇所ですね。

https://github.com/funatsufumiya/v-lab/blob/c579453b0ca0732a462c1fc1057588d07abf5d64/rectangles/main.v#L37-L43

`draw_rect_filled` が何をしているのか見てみましょう。

https://github.com/vlang/v/blob/0df443563544c3c35d5c8027d0f3a341f68c1eed/vlib/gg/draw.c.v#L257-L281

`$if macos` のところはmac用の特殊処理のようですね。（`$if`はC言語でいうところの `#ifdef`のようなプリプロセッサで、コンパイル時処理。）

以下の部分が核になっていて、`c4b` = `color 4 bytes` で色を描いて、`v2f` = `vertex 2 floats` で頂点を指定しています。 

https://github.com/vlang/v/blob/0df443563544c3c35d5c8027d0f3a341f68c1eed/vlib/gg/draw.c.v#L273-L280

## gg.begin() が何をしているか

ちなみに頂点などの話をするときに、カメラや行列がどんな設定になっているかが大事になってきますが、`gg.begin()` で何をしているかを見てみると、シンプルな平行投影 (`ortho`) になっています。

https://github.com/vlang/v/blob/0df443563544c3c35d5c8027d0f3a341f68c1eed/vlib/gg/gg.c.v#L600-L608

このコードによると、`sapp.width()` （ `import sokol.sapp` ）で画面の横幅がとれるようで、画面をリサイズしてもちゃんと横幅がとれていました。

また、その後の引数の最後の `-1.0` と `1.0` は、z座標のnearとfarの範囲 (culling) が指定されていて、ggのデフォルトではz座標は-1.0〜1.0までであることがわかります。

例えば、`gg.begin()` の後で以下のように書けば、Z範囲を好きにいじって、自分の好きな3Dを描くこともできますね。

```v
sgl.defaults()
sgl.matrix_mode_projection()
sgl.ortho(0.0, f32(sapp.width()), f32(sapp.height()), 0.0, -10000.0, 10000.0)
```

実際、以下のコードのように描くと、好きな3Dの三角形を2Dに混じって描くことができます。（詳しい解説はしませんが、Z座標を確認するために時間でスイープさせてみたりしています。デフォルトではZ = 0で、背景色もZ = 0なので、それより手前に描くとちゃんと見えます。）

![スクリーンショット 2025-09-24 5.26.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/c30685ab-58f4-46d4-8a9c-25f6d7c879bb.png)

https://github.com/funatsufumiya/v-lab/blob/40fa4918c7a9462c86fa5beeb54e7dc645db45e5/triangle6/main.v

## パイプラインとブレンドモードについて

`draw_rect_filled` が何をしているのかに少し話は戻ります。

https://github.com/vlang/v/blob/0df443563544c3c35d5c8027d0f3a341f68c1eed/vlib/gg/draw.c.v#L257-L281

唯一解説していなかった以下の部分ですが、透明〜半透明のときだけパイプラインを指定していますね。

https://github.com/vlang/v/blob/0df443563544c3c35d5c8027d0f3a341f68c1eed/vlib/gg/draw.c.v#L270-L272

パイプラインの初期化は `init_pipeline()` で行われていて、デフォルトでは `alpha` (アルファ合成用) と `add` (加算合成用) が準備されているようです。

https://github.com/vlang/v/blob/0df443563544c3c35d5c8027d0f3a341f68c1eed/vlib/gg/gg.c.v#L151-L179

`gfx.BlendState` のところは、OpenGLでいうところの glBlendFunc とglBlendFuncSeparate にあたるもので、この例に習って好きなパイプラインを事前に作っておいて `sgl.load_pipeline()` で読み込めば、Photoshop等でよくある好きなブレンドモードを自分で使い分けることができそうです。

https://github.com/vlang/v/blob/0df443563544c3c35d5c8027d0f3a341f68c1eed/vlib/gg/gg.c.v#L172-L176

参考:

https://memo.devjam.net/clip/538

ちなみに `sgl.push_pipeline()` と `sgl.pop_pipeline()` を使うと、パイプラインの状態保存もできるようです。

https://modules.vlang.io/sokol.sgl.html#push_pipeline

## まとめ

今回はこれくらいで終わります。`sokol.sgl` ([vdoc](https://modules.vlang.io/sokol.sgl.html)) にはOpenGL風のAPIが整備されているので、OpenGLに慣れていれば大抵のことは読み替えができるという印象です。

[Sokol](https://github.com/floooh/sokol) は現代的なGPU処理のラッパーなので、OpenGL時代にはなかった用語が多々ありますが、以下の記事にイラスト付きでわかりやすく用語がまとまっているので、理解の参考になるかと思います。

https://alain.xyz/blog/comparison-of-modern-graphics-apis

個人的に、Sokolの情報自体が結構少ないように思うので、ggのようにV言語がラッパーを描いてくれていて、かつvdocなどを整備してくれているのはありがたいですね。C言語等で直接sokolを使いたい場合にも大いに参考になりそうです。
