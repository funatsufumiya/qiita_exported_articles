![ゲームエンジンOSS対応表.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/fd73d9fc-96b9-55a7-dd6d-7d0683195ad6.png)

完全なる独断と偏見で、オープンソースのゲームエンジン・VJエンジンと商用エンジンの対応表（似ているものの相関図）を作ってみました。[^5]

2024年5月時点の情報なので、今後対応するOSや言語などは変わっていくと思われます。

（似たような記事や動画は多くあると思いますが、一応参考になりそうな他の動画を２つほど挙げておきます。）

https://www.youtube.com/watch?v=WxaXmB-tPBE

https://www.youtube.com/watch?v=2ardiWXqdk8

:::note warn
冒頭の図における「だいたい対応」というのは、思想や概念、使い勝手などが似ていることを指していて、移行が比較的簡単であることを意味しています。一方で、例えばファイル形式などに互換性があったり、公式サポートがあるわけではないので注意です。
:::

## 背景 (who am i)

一応これを作った背景を。元々筆者は複数のゲームエンジンやフレームワークを適材適所で利用して、仕事などで作品等を作ってきました。

私は最近では[Bevy Engine](https://bevyengine.org/)を主に使っていますが、開発ステージ的に安定版まではもうあと一歩であるということと、エディターレスであるということもあり、やはり複数のフレームワークやエンジンを適材適所で使うということを続けています。

長らく[Unity](https://unity.com/ja)を使っていたこともあり、最近は思想が比較的似ている[Godot](https://godotengine.org/)を使い始めたりもしていますが、VJ的な作品では[Touch Designer](https://derivative.ca/)や[Max](https://cycling74.com/products/max)などが向いている場合もあり、ゲームエンジン等にはそれぞれ向き不向きや哲学があります。

冒頭の図においては、矢印で繋いだものは比較的思想が似ているものを繋げたつもりです。いずれのエンジンも同等なことができたり、相互利用できたりすること、また似ていると思っていても違う哲学があったりすることを留意いただければと思います。

## 注意事項・補足

### VVVV ←→ Stride

今回の図にVJエンジンを含めた大きなポイントが[VVVV](https://visualprogramming.net/)にあります。VVVVは、元来のバージョンである`beta`と、2020年からスタートした`gamma`でだいぶ変更点があり、特に`gamma`では3D描画エンジンとして[Stride](https://www.stride3d.net/)が利用されています[^2]。

このことを配慮したのと、[TOOLL3](https://tooll.io/)をぜひ取り上げたかったということもあり、今回ゲームエンジンとVJエンジンを混ぜることにしました。（TOOLL3については後述。）

VVVVを商用エンジンに含めるかは迷いましたが、ライセンスとして商用用途は有償で販売されていることもあり、今回は商用エンジン扱いとしています。

### UE ←→ Stride

[Unreal Engine](https://www.unrealengine.com/ja)についても、ソースは公開されているので厳密にはオープンソースですが、VVVVと同様の理由で商用エンジン扱いとしています。

Strideが対応関係にあるかについては賛否両論あるかと思いますが、描画エンジンの方向性がフォトリアリスティックであることや、UEもStrideも3Dを基軸としていること（UnityやGodotには2Dモードが別にあること）などを踏まえて、対応関係としました。

ちなみに、フォトリアリスティックな表現ができるゲームエンジンは例えば[Wicked Engine](https://wickedengine.net/)などいろいろ選択肢がありますので、あくまで参考として肌感に合ったものを使われると良いと思います。

（追記: 2024年10月）[O3DE](https://o3de.org)もStrideと比較的近いゲームエンジンのように思います。UEのようなDeferred Renderingはまだサポートされていませんが、さらに調査して比較してみたいところです。

### Stride

https://www.youtube.com/watch?v=BiFLOs40BgM

[Stride](https://www.stride3d.net/)は、過去にXenkoと呼ばれていたゲームエンジンのようで、特にフォトリアリスティックな表現をするのに向いているエンジンという印象があります。執筆時点で最新版の4.2では、.NET 8 / C# 12への対応もできているとのことで、UE5の代替の大きな選択肢の一つではないでしょうか。

現時点では開発環境はWindowsのみ対応していますが、クロスプラットフォームに書き出しすることが可能で、エディタ自体をMac/Linuxに対応させようという動きも一応あるようです[^7]。

### Unity ←→ Godot

UnityとGodotは、一方がコンポーネントシステムでありもう一方がノードシステムであることなど、違いは多く存在します。今回はUEやTouch Designerなどと比較して大局をみたときに、類似点が多いことから対応関係としました。

Godotについてはすでに多くの情報があることもあり、書くとしても別の記事にしたいと思っていますので詳細は割愛します。個人的なGodotの魅力は、言語選択の自由度が高いこと、クロスプラットフォーム対応の選択肢が幅広いこと（描画エンジンが選べるなど）、そして後発ゆえにエディタや構造がシンプルで使いやすいことが挙げられると思います。

### OpenFrameworks ←→ Bevy

今回特に扱いに迷ったのが、エディタレスの汎用エンジンである[OpenFrameworks](https://openframeworks.cc/) (oF)と[Bevy](https://bevyengine.org/)です。

正直載せなくてもいいくらいではあったのですが、OpenFrameworksはよくVVVVやTouch Designerと同じ界隈で広く利用されていること、また、BevyはUnity DOTSと思想が似ていること、そして標準エディタがないだけで、描画エンジンや開発フレームワークとしてのポテンシャルが高いことを考慮に入れて表に載せました。

また今回、BevyをOpenFrameworksの「上位互換」[^6]とあえて書きましたが、あくまでベーススペック上の概観の話であって、開発思想やユーザ層は全く異なりますし、アドオンなどの広がりも方向性も異なっているので、C++とRustという言語の壁はありますがうまく相互利用できる関係にあると思っています。

#### "エディタレス" の意味合いについて

個人的には、BevyとGodotをうまく今後連携して利用していければと思っていて、[bevy_godot4](https://github.com/funatsufumiya/bevy_godot4)の存在を念頭において、「一応相互利用可能」と表記しました。

ただBevy界隈では、[Blender_bevy_components_workflow](https://github.com/kaosat-dev/Blender_bevy_components_workflow)を使ってBlender (GLTF)をエディタのように使ったり、[bevy_ecs_ldtk](https://github.com/Trouv/bevy_ecs_ldtk)を使って[LDTK](https://ldtk.io/)を利用したりすることの方が多いかなという印象で、Unityと連携している事例[^3]もありますが、他のゲームエンジンとの連携はこれからかなと思っています。

BlenderやLDTK、そして[egui](https://github.com/mvlabat/bevy_egui)や[ImGui](https://github.com/ocornut/imgui)なども考慮にいれると、BevyやoFはもはやエディタレスとは呼べないかもしれませんし、エディタがないゆえの開発上の利便性（抽象性やポータビリティ）や選択の自由度があるのは確かです。

エディタレスのゲームエンジンは、正直挙げればキリがないのですが、[Proecssing](https://processing.org/)や[LÖVE2D](https://love2d.org/)、[OpenFL](https://www.openfl.org/)など、一定層の根強い人気があり、クリエイティブコーディングの楽しさや気軽さ、抽象度の高さゆえの応用性の高さがあります。私はエディタレスのフレームワークは、個人的にUnityやUEとは別の方向性でも発展していくのではないかと感じています。前述のように、うまく相互利用していけるといいなと思っています。

### TOOLL3

https://www.youtube.com/watch?v=_zvzX0fZ8sc

[TOOLL3](https://tooll.io/)（あるいは単にT3）は個人的に使っていてすごく楽しいと思うツールの一つで、Touch Designerや[Max](https://cycling74.com/products/max)などに触れたことがある方であれば楽しく使えるのではないかと思います。

TOOLL3がオープンソースとして今後どんな発展を遂げていくかはわかりませんが、Touch DesignerがGodot同様にOpenGL/Vulkanをベースにしているのに対して、DirectXをベースにしていたり、C#でカスタムできたりと、思想が異なる部分を逆に興味深く感じています。

MacやLinuxへの対応も検討されている[^1]ようで、DirectXベースの部分が今後どう他のプラットフォームに展開されていくのか、あるいはWindowsのみで発展していくのかを注視したいところです。

[^1]: https://github.com/tooll3/t3/issues/32
[^2]: https://thegraybook.vvvv.org/reference/libraries/graphics-3d.html
[^3]: https://www.youtube.com/watch?v=lsc2X3uS8VU
[^5]: 図やこの記事の内容については自由に転載や改変、翻訳・翻案などしてもらえればと思います。[CC0](https://creativecommons.jp/sciencecommons/aboutcc0/)扱いとしておきます。
[^6]: BevyをoFの上位互換とした理由のもう一つが、oFやProcessingのRust言語版ともいえる[nannou](https://nannou.cc/)が、[Bevyプラグインとして再開発されようとしている](https://github.com/nannou-org/nannou/issues/953)という事情もあります。とはいえ、Bevyの描画エンジンの基盤となっている[wgpu](https://github.com/gfx-rs/wgpu)にも課題はありますし、ECSも万能ではありませんので、やはり適材適所が大事だと思っています。
[^7]: https://github.com/stride3d/stride/issues/63
