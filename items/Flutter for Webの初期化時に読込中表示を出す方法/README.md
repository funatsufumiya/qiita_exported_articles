![Timeline-1.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/13d3c9a2-c7d0-0fc6-385a-86a93c65872b.gif)

Flutter 2.5.1現在、Fluuter for Webのデバッグ時には上のgifのような読込中のインジケータ（ローディング画面）が表示されますが、現時点では**デバッグ時のみ**でリリースビルド時には表示されず、表示させるオプションは見当たりません。

リリースビルド後、現時点で初期読み込み時間が10秒程度かかってしまうことはよくあることなので、**リリース時にもデバッグ時と同じ読込中表示を出す方法**をご紹介します。

（根本的な問題である、`main.dart.js`のサイズが大きい問題や、マテリアルアイコンのフォントサイズが大きい問題には今回は触れません。(参考: [main.dart.js is too large #46589](https://github.com/flutter/flutter/issues/46589)) ）

## 方針

読込中インジケータにはいろんな種類がありますが、今回は実際にデバッグ時に使われているものをそのまま表示させたいので、`packages/flutter_tools/lib/src/web/bootstrap.dart` に記載のコードを極力コピーして再利用します。

https://github.com/flutter/flutter/blob/master/packages/flutter_tools/lib/src/web/bootstrap.dart

実装としては、windowのloadイベントのタイミングでインジケータを消す方法がindex.htmlをいじるだけなので簡単ですが、それだと実際にFlutterのUIが表示されるまで案外ラグが大きいので、今回はFlutterのmain関数が呼ばれるタイミングで消えるようにしました。

## index.html

まず、`web/index.html`の`<body>`内に次を書き加えます。（styleとscriptは`<head>`内でも構いません。）

```html:index.html
<style type="text/css">
.flutter-loader {
    width: 100%;
    height: 8px;
    background-color: #13B9FD;
    position: absolute;
    top: 0px;
    left: 0px;
}
.indeterminate {
    position: relative;
    width: 100%;
    height: 100%;
}
.indeterminate:before {
    content: '';
    position: absolute;
    height: 100%;
    background-color: #0175C2;
    animation: indeterminate_first 2.0s infinite ease-out;
}
.indeterminate:after {
    content: '';
    position: absolute;
    height: 100%;
    background-color: #02569B;
    animation: indeterminate_second 2.0s infinite ease-in;
}
@keyframes indeterminate_first {
    0% {
        left: -100%;
        width: 100%;
    }
    100% {
        left: 100%;
        width: 10%;
    }
}
@keyframes indeterminate_second {
    0% {
        left: -150%;
        width: 100%;
    }
    100% {
        left: 100%;
        width: 10%;
    }
}
</style>

<script>
function flutterReady(){
    // console.log("flutter app is ready");
    var el = document.getElementsByClassName('loading')[0];
    if(el){ el.remove(); }
};
</script>

<div class="flutter-loader"><div class="indeterminate"></div></div>
```

## main.dart

次に、`main.dart`に以下の記載を加えます。

```dart:main.dart
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:js/js_util.dart' as js_util;
import 'dart:html' as html;

void main(){
  if (kIsWeb) {
    if (js_util.hasProperty(html.window, "flutterReady")) {
      js_util.callMethod(html.window, "flutterReady", []);
    }
  }

  // runApp(App()); などメインの処理
}
```

これで完成です。あとは実際にリリースビルドすれば、初回ロード時だけインジケータが表示されると思います。（もし`build`以下のindex.htmlが書き換わっていない場合は`web`以下からコピーすればOKです。）

一度この雛形を組んでしまえば、index.htmlを好きにいじることで自分の好きなローディング画面を出すこともできるので、何かと便利だと思います。
