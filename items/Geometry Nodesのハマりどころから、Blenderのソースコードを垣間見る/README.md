## 本記事の目的

BlenderのGeometry Nodes（ジオメトリノード）には、いくつかハマりどころがあるのだけれど、それらのハマりどころについて、直感的に解説している記事は多いものの、ソースコードなどの出典が書かれている記事はかなり少ない。

そこで当記事では、Blenderのソースコード・リーディングのさわりとして、実際のGeometry Nodesの実装がどのソースコードに書かれているかなどの対応を確認していく。

:::note warn
筆者はBlenderのコミッターではないので、以下で書く考察については、あくまでソースコードに書かれたメモ（ドキュメントおよびコメント）やコードから類推しているに過ぎない。実際には読者自身がご自身の目でコードやIssue、メーリングリストやチャット等を確認されることを強く推奨する。
:::

## 利用するソースコード: GitHub

今回は、Blenderソースコードの公式ミラーである、GitHub版を使っていく。
[https://github.com/blender/blender](https://github.com/blender/blender)

Blenderには複数のバージョンがあるが、今回はGitHubのソースコード検索の関係で、執筆時点で最新のmain（執筆時点で v4.4.3 以降）を題材として取り上げる。（ただし、筆者Blender環境はv4.2 LTS。）

なお、手元で検索したい場合は、 `git clone --depth=1 https://github.com/blender/blender -b main` などを行い、VSCode等で検索を行うと良い。（定義ジャンプについてはこちらが便利。）

## Geometry Nodesとソースコードの対応

[source/blender/nodes/geometry/nodes](https://github.com/blender/blender/tree/50927827ff1c4da70ece03d609490b4cf0768cfe/source/blender/nodes/geometry/nodes) 以下に、各Geometry Nodeに対応するソースコードがズラッと並んでいる。

![スクリーンショット 2025-06-11 8.58.02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/ccbe17c0-ec28-4b11-b30d-ad43df0191c3.png)

ちなみにこれらの名前は、実際に利用する際の名前とは異なっているので、表示名で検索したい場合は、`ntype.ui_name = "Grid"` などで全文検索を行うと良い。

`ntype.ui_name = "Grid"` で実際に検索してみると、[node_geo_mesh_primitive_grid.cc](https://github.com/blender/blender/blob/50927827ff1c4da70ece03d609490b4cf0768cfe/source/blender/nodes/geometry/nodes/node_geo_mesh_primitive_grid.cc) が該当する。

## 前菜: Gridのソースコードに少し目を通してみる

最初の題材として、前項の流れで [Grid](https://docs.blender.org/manual/en/latest/modeling/geometry_nodes/mesh/primitives/grid.html) ([node_geo_mesh_primitive_grid.cc](https://github.com/blender/blender/blob/50927827ff1c4da70ece03d609490b4cf0768cfe/source/blender/nodes/geometry/nodes/node_geo_mesh_primitive_grid.cc)) をまず取り上げてみたい。

Geometry NodesのGridは、以下のようなものになっている。グリッド状のメッシュを生成するためのノードで、グリッドのサイズや頂点数を入力として受け付け、MeshやUV Mapを出力する。

![スクリーンショット 2025-06-11 9.04.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/22e0769a-a0f5-4df9-87df-40b66644d0fe.png)

[定義箇所に該当するコード](https://github.com/blender/blender/blob/50927827ff1c4da70ece03d609490b4cf0768cfe/source/blender/nodes/geometry/nodes/node_geo_mesh_primitive_grid.cc#L13-L37)（node_declare）を見てみると、確かにそのような記述がある。

```cpp
static void node_declare(NodeDeclarationBuilder &b)
{
  b.add_input<decl::Float>("Size X")
      .default_value(1.0f)
      .min(0.0f)
      .subtype(PROP_DISTANCE)
      .description("Side length of the plane in the X direction");
  b.add_input<decl::Float>("Size Y")
      .default_value(1.0f)
      .min(0.0f)
      .subtype(PROP_DISTANCE)
      .description("Side length of the plane in the Y direction");
  b.add_input<decl::Int>("Vertices X")
      .default_value(3)
      .min(2)
      .max(1000)
      .description("Number of vertices in the X direction");
  b.add_input<decl::Int>("Vertices Y")
      .default_value(3)
      .min(2)
      .max(1000)
      .description("Number of vertices in the Y direction");
  b.add_output<decl::Geometry>("Mesh");
  b.add_output<decl::Vector>("UV Map").field_on_all();
}
```

よく眺めてみると、`UV Map` にだけ、`field_on_all()` と書かれている。

Gridノードをよく見てみると、`UV Map`だけはひし形の形になっているようだ。

![スクリーンショット_2025-06-11_9_32_16a.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/ef14c95e-8b4c-48e9-8c80-3f558f9bab13.jpeg)

[field_on_all()の定義](https://github.com/blender/blender/blob/c463860a828b491e4a3b8edd9f4de18f6d65fe4c/source/blender/nodes/intern/node_declaration.cc#L662-L681)にジャンプしてみると、以下のような記述がある。今回は出力 = Output での利用であることに注目すると、これはどうやら出力が `Field` であることを明示するためのものらしい。

```cpp
BaseSocketDeclarationBuilder &BaseSocketDeclarationBuilder::field_on_all()
{
  if (this->is_input()) {
    this->supports_field();
  }
  if (this->is_output()) {
    this->field_source(); // <- 今回は出力なのでこちら
  }
  field_on_all_ = true;
  this->structure_type(StructureType::Field);
  return *this;
}

// field_source() の定義
BaseSocketDeclarationBuilder &BaseSocketDeclarationBuilder::field_source()
{
  BLI_assert(this->is_output());
  decl_base_->output_field_dependency = OutputFieldDependency::ForFieldSource();
  this->structure_type(StructureType::Field);
  return *this;
}
```

## ◆ハマりどころ1: Fieldとは何だろうか

さてここで、Geometry Nodesの最初のハマりどころともいえる、Fieldが早速登場した。

Fieldについての直感的な解説については、以下が詳しいので、併せて参照していただきたい。

https://www.ultra-noob.com/blog/2022/34/

では早速、Fieldの定義を見ていく。Fieldは、[`FN_Field.hh`](https://github.com/blender/blender/blob/50927827ff1c4da70ece03d609490b4cf0768cfe/source/blender/functions/FN_field.hh)に定義されている。

FN_Field.hh の冒頭に、以下のようなコメントがある。

```cpp
/** \file
 * \ingroup fn
 *
 * A #Field represents a function that outputs a value based on an arbitrary number of inputs. The
 * inputs for a specific field evaluation are provided by a #FieldContext.
 *
 * A typical example is a field that computes a displacement vector for every vertex on a mesh
 * based on its position.
 *
 * Fields can be built, composed and evaluated at run-time. They are stored in a directed tree
 * graph data structure, whereby each node is a #FieldNode and edges are dependencies. A #FieldNode
 * has an arbitrary number of inputs and at least one output and a #Field references a specific
 * output of a #FieldNode. The inputs of a #FieldNode are other fields.
 *
 * There are two different types of field nodes:
 *  - #FieldInput: Has no input and exactly one output. It represents an input to the entire field
 *    when it is evaluated. During evaluation, the value of this input is based on a #FieldContext.
 *  - #FieldOperation: Has an arbitrary number of field inputs and at least one output. Its main
 *    use is to compose multiple existing fields into new fields.
 *
 * When fields are evaluated, they are converted into a multi-function procedure which allows
 * efficient computation. In the future, we might support different field evaluation mechanisms for
 * e.g. the following scenarios:
 *  - Latency of a single evaluation is more important than throughput.
 *  - Evaluation should happen on other hardware like GPUs.
 *
 * Whenever possible, multiple fields should be evaluated together to avoid duplicate work when
 * they share common sub-fields and a common context.
 */
```

これを翻訳すると以下のようになる。（GPT-4oによるもの。誤りを含む場合もある。）

>このファイルは、任意の数の入力に基づいて値を出力する関数を表す#Fieldについて説明しています。特定のフィールド評価の入力は#FieldContextによって提供されます。
>
>典型的な例としては、メッシュ上の各頂点の位置に基づいて変位ベクトルを計算するフィールドがあります。
>
>フィールドは、実行時に構築、合成、評価することができます。これらは、各ノードが#FieldNodeであり、エッジが依存関係である有向木グラフデータ構造に格納されます。#FieldNodeは任意の数の入力と少なくとも1つの出力を持ち、#Fieldは#FieldNodeの特定の出力を参照します。#FieldNodeの入力は他のフィールドです。
>
>フィールドノードには2つの異なるタイプがあります：
>
>- #FieldInput: 入力がなく、正確に1つの出力を持ちます。評価時に全体のフィールドへの入力を表します。この入力の値は#FieldContextに基づいています。
>- #FieldOperation: 任意の数のフィールド入力と少なくとも1つの出力を持ちます。主な用途は、複数の既存のフィールドを新しいフィールドに合成することです。
>
>フィールドが評価されると、それらは効率的な計算を可能にする多機能手続きに変換されます。将来的には、以下のようなシナリオに対して異なるフィールド評価メカニズムをサポートする可能性があります：
>
>- 単一評価のレイテンシがスループットよりも重要である場合。
>- 評価がGPUなどの他のハードウェアで行われるべきである場合。
>
>可能な限り、複数のフィールドは共通のサブフィールドと共通のコンテキストを共有する場合に重複作業を避けるために一緒に評価されるべきです。

どうやら、Blenderのソースコードにはこのように丁寧なドキュメントが書いてあるらしい。ありがたい。（実際にはこれではなく、公式ドキュメントを閲覧したほうがわかりやすい場合も多いだろう。）

:::note info
なお、このコメントの意味がさっぱりわからなくても何の問題もない。以下で少しずつ深堀りしていく。
:::

## ◆ハマりどころ2: FieldとしてのIDやPositionはどうやって評価されているのか

さて、前項の内容（Field）の解説をするにあたって、例として、もう一つだけ別の題材を挙げておきたい。

Fieldの実体として、先に挙げた記事中でも紹介されている、IDやPositionなどの計算がわかりにくいという事例を例に挙げて深堀りしていく。

https://www.ultra-noob.com/blog/2022/34/

具体的に、以下のようなノードの接続があるとする。ここでは、ひし形のフィールド同士が接続されているようだ。

![スクリーンショット 2025-06-11 9.30.11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/67967fdb-e43b-4728-b886-197b9f880d06.png)

ここで先の記事で挙げられている疑問は、「PositionやIDというのは、何のPositionで、何のIDなのか」というもの。

では実際に、Positionノードの定義を見ていこうと思う。おさらいとして `ntype.ui_name = "Position"` でソースコードを検索すると、[node_geo_input_position.cc](https://github.com/blender/blender/blob/c463860a828b491e4a3b8edd9f4de18f6d65fe4c/source/blender/nodes/geometry/nodes/node_geo_input_position.cc) が該当する。

今回は定義部分（node_declare）ではなくて、node_geo_input_position.ccの[実際の処理コード](https://github.com/blender/blender/blob/c463860a828b491e4a3b8edd9f4de18f6d65fe4c/source/blender/nodes/geometry/nodes/node_geo_input_position.cc#L14-L18)（node_geo_exec）を見ていく。

```cpp
static void node_geo_exec(GeoNodeExecParams params)
{
  Field<float3> position_field{AttributeFieldInput::Create<float3>("position")};
  params.set_output("Position", std::move(position_field));
}
```

これを見ると実装自体はシンプルで、`AttributeFieldInput`というものを作って、出力に紐づけているだけのようだ。

`AttributeFieldInput` 自体は [`BKE_geometry_fields.hh`](https://github.com/blender/blender/blob/c463860a828b491e4a3b8edd9f4de18f6d65fe4c/source/blender/blenkernel/BKE_geometry_fields.hh) に定義されていて、 `GeometryFieldInput` を継承している。さらに、`GeometryFieldInput` は同ソース内で `fn::FieldInput` を継承している。

ということは、`XxxFieldInput` は、全て `fn::FieldInput` の派生クラスだと考えてよさそうだ。

なお、各 `FieldInput` の派生クラスには、それぞれ `get_varray_for_context` という関数が定義されている （ [親クラスである FieldInput の get_varray_for_context の定義はこちら](https://github.com/blender/blender/blob/c463860a828b491e4a3b8edd9f4de18f6d65fe4c/source/blender/functions/FN_field.hh#L282) ）。この `get_varray_for_context` という関数で、実際の処理内容が確認できる。

実例として、定義文が比較的短い、[IDノードの例](https://github.com/blender/blender/blob/50927827ff1c4da70ece03d609490b4cf0768cfe/source/blender/blenkernel/intern/geometry_fields.cc#L497C32-L510) (`IDAttributeFieldInput::get_varray_for_context`の実装) を見ていきたい。

```cpp
GVArray IDAttributeFieldInput::get_varray_for_context(const GeometryFieldContext &context,
                                                      const IndexMask &mask) const
{

  const StringRef name = get_random_id_attribute_name(context.domain());
  if (auto attributes = context.attributes()) {
    if (GVArray attribute = *attributes->lookup(name, context.domain(), CD_PROP_INT32)) {
      return attribute;
    }
  }

  /* Use the index as the fallback if no random ID attribute exists. */
  return fn::IndexFieldInput::get_index_varray(mask);
}
```

ここで注目したいのは、`context.domain()`というものを引数にとって、実際の計算をしていること。

このドメインの型となっている [`AttrDomain`の定義](https://github.com/blender/blender/blob/50927827ff1c4da70ece03d609490b4cf0768cfe/source/blender/blenkernel/BKE_attribute.hh#L62-L79)は以下のようになっている。

```cpp
enum class AttrDomain : int8_t {
  /* Used to choose automatically based on other data. */
  Auto = -1,
  /* Mesh, Curve or Point Cloud Point. */
  Point = 0,
  /* Mesh Edge. */
  Edge = 1,
  /* Mesh Face. */
  Face = 2,
  /* Mesh Corner. */
  Corner = 3,
  /* A single curve in a larger curve data-block. */
  Curve = 4,
  /* Instance. */
  Instance = 5,
  /* A layer in a grease pencil data-block. */
  Layer = 6,
};
```

つまり、`XxxFieldContext` (今回の場合は `GeometryFieldContext`) というフィールドのコンテクスト情報（文脈）を解釈し、コンテクストに応じたデータを返していることがわかる。

ここから類推するに、Field というのは何らかの情報の集合体で、FieldInput でコンテクストに応じて集合体の種類 (= どのFieldを返すべきか) を特定しつつ、必要な情報を受け渡す、一種の配列（バッファ）のようなもの[^1]であるというのがわかってくる。

## ◆ハマりどころ3: Geometry Nodesはどういうふうに計算されているのか

ここで、そもそもの疑問が湧いてくる。FieldInput がコンテクストに応じて情報を返すのであれば、Geometry Nodesというのは一体どんな順序で計算されているのだろうか。

これは、ノードの実行の実体が書かれている `node_geo_exec` 関数が、どんな呼び出し順序で呼ばれているかを丁寧に追っていくと、垣間見えてくる。

まず `node_geo_exec` 関数は、`node_register` 関数内で `ntype.geometry_node_execute` にアサインされている。[Gridノードでの実例](https://github.com/blender/blender/blob/c463860a828b491e4a3b8edd9f4de18f6d65fe4c/source/blender/nodes/geometry/nodes/node_geo_mesh_primitive_grid.cc#L69)を以下に挙げておく。

```cpp
static void node_register()
{
  static blender::bke::bNodeType ntype;

  geo_node_type_base(&ntype, "GeometryNodeMeshGrid", GEO_NODE_MESH_PRIMITIVE_GRID);
  ntype.ui_name = "Grid";
  ntype.ui_description = "Generate a planar mesh on the XY plane";
  ntype.enum_name_legacy = "MESH_PRIMITIVE_GRID";
  ntype.nclass = NODE_CLASS_GEOMETRY;
  ntype.declare = node_declare;
  ntype.geometry_node_execute = node_geo_exec; // <- この行
  blender::bke::node_register_type(ntype);
}
```

そしてこの `geometry_node_execute` は、LazyFunctionForGeometryNodeというクラスの中で実際には呼び出されている。 ([geometry_nodes_lazy_function.cc#L261](https://github.com/blender/blender/blob/c463860a828b491e4a3b8edd9f4de18f6d65fe4c/source/blender/nodes/intern/geometry_nodes_lazy_function.cc#L261))

この、LazyFunctionというのは何なのだろうか。[geometry_nodes_lazy_function.cc 冒頭の解説文](https://github.com/blender/blender/blob/c463860a828b491e4a3b8edd9f4de18f6d65fe4c/source/blender/nodes/intern/geometry_nodes_lazy_function.cc#L5-L21)を読んでみると、その一端が垣間見えてくる。

```cpp
/** \file
 * \ingroup nodes
 *
 * This file mainly converts a #bNodeTree into a lazy-function graph, that can then be evaluated to
 * execute geometry nodes. This generally works by creating a lazy-function for every node, which
 * is then put into the lazy-function graph. Then the nodes in the new graph are linked based on
 * links in the original #bNodeTree. Some additional nodes are inserted for things like type
 * conversions and multi-input sockets.
 *
 * If the #bNodeTree contains zones, those are turned into separate lazy-functions first.
 * Essentially, a separate lazy-function graph is created for every zone that is than called by the
 * parent zone or by the root graph.
 *
 * Currently, lazy-functions are even created for nodes that don't strictly require it, like
 * reroutes or muted nodes. In the future we could avoid that at the cost of additional code
 * complexity. So far, this does not seem to be a performance issue.
 */
```

GPT-4oによる翻訳（誤りを含む可能性あり）:

>このファイルは、主に#bNodeTreeを遅延関数グラフに変換し、それを評価してジオメトリノードを実行できるようにします。これは一般的に、各ノードのために遅延関数を作成し、それを遅延関数グラフに配置することで機能します。新しいグラフ内のノードは、元の#bNodeTreeのリンクに基づいて接続されます。型変換や複数入力ソケットなどのために、いくつかの追加ノードが挿入されます。
>
>#bNodeTreeにゾーンが含まれている場合、それらは最初に別々の遅延関数に変換されます。基本的に、親ゾーンまたはルートグラフによって呼び出される各ゾーンのために別々の遅延関数グラフが作成されます。
>
>現在、遅延関数は、リルートやミュートされたノードのように厳密には必要としないノードに対しても作成されています。将来的には、追加のコードの複雑さを代償にそれを避けることができるかもしれません。これまでのところ、これはパフォーマンスの問題にはなっていないようです。

わかったようなわからないような。

これだけでは少し掴みづらいので、実際にLazyFunctionを呼び出している[lazy_function_graph_executor.ccの冒頭の解説文](https://github.com/blender/blender/blob/bcfd276825f7791e3c1969309a16811d548dfdaf/source/blender/functions/intern/lazy_function_graph_executor.cc#L5-L21)も少しだけ読んでみることにする。

```cpp
/**
 * This file implements the evaluation of a lazy-function graph. It's main objectives are:
 * - Only compute values that are actually used.
 * - Stay single threaded when nodes are executed quickly.
 * - Allow spreading the work over an arbitrary number of threads efficiently.
 *
 * This executor makes use of `FN_lazy_threading.hh` to enable multi-threading only when it seems
 * beneficial. It operates in two modes: single- and multi-threaded. The use of a task pool and
 * locks is avoided in single-threaded mode. Once multi-threading is enabled the executor starts
 * using both. It is not possible to switch back from multi-threaded to single-threaded mode.
 *
 * The multi-threading design implemented in this executor requires *no* main thread that
 * coordinates everything. Instead, one thread will trigger some initial work and then many threads
 * coordinate themselves in a distributed fashion. In an ideal situation, every thread ends up
 * processing a separate part of the graph which results in less communication overhead. The way
 * TBB schedules tasks helps with that: a thread will next process the task that it added to a task
 * pool just before.

 （=== 後略 ===）

*/
 ```

 GPT-4oによる翻訳（誤りを含む可能性あり）: 

>このファイルは、遅延関数グラフの評価を実装しています。主な目的は以下の通りです：
>
>- 実際に使用される値のみを計算すること。
>- ノードが迅速に実行される場合はシングルスレッドを維持すること。
>- 任意の数のスレッドに効率的に作業を分散させること。
>
>このエグゼキュータは、FN_lazy_threading.hhを利用して、利益が見込まれる場合にのみマルチスレッドを有効にします。シングルスレッドモードとマルチスレッドモードの2つのモードで動作します。シングルスレッドモードでは、タスクプールやロックの使用は避けられます。マルチスレッドが有効になると、エグゼキュータは両方を使用し始めます。マルチスレッドからシングルスレッドモードに戻ることはできません。
>
>このエグゼキュータに実装されたマルチスレッド設計は、すべてを調整するメインスレッドを必要としません。代わりに、1つのスレッドが初期作業をトリガーし、その後、多くのスレッドが分散方式で自らを調整します。理想的な状況では、各スレッドがグラフの別々の部分を処理し、通信オーバーヘッドが減少します。TBBがタスクをスケジュールする方法がこれに役立ちます：スレッドは、直前にタスクプールに追加したタスクを次に処理します。

なるほど、なんとなくわかってきた。つまりGeometry Nodesというのは、単純に左から右に逐次実行されているのではなくて、必要な部分だけを必要に応じて計算する、いわゆる遅延評価が行われているようだ。（ということは、実際には左から右ではなくて、右から左に必要に応じて計算を深堀りしている、という見方もできるかもしれない。）

今回はこれ以上深くは深堀りしないが、さらに実装が気になる場合は、関連するソースコードを読んでいくと面白いかもしれない。

## ◆ハマりどころ4: Instance on Points で、Pick Instanceってどうやって実行されているの？

さて、この記事の趣旨は前項までにほとんど終えているので、**最後はちょっとしたオマケ**として、ある特定のGeometry Nodeのわからないところを深堀りする例として、軽くだけ触れていきたい。

Geometry Nodesでよく使われるノードに、Instance on Pointsというものがある。

![スクリーンショット 2025-06-11 10.22.28.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/6786f82f-d5b6-4819-a276-b502bb45c40c.png)

このInstance on Pointsは、入力として与えたポイントに、Instanceとして与えたものを（インスタンスとして）自動的に複製してくれるので、例えば草を生やしたり、同じものをとにかくたくさん増やしたいときに便利に使える。

このとき、同じインスタンスではなくて、別の種類のインスタンスも混ぜたい、ということがよくある。このとき使うのが `Pick Instance` という入力（フラグ、Boolean = 真偽値）となる。

※ なお、例のごとく機能の詳しい解説は行わないので、直感的な説明は以下を参照してもらいたい。

https://www.ultra-noob.com/blog/2022/33/

今回は軽く、このPick Instanceがどうやって機能しているかを見てみようと思う。

どのソースコードが該当しているか、まず簡潔に[該当箇所](https://github.com/blender/blender/blob/50927827ff1c4da70ece03d609490b4cf0768cfe/source/blender/nodes/geometry/nodes/node_geo_instance_on_points.cc#L119-L136)だけ挙げておく。

```cpp
    // (=== 前略 ===)
    
    const bool use_individual_instance = pick_instance[i]; 
    // メモ: ↑ ここでPick Instanceの値が使われている
    
    if (use_individual_instance) {
      if (src_instances != nullptr) {
        const int src_instances_num = src_instances->instances_num();
        const int original_index = indices[i];
        /* Use #mod_i instead of `%` to get the desirable wrap around behavior where -1
         * refers to the last element. */
        const int index = mod_i(original_index, std::max(src_instances_num, 1));
        if (index < src_instances_num) {
          /* Get the reference to the source instance. */
          const int src_handle = src_instances->reference_handles()[index];
          dst_handle = handle_mapping[src_handle];

          /* Take transforms of the source instance into account. */
          mul_m4_m4_post(dst_transform.ptr(), src_instances->transforms()[index].ptr());
        }
      }
    }

    // (=== 後略 ===)
```

正直、前後のコードをよく見てもらわないと意味がわからないとは思うのだけれど、確かに、Pick Instanceが有効になっている場合は、`handle_mapping`なるものを参照しながら、入力されたインスタンスから選び取って、出力先にハンドルを渡しているようだ。（今回はこれ以上深入りはしない。）

なおこのコードは、`add_instances_from_component`関数の一部なのだけれど、前述したノードの実装が書かれている`node_geo_exec`関数から内部的に呼び出されている。（これを辿れたのは、`Pick Instance`で検索して、それがどんなふうに変数に渡っているかを辿った結果。）

今回のこの短い項で伝えたかったのは、Geometry Nodesのなかで気になる機能があれば、ソースコード中で丁寧に機能を辿っていけば、該当箇所がソースに書かれているよ、ということ。こうやってできるだけ一次資料（原典）に触れることで、実装の実体を理解する一助になると嬉しい。

## まとめ

さて、今回は早足になったが、Geometry Nodesの実際のソースコードを、ほんの一部だけ眺めてみた。

Blenderのソースコードはかなり膨大なので、全てに目を通すのは事実上不可能だと思うのだけれど、例えばコアとなるBlenderカーネルには `BKE_` という接頭辞がついているとか、そうしたほんの少しの情報があれば、ちょっとずつでもソースコードの海を泳いでみることができるのではないだろうか。（難しいものを自分で解読するということ自体にも、楽しさがある気がする。）

なかなかこうした一次資料（原典）に触れるというのは、大変な作業なので気軽にはできないと思う。けれど、GitHubのソースコード検索機能なども年々使いやすくなっているし、こうした身近なところから、ソースコード・リーディングというものに触れてもらえれば、実体験と照らし合わせてコードが読めるので楽しいのではないかと思う。

[^1]: GVArray (= Generic Virtual Array) というのが Field に実際にアクセスする際のインターフェイスとなっている。GVArrayの実装は [BLI_generic_virtual_array.hh (#L30)](https://github.com/blender/blender/blob/13fa79aa37c647f7eb2856d5d338b95bf953e1cf/source/blender/blenlib/BLI_generic_virtual_array.hh#L30) に書かれており、仮想配列についての解説は [BLI_virtual_array.hh の冒頭の解説文](https://github.com/blender/blender/blob/13fa79aa37c647f7eb2856d5d338b95bf953e1cf/source/blender/blenlib/BLI_virtual_array.hh#L7C1-L26C4) が詳しい。以下にGPT-4oによる翻訳を挙げておく。<br><blockquote><p>仮想配列は、配列に似たデータ構造ですが、その要素は仮想メソッドを通じてアクセスされます。これにより、関数と呼び出し元の間の結合が改善され、関数はデータがメモリにどのように配置されているか、またはメモリに保存されているかを正確に知る必要がなくなります。データはその場で計算されることもあります。</p><p>仮想配列をパラメータとして受け取ることは、より具体的な非仮想型を受け取ることに比べていくつかのトレードオフがあります。個々の要素へのアクセスは、関数呼び出しのオーバーヘッドのために遅くなります。一方で、潜在的な呼び出し元はデータを関数に必要な特定の形式に変換する必要がなくなります。最終的にアクセスされる要素が少ない場合、この変換はコストがかかることがあります。</p><p>仮想配列を入力として受け取る関数は、異なるデータレイアウトに対して最適化を行うことができます。たとえば、配列が内部で連続したメモリを参照しているか、すべてのインデックスに対して同じ値であるかを確認できます。関数内で異なるデータレイアウトに対して最適化する価値があるかどうかは、ケースバイケースで判断する必要があります。コンパイル時間とバイナリサイズの増加がそれに見合うかどうかを確認するために、常にベンチマークを行うべきです。</p></blockquote>
