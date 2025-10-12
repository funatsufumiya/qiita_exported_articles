## 前置き: Git LFSを自由に扱いたい！

Gitを使っていて、次のような経験はないでしょうか。

- ちょっと大きなファイルをGitでも管理したいけど、Git LFSの容量が少ない(高い)。
- わざわざGit LFSのために月額課金したくないけど、1GB未満じゃ少ない。
- レンタルサーバーは持ってる。or Google Driveは契約してる。or S3 (互換含む) は使える。
- VPSはちょっと仰々しいし、サーバ設定は面倒。
- Git自体の置き場は変えたくない (GitHubやGitLabをそのまま使い続けたい)。

そんなとき、Git LFSのCustom Transfer Agentが便利です。

https://github.com/git-lfs/git-lfs/blob/main/docs/custom-transfers.md

Custom Transfer Agentというのは、Git LFS用のサーバを立てることなく、LFSの処理をカスタムしたプログラムに処理させるもので、自由に処理ができる割に、単にJSONの受け渡しを標準入出力 (stdin/stdout) で行うだけなので、非常にシンプルで拡張性があります。

非常に自由度が高いので、例えば[lfs-folderstore](https://github.com/sinbad/lfs-folderstore)のように単純に別フォルダをLFS置き場にすることもできますし、SSHやGoogle Driveなど、自分の好きな場所をLFS置き場にできます。

また、LFSの部分だけをHookすることができるので、元のGitは別のサーバにできるので、例えばGitはGitHub、Git LFSはレンタルサーバーといった運用もできます。

## 運用例1: レンタルサーバーや自鯖をGit LFS置き場にする

https://github.com/funatsufumiya/git-lfs-agent-scp

例えば、SSH (SCP) と連携するCustom Transfer Agentを作ると、単純なレンタルサーバーや社内サーバ (自鯖) をLFS置き場にすることができます。

やり方は[README](https://github.com/funatsufumiya/git-lfs-agent-scp)に書いている通りで、まず[Releases](https://github.com/funatsufumiya/git-lfs-agent-scp/releases)から実行ファイルの入ったzipをダウンロードし、中に入っている実行ファイルを `/usr/local/bin` や `C:¥Windows¥System32` に置きます。(パスさえ通っていればどこでも構いません。)

そして自分のリポジトリの作業ディレクトリで、以下を実行します。

```bash
$ git config lfs.standalonetransferagent scp
$ git config lfs.customtransfer.scp.path git-lfs-agent-scp
$ git config lfs.customtransfer.scp.args myserver:/path/to/any/folder
```

これらはリポジトリフォルダ以下の `.git/config` に保存されますが、`git config -f .lfsconfig` と書くことで別ファイルに保存して共有することもできます。（ただしセキュリティの関係で、`.lfsconfig` をリポジトリに置いても `.gitignore` などのように自動で反映はされないので注意です。）

`myserver:/path/to/any/folder` の部分はSSH (SCP) が解釈できるものであればOKです。この場合、`~/.ssh/config` に myserver が登録されていて、SSHサーバ上に `/path/to/any/folder` があればアップロードとダウンロードがなされます。

ちなみに一度cloneしたリポジトリの場合は、この設定をしたあとに `git reset --hard` する必要があります。

これだけで自分の持っているサーバを、SSHだけでGit LFSサーバにできるのだから便利ですね。

ちなみにGit LFSのCustom Transfer Agent側には、oidと呼ばれるファイルのハッシュ情報くらいしか渡されないため、ファイルは基本的にハッシュ名で同じフォルダにひたすら置くだけのような実装になります。したがってリポジトリごとに整理するなどは自分でフォルダ指定などの引数で行う必要があることに注意です。

## 運用例2: Google DriveやS3をGit LFS置き場にする

https://github.com/funatsufumiya/git-lfs-agent-rclone

さっきの応用例で、こちらはscpの代わりにrcloneを使うようにしたバージョンです。使い方は先程と全く同じで、rcloneでGoogle DriveやS3、One Driveなど好きなクラウドストレージをセッティングしておき、 `myserver:/path/to/any/folder` のように指定するだけです。

## 自分の好きなプログラムをCustom Transfer Agentとして登録する

Custom Transfer Agentの可能性は無限大で、仕様もシンプルなのでどんな風にでも書けます。ちなみに以下のように書くとどんなプログラムでも指定できるので、PythonやNode.js、シェルスクリプトなど、自分の好きな言語でカスタムすることが可能です。

```bash
$ git config lfs.standalonetransferagent myapp
$ git config lfs.customtransfer.myapp.path /path/to/your/bin
$ git config lfs.customtransfer.scp.args <引数としてほしいもの...>
```

なので、例えば自分が作ったssh/rcloneの例を、単純にcpに変えるとローカルのフォルダにコピーすることもできますし、本当に自由にカスタムできまｓ（rcloneが何でもいけるので、正直それ以上は必要ないかもですけどね。）

登録したプログラムには、標準入力にJSONが来るのですが、それがたった4種類しかなくて、

- initが来たら空のJSON `{}` を返して応答
- terminateには応答しなくてOK
- downlaodはoidが指定されるのでファイルパスを返す (moveされるので消してもOKなパスを渡す)
- uploadはoidとコピーするべきパスが渡されるので、アップロード完了したらoidを返す。(progressを返すとなお良いが必須ではない。)

と、至ってシンプルな仕様になっていますので、自分でプログラム書くのは簡単です。(https://github.com/git-lfs/git-lfs/blob/main/docs/custom-transfers.md)

なお `git config` は `git config --local` の略なので、`git config --global` とするとグローバル設定にすることもできますが、引数設定などはリポジトリごとにしたいはずなので、ローカル設定とすることが多いはずです。

毎回 `git config` するのが面倒という場合は、この3行を自動で行うスクリプトを、自分用や社内用に作っておくといいかもしれません。

## まとめ: Custom Transfer Agentを使いこなして快適なGit LFSライフ

GitHubの場合、Git LFSの容量などで課金されているので、今回のテクニックを使うと自分の好きな別の場所をGit LFS置き場にすることができます。もちろんGit LFSサーバを自分で立てることもできますが、SSHやrcloneだけで好きなストレージを使うことができるというのは便利ですね。

また、繰り返しになりますがCustom Transfer Agentの仕様はシンプルなので、プログラムが気に入らない場合や自分でカスタムしたい場合も、さほど苦もなく好きなプログラムを好きな言語で書くことができます。

https://github.com/git-lfs/git-lfs/blob/main/docs/custom-transfers.md

プログラムの標準入力にわかりやすいJSONが来て、処理を行い、結果を標準出力にprintなどでJSONで返すだけなので、ちょっとしたサーバプログラムをPythonなどで書いたことがある人はすぐ書けると思います。

ぜひ自分の好きなようにカスタムして、快適なGit LFSライフを送ってください。
