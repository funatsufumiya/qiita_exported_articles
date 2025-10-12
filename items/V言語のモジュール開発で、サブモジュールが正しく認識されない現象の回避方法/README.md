:::note warn
以下は V言語 0.4.12 時点の情報で、今後変更があるかもしれません。
:::

:::note warn
この記事でいうサブモジュールは `git submodule` のことではなくて、V言語の概念としてのモジュール内の別のモジュールのことです。
:::

## 先に結論 (TL;DR)

現時点では、モジュール内のexampleフォルダを `~/.vmodules` 以外の別フォルダにコピーして開発するしかない。~~シンボリックリンク等は不可。~~ **【追記】** `ln -s ~/Documents/mylib ~/.vmodules/mylib` **はOK**でした。逆はダメでした。

（あるいは、examples だけ `git submodule`にしたり別のリポジトリにしてしまう等。）

あわせて v-analyzer の最新版を使っているかも要確認。

https://qiita.com/funatsufumiya/items/4f2824d304f006d3a366

## ~/.vmodules でモジュール開発する上でのトラブル

V言語では外部モジュールは基本的に `~/.vmodules` 上に置かれていて、新しいモジュールを作るときはそこに新しいフォルダを作り、`v.mod`という空のファイルを置くことでモジュールとして認識されるので、そこからテストしていきます。[^2]

ただ、`example` フォルダなどを中に作ってモジュールを試そうとすると、現時点では場合によってエラーが発生します。

代表的なものは、**モジュール内に存在するはずのサブモジュールが正しく認識されない**というものです。

### 検証

https://github.com/funatsufumiya/v_mytestlib

例えば上記モジュールをインストールしてexampleを実行するとき、README通りの以下の手順ならうまくいくのですが [^1]:

```bash
$ git clone https://github.com/funatsufumiya/v_mytestlib ~/.vmodules/mytestlib

$ v run ~/.vmodules/mytestlib/example/main.v

# hello
# sub

$ v run ~/.vmodules/mytestlib/example2/main.v

# sub

```

相対パスで実行するとうまくいきません。

```bash
$ cd ~/.vmodules/mytestlib
$ v run example/main.v

# mytestlib.v:7:6: error: unknown function: mytestlib.sub.sub .
# 1 possibility: `mytestlib.hello`.
#     5 | pub fn hello(){
#     6 |     println("hello")
#     7 |     sub.sub()
#       |         ~~~~~
#     8 | }
# If the code of your project is in multiple files, try with `v example` instead of `v example/main.v`

$ v run example2/main.v

# example2/main.v:4:6: error: unknown function: mytestlib.sub.sub
#     2 |
#     3 | fn main(){
#     4 |     sub.sub()
#       |         ~~~~~
#     5 | }
# If the code of your project is in multiple files, try with `v example2` instead of `v example2/main.v

```

しかも実は、[v-analyzer](https://github.com/vlang/v-analyzer) 上の静的構文解析も、先の時点（`~/.vmodules` 以下にある時点）でうまくいきません。

![スクリーンショット 2025-09-23 16.12.01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/69a5865f-c754-4c25-85a8-a964a947276f.png)

## 解決策

`v run` 上だけで正しく認識されるだけでよければ、`v run ~/.vmodules/mytestlib/example/main.v` のように絶対パスで実行すれば正しい結果が得られます。（ここでもエラーが出る場合は、脚注 [^1] および `v up` などでV言語のバージョンを確認。）

ただ、これでは [v-analyzer](https://github.com/vlang/v-analyzer) 上の静的構文解析のエラーが回避できません。

:::note warn
v-analyzerは[この記事](https://qiita.com/funatsufumiya/items/4f2824d304f006d3a366)で最新版が正しくインストールされている前提です。そうでない場合はそもそも静的構文解析がうまくいきません。（私自身ここでハマりました… orz）
:::

ではどうすればいいのだろうと試行錯誤して、[vdev](https://github.com/rcqls/vdev)というツール（`v -path`の改変ツール）を使ってみたり（[フォーク](https://github.com/funatsufumiya/vdev)してみたり）、シンボリックリンクを作ったりしたのですが、~~結論からいうとうまくいきませんでした。~~ **【追記】** `ln -s ~/Documents/mylib ~/.vmodules/mylib` **はOK** でした。逆はダメでした。

~~現時点では、冒頭の結論 (TL;DR) にあるように、モジュール内のexampleフォルダを `~/.vmodules` 以外の別フォルダにコピーして開発するしかないようです。（フォルダのシンボリックリンクでは、v-analyzer は正しく認識しませんでした。）~~

`~/.vmodules` 以下にモジュール自体は置いて、V言語やv-analyzerにモジュールを認識させつつ、VSCodeではコピーしたファイルの方を開いてexampleをデバッグするという感じです。

![スクリーンショット 2025-09-23 15.42.02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/7235df2c-7dae-4ada-b4b1-0086019295e3.png)

~~フォルダのシンボリックリンクを使えればgitの管理などで都合が良いのですが、現時点では v-analyzer が正しく認識しないので仕方がありません。~~

（あるいは、examples だけ `git submodule`にしたり別のリポジトリにしてしまうといった感じで別管理にしてしまってもいいかと思います。）

おそらく、`v.mod` の検出方法とか、`~/.vmodules` の扱いなどに理由があるのだと思いますが、そもそもこうなってしまう原因は私にはよくわからず、今後のV言語やv-analyzerのバージョンアップに期待するしかなさそうです。（Rust言語の `cargo run --example xxx` みたいなものがあるといいんですけどね。）

## 余談

V言語のモジュール周りは2025年9月現在であまりしっかりドキュメントがまとまっておらず、仕様もはっきりせず、GitHubやDiscordでも同様の質問や意見が飛び交っている状況になっています。（特に `v.mod` ファイルへの混乱など。）

結局のところ公式のexampleや他のGitHubのリポジトリがどうしているのか真似してみたり、Discordなどから情報を得るしかないのですが、モジュールに関しては、他の言語のようにわかりやすい情報がまとまっているとは言い難いし、新旧の情報が入り乱れているし、ユーザ同士も自身のミスなのかバグなのか仕様なのかよくわからない状況で利用を続けているので、V言語自体がバージョン1.0に達していないので色々と混沌とした状況といえるかなとは思います。

ただ少なくとも私自身はこの検証過程でモジュールやサブモジュール自体は正しく動作することが確認できたし、最新のv-analyzerで補完等もちゃんと効くことがわかったので、一応安心してV言語を使っていけるんじゃないかなとは思っています。

今後、より良いモジュール開発の手順とか、現在水面下で開発が進んでいるとみられる、npmなどのようなモジュール依存関係の解決などが進んでいくといいなと思います。

[^1]: ただし、絶対パスで `v run` を実行した場合でも、その時に `cd` しているディレクトリが `~/.vmodules` 上のモジュール内だとうまくいかないので注意。
[^2]: https://zenn.dev/tkm/articles/de319625f17f25232636 にあるように、`v new` コマンドで新規作成すると確実ですが、`~/.vmodules` 以外に作るとフォルダ内のexample上でモジュール・サブモジュールが正しく認識されないことに現時点では変わりはありません。
