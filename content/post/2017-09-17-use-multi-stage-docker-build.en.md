+++
title = "2017 09 17 Use Multi Stage Docker Build"
date = 2017-09-17T13:38:11+09:00
tags = ["docker", "kubernetes", "typescript"]
+++

I had wrote Dockerfile like this, to deploy Node.js application written by TypeScrpt to k8s

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

The bad point is that, the image includes `typescript` and `@types/*` dependencies to transpile TypeScript.

And, to run CI test like

```
docker run --rm イメージ名:test yarn run mocha 'dist/test/**/*.js'
docker run --rm イメージ名:test yarn run tslint --project tsconfig.json
docker run --rm イメージ名:test yarn run prettier --list-different 'src/**/*.ts'
```

The image includes all dependencies only for test, such as `mocha`.

I knew it is not good, but did installed all `dependencies` and `devDependencies`.


Now I rewrite this Dockerfile with multi stage build

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

The final image includes only production `node_modules` via `yarn install --production`, and not includes `devDependencies`.

And this image can't be used to run test for CI, so I changed to run inside docker build process.

As a result, the image size was reduced from 1.6GB to 1.2GB 🎉

Reference

- [Use multi-stage builds | Docker Documentation](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)
- [Builder pattern vs. Multi-stage builds in Docker](https://blog.alexellis.io/mutli-stage-docker-builds/])
- [Create lean Node.js image with Docker multi-stage build](https://codefresh.io/blog/node_docker_multistage/)

