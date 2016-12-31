+++
date = "2017-01-01T00:00:00+09:00"
title = "Hello, Hugo."
tags = ["hugo"]
draft = false

+++


新年あけましておめでとうございます。

2017年になったこの機にブログを始める。~~（2年ぶり4回目...~~

今回は2017年なのでHugoで多言語対応。

### セットアップ

hugoのインストールから。

```
brew update
brew install hugo
```

サイト生成。

```
hugo new site yokomotod.io
```

### テーマ選び

多言語対応しているテーマを探したら意外と
[hugo-theme-bootstrap4-blog](http://themes.gohugo.io/hugo-theme-bootstrap4-blog/)と
[hugo-theme-foundation6-blog](http://themes.gohugo.io/hugo-theme-foundation6-blog/)だけしかない・・・？

今回はhugo-theme-bootstrap4-blogで。

```
git clone https://github.com/alanorth/hugo-theme-bootstrap4-blog.git themes/hugo-theme-bootstrap4-blog
```

### 設定

config.toml

```
baseurl = "http://yokomotod.io/"

theme = "hugo-theme-bootstrap4-blog"

# どの言語もサブディレクトリ（/en、/ja）に配置。
# これでルートディレクトリが自動的にルートページがデフォルト言語のURLにリダイレクトされる
# ・・・が、これがSEO的に問題ないかどうかが疑問
DefaultContentLanguage = "en"
defaultContentLanguageInSubdir = "true"

# [Language.ja]の下における方がベターだったけどそれだと認識されず。global optionな模様。
hasCJKLanguage = "true"

title = "yokomotod.io"
copyright = "&copy;2017 yokomotod"

[PermaLinks]
  post = "/:year/:month/:day/:title/"

[[Menu.Sidebar]]
  name = "github"
  url = "https://github.com/yokomotod"
[[Menu.Sidebar]]
  name = "twitter"
  url = "https://twitter.com/yokomotod"
[[Menu.Sidebar]]
  name = "qiita"
  url = "http://qiita.com/yokomotod"


[Params]
  Author = "yokomotod"
  # コメント機能
  # https://disqus.com/ でサイト登録してIDを発行する
  disqusShortname = "yokomotod-io"

[Params.Social]
  twitter = "yokomotod"

[Params.Sidebar]
  about = "yokomotod blog"

# 対応する言語
[Languages]
[Languages.en]
[Languages.ja]
# いくつかの項目は、ここに書くと言語ごとに設定を切り替えられる

```

### シンタックスハイライト

ドキュメントの[Syntax Highlighting](https://gohugo.io/extras/highlighting/#highlight-js-example)そのまま。

layouts/partials/head-custom.html

```
<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.6.0/styles/default.min.css">
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.6.0/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

### 記事作成

```
hugo new post/2017-01-01-hello-hugo.en.md
hugo new post/2017-01-01-hello-hugo.ja.md
```

### ローカルサーバ起動

```
hugo server
```

### Git コミット

```
git init
echo "/public/" >> .gitignore
echo "/themes/" >> .gitignore
git add --all
git commit
```

### 課題

ブログタイトルのURLが`.Site.BaseURL` (http://yokomotod.io/)になっていて、常に/enにリダイレクトされるので、/jaと分けたい。

RSS feedのURLがバグっている・・・。 ここも http://yokomotod.io/index.xml ではなく http://yokomotod.io/ja/index.xml に言語ごとに要切り替え。

### 次回

次はデプロイ部分の予定。Google Cloud Platformを使用。

Google Cloud Storageでホストしつつ、Load Balancerを使ってSSLに対応。
