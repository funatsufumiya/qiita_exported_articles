最近自分が積極的にウォッチしている、[RavEngine](https://github.com/RavEngine/RavEngine)というゲームエンジンと、その描画基盤の[RGL](https://github.com/RavEngine/RGL/)について、自分の思考整理も兼ねて情報をまとめておきたいと思う。

ただし、自分はコントリビューターでも何でもないため、おそらく多くの語弊や誤解があることを先に注意しておきたい。

## 動画や画像

文字よりも動画のほうがわかりやすいという方は、以下の[紹介動画](https://www.youtube.com/watch?v=J84grfCVIIA)がわかりやすい（ただし英語）。

<iframe width="560" height="315" src="https://www.youtube.com/embed/J84grfCVIIA?si=L7KXRVftYDr01nLn" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


また、いくつかRavEngineサンプルのスクリーンショットを先に掲載しておく。

![スクリーンショット 2025-01-08 18.07.16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/b6ce892e-4d6e-6683-097f-7cd225d525f5.png)

![スクリーンショット 2025-01-08 18.06.49.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/5f08c878-7ed5-8d00-863a-266fd2e2be43.png)

## RGL

https://github.com/RavEngine/RGL/

RGLは、MetalライクなAPIを基調に、DirectX12 (以下DX12) やVulkanにも対応範囲を拡大させ、クロスプラットフォームな描画基盤となるもの。

主にRavEngine向けに作られているものの、RavEngineとは独立していて、例えば[BGFX](https://github.com/bkaradzic/bgfx)のように自分の独自のフレームワークに組み込んで使うことも可能。

特にBGFX等と比較した印象では、実装がスッキリしていて読みやすく、シェーダ言語についてもデフォルトでマクロのないプレーンなGLSLが使えるというのは魅力。

実装がシンプルなのは、OpenGLやDirectX11などをバッサリと切り捨てている[^1]ためだと考えられる。そのおかげもあってか、後述のRavEngineも含めて、DX12/Metal/Vulkanのラッパーとして考えたときにレイヤーが非常に薄く、考え方によっては自分でもカスタムできるかもしれないという魅力がある。

前述の通りBGFXなど似たような描画基盤は複数存在するものの、それらのなかでも特筆して実装がシンプルな印象で、ゲームエンジンの基盤に留まらず、自分自身でフルスクラッチのコードを書いていく際にも使えるのではないかという感覚がある。これはおそらく、RavEngineを拡張していく際にも有利に働くのではないだろうか。

## RavEngine

https://github.com/RavEngine/RavEngine

前述のRGLに加えて、ECS (Entity Component System) や、PBR (Physically Based Rendering) のサポートなど、ゲームエンジンとしての上位レイヤーを加えたもの。

語弊を恐れずに書くならば、運営規模は全然違うものの、[Bevy Engine](https://bevyengine.org/)のC++版という印象がある。

Bevyとの違いを書くとすれば、Bevyは基本的には[wgpu](https://github.com/gfx-rs/wgpu)というFirefox由来の描画基盤を使っているのに対し、RavEngineは前述のRGLを基盤としていることから、この違いが鮮明となっている。

ただしBevyは、[Tiny Glade](https://store.steampowered.com/app/2198150/Tiny_Glade/)というBevyで作られた有名なゲームが、[bevy_render](https://crates.io/crates/bevy_render)を使わずに別のVulkanレンダラ([ash](https://github.com/ash-rs/ash))を使っていることでも知られているように、上位レイヤーと下位レイヤーを独立させている。

一方でRavEngineはRGLに実装を依存させていることが、個人的には逆にシンプルだと感じる。C++には既に多くのECS実装など上位レイヤーの代替候補は多くあるなかで独自にECSを実装しているのは、並列処理などの目的があってのことと推察する。

また、RGLと同じく、基本的に薄いレイヤーで構成されている。Bevyをはじめとして、最近のゲームエンジンは非常に厚いレイヤーでフレームワークが構成されていることが多く、その分だけどんな環境でも安定動作したり、抽象化されている安心感はあるものの、時には自分で下位レイヤーもいじりたいと思うことがある。これは特に、DX12/Metal/Vulkanの最新機能などをすぐ使いたいときなどに有利に働くのではないかと思う。

## ビルドシステム

RavEngine/RGLは、CMakeをビルドシステムに採用している。最近のフレームワークはビルドシステムが多様にあるなかで、標準的なCMakeが選ばれていることは個人的には評価したい。

その恩恵というべきか、他の既存のCMakeを使ったプロジェクトに組み込むのはさほど難しくない印象。

課題としては、基本設定通りにすると依存ライブラリのすべてをビルドさせ[^1]、スタティックリンクさせてしまうので、ビルド時間が長くなり、メモリも多く必要となる。（[READMEの要求スペック](https://github.com/RavEngine/RavEngine?tab=readme-ov-file#system-requirements-for-developers)が高いのはこのため）。これについては対応策は多くあると思うので、CMakeなどをどう活用していくか次第ではないかと思う。

（Bevyが使っているRust/Cargoのエコシステムでも、必要なライブラリは基本的に一から全てビルドするので、ビルドの速さの違いや、賛否両論はあるかもしれないけれど、安定してビルドできるという面では良いかもしれない。）

## コミュニティ

コミュニティについては現時点ではさほど大きくない印象で、[GitHub Discussions](https://github.com/RavEngine/RavEngine/discussions)を中心に利用している様子。

プラグインシステムなどについてはまだ未調査なものの、ECS自体がかなり拡張性を持っていることや、一般的なCMakeなどを使っていることから、拡張性は高いのではないかと思う。

ECSについて一般的にいわれていることとして、プラグインなどはそのフレームワークの専用の実装になりがちで、RavEngineは特にRGLに依存していたり、ECSが独自実装であることから、プラグイン等については[Bevy Assets](https://bevyengine.org/assets/#assets)のような感じで何かしらまとめてあったり、GitHubのタグ（トピック）があると嬉しいような気もする。（詳しくは今後調査。）

## まとめ

まだまだアーリーステージなプロダクトという印象はあるものの、現時点で非常に斬新でシンプルな実装が際立つので、個人的には現時点でも積極的に使っていきたいという印象を持つ。

ただし、OpenGLやDirectX11などの旧来のAPIはサポートされていないので、動作環境はかなり限られている（新しいPCやスマホに限定される）ことに注意したい[^2]。

サンプル等のビルドについては、いくつかハマりどころがあるので、それについては[別の記事](https://qiita.com/funatsufumiya/items/632d27454bdf51dace9b)などでまとめていければと思う。

## 追記: SDL3との関連性

執筆時点で、[SDL3](https://wiki.libsdl.org/SDL3/FrontPage)の正式リリースが間近になっていることもあり、SDL3について追記しておきたい。

SDL3の[新しいGPU API](https://wiki.libsdl.org/SDL3/CategoryGPU)は、RGLの向かっている方向性と（シェーダ言語の扱いを除いて）とてもよく似ていると私は感じている。（先取りしているという見方もできるかもしれない。）

おそらく[作者]()もそれは意識していると思われ、（SDL GPUではないものの）作者自身が[sdl3-sample](https://github.com/Ravbug/sdl3-sample)を更新していたり、RavEngine自体にもSDL3が一部既に使われていたりする。

SDL GPUについて作者自身がどう考えているかは情報が少なくてわからないが、今後SDL3が正式リリースされると、また動向が変わってくる可能性があるため、しばらくは破壊的変更などに注意が必要かもしれない。

（ただ、SDL3は最初のリリースではWebGPUなどをサポートしていなかったり、シェーダ言語に対する方向性の違いがあったりするので、RGLはおそらくそのまま開発が継続し、SDL3はうまく取り込んでいく可能性が大きいと予想。）


<iframe width="560" height="315" src="https://www.youtube.com/embed/UFuWGECc8w0?si=lbI195fkrtb7ks6h" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


[^1]: 余談として、RavEngineのCMakeのビルド過程で、関連ライブラリをまとめてビルドすることから、依存関係にあるSDLやOpenXRにより "OpenGL Support" というログがビルド途中に表示される。これによって自分はてっきりOpenGLサポートも実験的に含まれているのかと一瞬勘違いしたけれど、そうではない。OpenGLは切り捨てられているので、実験的なEmscriptenサポートでもWebGLはサポートされず、対応するのはWebGPUとなっている。
[^2]: 執筆時点での[最低動作環境](https://github.com/RavEngine/RavEngine?tab=readme-ov-file#system-requirements-for-developers)は、`DX12, VK 1.3, or Metal 3` とあるので、仮にVulkanやMetalが動く環境だとしても、現時点で最新のSDKや機能が動作しなければサポートされないので、十分な注意が必要。
