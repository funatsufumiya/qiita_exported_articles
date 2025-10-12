Windowsで普通に `pip install pycocoapi` としようとして色々詰まったのでメモ。

（以下の内容は主に https://github.com/cocodataset/cocoapi/issues/51 にあるトラブルシューティング＋α。Qiitaにある[他の記事](https://qiita.com/kekekekenta/items/ca9d5d1f197c373656ec)も参照したけど、コンパイラフラッグが足りなくてビルドがコケたので、いろいろ足した。）

## 基本はgitでクローンしてビルド

そもそもgitが入ってないときは[git for windows](https://gitforwindows.org/)。

```bash
git clone https://github.com/cocodataset/cocoapi.git
```

で、ビルドしようとして多分間違いなくコケる。

```
cd cocoapi\PythonAPI
python setup.py build_ext install

# 以下エラーメッセージがいろいろ
```

ので、ここからはエラーメッセージの内容に応じてトラブルシューティング。

## 1. cl.exe がない

`error: command 'cl.exe' failed: No such file or directory` みたいなエラーメッセージが出たら、cl.exeがPATHにない。

一番簡単な解決法は、Visual Studioをインストールして、**開発者コマンドプロンプト**を使う。（インストールの仕方はいろんな記事があるので[そちらを参照](https://arakan-pgm-ai.hatenablog.com/entry/2018/12/13/000000)。)

ちなみに開発者コマンドプロンプトにはいろんな種類があるが、今回は**x64 Native Toolsコマンドプロンプト**を使う。

起動するには、Visual Studioをインストールした上で、検索バーで`x64 Native`と打てば出てくる。

![SnapCrab_NoName_2020-3-30_12-36-21_No-00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/18ea582b-bc0c-00a4-5abb-187c9bbe5060.png)


## 2. Wno-cpp が使えない

さて、VS2017用 x64 Native Toolsコマンドプロンプトを起動して、

```
cd cocoapi\PythonAPI # cocoapiをgitでクローンしたパス
python setup.py build_ext install
```

を実行すると、`invalid numeric argument '/Wno-cpp'` みたいなエラーメッセージが出る。（ちなみにそれ以前にpythonがないというエラーが出る場合は、pythonのPATHが通っていない。）

なので、`cocoapi\PythonAPI\setup.py`の14行目あたりにある、`extra_compile_args`を以下のように空にする。

```python
extra_compile_args=[],
#extra_compile_args=['-Wno-cpp', '-Wno-unused-function', '-std=c99'],
```

書き換えたら再度トライ。

## 3. math.h がない、basetsd.h がない等々

今度はヘッダファイルが足りないと言われるので、さらに`cocoapi\PythonAPI\setup.py`の`include_dirs`を以下のように書き換える。

```python
include_dirs = [np.get_include(), '../common', 'C:/Program Files (x86)/Windows Kits/10/Include/10.0.17763.0/ucrt', 'C:/Program Files (x86)/Windows Kits/10/Include/10.0.17763.0/winrt','C:/Program Files (x86)/Windows Kits/10/Include/10.0.17763.0/um','C:/Program Files (x86)/Windows Kits/10/Include/10.0.17763.0/shared'],
#include_dirs = [np.get_include(), '../common'],
```

- ※1： `10.0.17763.0` の部分はWindowsのバージョンによって異なるので、エクスプローラーで `C:\Program Files (x86)\Windows Kits\10\Include` の中身を確認して存在するバージョンに書き換える必要がある。
- ※2： もし `C:\Program Files (x86)\Windows Kits` の中身が存在しない場合は、Visual Studio InstallerからWindows 10 SDKを追加する必要がある。

## 4. kernel32.lib がない

さて今度はリンカフラッグが足りないので、`include_dirs`の次の行に以下を**書き足す**。

```python
library_dirs = ['C:/Program Files (x86)/Windows Kits/10/Lib/10.0.17763.0/um/x64','C:/Program Files (x86)/Windows Kits/10/Lib/10.0.17763.0/ucrt/x64'],
```

※ `10.0.17763.0`の部分に関しては先程と同様。（もしx64コマンドプロンプトじゃなくてx86コマンドプロンプトを使ってる場合は、x64をx86に書き換える必要があると思う。）

ちなみに最終的なsetup.pyはこんな感じ。

```python
from setuptools import setup, Extension
import numpy as np

# To compile and install locally run "python setup.py build_ext --inplace"
# To install library to Python site-packages run "python setup.py build_ext install"

ext_modules = [
    Extension(
        'pycocotools._mask',
        sources=['../common/maskApi.c', 'pycocotools/_mask.pyx'],
        include_dirs = [np.get_include(), '../common', 'C:/Program Files (x86)/Windows Kits/10/Include/10.0.17763.0/ucrt', 'C:/Program Files (x86)/Windows Kits/10/Include/10.0.17763.0/winrt','C:/Program Files (x86)/Windows Kits/10/Include/10.0.17763.0/um','C:/Program Files (x86)/Windows Kits/10/Include/10.0.17763.0/shared'],
        library_dirs = ['C:/Program Files (x86)/Windows Kits/10/Lib/10.0.17763.0/um/x64','C:/Program Files (x86)/Windows Kits/10/Lib/10.0.17763.0/ucrt/x64'],
        extra_compile_args=[],
    )
]

setup(
    name='pycocotools',
    packages=['pycocotools'],
    package_dir = {'pycocotools': 'pycocotools'},
    install_requires=[
        'setuptools>=18.0',
        'cython>=0.27.3',
        'matplotlib>=2.1.0'
    ],
    version='2.0',
    ext_modules= ext_modules
)
```

## 5. rc.exe がない

これが最後の砦。`C:\Program Files (x86)\Windows Kits\10\bin\x64` をPATHに追加して、もう一度x64 Native Toolsコマンドプロンプトを開き直し、同じコマンドを実行すればOK。

( もしPATHにこのフォルダを追加したくない場合は、中にある `rc.exe` と `rcdll.dll` だけをコピーしてPATHの通ったフォルダに置けばOK。)

## 6. やっとビルドが通る

あぁ長い道のりだった。ちゃんとインストールできてるかは以下のコマンドでわかる。

```
python
>>> from pycocotools.coco import COCO
>>> 
```

エラーが出なければOK。
