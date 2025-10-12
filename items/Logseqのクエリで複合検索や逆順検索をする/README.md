## 先に結論

TODOの最新30を取得し、優先度 (A,B,C) をアルファベット順、日付を降順（新しい順）にする例。

（記事の後半に[別解](#別解)も紹介。）

```cljs
#+BEGIN_QUERY
{:title "TODO (直近30)"
 :query (todo TODO)
 :result-transform (fn [result]
  (let [rev-pri (fn [p] (case p "A" "Z" "B" "Y" "C" "X" "Z" "B" "A"))]
   (->> result
    (sort-by (fn [r]
        [
          (rev-pri (get r :block/priority "Z"))
          (get (get r :block/page) :block/journal-day 19000101)
        ]
     ))
    reverse
    (take 30)
    ;; 日付デバッグ用
    ;; (map (fn [m] (update m :block/properties
    ;;   (fn [u]
    ;;      (assoc u
    ;;           :journal-day (get (get m :block/page) :block/journal-day)
    ;;       )
    ;;   ))))
    )))
 :breadcrumb-show? false
}
#+END_QUERY
```

## 解説

まず、`->>` はThread-Lastマクロと呼ばれていて、関数を順番に実行して適用するという意味。Thread-Firstマクロ (`->`) との違いは、引数が関数の最後に渡されるか最初に渡されるかの違い。

わかりやすくいえば次のように置き換えられる。

```cljs
(->> result fn_a fn_b fn_c)
; 上と下は同じ意味
(fn_c (fn_b (fn_a result)))
```

もう少し具体的な例を[ClojureDoc](https://clojuredocs.org/clojure.core/-%3E%3E)から引用しておく。

```cljs
;; An example of using the "thread-last" macro to get
;; the sum of the first 10 even squares.
user=> (->> (range)
            (map #(* % %))
            (filter even?)
            (take 10)
            (reduce +))
1140

;; This expands to:
user=> (reduce +
               (take 10
                     (filter even?
                             (map #(* % %)
                                  (range)))))
1140
```

これを踏まえると、`:result-transform` の中身は次のように分けて考えられる。

```cljs
(fn [result]
  ; 先に使いたい関数を定義
  (let [rev-pri (fn [p] (case p "A" "Z" "B" "Y" "C" "X" "Z" "B" "A"))]
   ; 得られた結果を、
   (->> result
    ; まずソートして
    (sort-by (fn [r]
        [
          (rev-pri (get r :block/priority "Z"))
          (get (get r :block/page) :block/journal-day 19000101)
        ]
     ))
    ; 逆順にして
    reverse
    ; 30個とってきて、
    (take 30)
    ; ついでに表示データに :journal-day を加える (※デバッグ用で、必須ではない。)
    (map (fn [m] (update m :block/properties
      (fn [u]
         (assoc u
              :journal-day (get (get m :block/page) :block/journal-day)
          )
      ))))
    )))
```

わかりやすく、ソートの部分だけを取り出しておく。

```cljs
(sort-by (fn [r]
    [
      (rev-pri (get r :block/priority "Z"))
      (get (get r :block/page) :block/journal-day 19000101)
    ]
 ))
```

ここでは、複数のソート条件を配列で渡している。（ちなみに`get`関数の最後の引数にある`Z`や`19000101`は、フォールバックというもので、見つからなかったときにその文字列で間に合わせるというやつ。）

ちなみに、`sort-by`や`reverse`を2回やってしまうと、結果全体が新規に書き換わってしまうのでうまくいかない。[Logseqフォーラム](https://discuss.logseq.com/t/sort-by-multiple-columns/6563/8)にもあるように、`juxt`等も同様の理由で今回は役に立たない。

そこで、ここでは作為的に、`rev-pri`なる、優先順位のアルファベットを逆転させる関数を先に`let`で定義して、`A`->`Z`, `B`->`Y`... と先に置き換えておくことで、逆順ソートを実現している。

Thread-Lastマクロの最後で、結果を逆転=`reverse`しているので、最終結果としては、**優先度 (A,B,C) がアルファベット順、日付が降順（新しい順）** になるようにソートされる。

なお、今回は「優先度」を配列の先に置いているので、「優先度」が優先されてソートされるが、逆にすれば日付が優先される。これは例えば`DONE`などの終わったものを順に表示させるときに良いと思う。

ちなみに今回は`sort-by`に配列を渡したけれど、配列ではなくて文字列結合(`str`)でも可。こちらのほうが内部で何が起きているかわかりやすいと思う。

## 別解

`journal-day`が`20241001`のような数値で返ってくることを活かして、

```cljs
(sort-by (fn [r]
    [
      (get r :block/priority "Z")
      (- (get (get r :block/page) :block/journal-day 19000101))
    ]
 ))
```

として `reverse` を取り除いても良い。

ただこの方法はマイナス値として扱えるような数値 (つまり整数) が来た場合に限る。

## 余談

複合検索とか、逆順検索みたいなのは、標準機能としてあってくれてもいいかなーと若干思う。

ただ、アルファベットを数値にする方法がわかれば[^1]、例えば日付の逆順なんかはマイナス値にすれば良いので、より汎用的な方法はありそう。（ただ、数値とかアルファベット1文字とかじゃない、`2024-10-01` みたいな非数値の文字列が現れたときはたぶん悩む。）

（追記）Logseqフォーラムに[Feature Request](https://discuss.logseq.com/t/combined-sort-function-on-query/29673)も一応出してみた。もしかしたらより良い方法があって教えてもらえるかも。[^2]

[^1]: `cljs`なので`int`関数は使えないし、Logseqは使える関数が制限されている様子。「優先度」(A,B,C) のように特定の文字列であれば、数値に変換できる範囲に変換してから`read-string`する等もできる…？

[^2]: ただ、執筆時点現在LogseqはDB版という新しい機構に開発リソースを割いている様子なので、もしそちらが公開されればクエリなどの方法も刷新されそう。その際はこの議論は古くなってしまう可能性も。
