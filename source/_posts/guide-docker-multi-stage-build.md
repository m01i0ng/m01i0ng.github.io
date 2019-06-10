---
title: Docker 多阶段构建
date: 2019-01-25 09:16:33
tags:
  - Docker
  - DevOps
categories: Docker
---

Docker 镜像内部寸土寸金，如何能尽量减小镜像的大小，其中一个方法是多阶段构建
一些语言项目例如前端、Go 等，其构建阶段需要一堆依赖如 `node_modules`、`vendor` 等，但运行阶段只需要 build 出来的静态页面或二进制文件，所以要将构建和运行分开为两个阶段，最终镜像中只包含 build 的结果

以前端项目举例，目录结构如下：

```
├── .docker
├── .dockerignore
├── Dockerfile
├── README.md
├── node_modules
├── package.json
├── public
├── src
└── yarn.lock
```

.docker 文件夹中存放了 nginx 的配置
.dockerignore 文件写不希望添加到镜像中的文件，比如 `node_modules`
Dockerfile 是主角，内容如下：

```Dockerfile
# 第一阶段，拉取 node 基础镜像并安装依赖，执行构建
FROM node:11-alpine as builder

WORKDIR /tmp
COPY . .
RUN npm config set registry https://registry.npm.taobao.org \
    && npm i -g yarn
RUN yarn && yarn build

# 第二阶段，将构建完的产物 build 文件夹 COPY 到实际 release 的镜像中，会丢弃第一阶段中其他的文件
FROM nginx:alpine

COPY .docker/conf/default.conf /etc/nginx/conf.d/
COPY --from=builder /tmp/build /usr/share/nginx/html

EXPOSE 80
```

执行 `docker build -t m01i0ng/demo .` 就会生成镜像

跟最终 release 镜像无关的文件依赖等等都要放在构建阶段
