+++
tags = ["hugo","gcp"]
date = "2017-01-02T22:41:23+09:00"
title = "Deploy Hugo on Google Cloud Storage"

+++

[前回](/ja/2017/01/01/hello-hugo./)作成したHugoブログをデプロイするところ。

### デプロイ先：Google Cloud Storage

HugoだとGithub Pagesへのデプロイの記事をよく見かける気がする。

一番手軽だし無料でいけるけど、なんとなく今回のブログを自分のgithub user pageにする気分じゃなかったので回避。

今回はGCPのGoogle Cloud Storageを使ってデプロイしてみる。

AWS S3でないことに特に理由はない。

Google Cloud Storage単体にも[静的ウェブサイトのホスト](https://cloud.google.com/storage/docs/hosting-static-website)機能はあるけど、SSLに対応していないので、まだアルファ版の[ロードバランサのバックエンドとしての Cloud Storage バケットの使用](https://cloud.google.com/compute/docs/load-balancing/http/using-http-lb-with-cloud-storage)の仕組みを使ってhttpsを実現させる。

個人ブログなので、アルファ版でも気にしない。

### アルファ機能のWhitelist申請

https://cloud.google.com/compute/docs/load-balancing/http/using-http-lb-with-cloud-storage

から、アルファ機能利用の申請が必要。


### Google Cloud Storageにデプロイ

[ドキュメント](https://cloud.google.com/storage/docs/hosting-static-website)のとおり、まずは普通にデプロイする。


バケット作成

```
gsutil mb -l asia gs://yokomotod.io
```

デフォルトACL設定

```
gsutil defacl ch -u allUsers:READER gs://yokomotod.io
```

コンテンツアップロード

```
gsutil -m rsync -R public gs://yokomotod.io
```

Indexページ、Not Foundページを割り当てる

```
gsutil web set -m index.html -e 404.html gs://yokomotod.io
```

### SSL証明書取得

Let's Encryptを使う。

Macはまだサポートされていないので、一時的にComputeインスタンスを立てた。

```
sudo apt-get update
sudo apt-get install certbot -t jessie-backports
sudo certbot certonly --manual --agree-tos --manual-public-ip-logging-ok --email yokomotod@gmail.com --domain yokomotod.io
```

コンソールにACMEチャンレンジ用の情報が表示されて待受状態になるので、それに従ってコンテンツを作る。

```
mkdir -p public/.well-known/acme-challenge/
echo -n m93MIV9iKfPGoZNKmDC-Una1cjD3w_FUWZYGbRsqBz0.DIv0_zl8TCm68ozhMeQWjeLCmlY0aj0PztQApIP1Lhw > public/.well-known/acme-challenge/m93MIV9iKfPGoZNKmDC-Una1cjD3w_FUWZYGbRsqBz0
gsutil -m rsync -R public gs://yokomotod.io
```

コンテンツの準備が出来たら、certbot側でエンターキーを押す。
きちんとレスポンスを返すことができれば証明書が作成されるので、GCPに登録する。

[Networking] > [Load Balancer] > [advanced menu] > [Certificates] > [CREATE SSL CERTIFICATE] と進み、作成された `cert.pem`、`chain.pem`、`privkey.pem` をそれぞれ登録。


### Load Balancer構築

backend backet作成（ここがアルファ機能の権限が必要）

```
gcloud alpha compute backend-buckets create yokomotod-io-backend-bucket \
    --description "" \
    --gcs-bucket-name "yokomotod.io"
```

url map作成

```
gcloud alpha compute url-maps create yokomotod-io-url-map \
    --default-backend-bucket=yokomotod-io-backend-bucket
```

target proxy作成

```
gcloud compute target-http-proxies create yokomotod-io-target-http-proxy \
    --url-map yokomotod-io-url-map
gcloud compute target-https-proxies create yokomotod-io-target-https-proxy \
    --url-map yokomotod-io-url-map --ssl-certificate yokomotod-io-ssl-certificate-2017-01-01
```

global ip address作成

```
gcloud compute addresses create yokomotod-io-address --global
```

global forwarding rule作成

```
gcloud compute forwarding-rules create yokomotod-io-http-forwarding-rule \
    --address 35.186.204.77 --global \
    --target-http-proxy yokomotod-io-target-http-proxy --ports=80

gcloud compute forwarding-rules create yokomotod-io-https-forwarding-rule \
    --address 35.186.204.77 --global \
    --target-https-proxy yokomotod-io-target-https-proxy --ports=443
```

これで、DNSで yokomotod.io のAレコードに35.186.204.77を設定することで https://yokomotod.io 開通。

### 末尾スラッシュリダイレクト問題

が、これで https://yokomotod.io にアクセスしてみると、何故か https://yokomotod.io/en/index.html にリダイレクトされる。

ローカルで `hugo server` では https://yokomotod.io/en/ なのに・・・。

調べてみると、

```
https://yokomotod.io → https://yokomotod.io/en → https://yokomotod.io/en/index.html
```

とリダイレクトされている。

[Stack Overflow](http://stackoverflow.com/questions/24362594/google-cloud-storage-301-redirect)によると、
`/en`を`/en/`じゃなく`/en/index.html`にリダイレクトするのはGCSの仕様っぽい。

そもそも最初から`https://yokomotod.io → https://yokomotod.io/en/`にリダイレクトされればいい気がするんだけど。。。

とりあえず、生成された`public/index.html`を直接書き換えて暫定対応。

```
<!DOCTYPE html><html><head><title>https://yokomotod.io/en</title><link rel="canonical" href="https://yokomotod.io/en"/><meta http-equiv="content-type" content="text/html; charset=utf-8" /><meta http-equiv="refresh" content="0; url=https://yokomotod.io/en/" /></head></html>
```

↓

```
<!DOCTYPE html><html><head><title>https://yokomotod.io/en/</title><link rel="canonical" href="https://yokomotod.io/en/"/><meta http-equiv="content-type" content="text/html; charset=utf-8" /><meta http-equiv="refresh" content="0; url=https://yokomotod.io/en/" /></head></html>
```

### 課題

- 末尾スラッシュ問題
  - 最初からHugoが`/en/`にリダイレクトするのが正解な気がするのでIssueを出してみようかな
- Let's EncryptでのCertificates更新を自動化
  - Compute Instanceにcronを仕込めば自動化はすぐできるけど、そのためだけにインスタンスを持ちたくない・・・。
  - Cloud Functionでcron的にジョブ実行できるようになると嬉しい。それまではGAEのCron Serviceとかかな。（未調査）
- httpのリクエストをhttpsに301 Redirect
  - AWS Cloud Frontだと http→httpsリダイレクト機能が用意されているけどGCPだとまだない。早く来て欲しい。
  - これも、自前でnginxを立てれば対応できるけど、そのためだけに（以下略）
  - あとはCloud FunctionがLoad Balancerのバックエンドにできるようになるとか来ないかな。


