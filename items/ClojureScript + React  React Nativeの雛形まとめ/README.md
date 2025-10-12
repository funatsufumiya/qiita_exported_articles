ポイントとして、shadow-cljsを使うか、figwheelを使うかで分かれる。またReact Nativeの場合はExpoを使うか使わないかでも分かれる。

## React

### shadow-cljs

https://github.com/filipesilva/create-cljs-app

丁寧な日本語の解説記事があってわかりやすい。 

https://qiita.com/lagenorhynque/items/7c049f3c3b967ee777ac

#### ◎ 使い方

```bash
$ npx create-cljs-app hello_react
$ cd hello_react
$ npm install
$ npx shadow-cljs watch app
```

### figwheel

https://github.com/bhauman/figwheel-template

#### ◎ 使い方

```bash
$ lein new figwheel hello_react -- --reagent
$ cd hello_react
$ npm install
$ lein figwheel
```

## React Native

### shadow-cljs + Expo

https://github.com/PEZ/rn-rf-shadow

雛形生成コマンドではなく、サンプルプロジェクト。READMEがとてもわかりやすい。

#### ◎ 使い方

```bash
$ git clone https://github.com/PEZ/rn-rf-shadow
$ cd rn-rf-shadow
$ npm install
$ npx shadow-cljs watch app
# 最初のコンパイルが終わるのを待ち、別のタブを開いて
$ npm start
```

### shadow-cljs + react-native

https://github.com/tomthought/react-native-init-shadow

スター数が少ないのが少し気になる。

#### ◎ 使い方

```bash
$ npx react-native-init-shadow MyAwesomeProject
$ cd MyAwesomeProject
$ npm install
$ npx pod-install
$ npx shadow-cljs watch dev
$ npx react-native run-ios # or: npx react-native run-android  # or: npx react-native run-web

```

### figwheel

https://github.com/bhauman/react-native-figwheel-bridge

#### ◎ 使い方

react-nativeとExpoが初期化時に選べる。

```bash
# react-nativeの場合
$ npx react-native init MyAwesomeProject
$ cd MyAwesomeProject
$ npx react-native run-ios # or: npx react-native run-android  # or: npx react-native run-web

# expoの場合
$ npx expo init MyAwesomeProject
$ cd MyAwesomeProject
$ yarn ios # or: yarn android # or: yarn web
```
