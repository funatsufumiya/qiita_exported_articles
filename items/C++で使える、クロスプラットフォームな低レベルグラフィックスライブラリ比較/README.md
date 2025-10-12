(英語版も書きました: [Comparison of C++ Low-Level Graphics, Cross-Platform Frameworks and Libraries](https://dev.to/funatsufumiya/comparison-of-c-low-level-graphics-cross-platform-frameworks-and-libraries-58e5))

最近OpenGLばっかり書いていて、ふと、MetalやDirectXでも動くクロスプラットフォームなアプリも書きたいなと思ったので調べてみた。

今回調べてみたライブラリの特徴は、

- **C++で実装**されている ( C++14/17 など詳細は別記 )
- **低レベルグラフィックスAPI**に対応 ( Vulkan、DirectX11/12、Metal など )
- **クロスプラットフォーム**で動く ( Win/Mac/Linux、iOS/Android など )
- UnityやUE4に比べて、**軽量もしくは拡張性・組み込み性が高く**、フレームワークやライブラリとして自由に使えるもの。

なお、今回は低レベルグラフィックスAPIへの対応を重視しているので、ゲームエンジンとしての比較は他記事を参照。今回紹介するライブラリをベースにした、独自のゲームエンジンが多数存在する。

**追記:** 本文が結構長くなったので、目次として [まとめ](#まとめ) の表をここに記載。（リンクは各見出しに飛びます。）

| |良い点 :white_check_mark: |悪い点 :x: |
|---|---|---|
|[LLGL](#llgl-low-level-graphics-library)|豊富なAPIに対応|個別対処・難易度 高|
|[The Forge](#the-forge)|対応端末数・完成度|個別対処・難易度 高|
|[Diligent Engine](#diligent-engine)|同一ソースで複数対応|Metal非対応|
|[bgfx](#bgfx)|同一ソースで複数対応・簡単|テッセレーション対応△|
|[oryol / sokol_gfx](#oryol--sokol_gfx)|組み込み・Web向き|計算シェーダ非対応|
|[Methane Kit](#methane-kit)|Windows / Mac に特化|モバイル非対応|
|[bs::framework](#bsframework)|ゲームエンジン基盤|Windows以外は開発途上|

## LLGL (Low Level Graphics Library)

https://github.com/LukasBanana/LLGL

![4506fdd3-2291-2b76-1390-89142d4c64f8-min.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/2a2992f1-535c-9cea-bb82-86b9e37cbdb7.jpeg)


- ライセンス: **BSD-3 clause**

- 対応プラットフォーム:

|  | DirectX | Metal | Vulkan | OpenGL | OpenGL ES | 
|---|---|---|---|---|---|
| Windows | :white_check_mark: | - | :white_check_mark: | :white_check_mark: | - |
| Mac | - | :white_check_mark: | - | :white_check_mark: | - |
| Linux | - | - | :white_check_mark: | :white_check_mark: | - |
| Android | - | - | :x: | - | :white_check_mark: |
| iOS | - | :white_check_mark: | - | - | :x: |

- シェーダ対応:

| Vertex | Fragment | Compute | Geometry | Hull (TC) | Domain (TE) | 
|---|---|---|---|---|---|
| :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |

※ シェーダはクロスコンパイル式ではなく、**プラットフォーム別**に準備が必要（glsl/hlsl/metal/spv）

- 所感:

依存ライブラリが少なく、ビルドは短時間で簡単 (cmake)。
DirectX / Metal / Vulkan / OpenGL (ES) に対応していて、各種シェーダも全て使えるので、**最も網羅的に使える**印象。

低レベルアクセスについては、共通するAPIは最低限だけ提供して、それぞれのプラットフォームに必要な処理はマクロ等で切り替えて**個別のプラットフォームごとに対応する**スタイル。そのため性能要求に応えやすくメンテしやすいが、**必要な知識と難易度は高め**。（とはいえ、APIのラッピングと共通化は行ってくれているので、可読性とメンテナンス性は高い。）

シェーダも個別に準備する必要があるが、他のライブラリのように自分で `spirv-cross` や `shaderc` を利用してクロスコンパイルすれば問題ない。逆に捉えれば、API同様、各プラットフォームごとに特化した処理を書くことができる。

## The Forge

https://github.com/ConfettiFX/The-Forge

![a7491fa8-f2cd-3c28-b809-5bbb3cf4922a-min.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/8b1ff4eb-4c7f-bc84-b484-4855b512d5bb.jpeg)


- ライセンス: **Apache-2.0**

- 対応プラットフォーム:

|  | DirectX | Metal | Vulkan | OpenGL | OpenGL ES | 
|---|---|---|---|---|---|
| Windows | :white_check_mark: | - | :white_check_mark: | :x: | - |
| Mac | - | :white_check_mark: | - | :x: | - |
| Linux | - | - | :white_check_mark: | :x: | - |
| Android | - | - | :white_check_mark: | - | :x: |
| iOS | - | :white_check_mark: | - | - | :x: |

※ **XBOX, PS4, Switch, Stadia にも公式対応**。

- シェーダ対応:

| Vertex | Fragment | Compute | Geometry | Hull (TC) | Domain (TE) | 
|---|---|---|---|---|---|
| :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |

※ シェーダはクロスコンパイル式ではなく、**プラットフォーム別**に準備が必要（glsl/hlsl/metal/spv）

- 所感:

かなり**完成度の高い**フレームワークで、**ゲームエンジンのレンダリング基盤**としての使用を強く意識して作られている。ただし、**OpenGL (ES)には非対応**なので、古いモバイル機器やRaspberry Piなどでは使えない。

非常に作り込まれている印象で、CIや単体テストを多用していることからも**安定性の高さ**が伺える。実際の導入実績も多い。

使い勝手は前述の[LLGL](#llgl-low-level-graphics-library)に近く、**個別のプラットフォームに対する知識は必要**だが、高い要求性能に応えることができるフレームワークだと感じる。


## Diligent Engine

https://github.com/DiligentGraphics/DiligentEngine

![ef1bf02d-23cd-8910-78a7-01d3bbf60458-min.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/af1b0343-7a3e-d48b-b6ca-d9e7f0d0f890.jpeg)

- ライセンス: **Apache-2.0**

- 対応プラットフォーム:

|  | DirectX | Metal | Vulkan | OpenGL | OpenGL ES | 
|---|---|---|---|---|---|
| Windows | :white_check_mark: | - | :white_check_mark: | :white_check_mark: | - |
| Mac | - | :x: | :white_check_mark: (MoltenVK) | :white_check_mark: | - |
| Linux | - | - | :white_check_mark: | :white_check_mark: | - |
| Android | - | - | :x: | - | :white_check_mark: |
| iOS | - | :x: | :white_check_mark: (MoltenVK) | - | :white_check_mark: |

- シェーダ対応:

| Vertex | Fragment | Compute | Geometry | Hull (TC) | Domain (TE) | 
|---|---|---|---|---|---|
| :white_check_mark: | :white_check_mark: | :white_check_mark: (※2) | :white_check_mark: | :white_check_mark: | :white_check_mark: |

※1 HLSLをクロスコンパイルさせる方式。
※2 OpenGL4.3未満ではコンピュートシェーダ使用不可のため、MacのOpenGLでは使えない。

- 所感:

依存ライブラリが少なく、ビルドは短時間で簡単 (cmake) [^注1]。名前にエンジンと付いてはいるが、**軽量で使いやすい**フレームワーク。

APIとシェーダは各プラットフォームで**完全に共通化**されており、同じソースコードで各プラットフォームに対応できるのが特徴。

ただし、**Metalに対応していない**のがすごく残念…。そのため、**MacのOpenGLではコンピュートシェーダが使えない**。その点を除けば、後述の bgfx と同様の軽量フレームワークとして、使い勝手が良いと感じる。（bgfxと比べると、後発だけあってAPIがモダンで実装効率もよく、拡張性が高い。）

## bgfx

https://github.com/bkaradzic/bgfx

![762db1a0-0944-f7d2-9b2d-4b9dccd73c6f-min.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/732622f0-dab1-5402-6902-433a3cd330c8.jpeg)


- ライセンス: **BSD-2 clause**

- 対応プラットフォーム:

|  | DirectX | Metal | Vulkan | OpenGL | OpenGL ES | 
|---|---|---|---|---|---|
| Windows | :white_check_mark: | - | :white_check_mark: | :white_check_mark: | - |
| Mac | - | :white_check_mark: | :question: | :white_check_mark: | - |
| Linux | - | - | :white_check_mark: | :white_check_mark: | - |
| Android | - | - | :question: | - | :white_check_mark: |
| iOS | - | :white_check_mark: | :question: | - | :white_check_mark: |

※ **WebGLに対応**。Emscriptenやasm.js、Raspberry Piなどにも対応。

- シェーダ対応:

| Vertex | Fragment | Compute | Geometry | Hull (TC) | Domain (TE) | 
|---|---|---|---|---|---|
| :white_check_mark: | :white_check_mark: | :white_check_mark: | :x: | :x: (※2) | :x: (※2) |

※1 GLSLのサブセットの独自言語を、クロスコンパイルさせる方式。
※2 DirectXとOpenGLについては、hull/domainシェーダーに対応させる[非公式フォーク](https://github.com/LSBOSS/bgfx)あり。メンテナンスされていないが…。

- 所感:

2015年から開発されていて、導入実績と実装例、質問投稿などがすごく豊富。コンパイルも簡単で、cmake以外にもGENieという独自のビルドマネージャー搭載。

**同じソースでマルチプラットフォームで動作**するのが特徴。**APIがとてもシンプル**で使いやすく、拡張性が高い。一方で、**テッセレーション系シェーダに非対応**なのが残念だが、コンピュートシェーダでテッセレーションに対応させることは可能 (公式Exampleも存在) 。Diligent Engineと違ってMetalに対応しており、**MacやiOSでもコンピュートシェーダが使える**のは大きい。

シェーダーにはGLSLに似た独自言語が使えるが、専用のヘッダファイルなどを定義する必要がある。ちょっと独特だが、各プラットフォームで共通のシェーダーが使えるのは嬉しい。

開発方針として、同一ソースにおける、各プラットフォームでの**互換性が重視**されているため、テッセレーションの件も含め、コマンドバッファなどに**一部制限**がある。また、今回紹介しているライブラリの中では最も登場時期が早いため**API設計がやや古く**、Dilligent Engineと比べると実装効率の面などで劣る点もあるが、今も活発に更新されているので改善に期待。

## oryol / sokol_gfx

https://github.com/floooh/oryol
https://github.com/floooh/sokol

![ed19ec55-2d89-29e9-cfc8-b0adbfc78f3c-min.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/b2e6ba87-4fe2-5552-fe93-37c5c0a07abb.jpeg)

- ライセンス: **MIT** (oryol)、**zlib** (sokol)

- 対応プラットフォーム:

|  | DirectX | Metal | Vulkan | OpenGL | OpenGL ES | 
|---|---|---|---|---|---|
| Windows | :white_check_mark: | - | :x: | :white_check_mark: | - |
| Mac | - | :white_check_mark: | :x: | :white_check_mark: | - |
| Linux | - | - | :x: | :white_check_mark: | - |
| Android | - | - | :x: | - | :white_check_mark: |
| iOS | - | :white_check_mark: | :x: | - | :white_check_mark: |

※ WebGLにも対応
※ Raspberrry Piにも正式対応 (OpenGL ES)

- シェーダ対応:

| Vertex | Fragment | Compute | Geometry | Hull (TC) | Domain (TE) | 
|---|---|---|---|---|---|
| :white_check_mark: | :white_check_mark: | :x: | :x: | :x: | :x: |

※1 oryolは、GLSLに似た独自言語を、クロスコンパイルさせる方式。sokol_gfxは個別に準備が必要。

- 所感:

oryol と sokol_gfx は、どちらも同じ作者が制作したライブラリで、sokol_gfxが **C言語用のシングルヘッダーライブラリ** であるのに対し、oryolは **C++用の軽量ライブラリ**。C++11未満でも使えるよう配慮されており、**組み込み用途を強く意識**しているのが特徴。

**非常に軽量なライブラリ**でありながら、必要な機能は十分備えている。さらに**WebGLに対応**しており、asm.js/wasmによってウェブサイト用にコンパイルして使うことができ、その際のファイルサイズも小さい。

**コンピュートシェーダーが使えない**のだけが残念だが、作者は別のアドオン的なライブラリとしての制作を検討している模様。今後に期待。

## Methane Kit

https://github.com/egorodet/MethaneKit

![087fc62c-afc9-b0fa-daf2-2aeb42ff3f60-min.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/9adec764-bc24-2e4c-19fb-97e1162e1553.jpeg)

- ライセンス: **Apache-2.0**

- 対応プラットフォーム:

|  | DirectX | Metal | Vulkan | OpenGL | OpenGL ES | 
|---|---|---|---|---|---|
| Windows | :white_check_mark: | - | :x: | :x: | - |
| Mac | - | :white_check_mark: | :x: | :x: | - |
| Linux | - | - | :white_check_mark: (※1) | :x: | - |
| Android | - | - | :x: | - | :x: |
| iOS | - | :x: | :x: | - | :x: |

※1 Linux/Vulkan 版は現在開発途中。

- シェーダ対応:

| Vertex | Fragment | Compute | Geometry | Hull (TC) | Domain (TE) | 
|---|---|---|---|---|---|
| :white_check_mark: | :white_check_mark: | :x: | :x: | :x: | :x: |

※ HLSL 5.1を、クロスコンパイルさせる方式。

- 所感:

C++17を基準に作られた、**デスクトップに特化**したライブラリで、**Windows/Macでの互換性・再現性の高さ**が特徴。Windows/Macともに同じコード・シェーダーで書くことができる。最近になってLinux/Vulkan版が発表された。

GitHubに公開されてまだ1年で、活発に更新が行われている。


## bs::framework

https://github.com/GameFoundry/bsf

![3997370c-2403-14ef-35c6-d97f135fc2b1-min.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/7cde6879-5e94-efbf-ca01-307e630cc603.jpeg)

- ライセンス: **MIT**

- 対応プラットフォーム:

|  | DirectX | Metal | Vulkan | OpenGL | OpenGL ES | 
|---|---|---|---|---|---|
| Windows | :white_check_mark: | - | :white_check_mark: | :white_check_mark: | - |
| Mac | - | :x: | :x: (※1) | :white_check_mark: (※2) | - |
| Linux | - | - | :white_check_mark: | :white_check_mark: | - |
| Android | - | - | :x: | - | :x: |
| iOS | - | :x: | :x: (※1) | - | :x: |

※1 MoltenVK経由での実装の議論は、執筆時点で進行中。
※2 執筆時点でMac版の動作に難あり。

- シェーダ対応:

| Vertex | Fragment | Compute | Geometry | Hull (TC) | Domain (TE) | 
|---|---|---|---|---|---|
| :white_check_mark: | :white_check_mark: | :white_check_mark: (※2) | :white_check_mark: | :white_check_mark: | :white_check_mark: |

※1 HLSLのサブセットである独自言語 (BSL) を、クロスコンパイルさせる方式。
※2 MacでMetal/Vulkan非対応のため、Macではコンピュートシェーダ利用不可。

- 所感:

今回の他のライブラリに比べて、これだけ少し規模感が異なり、**ゲームエンジンとしての機能を統合的に提供するフレームワーク**。[Banshee 3D Engine](https://www.banshee3d.com/)の作者が作っていて、ゲームエンジンの基盤として意識して作られている模様。初期バージョンがリリースされてからまだ１年で、未だベータ版。

C++14を基準に作られており、UnityやUE4のようなシーン・コンポーネントシステムや物理演算、各種ファイル読み込み機能、アニメーション機能、音声再生等、低レベルグラフィックAPIだけにとどまらず、１つのフレームワークで**幅広い機能**を提供している。

ドキュメントは比較的豊富だが、全体的に**開発途上で、安定性に欠ける**。**モバイル未対応**なのと**Metal非対応**なのも辛い。現状ではWindows版のみ安定動作している印象[^注2]。

## その他

- Acid (https://github.com/EQMG/Acid) ・・・ Vulkan / MoltenVK に特化したゲームエンジン。
- magnum (https://github.com/mosra/magnum) ・・・ OpenGL / WebGL に対応。Vulkanへの対応が検討されている。
- Rizz (https://github.com/septag/rizz) ・・・ 描画にsokol_gfxを利用した、C言語ベースの軽量フレームワーク。特筆すべきは、[glslcc](https://github.com/septag/glslcc)を利用しているのと、DirectXで実験的にコンピュートシェーダが使える。
- filament (https://github.com/google/filament) ・・・ The Forgeやbgfxのような、google製レンダリングエンジン。Vulkan / Metal / OpenGL(ES) / WebGL をサポート。
- LinaGX (https://github.com/inanevin/LinaGX) ・・・ 詳細は未確認も、Windows / Mac / Android / iOS をサポートしている様子。

**注意:** 今回はテーマの都合上、OpenGL系の [openFrameworks](https://openframeworks.cc/) や [Cinder](https://libcinder.org/) などは意図的に除外したが、実はそれぞれVulkanでの実装 ([openFrameworks-vk](https://github.com/openframeworks-vk/openFrameworks), [Cinder-Vulkan](https://github.com/cinder/Cinder/tree/vulkan)) も一応存在する。ただ、いずれも実験的で、メンテナンスされているかというと微妙な感じ。

ちなみにOpenGL系のフレームワークについては [awesome-creative-coding](https://github.com/terkelg/awesome-creative-coding) などの記事がおすすめ。 

## その他、C++以外で類似のフレームワーク

- gfx-rs (https://github.com/gfx-rs/gfx) ・・・ Rust言語で実装され、今回のテーマでは最も完成度の高いフレームワークの一つ。C言語APIを通してC++から使うこともできる？
- rendy (https://github.com/omni-viral/rendy) ・・・ 同じくRust言語で、gfx-rsをベースにさらに自作レンダラーを作りやすくしたフレームワーク。The Forgeライクなもの？
- veldrid (https://github.com/mellinoe/veldrid) ・・・ .NET実装。
- Kha (https://github.com/Kode/Kha) ・・・ Haxe実装。かなり対応端末と対応APIが多く、利用も簡単そう。C++にクロスコンパイルすればC++と一緒に利用できるのでは。

## まとめ

書いてみると意外と分量が多くなった。簡潔に比較すると以下の通り。

| |良い点 :white_check_mark: |悪い点 :x: |
|---|---|---|
|[LLGL](#llgl-low-level-graphics-library)|豊富なAPIに対応|個別対処・難易度 高|
|[The Forge](#the-forge)|対応端末数・完成度|個別対処・難易度 高|
|[Diligent Engine](#diligent-engine)|同一ソースで複数対応|Metal非対応|
|[bgfx](#bgfx)|同一ソースで複数対応・簡単|テッセレーション対応△|
|[oryol / sokol_gfx](#oryol--sokol_gfx)|組み込み・Web向き|計算シェーダ非対応|
|[Methane Kit](#methane-kit)|Windows / Mac に特化|モバイル非対応|
|[bs::framework](#bsframework)|ゲームエンジン基盤|Windows以外は開発途上|

もし自分が実践で使うとしたら、今のところ簡単さと互換性で bgfx かなぁ…。

要求性能が高いところでは LLGL か The Forge という感じだけど、この２つはもはや新しいゲームエンジンを作成する基盤として使えるレベル。ある意味で Unity / UE4 との比較になってくる気がする。

## 参考記事

- awesome-vulkan (https://github.com/vinjn/awesome-vulkan#Libraries)

[^注1]: 執筆時点では、Windows用のスクリプトが準備されていないので、ビルド時・テスト時には、Mac/Linux用のシェルスクリプトを参考にしながらコマンドを打つ必要あり。
[^注2]: ビルドについては、[Hello World](http://docs.bsframework.io/latest/build.html)はすぐビルドできるものの、[Examples](https://github.com/gamefoundry/bsfExamples)はビルド時に追加ファイルが多く、ダウンロードに失敗するケースもあり、ビルドまでにcmakeを修正したりと時間と手間がかかる。また、Macでは配布されているバイナリv1.1.0では執筆時点で[正しく動作しないケースがあるようで](https://github.com/GameFoundry/bsf/issues/129)、筆者環境では配布バイナリでもmasterからビルドしても正常動作せず。
