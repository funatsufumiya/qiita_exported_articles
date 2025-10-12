おそらく執筆時点でのノウハウに過ぎないと思われる、本家での解決までに使える一時的なノウハウを集めてみた。

ちなみに、サンプルの内容には全く言及しない。あくまでビルドが通るまでのノウハウであることに注意。

※ RavEngineとRGLの概要については自分の書いた以下の記事を参考にしていただければ。

https://qiita.com/funatsufumiya/items/4a154378e490fdde42a8

なお、Windows/Mac/Linuxなどの様々なプラットフォームの問題が混在するので、見出しでわかるようにしておくので都度読み飛ばしてほしい。

## (共通) ビルド済みのサンプルが動かない

[README の How to try the samples without building](https://github.com/RavEngine/Samples?tab=readme-ov-file#how-to-try-the-samples-without-building) に、ビルドせずにビルド済みのバイナリを動かす方法（例: [#669 の CIビルド済ファイル](https://github.com/RavEngine/Samples/actions/runs/12626631480)をダウンロードなど）が紹介されているが、これが動かないことがある。

動けばバンザイでOKなのだけれど、おそらく環境要因等で動かない場合は、自分でビルドするしかない。（ただし、[最低動作環境](https://github.com/RavEngine/RavEngine?tab=readme-ov-file#system-requirements-for-developers)が非常に厳しいので、まず先に確認する。）

手動ビルドは、具体的なコマンドがドキュメント不足でわかりづらい点が多々あるので、必要に応じて [`.github/workflows/build.yml`](https://github.com/RavEngine/Samples/blob/master/.github/workflows/build.yml) などを参考にすることをおすすめする。

ちなみに、mainブランチ (masterブランチ) の更新は、時にはビルドが失敗するものも含まれているはずなので、コミットログのCIの成功・失敗は一応参考にすると良い。

## (共通) ヒープ領域を使い果たしたと言われてビルドに失敗する

これはいわゆるメモリ不足。本当にメモリが足りない場合もあるし、失敗した時点からやり直せば解決する場合もある。

本質的な対応策としては、ライブラリをstatic linkにしない、外部ライブラリを動的リンクするなどの対応策があるはずだけれど、これについては未調査。

## (Win) ロングパスの有効化

ビルド時に、パスが256文字を超えているためにファイルが保存できないという主旨のエラーが出る。これは以下のようにロングパスを有効化することで回避。

https://qiita.com/masinc000/items/934166ac13d894ae28d1

## (Win) DX12のバージョン問題 ( Agility SDK )

RavEngineではなく、[RGLのサンプル](https://github.com/RavEngine/RGL-Samples)をビルドする際に、実行時にDX12の構成エラーが出る。

これは、[Agility SDK](https://qiita.com/keruya/items/f89c7c8936dac622bdf0)が同梱されていないのが問題で、`C:¥Windows¥System32` にあるDX12関連のdllを読もうとして、実行時のバージョンのミスマッチにより動作に失敗する。

結論としては、[RavEngine/Samples](https://github.com/RavEngine/Samples)の依存関係の方のRGLには、Agility SDKが同梱されているので、そちらをコピーして使う。（`deps/RGL`ごとコピーして基本的に問題ないはず。）[^1]

## (Mac) metalのビルドに失敗する

これは、CMakeの呼び出し時に `-G Xcode` を指定していないのが問題。Metalシェーダのビルドには、現時点でXcodeが必須のようで、[READMEのSupported Platforms](https://github.com/RavEngine/RavEngine?tab=readme-ov-file#supported-platforms)でMacだけmakeやninjaが存在せずに、Xcodeだけがサポートされているのはこれが理由。

（エラーメッセージとしては、`LANGUAGE METAL` がサポートされていないなどのエラーが出る。）

## (Linux) volkフォルダがない (volk copyとなってしまう)

これはタイトル通りで、Vulkan SDKの一部であるvolkが、なぜか `deps/volk copy` という名前のフォルダになってしまう。これは単純にリネームすればOK。（気付くのにちょっと時間がかかる。）

## (Linux) CreatePipelineLayoutの作成に失敗する

おそらくgcc (clang)のバージョンが関連していると思われる、`device->CreatePipelineLayout({ ... });` の構文読み取りエラーが起こることがある。

これは一度 `RGL::PipelineLayoutDescriptor pld{ ... };` として変数に格納して、その後 `pipelineLayout = device->CreatePipelineLayout(pld);` とすれば解決する。

## (Linux) RVE_VFS_get_name がないといわれる

これは、VirtualFileSystem.cppで、`extern const`となっている[2種類の関数](https://github.com/RavEngine/RavEngine/blob/1fdfe9183a1655fe9a36a892be816a71e1e0e404/src/VirtualFileSystem.cpp#L129-L132)が、CMakeによる動的生成によって実体が呼び出されるのが原因。

実は自分はまだこれをちゃんとは解決できていないのだけれど、[HelloCube](https://github.com/RavEngine/HelloCube)の方のサンプルを動かすか、あるいはCMakeから不必要な `add_sample(...)` の行を除いて、`extern const`の行をコメントアウトしたり、あるいはサンプルのcppに追加の行やファイルを追加し、CMakeLists.txtから正しいものを目で読み取って記載するという手はある。

（Win/Macでは問題ないので、IssueやDiscussionを探せば解決策があるかもしれないし、CMakeのオプションなど、[GitHub Workflowの記述](https://github.com/RavEngine/Samples/blob/master/.github/workflows/build.yml)を参考にするべきかもしれない。）

## (Linux) Vulkanの実行時エラーで〇〇の機能がないといわれる

例えば、`Cannot init - Fragment Shader Interlock is not supported` のようなもの。

こればっかりは正直どうしようもなくて、WSLを使っていると起こりがち。デバッグ関連の最新機能が多いので、コメントアウトできる部分もありそうな気はするけれど、仮想環境ではないUbuntu 24.04を使うと解決することが多いけれど、おそらく使っているGPUにも依存する。

![スクリーンショット 2025-01-08 16.42.30.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/7d383652-b1f2-9cb1-783e-a26281e61915.png)

[^1]: Agility SDKが存在しないように見える時は、gitのサブモジュールを再帰的に取得するのを忘れている。余談として、サブモジュールのサイズや転送量を削減するには次の記事を参照: https://qiita.com/tsuyoshi_cho/items/8ee621805332249a6fa5
