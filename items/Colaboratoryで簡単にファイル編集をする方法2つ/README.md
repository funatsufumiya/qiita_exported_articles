機械学習やPythonの習得にとても便利な[Colaboratory](https://colab.research.google.com)ですが、実はファイル編集だけはちょっと面倒です。
そこで、今回はColaboratoryでファイル編集をする場合に便利な方法を2つピックアップしました。

- 【方法1】 Google Drive + Anyfile Notepad **(← おすすめ)**
- 【方法2】 bashコマンド(cat, sed) **(← サクッとファイル編集したいときに便利)**

# 【方法1】 Google Drive + Anyfile Notepad

Google Drive連携自体は、よく使われる方法ですね。これに加えて、[Anyfile Notepad](https://anyfile-notepad.semaan.ca/)というGoogle Driveアプリを使うことで、どんなファイルでも簡単に編集できるようになります。

（ちなみに、Anyfile Notepadを使わなくても、[Googleドライブファイルストリーム](https://www.google.com/intl/ja_ALL/drive/download/)等を使ってPCから直接編集することもできます。PC上のアプリでファイル編集したい場合はそちらがおすすめです。)

### Google Driveの認証とマウント

Google Driveをマウントするには以下のコマンドを使います。

```python
from google.colab import drive
drive.mount('/content/drive')
```

あとは以下のようなURLが表示されるので、URLを開き、最後に出てくる暗号のような文字列を入力欄にコピペすれば完了です。

![スクリーンショット 2019-10-03 13.08.49.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/154c9a9b-31b1-c8a3-b8a6-a05926a141c6.png)

### Anyfile Notepadのインストール

Anyfile Notepadのインストールは、ブラウザ版Google Driveの右クリックメニューから、アプリの追加を選択し、Anyfile Notepadを検索すればすぐインストールできます。

![スクリーンショット 2019-10-03 13.15.27.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/2e52d832-75bc-1d74-1063-24a69fbe9c61.png)

![スクリーンショット 2019-10-03 13.11.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/d05c4059-da87-43fc-ec6f-dd228b8c3428.png)

あとは、編集したいファイルを右クリックして、「アプリで開く > Anyfile Notepad」を選択すればOKです。簡単ですね。

![スクリーンショット 2019-10-03 13.15.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/c050e775-e52a-fc0f-65c3-cfee3b2b0fe3.png)

実際にファイルを開いてみた様子です。ここでは`LICENSE.txt`を開いてみていますが、どんな拡張子のファイルでも編集できます。

![スクリーンショット 2019-10-03 14.20.03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/d6b3c1b9-15b7-c009-479c-f8f2261c27e1.png)

ここで1点だけ注意ですが、**Google Driveでの編集が実際に反映されるのに、時間差があるときがあります。**

なお、この方法を応用すれば、画像ファイルやExcelファイルも追加アプリで直接編集できるので、いろいろと応用の効く方法ですね。

ちなみにGoogle Drive連携を使う方法には他にもいろいろメリットがあって、Colaboratory上のマシンが停止してもファイルはDriveに残るので、学習後に継続してファイルを使う場合等にとっても便利です。セッションが切れてもすぐ継続して学習を進められるのは嬉しいですね。

## 【方法2】 bashコマンド (cat, sed) 

Google Driveを使う方法でだいたいは事足りるのですが、ちょっとした編集のとき、毎回Google Drive上で編集するのは面倒です。（[Googleドライブファイルストリーム](https://www.google.com/intl/ja_ALL/drive/download/)を使えば多少は楽ですが。）

そんなとき、**Colaboratory上だけでサクッと**ファイル編集したいときは、以下の方法がおすすめです。

### ファイルの新規作成と、中身の確認 (catコマンド)

`%%bash`や`!`で始まるセルは、bashコマンド（Linux上のコマンド）になることを活用して、`cat`コマンドを使ってファイルを新規作成します。

```bash
%%bash
cat <<EOF > test.txt
hello
this
is
test
file
EOF
```

EOFからEOFの間は自由に記述できます。ファイルの中身を確認したい場合も、同じくcatコマンドを使います。

```bash
!cat test.txt # ちなみにcatは、catenate(連結)の意味
```

![スクリーンショット 2019-10-03 13.28.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/fbbdf814-6b31-c662-e85a-22ae8ff5a0a9.png)

大抵のテキストファイルの新規作成はこれで事足りますし、**ファイル全体を書き換えるときは同じ方法が使えます**。

（ちなみに蛇足ですが `!sudo apt install imagemagick` でImageMagickをインストールすれば、ちょっとした画像ファイルの作成や編集なんかもできます。もちろん`ffmpeg`とかも同様に使えますし、さらには`gcc`や`ruby`、`localtunnel`なんかもインストールできます。Colaboratoryは拡張性がすごいですね。）

### ファイルの書き換え (sedコマンド)

Colaboratory上では、EmacsやVim等の対話型エディタを開くことはできないので、`sed`コマンドを使います。

#### まず行番号の確認

まず編集する前に、先ほど作成した `test.txt`を、行番号付きで中身を確認します。

```bash
!cat -n test.txt # -n はnumberの意味
```

![スクリーンショット 2019-10-03 13.34.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/62306dda-c0d8-5b13-2a40-207e10e6bc4f.png)

もし表示される行数が多すぎる場合は、`head`コマンドや`tail`コマンドを使って表示行数を制限できます。（行の範囲を指定することもできますが、ここでは割愛します。）

```bash
%%bash
cat -n test.txt | head -n 2 # 最初の2行を表示
echo; echo # 空行を2つ表示
cat -n test.txt | tail -n 1 # 最後の1行を表示
```

![スクリーンショット 2019-10-03 13.37.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/3b3a2cb0-e351-c2bd-af04-07ed0164ed37.png)

#### 行の書き換え

では早速、`sed`コマンドを使って、2行目を`world`に書き換えます。

```bash
%%bash
sed -i -e "2c world" test.txt # ここで、2c は2行目を書き換える (change) の意味

cat -n test.txt # 結果の確認
```

![スクリーンショット 2019-10-03 14.15.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/c9c64e78-0af2-cd1a-c99d-713bd057eb52.png)


書き換わりました。行番号の確認さえできれば、簡単ですね。

（ちなみに`sed`は**s**tream **ed**itorの略です。`-i -e`オプションはそれぞれ、`--in-place --expression`の略で、ファイルの置き換えと実行コマンドを示すオプションの意味です。）

### 行の挿入

ちなみに挿入したいときは、`c` (change) の代わりに `i` (insert) や `a` (append) を使います。

```bash
%%bash
sed -i -e "2i great" test.txt # 2行目にgreatを挿入する

cat -n test.txt # 結果の確認
```

![スクリーンショット 2019-10-03 13.52.18.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/380aa468-d480-556e-23cd-e0abc04d90ee.png)

```bash
%%bash
sed -i -e "2a and wonderful" test.txt # 2行目の次に、and wonderfulを挿入する

cat -n test.txt # 結果の確認
```

![スクリーンショット 2019-10-03 13.53.10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/1c441895-193c-4bf9-193c-dd451e27bdc9.png)

### 行の削除

削除するときは、`d` (delete) を使います。

```bash
%%bash
sed -i -e "5,7d" test.txt # 5〜7行目を削除する

cat -n test.txt # 結果の確認
```

![スクリーンショット 2019-10-03 13.54.38.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/59bac9ab-d8d2-3799-8baf-14981f641b6d.png)

### 文字列の置き換え

文字列を置き換えるときは、`s` (substitute) を使います。

```bash
%%bash
sed -i -e "1s/hello/good morning/" test.txt # 1行目のhelloをgood morningに置き換える

cat -n test.txt # 結果の確認
```

![スクリーンショット 2019-10-03 13.57.32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/49808f13-c15a-533c-c7e7-8df73cfe52ad.png)

ちなみにファイル全体の中で置き換えることもできます。

```bash
%%bash
sed -i -e "s/d/D/g" test.txt # dをDに置き換える

cat -n test.txt # 結果の確認
```

![スクリーンショット 2019-10-03 13.59.00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/8996c695-82a6-80ab-e19d-484b26026f25.png)

便利ですね。

なんだか後半は単なる`sed`コマンド講座になってしまいましたが、`sed`コマンドはこういう対話シェルが使えない環境では本領発揮しますね。これを応用すれば、`awk`コマンドなんかも組み合わせて、テキストファイルをまとめて編集するときなんかに活躍しそうです。
