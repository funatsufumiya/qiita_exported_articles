## 結論

ターミナルから

```
$ xcodebuild -project xxx.xcodeproj -configuration Debug
```

すると、リンカー(`ld`)のエラーメッセージが見れる。

## 解説

Xcodeで、`linker command failed with exit code 1 (use -v to see invocation)` というエラーが出ると、`Reveal in Logs`でも、ビルド履歴（ナビゲータ領域[^1]の一番右のアイコン）からも詳細が見れなくて、どうしたらいいかわからず詰まってしまうことがあります。

そういうときはターミナルから使える、いわゆるCLIコマンドを打つと、より詳細なエラーログが見れることがあります。（Xcodeプロジェクトの場合は上記のとおりですが、場合によっては`cmake`や`make`など、別のビルドコマンドが必要な場合があります。）

さらに、`Build Settings`の`Other Linker Flags`などに`-v`を指定すれば、より詳細なエラーログを見れます。

[^1]: ファイルツリーとか見るところ
