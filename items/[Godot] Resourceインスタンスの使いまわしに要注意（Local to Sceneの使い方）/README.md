## まず結論

Godotで使われるResourceは、Local to Sceneを適切に設定しないと、Materialなどのインスタンスは使い回されるので注意。

![スクリーンショット 2024-10-14 21.41.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/a6895633-92d8-1e80-7291-25060805d667.png)

## 概要

Godotを使っていると自然に気づくことなのだけれど、Materialなど、各ノードのパラメータ等で指定する`Resource`は、何もしなければ別のノードでも使い回される。

![スクリーンショット 2024-10-14 21.47.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/2aa9d980-9c37-4003-eff0-ed8283dedc0e.png)

↑Resourceの見分け方は、「> Resource」がついているのが特徴。

例えば、ある作成済みのノードをコピーして、元のノードのマテリアルを編集すると、新しいノードでもその値が使われる。つまり、**いわゆるシャローコピー (Shallow Copy) の状態**になっている。

これを避ける簡単な方法として、「ユニーク化」がある。

![スクリーンショット 2024-10-14 21.50.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/b7292419-8c93-3bbd-ea48-97a0d3d26f30.png)

ユニーク化すれば、別のリソースとなるので、別のノードで別のパラメータなどを指定しても、リンクすることはない。

これで万事解決か？と思っていると、実は**同じノードをPrefab的に、何度もリスポーンさせると、リソースの使い回しが発生する**。

## 検証用コード

検証のために、以下のような簡単なプロジェクトを作ってみる。（執筆時点でGodot 4.3にて検証。）

![スクリーンショット 2024-10-14 21.40.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/743dabcd-0625-7d0a-40aa-dcf1406e2d40.png)

まずメインシーンとなる `test_scene.tscn` は以下の通り。

![スクリーンショット 2024-10-14 21.40.10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/686d8605-2e88-2faa-f181-0c6869024591.png)

ここで `TestSceneController` (`test_scene_controller.gd`) は以下の通り。

```gdscript
class_name TestSceneController
extends Node

@export var vw: int = 640
@export var vh: int = 480

@onready var scene_root: Node2D = self.get_parent()

var is_first_time = true

# Called when the node enters the scene tree for the first time.
func _ready() -> void:
	pass # Replace with function body.


# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta: float) -> void:
	pass

func _spawn_obj() -> void:
	var res: Resource = load("res://test_node.tscn")
	var obj: Sprite2D = res.instantiate()
	obj.position = Vector2(randf_range(0, vw), randf_range(0, vh))

	if is_first_time:
		var mat: CanvasItemMaterial = obj.material as CanvasItemMaterial
		mat.blend_mode = CanvasItemMaterial.BLEND_MODE_ADD
		is_first_time = false

	scene_root.add_child(obj)

func _unhandled_input(event: InputEvent) -> void:
	if event is InputEventKey:
		if event.is_pressed():
			if event.keycode == KEY_A:
				_spawn_obj()

```

このコードは、`A`キーが押されるたびにランダムな位置に`TestNode`を作る。ただし、**初回だけ**マテリアルのブレンディングをADDにする。

ちなみに`TestNode` (`test_node.gd`) は、以下のようなシンプルなSprite2Dで、`CanvasItemMaterial`がマテリアルに設定してある。

![スクリーンショット 2024-10-14 21.40.28.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/a79af976-0e67-0e11-673f-dbe4cc7f7d40.png)

![スクリーンショット 2024-10-14 21.40.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/82bcf61e-d929-6acb-9665-3555013acfa1.png)

![スクリーンショット 2024-10-14 22.01.13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/4edcea53-b9ab-167f-5671-6e0ab06f24d5.png)

## 検証用コードの実行

さて、このコードを素朴に実行すると、次のようになる。

![スクリーンショット 2024-10-14 22.02.07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/0095e813-036a-a014-8fde-44dd870ea833.png)

初回だけにブレンディングをADDに変更したにもかかわらず、リソースが使い回された結果、すべてに適用されてしまっている。

しかも、これらのリソースはコピーされたわけではなく**同じインスタンスを指している**ので、一つの値を変えると残り全ても変わってしまう。

### Local to Sceneの有効化

では次に、Local to Sceneをオンにした上でもう一度実行する。

![スクリーンショット 2024-10-14 22.02.18.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/7c47b16c-82af-6437-d8ff-743bd0172d7c.png)

![スクリーンショット 2024-10-14 22.02.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/fbd9d45a-37d6-748e-abef-3e8a26fc6155.png)

今度は、ちゃんと初回だけがADDになって、2回目以降は通常のブレンディングになっている。

ちなみに Local to Scene が何を意味するかはドキュメントに書いてあり、今回の目的通り、各インスタンスが同じものを指すのではなくちゃんとコピーしたい場合に`true`にするとある。

![スクリーンショット 2024-10-14 21.41.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/fdce5601-8d28-e2f7-a82b-a13209417ff4.png)

（これを再帰的にやってくれるオプションがあれば、さらに子のリソースもディープコピー (Deep Copy) されて便利な気もするけど、そういうオプションってあるのだろうか…？）

## まとめ

今回はマテリアルを例として取り上げたものの、すべての`Resource`について同じことがいえるので、例えば動画・音声プレイヤーなどで再生対象のファイルを動的に変えたりするようなシチュエーションでは、注意が必要。

特に、ゲームの2回目の実行時は、一度使われたものを再度リスポーンすることが多いので、これにハマってしまうケースがあるので要注意。

他の言語のシャローコピーやディープコピーと同じく、どちらが良いかはケースバイケースなので、例えば今回のようにコピーするのではなく、`ready()`のときに必要なパラメータをリセットするコードを呼ぶ、などとの使い分けが必要。
