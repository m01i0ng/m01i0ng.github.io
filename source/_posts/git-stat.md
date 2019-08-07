---
title: 使用 Git 统计代码
date: 2019-08-07 08:27:23
tags: Git
categories:
  - Git
---

## 原生 Git 配合 shell 命令

### 统计个人代码量

```bash
git log --author=`git config --global --get user.name` --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -
```

![pic](https://gitee.com/alley9469/pic/raw/master/content/IYHAdd.png)

### 贡献值统计

```bash
git log --pretty='%aN' | sort -u | wc -l
```

![pic](https://gitee.com/alley9469/pic/raw/master/content/jEQuCK.png)

### 查看排名前 5 的贡献者

```bash
git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5
```

![pic](https://gitee.com/alley9469/pic/raw/master/content/dNHBcr.png)

## 使用 git_stats

首先全局安装

```bash
gem install git_stats
```

然后运行

```bash
git_stats generate
```

打开 `git_stats` 目录

```bash
cd git_stats
open index.html
```

效果示例：

![pic](https://gitee.com/alley9469/pic/raw/master/content/ukOJNC.png)

![pic](https://gitee.com/alley9469/pic/raw/master/content/nlW6Kq.png)

## 使用 cloc

安装

```bash
npm i -g cloc
```

进入项目，执行

```bash
cloc .
```

结果示例：

![pic](https://gitee.com/alley9469/pic/raw/master/content/uJ5SbX.png)
