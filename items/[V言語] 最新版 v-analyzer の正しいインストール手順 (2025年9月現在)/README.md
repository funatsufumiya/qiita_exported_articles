VSCode等でV言語の開発をする上で必須になってくる [v-analyzer](https://github.com/vlang/v-analyzer) ですが、歴史的経緯でリポジトリが複数あり、かつマーケットプレイスで配布されているものが古い方のリポジトリという事情があって、混乱が見られるので、以下に2025年9月現在の最新手順を記載します。

## まずv-analyzerのインストール

Googleでv-analyzerと検索すると、[v-analyzer/v-analyzer](https://github.com/v-analyzer/v-analyzer) がヒットしますが、**こちらはメンテナンスされていない古いリポジトリです！**

現時点でメンテナンスされている新しいリポジトリは、[vlang/v-analyzer](https://github.com/vlang/v-analyzer) の方です。

https://github.com/vlang/v-analyzer

インストール手順は、[README.md#installation](https://github.com/vlang/v-analyzer?tab=readme-ov-file#installation)に書かれているように、

```bash
v download -RD https://raw.githubusercontent.com/vlang/v-analyzer/main/install.vsh
```

でインストールされ、このコマンドを実行すると最後に、

```
For example in VS Code settings.json:
{
    "v-analyzer.serverPath": "/Users/fu/.config/v-analyzer/bin/v-analyzer"
}
```

のように `settings.json` への記載事項が表示されます。これをコピペしておきます。

なお、必須ではないのですがv-analyzerをPATHに通しておくことで、`v-analyzer up` で最新版に更新したりできて便利です。

## VSCode Extensionのインストール

次にVSCode Extensionをインストールします。[vlang/v-analyzer](https://github.com/vlang/v-analyzer) からも古いリポジトリから配布されているものがリンクされていて、Extensionは古いもので大丈夫のようです。（つまり、普通に検索して出てくるv-analyzerでOK。）

https://marketplace.visualstudio.com/items?itemName=VOSCA.vscode-v-analyzer

## settings.json の編集

最後に、v-analyzerインストール時に表示された以下の部分（**環境やユーザによって違います**）をsettings.jsonに設定して完了です。

```json
"v-analyzer.serverPath": "/Users/fu/.config/v-analyzer/bin/v-analyzer"
```

## 検証: 古いものと新しいものの違い

古い方のv-analyzerはメンテされていないので、いろいろとバグがあったりするのですが、特に使っていて困っていたのはサブモジュールの扱いでした。

https://github.com/funatsufumiya/v_mytestlib

例としてこのモジュールをインストールしたあと、`~/.vmodule` 以外のどこかのリポジトリに適当に、

```v
import mytestlib.sub

fn main(){
	sub.sub()
}
```

と書いてVSCodeで見てみると、新しいv-analyzerではきちんとサブモジュールが検出され、補完できたり、docが表示されたり、定義にジャンプすることができます。

![スクリーンショット 2025-09-23 11.20.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/2d8cf97d-8838-4709-bed6-cc0246f320c1.png)

一方で古い方では、サブモジュールが正しく検出されないので、同様の現象で困っている人はぜひ試していただければ。
