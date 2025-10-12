![screenshot.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/2bafa837-39bc-044c-fd28-4689d3494914.jpeg)

ゲームエンジン、とタイトルにつけようか迷ったけれど、[minimax](https://github.com/roman01la/minimax)という名前の通り、ミニマリスト的なフレームワークなので、ゲームフレームワークとタイトルにはつけた。[^3]

ちなみに執筆時点の今現在では、本家はmacOSでしか動作確認が取られていなかったが、自分自身が動作確認と修正を行い、[こちらのフォーク](https://github.com/funatsufumiya/minimax/tree/for_pr)でWin/Mac/Linuxに対応させた。

個人的にはこのフレームワークがクロスプラットフォーム対応になったのは、ちょっと夢のようであって、Clojureで使えて、かつOpenGLではなく（OpenGLにも対応しているが）それ以降のモダン描画APIに対応したクロスプラットフォームなゲームフレームワークというのは、自分自身がずっと待望していたものだった。

## BGFX と LWJGL3 について

なぜ待望していたかの一つに、minimaxがLWJGL3を通して[BGFX](https://github.com/bkaradzic/bgfx)に対応しているというのがある。

https://github.com/bkaradzic/bgfx

BGFXというのは、モダン描画APIに対応しながらも、OpenGLのような気軽な使い勝手を目指して作られた描画ライブラリ。

そしてそのBGFXの持つクロスプラットフォームな特性から、[LWJGL3](https://github.com/LWJGL/lwjgl3)に含まれてJavaバインディングされるに至ったのだと思う。

https://github.com/LWJGL/lwjgl3

LWJGLといえば、[Minecraft（マインクラフト）](https://www.minecraft.net/ja-jp)（Java版）が使っているフレームワークであることで有名で、自分はLWJGL2だった時代のことしか知らなくて、今もMinecraftがLWJGLを使っているかどうかは知らないのだけれど、LWJGLはC++との親和性を強く意識して作られていて、Javaで書いたとしてもメモリの圧迫（ヒープ領域の圧迫）をしないようにメモリ管理が（必要なら）独自でできるように意識して作られているし、ライブラリのバインディングもほぼC++と似るように作られていて、C++版のサンプルコードを参考にしたりもできる。（[lwjgl3-demos](https://github.com/LWJGL/lwjgl3-demos)ももちろん良い参考になる。）

しかもここ数年のJavaのアップデートにより、[アリーナ・アロケータやスタック・アロケータ](https://docs.oracle.com/javase/jp/21/core/slicing-allocators-and-slicing-memory-segments.html)といった近代的で効率の良い、[Zig言語](https://ziglang.org/)等でも使われているメモリ管理手法がJavaに導入されているが、これとも相性が良いし、LWJGL3自身がメモリ関連の使いやすいユーティリティを提供してくれている。

LWJGLにはいろんなライブラリが含まれていて、必要に応じて取捨選択できるようになっている。例えばlwjgl-vulkanを使えばVulkan APIで開発を進めることもできる。(参考: [lwjglgamedev/vulkanbook](https://github.com/lwjglgamedev/vulkanbook))

そういう意味ではBGFXは選択肢の一つでしかないのだけれど、書きやすさと、MetalやDirectXやVulkanに自動で対応してくれる臨機応変さが、とてもありがたい。

ちなみにLWJGLを経由して使う利点としては、C++から使うのに比べてビルドが簡単というのがいえる。これは[Maven Central](https://mvnrepository.com/repos/central)や[Gradle](https://gradle.org/)などのビルドシステムの充実のおかげ。（minimaxでは、ClojureだけインストールすればOK。Clojureの持つビルドシステムで、Maven Centralから依存関係を自動解決する。）

一方でLWJGLを使うデメリットは、JavaでありながらJavaで書かれたライブラリではなくネイティブなものを使うので、環境依存の描画問題などが起こりやすいこと。リソース管理についても他のJavaライブラリに比べるとやや煩雑な分だけ、GCに依存しないようにできるなど自由度は高いが、諸刃の剣。（[自動でメモリリークを検出する機能](https://stackoverflow.com/questions/52704046/how-to-find-and-eliminate-memory-leaks-in-lwjgl)などはある。）

【追記】LWJGLの最大のデメリットに、Web対応ができないというものがある。これはJavaの最大の宿命かもしれない。（iOS対応はGraalVMでできるはずだけど、LWJGL自体が対応しきれるかは不明。）

## なぜClojureか (ほぼポエム)

（興味のある人だけ読んでもらえれば。次の章に進んでもらっても可。）

- [ハッカーと画家](https://www.amazon.co.jp/dp/4274065979/)
- [プログラミングClojure 第2版](https://www.amazon.co.jp/dp/4274069133/)

上記の本や、他の記事の方が詳しいと思うのだけれど、自分はClojureが過去のLispの中で最も実用的で最も完成度が高いと勝手に思っている。

Lispから見たらそうだけれど、Javaから見ると、Kotlinのような使い勝手で使えつつ、動的言語でありながら凄まじいパフォーマンスを誇る。こう考えるとJavaScriptみたいなもんだと思ってもらえればいいかもしれない。

ちなみにゲームエンジンをスクリプト言語で書きたければ、JavaScriptではなくて、C++と相性の良いLuaを使うのが普通だと思う。好きなフレームワークからLuaを呼んだり、Lua(JIT)から直接[LÖVR](https://lovr.org/)や[Love2D](https://love2d.org/)を使えば良いし、Luaなら[Fennel](https://fennel-lang.org/)という、Clojureにちょっと似たLispを使うこともできる。（[Godot](https://godotengine.org/)もGDScriptというすごく書きやすいスクリプト言語を持っているのでこちらもおすすめ[^2]。）

だけれどわざわざClojureを使う意味は、やっぱりClojureが書きたいからに他ならないかなと思う。Clojureの生産性の高さとREPL駆動開発による楽しさ、[哲学的なまでのシンプルさ](https://logmi.jp/main/technology/331309?n=1&e=321965)は、Clojureでしか味わえない。（ただ、minimaxのREPL連携はまだ未調査。）

Clojureには[babashka](https://github.com/babashka/babashka)や[nbb](https://github.com/babashka/nbb)という使いやすい気軽な環境がJVM以外にも[^1]あって、特にnbbはNode.jsのnpmが使えるので、[nvk](https://github.com/maierfelix/nvk)のようなVulkan実装を使っても楽しいかもしれないけれど、今のところ最も安定して使えるのはやはりJVMであるし、そうなると描画系はLWJGL3に行き着くと思う。

ちなみにClojureには[Jank言語](https://jank-lang.org/)という新しい亜種があって、これも期待しているのだけれど、最近公式ページに書かれているC++ interopではなくLLVMに移植され、過渡期のようなので、完成度が高まってきたらぜひ使いたいと思っている。

自分自身、Clojure（やJava）をネイティブに近い分野で実用するのは、GCがある関係で避けてきたのだけれど、Minecraftのような前例もあるし、近年のJVMの進化、[GraalVM](https://www.graalvm.org/)の存在、前述のアリーナ・アロケータやスタック・アロケータなどのJavaへの採用などもあって、そろそろ実用して良い頃合いなのではないかと思っている[^a]。

Clojure（およびJava）自体は、サーバウェアを書くにはたぶんハッピーな言語で、過去にはnbbに似た[lumo](https://github.com/anmonteiro/lumo)というClojureScriptのJSネイティブ実装もあったりした（nbbは[SCI](https://github.com/babashka/sci)というClojureの部分実装）ので、ブラウザ関係ではバリバリ使える言語だったはずで、だからこそClojurianは生き続けている (?) のだと思う。

Clojureをネイティブ寄りで使おうというのはちょっと挑戦ではあるけれど、ClojureScriptにも一応少し言及すると[bun](https://bun.sh/)が最近人気が出ていて、エッジ開発が強く意識されていて、C/C++との相性がとてもよく、nbbもbunと一緒に使うことができることから、将来的にはそちらへのシフトを念頭に置きつつ、JVMでいま使える選択肢として[minimax](https://github.com/roman01la/minimax)を挙げてみた。（将来的には、npmで使える選択肢が自分の中で出てくればとも思う。）

## minimaxコードリーディングツアー

このままminimaxについては何も書かずに終わろうと思ったけど、軽くコードリーディングツアーだけやっておく。（minimaxの特徴については[README](https://github.com/roman01la/minimax)に書かれているのであえて割愛する。）

なお、VSCode + [Calva](https://calva.io/) を使うと、定義にジャンプできるようになったり、エラーがチェックされたりして快適になるので、ぜひ手元にご準備の上。

冒頭部にあるスクリーンショットのコードは [src/fg/core.clj](https://github.com/roman01la/minimax/blob/main/src/fg/core.clj) にあたる。

個人的にClojureらしさが出ていると思う部分を以下に一部抜粋する。雲をふわふわさせたり、シーンを回転させている部分が中心。

いわゆる副作用があるコードは `do` や `doseq` で囲まれていて、状態変化には `atom` が使われている。（`vreset!` はatomを更新するためのマクロ。）

```clojure
;; src/fg/core.clj より一部抜粋

(def model
  (do
    (log/debug "Importing model...")
    (log/time (md/load-model "models/castle.glb"))))

;; debug
(def selected-object (atom nil))
(def debug-box @debug/debug-box)

;; scene
(def scene
  (atom (scene/create
         {:name "MainScene"
          :children [d-light (:scene model) debug-box]})))

(def castle-obj
  (obj/find-by-name @scene "castle_root"))

(def cloud-1-obj
  (obj/find-by-name @scene "cloud_1"))

(def cloud-2-obj
  (obj/find-by-name @scene "cloud_2"))

(defn render [dt t]
  (let [t (* t 10)
        pos1 (obj/position cloud-1-obj)
        pos2 (obj/position cloud-2-obj)
        y (-> (Math/sin (/ t 2))
              (/ 100))
        x (-> (Math/sin (/ t 10))
              (/ 100))
        z (-> (Math/cos (/ t 10))
              (/ 100))]

    (obj/rotate-y cloud-1-obj (* dt 0.3))
    (obj/set-position-y cloud-1-obj (+ (.y pos1) y))
    (obj/set-position-x cloud-1-obj (+ (.x pos1) x))
    (obj/set-position-z cloud-1-obj (+ (.z pos1) z))

    (obj/rotate-y cloud-2-obj (* dt -0.3))
    (obj/set-position-y cloud-2-obj (+ (.y pos2) y))
    (obj/set-position-x cloud-2-obj (+ (.x pos2) z))
    (obj/set-position-z cloud-2-obj (+ (.z pos2) x))

    (obj/rotate-y castle-obj (* dt 0.1))

    (vreset! (:visible? debug-box) (some? @selected-object))

    (when-let [obj @selected-object]
      (let [[root & objs] (reverse (obj/obj->parent-seq obj []))
            mtx (->> objs
                     (reduce
                      (fn [mtx obj]
                        (obj/apply-matrix* mtx (:lmtx obj) mtx))
                      (.set (Matrix4f.) ^Matrix4f (:mtx root))))]
        (debug/set-object-transform obj debug-box mtx)))))

(defn render-ui []
  (fg.ui.core/test-root @state/state @scene selected-object))
```

ちなみに `def` というのがいわゆるグローバル変数で、`let` は局所変数にあたる。`defn` は関数。

シーンの部分だけ以下に抜粋してさらにみてみると、ednという、JavaScriptでいうところのjsonで基本的には書かれていることがわかる[^4]。jsonと違うのは、キーは文字列ではなくキーワード (`:name` や `:children` など) になっていて、これはそのままEnum（列挙型）のように使えるスグレモノ。

```clojure
;; scene
(def scene
  (atom (scene/create
         {:name "MainScene"
          :children [d-light (:scene model) debug-box]})))
```

ちなみにシーン自体の定義は [`objects/scene.clj`](https://github.com/roman01la/minimax/blob/ffd42cf1af15f3f7bb77b6a0a5fdeb6ea3a90a7a/src/minimax/objects/scene.clj) にあって、あえて全文を掲載するけれど、定義は至ってシンプル。

（ここで出てくる `defrecord` というのは、クラスの代わりだと思ってもらって差し支えはない。`[name children mtx]` がパラメータ、つまりは変数みたいなものだと思ってもらえれば。 ）

```clojure
;; objects/scene.clj の全文

(ns minimax.objects.scene
  (:require [minimax.object :as obj]
            [minimax.objects.light]
            [minimax.util.scene :as util.scene])
  (:import (minimax.objects.light DirectionalLight)
           (org.joml Matrix4f)))

(set! *warn-on-reflection* true)

(defrecord Scene [name children mtx]
  obj/IRenderable
  (render [this id]
    (let [lights (obj/find-by-type this DirectionalLight)]
      ;; lights are rendered first to set up uniforms
      (run! #(obj/render % id) lights)
      (obj/render* id mtx children)))
  obj/IObject3D
  (add-child [this child]
    (obj/add-child* this child))
  (remove-child [this child]
    (obj/remove-child* this child))
  (children [this]
    children)
  (find-by-name [this name]
    (obj/find-by-name* this name))
  (find-by-type [this obj-type]
    (obj/find-by-type* this obj-type))
  (replace-by-name [this name obj]
    (obj/replace-by-name* this name obj)))

(defn create [{:keys [name children]}]
  (util.scene/add-parent-link
   (map->Scene
    {:name name
     :mtx (Matrix4f.)
     :children children
     :parent (volatile! nil)})))
```

最後の `create` 関数のところで、シーンを作るときと似たようなedn（つまりjsonもどき）があることに気付くと思うけれど、Clojureの文法および構成要素は非常に少ない。新しい文法に出会うことは、ユーザ定義のマクロを除いてほぼないと思ってもらっていい。

これが前述した[シンプル](https://logmi.jp/main/technology/321965)ということ。そしてこれが自分がClojureを書き続けたい理由。

## まとめ (?)

さて、minimax自体の特徴にはさほど言及しないまま、Clojureについてのポエムを書きたいだけ書いて終わった感が否めないけれど、きっと良さは伝わった (?) んじゃないかな。たぶん。

minimax自体は、GLTFが読み込めたり、PBRに対応していたり、UI表示に対応していたり、サウンドにも標準対応していたりと、minimaxといいながら結構至れり尽くせりな感じ。

冒頭でも紹介したように、[自分のフォーク](https://github.com/funatsufumiya/minimax/tree/for_pr)によってWin/Mac/Linux対応が完了していて、そのうち本家にマージされたらいいなと感じているので、ぜひ試してもらえると嬉しい。

## ライセンスについて

ちなみに、minimaxが採用している、準コピーレフトなライセンスである[Eclipse Public License (EPL v2) ](https://www.eclipse.org/legal/epl-2.0/)に抵抗のある人もいるかもしれないけれど、少し調べてみるとわかるように[LGPL](https://ja.wikipedia.org/wiki/GNU_Lesser_General_Public_License)よりもさらにゆるくて、基本的にコピーレフトが適用されるのはライブラリ部分のみ。商用利用等の派生作品などにはコピーレフトは伝播しない（ただしライブラリ部分のソース公開など一定の条件あり）ので、安心して使ってもらえればいいなと思う。

[^3]: GodotやUnityのようなエディタはminimaxには付いていないのだけれど、[Blenvy (Blender <-> Bevy)](https://github.com/kaosat-dev/Blenvy)と同じようなノリで、GLTF自体を使ってBlenderをエディタのように使うことはできるし、[ImGuiバインディング](https://github.com/kotlin-graphics/imgui/)も使える、はず。

[^2]: Godotには、[Kotlinバインディング](https://godot-kotl.in/en/stable/)や、実験的な[Kotlin/Nativeバインディング](https://godot-kotlin.readthedocs.io/en/latest/)がある様子。[Clojureから使っている例](https://github.com/tristanstraub/thecreeps-godotclj)も一応あった。さすがGodot、人気さがうかがえる。ちなみに自分はC++で拡張機能を書きつつGDScriptで書く元々のスタイルがシンプルで好きだけど、Godotはいろんな言語から使えるのがホントすごい。

[^1]: わかりやすく「JVM以外にも」と書いたけれど、babashkaはGraalVMでネイティブ化されているだけで基本的にはJVM。ただ、GraalVMのネイティブ化は、JVMと思えないくらいの快適さがある。

[^4]: JavaScriptのオブジェクトとJSONが厳密には異なるのと同じく、ednはそのままイコールClojureのデータ型とは異なるが、ここではわかりやすさを優先した。

[^a]: 【追記】後日談として、ここに書いていることは何も間違いではないのだけれど、検証を進めていくうちに、GC言語内で手動のメモリ管理をする辛さが理解されてきて、GDScriptなどの（準）参照カウンタベースの言語とはやはり根本的に違うというのは痛感しているところ。ZigやOdinやGoのような、deferなどがなく（追記: [あった](https://github.com/nijohando/deferable)）、参照カウントを使うにしてもある程度自作することになるのはちょっと大変そう（追記: これも一応[あった](https://github.com/cdeln/lexref-clj)）。ちなみにminimaxには、この辺を意識した [`mem/slet`](https://github.com/funatsufumiya/minimax/blob/808de37a302c42483fe95280a229de38f33e3b74/src/minimax/mem.clj#L16C11-L16C15) という使いやすいスタック・アローケータがあり、実際書いていくときはこのあたりを駆使していけば良いとは思う。同様の考え方で参照カウンタなどもすぐ実装できそうとは思う。
