## 概要

Bevy Engineを初めて使う人にとって、Rust言語特有の壁があります。今回は特にハマりがちな点を3つ挙げてご紹介します。

## 前置き

[Bevy Engine](https://bevyengine.org/)は、ゲームエンジンと紹介されることが多いですが、個人的には[openFrameworks](https://openframeworks.cc/ja/)などと同様に、汎用的な開発フレームワークと捉えています。

Bevy Engineはとてもよく設計されていて、幅広い用途に活用できるのですが、良くも悪くも現状はRust言語専用に作られていて、Rust言語以外でも使えるようにしようとする動きはなくはないものの、現状ではRustで書かざるをえません。

Rustは、C++やJavaと比べても難易度は高く、Bevyは難易度を緩和する仕組みを設けてくれてはいますが、BevyとRustにはいくつかのハマりポイントがあります。

https://zenn.dev/msakuta/articles/40c1ad41b1c62e

当記事では、特にRust言語特有の事情に焦点をあて、Bevyに初めて触れる際に、特に抑えておくべき、Tipsともいえるコツをいくつか挙げていきます。

## 1. 可変参照はスコープで1つだけ

所有権は、RustをRustたらしめているものでありながら、プログラマを最も苦しめるものでもあります。

特に、他のプログラミング言語ではほとんど意識することがない事柄を意識せねばならず、意外な箇所でエラーに阻まれ、思ったようなコードが書けないことがあります。

https://doc.rust-jp.rs/book-ja/ch04-02-references-and-borrowing.html

今回は特に、 **「可変参照はスコープで1つだけ」** というルールを挙げておきます。下記のSlackOverflowのやり取りがわかりやすいので、コードを参考にして少し改変しながらみていきます。

https://stackoverflow.com/questions/51511114/borrowing-mutable-twice-while-using-the-same-variable

以下のコードは、`do_stuff`の2回目でエラーになり、コンパイルが通りません。

```rust
fn do_stuff(n: &mut usize) {
    *n += 1;
}

fn main() {
    let mut v = vec![1, 2, 3, 4];
    let current1 = &mut v[1];
    do_stuff(current1);
    
    let current2 = &mut v[0];
    do_stuff(current2);
    
    do_stuff(current1); // error[E0499]: cannot borrow `v` as mutable more than once at a time
    
    println!("{:?}", v);
}
```

```text
error[E0499]: cannot borrow `v` as mutable more than once at a time
  --> src/main.rs:10:25
   |
7  |     let current1 = &mut v[1];
   |                         - first mutable borrow occurs here
...
10 |     let current2 = &mut v[0];
   |                         ^ second mutable borrow occurs here
...
13 |     do_stuff(current1);
   |              -------- first borrow later used here

```

このケースの場合は、1回目の借用と2回目の借用で、スコープが被ってしまっているのが原因です。以下のように修正すればコンパイルが通るようになります。

```rust
fn do_stuff(n: &mut usize) {
    *n += 1;
}

fn main() {
    let mut v = vec![1, 2, 3, 4];
    {
        let current1 = &mut v[1];
        do_stuff(current1);
    }
    {
        let current2 = &mut v[0];
        do_stuff(current2);
    }
    {
        let current3 = &mut v[1];
        do_stuff(current3);
    }
    
    println!("{:?}", v); // [2, 4, 3, 4]
}
```

この、`&mut`を使いたいのにコンパイルが通らないというのが、Bevyを使っていて一番遭遇率が高いケースだと思いますので、これを知っているか否かでハマり時間がだいぶ変わると思います。

ちなみにどうしてもこれによりコンパイルが通らない場合、Bevyでは**システムを分割する**ことで対応できることが多くあります。

なおBevy固有の事項としては **クエリ (Query)** でも、同様の問題が発生することがあります。こちらは`Without`を利用することで回避できます。詳しくは下記記事を参照してください。

[[Rust] Bevyのはまりどころ - Componentの同時アクセス](https://zenn.dev/msakuta/articles/40c1ad41b1c62e#component-%E3%81%AE%E5%90%8C%E6%99%82%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9)

## 2. 再借用トリック: 可変参照と不変参照の共存

再び所有権まわりですが、先ほどの問題と似ていて次に多いエラーが、**一度immutable (不変) で借用するとmutable (可変) で借用できない**というものです。逆パターンのこともあります。

https://bevy-cheatbook.github.io/pitfalls/split-borrows.html

コード例：

```rust
#[derive(Component)]
struct MyThing {
    a: Foo,
    b: Bar,
}

fn my_system(mut q: Query<&mut MyThing>) {
    for thing in q.iter_mut() {
        helper_func(&thing.a, &mut thing.b); // ERROR!
    }
}

fn helper_func(foo: &Foo, bar: &mut Bar) {
    // do something
}
```

エラー内容：

```text
error[E0502]: cannot borrow `thing` as mutable because it is also borrowed as immutable
  --> src/main.rs:14:36
   |
14 |         helper_func(&thing.a, &mut thing.b); // ERROR!
   |         -----------  -----         ^^^^^ mutable borrow occurs here
   |         |            |
   |         |            immutable borrow occurs here
   |         immutable borrow later used by call

error[E0596]: cannot borrow `thing` as mutable, as it is not declared as mutable
  --> src/main.rs:14:36
   |
14 |         helper_func(&thing.a, &mut thing.b); // ERROR!
   |                                    ^^^^^ cannot borrow as mutable
   |
help: consider changing this to be mutable
   |
13 |     for mut thing in q.iter_mut() {
   |         +++

```

ご丁寧に解決策のヒントまで書かれているのですが、このヒントでは実は解決しません。（Rustはこういうことが多いのです…。）

本当の解決策は先程の記事にかかれていて、**再借用トリック**を使います。

```rust
for thing in q.iter_mut() {
    let thing = thing.into_inner();
    // または
    // let thing = &mut *thing;
    helper_func(&thing.a, &mut thing.b);
}
```

このトラブルの原因は、Bevyが提供する`Res<T>`や`ResMut<T>`などのスマートポインタに起因するものです。詳細は[元記事](https://bevy-cheatbook.github.io/pitfalls/split-borrows.html)に[解説](https://bevy-cheatbook.github.io/pitfalls/split-borrows.html#explanation)がありますので、脚注にその翻訳だけ載せておきます [^3]。

## 3. システムがどうしてもコンパイルを通らない

次に多いトラブルが、自作の関数をBevyのシステムとして `add_systems()` したいのにコンパイルを通せない、あるいは `.run_if()` の前の部分でエラーが出るというものです。

https://bevy-cheatbook.github.io/pitfalls/into-system.html

これはBevy固有の内容が多いので、上記記事の翻訳をそのまま載せておきます。

> 初心者にありがちなミス
> - `mut commands: Commands` の代わりに `commands: &mut Commands` を使う。
> - `Query<&MyStuff>` や `Query<&mut MyStuff>` の代わりに`Query<MyStuff>` を使用する。
> - `Query<(&ComponentA, &ComponentB)>` の代わりに  
 `Query<&ComponentA, &ComponentB>` を使用する（タプルを忘れる）。
> - `Res`や`ResMut`を使わずにリソース型を直接使用する。
> - コンポーネント型を`Query`を使わず直接使用する。
> - クエリでバンドル型を使用する。（個々のコンポーネントを使うのが正しい。）
> - 関数内で他の任意の型を使用する。
>
> ただし、エンティティーは特別なものであり、コンポーネントではないので、`Query<Entity>`は正しい。

上記には含まれていませんが、システムの引数が16個を超えると謎のエラーが出たりします。これについては別記事で解説しています。

https://qiita.com/funatsufumiya/items/4b3ba3b8b2d23eca7b05

また、`ResMut`にはしているものの`mut`を付け忘れるなんてこともよくあります。

この他、Rust固有ではないBevy Engineで抑えておくべきポイントについては、下記記事を参照してください。

https://qiita.com/funatsufumiya/items/a59603ac8d636362f3d7

## まとめ: Tipsを挙げるとキリがないけれど…

今回はあくまで最低限のTipsを挙げるならという観点で、3つだけ厳選してみました。

これだけで避けられるエラーは多いはずですが、他にもライフタイム周りなど、Rustはハマりどころが本当に多いです。

特にライフタイムについてはBevyの恩恵を一番受けられるところだと思いますが、ライフタイム周りのエラーについて挙げるとキリがなく、かつケースバイケースのことが多いので、それについてはまた別記事でまとめる機会があればと思います。

[^3]: 「再借用トリック」[解説文](https://bevy-cheatbook.github.io/pitfalls/split-borrows.html#explanation)の機械翻訳: <br> <blockquote>Bevy は通常、特別なラッパー型 ( [`Res<T>`](https://docs.rs/bevy/0.13.0/bevy/ecs/system/struct.Res.html), [`ResMut<T>`](https://docs.rs/bevy/0.13.0/bevy/ecs/system/struct.ResMut.html)や[`Mut<T>`](https://docs.rs/bevy/0.13.0/bevy/ecs/world/struct.Mut.html)(コンポーネントをミュータブルに問い合わせる場合) ) を使ってデータにアクセスできます。 <br><br> これにより、Bevyはデータへのアクセスを追跡することができます。これらはRustの[`Deref`](https://doc.rust-lang.org/stable/std/ops/trait.Deref.html)特性を使用する「スマート・ポインタ」型です。通常はシームレスに動作するので、気づかないことも多いです。 <br><br> しかし、ある意味、コンパイラにとっては不透明です。Rust言語では、構造体に直接アクセスできる場合は、構造体のフィールドを個別に借用できますが、構造体が別の型にラップされている場合は、これが機能しません。 <br><br> 上に示した再借用トリックは、効果的にラッパーを通常のRust参照に変換します。`*thing`は[DerefMut](https://doc.rust-lang.org/stable/std/ops/trait.DerefMut.html)でラッパーを参照解除し、`&mut` がそれをミュータブルに借用します。これで、`Mut<MyStuff>` の代わりに`&mut MyStuff` を持つことになります。</blockquote>
