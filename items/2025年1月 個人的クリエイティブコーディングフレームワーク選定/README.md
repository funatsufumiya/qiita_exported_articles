**完全なる独断と偏見**で、自分が個人的に2025年に使っていきたい、クリエイティブコーディング関連のフレームワークについてまとめていこうと思う。

（プロプライエタリ = 非オープンソース のものは、TouchDesignerのみ一応紹介して、あとは選定から外している点に注意。）

なお、半年以上前にまとめた以下の記事も参考になるはず。

https://qiita.com/funatsufumiya/items/9088905c7e6f9a1531bc

## とにかくスクラッチで書きたい！ → LÖVR

<iframe width="560" height="315" src="https://www.youtube.com/embed/V31VfV84FvY?si=nxDbUUuyCbtQN0xr" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

https://lovr.org/

上記動画で、[LÖVR](https://lovr.org/)が最も簡単な3Dゲームエンジンではないかと紹介されているけれど、自分も同意する。

元々人気のあったLua用ゲームエンジンである[Love2D (LÖVE)](https://love2d.org/)を、Vulkan対応などによって3Dに進化させたもの。同様の亜種は複数あるものの、ドキュメントが豊富で、スター数なども考慮して、一番安定して使いやすいものではないかと思う。

現時点のデメリットとしてはiOS非対応なことだけれど、Androidには既に対応しているし、時間の問題かもと一瞬思ったが、このフレームワークの主眼はVR/XRにあるようで、[今のところ対応予定はない](https://github.com/bjornbytes/lovr/issues/654)とのこと。ただ、visionOSの存在もあるので、今後の動向に注目。

## エディタがやっぱりほしい！ → Godot

https://godotengine.org/

ゲームエンジンについては多種多様に存在するので、正直自由に選んでもらえればと思うのだけれど、GDScriptの完成度や、C#も併用したり、C++（や他の言語）で自由に拡張したりといった総合評価で、自分はGodotをおすすめしたい。

前述のLÖVR同様、書いていて楽しいと思うのはGodotも間違いないと思う。例えばシェーダ言語が書けなくても、ビジュアルエディタが豊富にあり、アニメーションやスプライトについてもエディタでサクサク作っていけるので、初心者にも上級者にもおすすめできると感じる。

ただし、LÖVRもそうかもしれないけれど、Vulkan対応等によって必須スペックは過去のゲームエンジンに比べて上がっていて、もし旧来のマシンやGPU、古いスマホにも対応したいという思惑があるのであれば、レンダラーを変更したり、他のOpenGL対応のゲームエンジンも検討するなどしてほしい。

## ECSでバリバリ書いていきたい！ → Bevy

https://bevyengine.org/

LÖVRの存在を知るまでは、スクラッチで書くならこれしかないとすら思っていた。先進的な機能と拡張性を併せ持っていて、Rust以外の何もインストールを必要としないのも嬉しい。(VSCodeさえあれば完璧に補完が効くのもありがたい。)

注意点としては、Bevyフレームワークのおかげでコーディングが比較的簡単になっているとはいえ、Rustの難易度は低くはないこと。そして、Rustの特徴として関連ライブラリを含めてすべてビルドするので、プロジェクトフォルダのサイズが大きく、（最初の）ビルド時間が大きい（【2025/5/6 追記】改善方法があったので脚注にて紹介[^3]）。これについては諸刃の剣であるし、sccache等の対応策も一応ある。

また、先進的な機能を次々に取り入れている結果、破壊的変更が多いことに注意。セマンティックバージョニング (`0.xx.yy`) で、`0.xx` (例えば 0.14) が変わらなければAPIは維持されているが、例えば 0.14 と 0.15 はもはや別物レベルで違う。（ただし、[マイグレーションガイド](https://bevyengine.org/learn/migration-guides/0-14-to-0-15/)という丁寧なドキュメントや、豊富なコミュニティサポートにより、移行自体は個人的にはそこまで苦労はしないと感じている。ただ、英語中心。）

古き良き[OpenFrameworks](https://openframeworks.cc/)などと同じように、[Bevy Assets](https://bevyengine.org/assets/)と呼ばれる拡張機能が多数存在するので、「これってできないのかな？」と思うことはたいてい拡張機能で対応できるのも嬉しい。コミュニティが厚いというのはこの辺が嬉しいところ。[Examples](https://bevyengine.org/examples/)もブラウザで動いているものを確認しながらコードも見れるのが楽しい。

ちなみにBevyは、ECSのおかげもあって[自動的にマルチスレッドかつパイプライン対応](https://bevy-cheatbook.github.io/setup/perf.html)してくれるのがありがたい。つまり、自然なコードを書けばフレームワーク側が自動でパフォーマンスを最適化してくれるので、ゲーム開発者にとっては助かる部分だと思う。

## バランス良く書いていきたい！ → Delve Framework (および zig-gamedev)

![300164821-45b64806-7829-4542-80d5-5a892eebf80d.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/b02ef8cc-6018-2098-213b-019cea8b0d64.png)


![スクリーンショット 2025-01-14 13.16.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/58241be6-dc52-09bd-47ad-04ad4d3d9e20.png)

![スクリーンショット 2025-01-14 14.32.29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/74f50e54-d4da-0aa6-1755-7c03015e05dc.png)

https://github.com/Interrupt/delve-framework

https://github.com/zig-gamedev

こちらは追記にあたるのだけれど、元々後述の「今後に期待シリーズ」に入れていたもののなかで、[Delve Framework](https://github.com/Interrupt/delve-framework)は特に光るものがあったので、ピックアップしておきたい。

[Zig言語](https://ziglang.org/)というのは言語のなかでも比較的新しいもので、近年ではRustやC/C++に次いでよく使われているのではないかと思う。（[bun](https://bun.sh/)というJavaScript処理系がZigで書かれていることでも知られている。）

手動でメモリ管理を行う言語ではありながら、自分が以前書いたアリーナ・アロケータに関する記事のように、いくつか注意していれば、普通に書き進めていくことのできるバランスのとれた言語。（その他、deferといったメモリ解放忘れを防ぐ仕組みが元々あったり、[zigrc](https://github.com/Aandreba/zigrc)のようなスマートポインタ実装も最近は登場している。）

https://zenn.dev/funatsufumiya/articles/b3935504dd1e97

今回、2025年らしい先進的なフレームワークに的を絞って集めたつもりなのだけれど、Delve Frameworkは特にバランスが良いと感じていて、Zigのおかげでビルドはすごく高速だし、Luaを使うこともできる。

Delve Frameworkでは[Sokol](https://github.com/floooh/sokol)という描画エンジンが使われていて、Metal/DX11/OpenGL/WebGL（[Vulkanは非対応](https://x.com/FlohOfWoe/status/1005377054320332800)）に自然と対応しながらも、とてもサクサク書き進めていける。一方でSokolは並列での描画には基本的に向いていないので、Delve Framework自身がなにか今後工夫をする可能性はありつつ、本気でマルチスレッドのパフォーマンスやスケーラビリティを求める場合は前述のBevyの方が良いかなとは思う。

プログラミング言語の好みもあると思うので、そういう意味でもZig言語のゲームエンジンを一つ紹介してみた。Zig言語の他のゲームエンジンとしては、本当はいつか[Mach Engine](https://machengine.org/)が完成してほしいと期待している。（Rustの難しさに悩んでいる人は、Zigを一度試してみると良いかもしれない。）

## ノーコードがいい！ → TouchDesigner

https://derivative.ca/

TouchDesignerはオープンソースでも無料でもないので、紹介程度に留めておきたいけれど、ゲームエンジンでいうところのUnityくらい、デファクト・スタンダードとなっているので、紹介だけはしておくべきかなと思う。

TouchDesignerのオープンソース的代替候補については[冒頭で紹介した記事](https://qiita.com/funatsufumiya/items/9088905c7e6f9a1531bc)で挙げているし、きっと他にもいろいろあるはず。ちなみに元々TouchDesignerはバックエンドとしてOpenGLを使っていたが、2022年頃にVulkanに刷新された。

プロプライエタリとはいえ、コミュニティは大きいので、ノーコードで制作したい、VJやりたいというときには真っ先に選択肢に挙げて良いものだと思う。

## その他 (今後に期待シリーズ)

以下、自分が個人的にウォッチしていて、今後のアップデートや動向次第では積極的に使っていきたいものリスト。

- [Cocos Creator](https://www.cocos.com/en/creator) ※ Cocos2d-xとは別物
- [SDL3](https://wiki.libsdl.org/SDL3/NewFeatures) (C++ほか。)
- [RavEngine](https://github.com/RavEngine/RavEngine/) (C++。[紹介記事](https://qiita.com/funatsufumiya/items/4a154378e490fdde42a8))
- [Sinen](https://github.com/astomih/sinen) (C++。[紹介記事](https://qiita.com/funatsufumiya/items/db1e13033ca071425cf6))
- [minimax](https://github.com/roman01la/minimax) (Clojure言語。[紹介記事](https://qiita.com/funatsufumiya/items/e41df266eb72b097074a))
- [OGRE-Next](https://github.com/OGRECave/ogre-next) (C++)
- [Mach Engine](https://machengine.org/) (Zig言語)
- [Odin Language](https://odin-lang.org/) (Odin言語 [^1])
- [nannou - Bevy Plugin Rework](https://github.com/nannou-org/nannou/issues/953) (Rust言語)

特にSDL3については、近日中に正式リリースになるので（【追記】2025/1/22に正式リリースされました）、特に[SDL GPU](https://wiki.libsdl.org/SDL3/CategoryGPU)をベースにした新しいフレームワークが次々と誕生するのではないかと予想。

ちなみに今回は挙げていないけれど、[Processing](https://processing.org/)や[OpenFrameworks](https://openframeworks.cc/)のような、旧来のOpenGL系の枯れたフレームワーク群も、まだまだ使えるし今後もおそらく使えるので、特に古いマシンやスマホとの互換性を保つ場合には是非検討してほしいと思う。（Web系も同様[^2]。）

[^1]: [Odin言語](https://odin-lang.org/)については、各種描画APIへのバインディングが公式で既になされているので、ライブラリではなく言語自体を列挙した。が、[BGFX](https://github.com/bkaradzic/bgfx)や[RGL](https://github.com/RavEngine/RGL/)のようなクロスプラットフォーム描画APIへのバインディング、かつ安定したものがあるとさらに使いやすくなりそう。

[^2]: Web系はもしかすると[three.js](https://threejs.org/)一択といえるのかもしれないし、最近WebGPUなどの動きが活発なので、[BevyのExampleもWebで動くものが見れる](https://bevyengine.org/examples/)し、選択肢は豊富にあるともいえるかもしれない。

[^3]: 次のBevyを紹介しているYouTube動画の、4:15あたりから、ビルド時間改善方法が紹介されている。 https://www.youtube.com/watch?v=5k66KB6DisI&t=251s （具体的には、動的リンクに変更する方法と、リンカを変える方法がしょうかいされている）
