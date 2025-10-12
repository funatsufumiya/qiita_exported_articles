:::note warn
【追記】以下のPRによりレンダラーが[Paranoixa](https://github.com/astomih/paranoixa)に変更されたようなので、バックエンドにSDL GPUおよびDX12、WebGPUが追加された。これについても今後追記していきたい。

https://github.com/astomih/sinen/pull/24
:::

本当は日中に執筆しようと思ったのだけれど、今から紹介する[Sinen](https://github.com/astomih/sinen)が、MoltenVKによって[Macに対応](https://github.com/astomih/sinen/issues/14)したことによって、Win/Mac/Linuxのクロスプラットフォーム対応になったことにワクワクしてしまって寝付けないので、思い切ってその勢いで記事を書きたい。

https://github.com/astomih/sinen

Sinenは、Vulkanをベースにしたクリエイティブコーディングフレームワークで、2D/3Dに対応し、フォントの読み込み、GLTFモデルの読み込みなど、基本的な機能は一通り揃っていて、シンプルでとても使いやすい印象を持つ。言語はLuaとC++に対応している。

2021年から開発が進められていた様子で、当初はOpenGLを使っていたのが、2023年頃にVulkanレンダラーに刷新された様子。ちなみに作者によると、[Paranoixa](https://github.com/astomih/paranoixa)という新しい描画基盤(RHI)を開発中とのことで、これにも期待していきたい。

https://github.com/astomih/paranoixa

## Hello World (2D)

```lua
local hello_texture = {}
local hello_font = {}
local hello_drawer = {}

-- Create a texture
hello_texture = texture()
-- Create a draw2D
hello_drawer = draw2d(hello_texture)
-- Create a font
hello_font = font()
-- Load a font from file(96px)
hello_font:load("mplus/mplus-1p-medium.ttf", 96)
-- Render text to texture
hello_font:render_text(hello_texture, "Hello Sinen World!",
    color(1, 1, 1, 1))
-- Set scale to texture size
hello_drawer.scale = hello_texture:size()

function update()
    -- Draw
    hello_drawer:draw()
end
```

![hello_world.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/ef8abca5-2812-1b07-df11-403d1fb6fbfd.png)

## 3D Example (works)

https://github.com/astomih/sinen/tree/main/works

![スクリーンショット 2025-01-09 21.26.06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/8d7c6786-f734-9eca-91eb-700aeb813b0b.png)

## Editor (Experimental)

https://github.com/astomih/sinen/tree/main/editor

![editor_sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/460bdb92-c1fa-bcc1-bf02-7d1f3ae556a2.png)

## API

Lua APIとC++ APIは、以下より確認が可能。

https://github.com/astomih/sinen/blob/main/docs/docs/lua_api.md

https://astomih.github.io/sinen/doxygen/annotated.html

## 解説

APIやサンプルを見ていただければ分かると思うが、個人的には、クリエイティブコーディングの基盤としてちょうど良いミニマル具合[^1]。というか、まさにこういうものがほしかったという印象を持っている。

~~現時点ではまだ、メッシュなどの中間レベルのAPIが存在していないため~~ (現時点でVertexArray等でメッシュ等に対応済みの様子なので、追って追記。) 例えば、球体や立方体を描くにはGLTFなどのモデルのロードが必要で、円や長方形を描くには、画像を読み込むかImGuiなどのGUIを利用するしかないのだけれど、個人的には正直それで十分な気がしていて、Vulkan APIが使えるのでそのあたりは今後簡単に対応・拡張していけるのではないかと感じている。

3Dを扱う上では、まだライティングに関するAPIが本当に最低限しかないものの、シェーダには対応しているので、これも必要に応じて拡張していけると思うし、3D Example (works) のスクリーンショットを見ていただければわかるように、現時点でもいわゆるUnlit風な描画であれば特に問題なく使っていくことができる。

OpenGLからVulkanになって以降、ゲームエンジンのような厚いレイヤーのものは多くあるものの、こういうミニマルなものが少なかった印象なので、[Processing](https://processing.org/)や[OpenFrameworks](https://openframeworks.cc/)、[Love2D](https://love2d.org/) [^2] のようなものが欲しかったという層にはまさにクリーンヒットするフレームワークではないだろうか。

~~前述のメッシュやライティングについては、個人的に今後コントリビュートしていきたいという思いがあるので、是非今後に期待してほしいと思う。~~ (現時点でメッシュは対応済みの様子なので、使い方について追記していきたい。)

[^1]: 方向性は少し違うものの、ミニマルという意味では、Clojure/LWJGL3/BGFXで書かれた[minimax](https://github.com/roman01la/minimax)というゲームエンジンと近しいようにも思う。ちなみにminimaxは現時点でMacでしか動かないが、ライブラリ自体はクロスプラットフォーム対応のはずなので、こちらも今後自分自身がコントリビュートしてWin/Linuxでも動くようにしたいなと思っている。
[^2]: Love2Dに関しては、最近同様のAPIでVulkan実装の3D対応版（[LÖVR](https://lovr.org/)）がリリースされているので、こちらも比較対象とすると良いと思う。 https://lovr.org/
