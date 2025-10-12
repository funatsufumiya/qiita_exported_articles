以下、2025年8月時点の備忘録。同様の主旨の記事は多くありながら、どれでも解決できなかったケースについてまとめた。

## 先に結論（TL;DR）

オプション機能から公式にインストールできるならそれが一番良い。できないなら[winget](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)か[chocolatey](https://qiita.com/kujiraza/items/4cd5a2db98a9c5bdc21a)から入れる。GitHub版を直はあまりおすすめしない。

## 1. そもそもOpenSSHサーバが有効化できない

一番ハマるのがこれで、このあとの対応で明暗が分かれることが多い。

WSUS、グループポリシーの変更などでインストールできるようになることもあるので、それで解決できる場合はOK。Windows Updateなどで解消できる場合もある。

自分の場合はその他Dockerなどの兼ね合いから、Windows HomeではなくWindows ProであればOpenSSHサーバのインストールも不思議とうまくいくことが多かったので、それで難を逃れていたこともあった。

以下で扱うのは、どうしてもインストールできなかった場合。

## 2. GitHub版から直でOpenSSHをインストールして、実際のSSH接続がうまくいかない

1で真っ先に考えるのは、いわゆるGitHub版からインストールするということ。

https://qiita.com/ShortArrow/items/a17a2071fe9cfffe4dbb

1の中で書いた「インストールできる」ケースは、「公式版」や「オプション機能版」などと呼ばれていて、中身は実は同じものではない。

GitHub版からインストールすると、[Releases](https://github.com/PowerShell/Win32-OpenSSH/releases)にあるのがPreview版やBeta版ばかりでありどれを選べばいいかわからないため、実際はSSH接続に失敗するなどのトラブルに見舞われることがある。そして、どれならうまくいくかなどは、正直、OSのバージョンとの組み合わせが様々にあって、ノウハウのレベルになってくるし、[Win32-OpenSSH Issue#1317](https://github.com/PowerShell/Win32-OpenSSH/issues/1317) にある以下の図にあるように、Windowsから配布されているOpenSSHサーバは、必ずしもGitHubの内容とは一致しない。

![50652818-88dab600-0f3c-11e9-9523-30f3474f5824.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/d7b26381-0c4b-410b-929b-1b292d594063.png)

で、それも踏まえてとりあえず適当なGitHub版を入れたとする。それで万事うまくいったなら何も問題ない。

ここからがハマりどころなのだけれど、設定したあとにSSH接続がうまくいかなかったりして、 `ssh xxxx -v` でログを見ると `debug1: SSH2_MSG_KEXINIT sent` などでハングしたりして、その解決策としてMTUの変更や、インストールパスをProgram Files以外にすると良いなどの記事が出てくる。が、実際はこれでは直らないことがある。

## 3. wingetやchocolateyなどからインストール

これが結論になるのだけれど、**GitHub直よりもマシというレベルの話で**、wingetやchocolateyから入れると良い。例えばwingetから入れる場合は[Install Win32 OpenSSH - Install using WinGet](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH) などを参照。

正直このレベルで詰まってしまうPCの場合は、wingetも入っていないとかうまく入れられないようなPCも多いので、その場合はchocolateyを使うほかない。

wingetやchocolateyを使えば、GitHub直から入れたときに、手持ちのPCに対してハズレを引く確率は減る。もちろん時期によっては良くないリリースを引く可能性もあるけれど、インストール時に `-pre` や `Preview` などを付けない限りは、自分でバージョン選択するより安定したバージョンを選択してくれる確率が高い。

例えばchocolateyから入れる場合は以下のようなコマンドになる。（詳しくは https://qiita.com/kujiraza/items/4cd5a2db98a9c5bdc21a などを参考。）

```bash
choco install openssh -params '"/SSHServerFeature"' -confirm
```

ただ注意が一つあって、このコマンドも結局GitHub版のどれかをとってきてインストールスクリプトを実行するので、それまでの試行錯誤でPATHなどがめちゃくちゃになっていると、うまくいかないことも多い。そのため、実行前に過去のOpenSSHを削除したりアンインストールスクリプトを実行したり、PATHからOpenSSHを削除するなどを推奨。

… MTU設定などでドツボにはまってしまう人がこれで一人でも減ってくれたら幸いだなと思う。
