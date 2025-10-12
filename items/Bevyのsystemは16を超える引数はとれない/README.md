## 結論: タプルあるいはSystemParamを使えば解決

https://github.com/bevyengine/bevy/blob/main/examples/ecs/system_param.rs

（追記）：以下に「16を超える引数はとれないよ」って書いてあって、対応策もちゃんと書いてあった。ただ実装上の関係か、タプルを使うと各16パラメータまでとのこと。（16 x 16 = 256なので、`SystemParam`を使わなくてもこれだけあれば十分。)

[https://bevy-cheatbook.github.io/programming/systems.html](https://bevy-cheatbook.github.io/programming/systems.html)

## 背景： systemに17以上の引数を与えると、意味不明なエラーが出る。

systemに互換のない引数を定義するとエラーが出る（詳細は下記）のだけれど、引数が16を超えると、互換があるはずなのに同様のエラーが出て、にっちもさっちもいかなくなる。特徴は、`'a, 'b, 'c, 'd, 'e, ...` が妙に多いことくらいで、実はこの数は16にはならないのでさらに解明を難しくする。

[https://bevy-cheatbook.github.io/pitfalls/into-system.html](https://bevy-cheatbook.github.io/pitfalls/into-system.html)

トレイトの定義も複雑なので、やっとのことで該当行にたどり着いた。

https://github.com/bevyengine/bevy/blob/fcd87b25281e7d1f6e857fb8096eea1ddfefb98c/crates/bevy_ecs/src/system/function_system.rs#L701

確かに `16` という数字がコード中にあり、これ以上の引数はとれないので `SystemParam` を使ってねという趣旨のメモが残されている。

```rust
// Note that we rely on the highest impl to be <= the highest order of the tuple impls
// of `SystemParam` created.
all_tuples!(impl_system_function, 0, 16, F);
```

Rustのタプル自体は数の上限はないはずなので、何かしら別の実装上の要因があってのことと思われるが、詳細は未調査。

もしかしたら今後改善されるかもしれないけれど、たぶん望み薄なので、引数が多いときは素直にタプルか `SystemParam` を使ったほうがよさそう。
