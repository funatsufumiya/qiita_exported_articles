M1 MacではRosetta 2を使って x86, x86_64 (以下x64) のIntelアプリやライブラリを実行できるのは有名だが、ターミナルを切り替えて使うことで、ARM非対応のツールやライブラリを使ってIntel版の開発を進めることができ、かつARM版と共存させることができる。

ARMとIntelのターミナルを切り替えて使う記事は既にあるものの、2021/2/5にARMに正式対応したHomebrew 3.0.0以前の記事しか見当たらず、**Homebrew 3.0.0以降の正式対応後の手順と設定**を改めてまとめてみることにした。

## iTermをインストール

デフォルトのターミナルでも良いのだけれど、後述の見た目を分けることができなかったり、名称変更後のターミナルがSpotlightで呼び出しにくいなどで不便が多いので、[iTerm](https://iterm2.com/)をインストールしている前提で以下説明する。

## Intel版 (x64版) のターミナルを作る

`/Applications/iTerm.app` をコピーして、「iTerm_x64.app」などに名称変更する。（ターミナル.appの場合は、`/Applications/Utilities/Terminal.app`を複製。）

![スクリーンショット 2021-08-12 15.26.45.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/3cf0e9c1-d3bd-43a8-ba70-a6616240ae39.png)

複製した方の「iTerm_x64.app」を右クリックして「情報を見る」を選択し、「Rosettaを使用して開く」にチェックを入れる。

![スクリーンショット_2021-08-12_15_23_53u.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/41488ec1-34d6-8acd-0cef-fcd266d78cce.png)

## ターミナルの見た目をそれぞれで変える

２つのターミナルをそのまま使うと非常に紛らわしいので、見た目を変える。`.bash_profile`などから切り替えてもいいのだけれど、簡単な方法として、プロファイルの背景色を変えるのがおすすめ。

![スクリーンショット 2021-08-12 15.30.45.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/b4f0f36e-0b98-aa84-3633-483855f7889f.png)

変えたい方のターミナルを一度起動して、メニューの iTerm > Preferences から、Profiles > Colors > Background で変更できる。（ターミナル.appだと、この設定を変えると両方に反映されてしまう。）

**追記:** プロファイルが一つだと、iTermでも設定が共有されてしまうことがあるようなので、ちゃんと切り替える方が良いみたい。２つプロファイル、例えば「ARM」と「Intel」を作成して、以下のように.bash_profileに追記することで自動切り替えを実現する。

```bash:.bash_profile
alias change_profile='(){echo -e "\033]1337;SetProfile=$1\a"}'

if [ "$(uname -m)" = "arm64" ]; then
  # arm64
  change_profile ARM
else
  # x86_64
  change_profile Intel
fi
```

新規プロファイルを作る際は、Defaultのプロファイルを「Other Actions...」から複製して、Generalタブから名称変更するのが楽。

ちなみに上記設定を追記すると、最初の実行時に警告が出るので、Always Allowを選択する。

![スクリーンショット 2021-08-18 21.13.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/15f85a8c-96d8-52ed-06d8-0b808da16dd7.png)

後でこの設定を変更したい場合は、Preferences > Advanced から、 Show only non-default valuesにチェックを入れ、Prevent control sequences from changing the current profile? の値を変更する （デフォルトはUnspecified）。

![スクリーンショット 2021-08-18 21.13.17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/e791c768-2312-a826-5009-ce2b24a00076.png)


## ちゃんとx64になっていることを確認

それぞれのターミナルを起動して、`uname -m` を実行すると、ARM版では`arm64`、Intel版では`x86_64`と表示される。

![スクリーンショット 2021-08-12 15.55.45a.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/932d55c0-02bc-1bc0-0eb2-f0e31b98aa69.png)


## Homebrewをそれぞれでインストールする

次に、各ターミナルを起動して、Homebrewを**それぞれで**インストールする。（ARM版のHomebrewを既にインストール済みの場合は、Intel版のターミナルでのみインストールする。）

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Homebrew 3.0.0以降でARM版に正式対応したので、ARM版ターミナルでは `/opt/homebrew`、Intel版では `/usr/local/` 以下にそれぞれ自動判別してインストールされる。

最後にパスの設定を促されるのだが、今回は共存させる必要があるので、bashの場合は`~/.profile`、zshの場合は `~/.zprofile`に以下のように設定する。（既に`eval "$(/opt/homebrew/bin/brew shellenv)"`が記載されている場合は置き換える。）

```bash:.profile
if [ "$(uname -m)" = "arm64" ]; then
  eval "$(/opt/homebrew/bin/brew shellenv)"
  export PATH="/opt/homebrew/bin:$PATH"
else
  eval "$(/usr/local/bin/brew shellenv)"
fi
```

これでそれぞれのターミナルを再起動すると、きちんとbrewがそれぞれのアーキテクチャで使えるようになっている。`which brew`でbrewのパスを確認できる。

![スクリーンショット 2021-08-12 15.55.45ax.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/29a9c702-e479-2161-5945-18a521b3fdfa.png)

あとは、それぞれのbrewはまるで別々のパソコンで使っているかのように別のフォルダに棲み分けされるので、必要なツールやライブラリはそれぞれでインストールする。

注意として、**パスが通っているとARMかIntelかに関係なく実行できる**のが、便利だが紛らわしい。ARMとIntelが混ざっていても問題ない場合もあるが、きちんとフォルダと設定を分けておいたほうが無難。

```bash:.bash_profile
if [ "$(uname -m)" = "arm64" ]; then
  # arm用の設定...
else
  # intel用の設定...
fi
```

## *envを棲み分けする

ARMとIntelで設定を分けたほうがいい具体例として、pyenv、nodenv、rbenvなどの*envがある。

### anyenvの場合

自分の場合はこれらをまとめて設定できる[anyenv](https://github.com/anyenv/anyenv)を使っているが、何も考えずに使ってしまうと`~/.anyenv`に両方混ざっておかしなことになるので、例えば以下のようにする。

```bash
git clone https://github.com/anyenv/anyenv ~/.anyenv_arm64
cp -r ~/.anyenv_arm64 ~/.anyenv_x64
```

```bash:.bash_profile
if [ "$(uname -m)" = "arm64" ]; then
  # arm64
  export ANYENV_ROOT="$HOME/.anyenv_arm64"
  export PATH="$HOME/.anyenv_arm64/bin:$PATH"
  eval "$(anyenv init -)"
else
  # x86_64
  export ANYENV_ROOT="$HOME/.anyenv_x64"
  export PATH="$HOME/.anyenv_x64/bin:$PATH"
  eval "$(anyenv init -)"
fi
```

### pyenvの場合

（anyenvを使っている場合は前述の設定だけで問題ないので、追加設定は不要。）

```bash
git clone https://github.com/pyenv/pyenv ~/.pyenv_arm64
cp -r ~/.pyenv_arm64 ~/.pyenv_x64
```

```bash:.bash_profile
if [ "$(uname -m)" = "arm64" ]; then
  # arm64
  export PYENV_ROOT="$HOME/.pyenv_arm64"
  export PATH="$HOME/.pyenv_arm64/bin:$PATH"
  eval "$(pyenv init -)"
else
  # x86_64
  export PYENV_ROOT="$HOME/.pyenv_x64"
  export PATH="$HOME/.pyenv_x64/bin:$PATH"
  eval "$(pyenv init -)"
fi
```

nodenvやrbenvなどの場合も、node-buildやruby-buildのcloneが追加で必要なくらいで、設定は同様。

## ARMとIntelが混ざってしまった場合

前述の通り、パスが通っているとARMかIntelか関係なく呼び出せてしまうので、アーキテクチャが混ざっていると困まったことがよく起こる。

コマンドの場合は、`which`や`where`などでどれが呼ばれているかを特定して、場合によって`lipo -info`や`otool`を使ってアーキテクチャを判断してケースバイケースで対処。

ライブラリの場合も同様で、`brew doctor`や`locate`などのコマンドで判別しつつ、`C_INCLUDE_PATH`や`LD_LIBRARY_PATH`などの環境変数を切り分けたり、brewなどで調整したりする。

いずれにしても、片方のアーキテクチャ**のみで**インストールされているツールやライブラリが問題になることが多いので、きちんと切り分けていれば大丈夫。

## arch -x86_64 をうまく使うと便利

ちなみに一度Rosettaがインストールされていれば、

```bash
$ arch -x86_64 uname -m
```

のように `arch -x86_64` をつけてコマンドを実行すればRosettaで実行でき、ちょっとしたことはこれで十分。（逆にARM64で実行したい場合は `arch -arm64e`。 ）

この方法を使えば、ターミナルごと分けなくても、

```bash
$ arch -x86_64 zsh
```

とか

```bash
$ arch -x86_64 bash
```

でRosettaのシェルに切り替えることもできる。これでも今までの設定はちゃんと読み込まれるので、どちらがいいかは好み。（ARCHでシェルを切り替える方が、ターミナルのGUI自体はネイティブに実行される分、電池消費はいいかもしれない。）
