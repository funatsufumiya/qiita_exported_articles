Flutterでスクロールバーを表示させたい場合、いくつか選択肢がある。が、「**常にスクロールバーを表示**させたい」なーと思って`ScrollBar`に`isAlwaysShown: true`を設定すると、なぜか途端にエラーの山にのまれて大変なことになるので、迷わないように定型化してみた。

## 結論

```dart
final _scrollController = ScrollController(); // ScrollControllerは必須

// 中略

Scrollbar(
  isAlwaysShown: true,
  controller: _scrollController, // <- 同じScrollControllerを配置

  child: SingleChildScrollView(
    controller: _scrollController, // <- 同じScrollControllerを配置
    
    // 以下、ScrollView内に配置したいWidget
    child: ListView( 
      // ListViewを配置する場合、以下2行は必須
      shrinkWrap: true,
      physics: NeverScrollableScrollPhysics(),
      children: [
        // ...
      ]
    )

  )
)
```

実際の動作は以下のDartPadで確認できる。
https://dartpad.dartlang.org/?id=d1330072fc1e7f2b9c9af52a1bd87edf&null_safety=true

<iframe src="https://dartpad.dev/embed-dart.html?id=d1330072fc1e7f2b9c9af52a1bd87edf"></iframe>

## 解説

特にListViewと併用する場合などに詰まってしまうパターンがよくあると思うので、まとめてテンプレ化した。

解説は以下参考ページに詳しく書かれているので割愛。`isAlwaysShown: true` する場合もしない場合も、このテンプレを基本にしておくと役立つと思うので、まとめてWrapしたWidgetにしておくと便利かもしれない。

同じページに複数のスクロール画面がある場合は、もちろん各場所でScrollControllerは分ける必要があるけれど、その場合も`Scrollbar`と`SingleChildScrollView`では共通のScrollControllerが必要。

## カスタムWidgetにまとめる

ということで、こんな感じにWidget化しておくと使いやすい。使うときは`SimpleScrollView`あるいは`ScrollListView`を呼べばよいので迷いがない。

```dart
import 'package:flutter/material.dart';

void main() => runApp(App());

class SimpleScrollView extends StatelessWidget {
  SimpleScrollView({required this.child, this.isAlwaysShown = true});

  final Widget child;
  final bool isAlwaysShown;

  final _scrollController = ScrollController();

  @override
  Widget build(BuildContext context) {
    return Scrollbar(
        isAlwaysShown: isAlwaysShown,
        controller: _scrollController,
        child: SingleChildScrollView(
          controller: _scrollController,
          child: child,
        ));
  }
}

class ScrollListView extends StatelessWidget {
  ScrollListView({required this.children});

  final List<Widget> children;

  @override
  Widget build(BuildContext context) {
    return SimpleScrollView(
        child: ListView(
            shrinkWrap: true,
            physics: const NeverScrollableScrollPhysics(),
            children: children));
  }
}

class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        home: Material(
      child: Scaffold(
        body: ScrollListView(children: [
          for (var i = 0; i < 50; i += 1) ListTile(title: Text("Item $i"))
        ]),
      ),
    ));
  }
}

```

DartPad
https://dartpad.dartlang.org/?id=bbd86b3e8515a4559255238b7588bf95&null_safety=true

## 参考

https://stackoverflow.com/questions/54963284/always-show-scrollbar-flutter?utm_source=pocket_mylist

https://qiita.com/tabe_unity/items/4c0fa9b167f4d0a7d7c2
