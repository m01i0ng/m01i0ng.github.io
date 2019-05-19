---
title: Git 设置 HTTP 代理加速上传下载
date: 2019-05-19 14:25:46
tags:
  - Git
categories: Git
---

```bash
git config --global http.proxy 'socks5://127.0.0.1:1086'
# 或
git config --global http.proxy 'http://127.0.0.1:1087'
```

以上两条命令都可以，具体参数值取决于你的本地代理地址

有的文章会说还需要设置 `https` 代理，其实没必要，因为 `git@github.com` 的方式不走 https
