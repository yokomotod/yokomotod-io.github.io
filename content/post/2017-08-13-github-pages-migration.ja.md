+++
date = "2017-08-13T00:00:00+09:00"
title = "結局Github Pagesに移行"
tags = ["hugo"]
draft = false

+++

GCPのロードバランサーが、ただブログをGSでホスティングするためだけには金食い虫なので結局GitHub Pagesに移行。

以下メモ。

GitHub Pages側はプロジェクトページ(USER.github.io/PROJECT.github.io)にして、`/docs` 配下を配信する設定に。
どうやらユーザページ(USER.github.io)だとmasterブランチ直下配信しか選択させてもらえない。

hugoの`config.toml`に以下追加。

```
publishDir = "docs"
```

ビルドしてpushしたら https://yokomotod.github.io/yokomotod-io.github.io が見えるように。

カスタムドメインにしてかつSSLは有効にしたいので、CloudFlareさんを噛ませる。

`yokomotod.io`(apexドメイン)で配信したいので、 https://help.github.com/articles/setting-up-an-apex-domain/ を参考に。

CloudFlareで新規サイト追加。ドメインは`yokomotod.io`。Aレコードで`192.30.252.153`と`192.30.252.154`を登録。DNSプロバイダ側で、ネームサーバをCloudFlare指定のものに変更。

最後に、GitHubのレポジトリ設定ページで、カスタムドメイン欄に`yokomotod.io`を追加。

DNSの設定変更が浸透してCloudFlareに認識されたら http://yokomotod.io で見えるようになった。

DNS設定だけだと`USER.github.io/`に飛ばせても、パスの`/PROJECT.github.io`に飛ばせなくて困りそうな予感してたけど、GitHub側がリクエストのserver nameを見てよしなにやってくれてるんだろか？

あとCloudFlareの設定でSSL有効にしたら一瞬でSSL対応完了。CloudFlareさんすごい。
