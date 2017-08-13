+++
date = "2017-08-13T00:00:00+09:00"
title = "Migrated to Github Pages"
tags = ["hugo"]
draft = false

+++

I migrated this blog to github pages hosting, as a result. The CGP load balancer is expencive for just routing GS hosted content.

This is memo for that.

On GitHub Pages, choose project page(USER.github.io/PROJECT.github.io) and publish `/docs` setting.
It seems that we can use only top directory pushlish setting, if we choose user page(USER.github.io).

On hugo, add one line in `config.toml`.

```
publishDir = "docs"
```

build and push, then, https://yokomotod.github.io/yokomotod-io.github.io becomes to visible.

Next, to use custom domain with SSL, I chose Cloud Flare.

Refer https://help.github.com/articles/setting-up-an-apex-domain/ .

Add new site on CloudFlare with `yokomotod.io` domain. Register `192.30.252.153` and `192.30.252.154` as A record. Then change Name Server to Cloud Flare's one on DNS provider settings.

Finally, add `yokomotod.io` in custom domain field, on GitHub repository setting.

After Cloud Flare recognize DNS settings change, http://yokomotod.io becomes to visible.

The reason why it shows `USER.github.io/PROJECT.github.io`(not `USER.github.io/`) is that GitHub handle it according to request server name? Cool.

In addition, just enabling SSL settings on Cloud Flare, https support completed. Cool, Cloud Flare.
