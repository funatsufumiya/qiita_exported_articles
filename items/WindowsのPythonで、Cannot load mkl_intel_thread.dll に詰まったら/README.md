何番煎じかわからないけれど、Windows上のPython (Anaconda/Miniconda) で `Intel MKL Fatal ERROR: Cannot load mkl_intel_thread.dll` で自分も詰まって、色々調査して解決したので、自分の思うベストソリューションをまとめてみる。

## 1. まず、どのdllが読み込まれているかを確認する

Anaconda PromptやPowershellなどで、`where.exe mkl_intel_thread.dll` を実行するとどのdllが読み込まれているかがわかる。

```powershell
> where.exe mkl_intel_thread.dll

C:\Miniconda3\envs\mkl_test\Library\bin\mkl_intel_thread.dll

# メモ: Powershellで実行する場合は、where じゃなくて必ず where.exe !
```

ここでAnacondaやMinicondaでインストールされている以外のdllが表示されているときは、間違いなく**バージョンのミスマッチ**でうまくいかないので、削除するか別のフォルダに移動する。（ちなみに手動でIntelのサイトから最新バージョンのMKLを入れている場合も、一旦Pathのフォルダから消したほうが良い。自分はここでハマッた。）

なお、`conda activate xxx` を行うと `where.exe` の表示も変わるので、condaで独自環境を作っている場合は先にactivateしてから実行すべき。

## 2. dllを整理してもうまく行かなかったら、新規に環境を作る

1.の手順で、余計なdllを削除して再度実行してもうまく行かない場合は、新規環境を作るのが良い。例えばPython3.6.5で環境を新規に作る場合は以下。`mkl_test`のところは好きな名前でOK。

```bash
conda create -n mkl_test python=3.6.5
conda activate mkl_test

# NOTE: Anaconda PromptではなくPowershellで最初にactivateを行うときは、
#   `conda init powershell` してPowershellの再起動が必要。
```

ただしこの `create` のとき、いつものように最後にanacondaを入れると、デフォルトのnumpyとかscipyとかが入ってしまうので、入れるべきではない。**あくまでMKLから先にインストールして、そこから自分の必要なライブラリをどんどん追加していく**。

MKLのバージョンについては、自分は2019.4ではうまく行かず、2019.1や2018.0.3ではうまくいった。

```bash
conda install mkl=2019.1
```

ここから、必要なライブラリを追加していく。

```bash
conda install numpy scipy
conda install -c pytorch pytorch=0.4.1
# などなど
```

そして最終的に `where.exe mkl_intel_thread.dll` でdllを確認して、今回作った環境のenvs以下のdllのみが表示されているのが正しい。

```
> where.exe mkl_intel_thread.dll

C:\Miniconda3\envs\mkl_test\Library\bin\mkl_intel_thread.dll
```

この状態でもう一度 `import numpy` などを実行すれば、多分うまくいく。
