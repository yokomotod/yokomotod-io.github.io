+++
title = "Multi Stage Docker Build 使ってみた"
date = 2017-09-17T13:38:11+09:00
tags = ["docker", "kubernetes", "typescript"]
+++

TypeScrptで書いたNode.jsアプリをk8sにデプロイするのに、今まではこんなDockerfileを書いてた。

```
FROM node:8.5.0

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    apt-transport-https \
  && rm -rf /var/lib/apt/lists/*

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    いろいろ
  && rm -rf /var/lib/apt/lists/*

EXPOSE 3000

RUN mkdir -p /usr/src/app

WORKDIR /usr/src/app
COPY package.json yarn.lock /usr/src/app/
RUN yarn install

COPY . /usr/src/app
RUN yarn run compile

CMD ["node", "dist/main/app.js"]
```

気持ち悪いのが、TypeScriptのトランスパイルのために`typescript`パッケージやもろもろの`@types/*`パッケージも全部インストールされたイメージができているところ。

さらにCIで

```
docker run --rm イメージ名:test yarn run mocha 'dist/test/**/*.js'
docker run --rm イメージ名:test yarn run tslint --project tsconfig.json
docker run --rm イメージ名:test yarn run prettier --list-different 'src/**/*.ts'
```

とやってテストもしたいので、`mocha`なんかのテスト用の依存も全部入ってしまっている。

気持ち悪いと思いながら、とりあえずdependenciesもdevDependenciesもインストールしていた。


今回multi stage build使って書き直した結果

```
FROM node:8.5.0 AS base

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    apt-transport-https \
  && rm -rf /var/lib/apt/lists/*

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    いろいろ
  && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /usr/src/app

WORKDIR /usr/src/app


FROM base AS builder

COPY package.json yarn.lock /usr/src/app/
RUN yarn install --production

RUN cp -R node_modules node_modules_production

RUN yarn install

COPY . /usr/src/app
RUN yarn run compile

RUN yarn run mocha 'dist/test/**/*.js'
RUN yarn run tslint --project tsconfig.json
RUN yarn run prettier --list-different 'src/**/*.ts'


FROM base

COPY --from=builder /usr/src/app/node_modules_production ./node_modules
COPY --from=builder /usr/src/app/dist ./dist

COPY res ./res
COPY database.json ./database.json

EXPOSE 3000

CMD ["node", "dist/main/app.js"]
```

最終的にデプロイされるイメージには`yarn install --production`したnode_modulesのみ（devDependenciesを含まない）にできた。

このイメージを使って外部からテスト実行するやりかたはできなくなるので、Dockerfile内でやってしまうことに変更。

イメージサイズは1.6GBだったのが1.2GBくらいにまで減った 🎉

参考

- [Use multi-stage builds | Docker Documentation](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)
- [Builder pattern vs. Multi-stage builds in Docker](https://blog.alexellis.io/mutli-stage-docker-builds/])
- [Create lean Node.js image with Docker multi-stage build](https://codefresh.io/blog/node_docker_multistage/)

