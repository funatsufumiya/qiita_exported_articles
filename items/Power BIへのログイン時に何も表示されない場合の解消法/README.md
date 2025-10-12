※ この現象は近々修正予定とのことで、直るまでの暫定措置。

## 現象

Power BIにログインしようと、https://app.powerbi.com/ にアクセスした際に、ライセンスが失効していると、`https://app.powerbi.com/SignupRedirect?pbi_source=websignup&responseError=UserNotLicensedru=https%3A...` のようなURLだけ表示され、何も画面に表示されないというトラブルが発生することがある。

## 回避策

URLの `https://app.powerbi.com/SignupRedirect?pbi_source=websignup&responseError=UserNotLicensedru=https%3A...` の、 `=UserNotLicensedru=` の箇所を `=UserNotLicensed&ru=` に置き換える。

すると次の画面に遷移できるようになり、新しくサインアップなどを行うことができるようになる[^1]。

[^1]: ライセンスが切れているのでエラーしても仕方ないかと思っていたが、ここで止まると新たなサインアップもできないため、Azureのサポートリクエストにより問い合わせたところ、この解決策を教えてもらえた。本当に感謝。
