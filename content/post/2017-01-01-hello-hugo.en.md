+++
date = "2017-01-01T00:00:00-05:00"
title = "Hello, Hugo."
tags = ["hugo"]
draft = false

+++


A happy new year.

I start my blog from this year, with Hugo, with multilingal feature.

### Setup

Install hugo.

```
brew update
brew install hugo
```

Generate new site.

```
hugo new site yokomotod.io
```

### Choose theme

By searching theme which support multilingal mode, I found
[hugo-theme-bootstrap4-blog](http://themes.gohugo.io/hugo-theme-bootstrap4-blog/) and
[hugo-theme-foundation6-blog](http://themes.gohugo.io/hugo-theme-foundation6-blog/).

This time I chose hugo-theme-bootstrap4-blog.

```
git clone https://github.com/alanorth/hugo-theme-bootstrap4-blog.git themes/hugo-theme-bootstrap4-blog
```

### Configuration

config.toml

```
baseurl = "http://yokomotod.io/"

theme = "hugo-theme-bootstrap4-blog"

# Put all language under sub directories (/en, /ja).
# Access to root directory will be redirected to default language directory.
# But my question is how this is for SEO...
DefaultContentLanguage = "en"
defaultContentLanguageInSubdir = "true"

# This is for Japanese conetnt.
# I thhought it's better to put [Language.ja] section, but it won't be recognized.
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
  # Comment function.
  # Register the site in https://disqus.com/ and get the ID.
  disqusShortname = "yokomotod-io"

[Params.Social]
  twitter = "yokomotod"

[Params.Sidebar]
  about = "yokomotod blog"

# Supported languages
[Languages]
[Languages.en]
[Languages.ja]
# Here, some configurations can be switched by language

```

### Syntax Highlighting

Just follow Document [Syntax Highlighting](https://gohugo.io/extras/highlighting/#highlight-js-example).

layouts/partials/head-custom.html

```
<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.6.0/styles/default.min.css">
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.6.0/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

### Write posts

```
hugo new post/2017-01-01-hello-hugo.en.md
hugo new post/2017-01-01-hello-hugo.ja.md
```

### Run local server

```
hugo server
```

### Commit to git

```
git init
echo "/public/" >> .gitignore
echo "/themes/" >> .gitignore
git add --all
git commit
```

### TODOs

Link of blog title is now `.Site.BaseURL` (http://yokomotod.io/) in all languages and always redirect to /en, but it should be depends on current showing page language.

RSS feed URL is now wrong. It should be like http://yokomotod.io/ja/index.xml but now http://yokomotod.io/index.xml, according to language.

### Next

Next would be deployment part. I used Google Cloud Platform.

Hostting contents on Google Cloud Storage, and use Load Balancer for support SSL.
