---
title: 规范你的 Commit Message
date: 2019-05-10 12:31:20
tags:
  - Git
categories: Git
---

# Git 提交信息规范

## Commit Message 格式

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

参考 [Angular](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines) 规范

通过 git commit 命令带出的 vim 界面填写的最终结果应该类似如上这个结构, 大致分为三个部分(使用空行分割)：

- 标题行：必填, 描述主要修改类型和内容
- 主题内容：描述为什么修改, 做了什么样的修改, 以及开发的思路等等
- 页脚注释：放 Breaking Changes 或 Closed Issues

分别由如下部分构成：

- type: commit 的类型
- feat: 新特性
- fix: 修改问题
- refactor: 代码重构
- docs: 文档修改
- style: 代码格式修改，注意不是 css 修改
- test: 测试用例修改
- chore: 其他修改，比如构建流程，依赖管理
- scope: commit 影响的范围，比如：route, component, utils, build...
- subject: commit 的概述，建议符合 50/72 formatting
- body: commit 具体修改内容，可以分为多行，建议符合 50/72 formatting
- footer: 一些备注，通常是 BREAKING CHANGE 或修复的 bug 的链接

## Git Commit 模板

修改 ~/.gitconfig，添加：

```
[commit]
template = ~/.gitmessage
```

新建 ~/.gitmessage 内容可以如下：

```
# head: <type>(<scope>): <subject>
# - type: feat, fix, docs, style, refactor, test, chore
# - scope: can be empty (eg. if the change is a global or difficult to assign to a single component)
# - subject: start with verb (such as 'change'), 50-character line
#
# body: 72-character wrapped. This should answer:
# * Why was this change necessary?
# * How does it address the problem?
# * Are there any side effects?
#
# footer:
# - Include a link to the ticket, if any.
# - BREAKING CHANGE
#
```

## Commitizen：替代你的 Git Commit

[commitizen/cz-cli](https://github.com/commitizen/cz-cli)，我们需要借助它提供的 git cz 命令替代我们的 git commit 命令, 帮助我们生成符合规范的 commit message

除此之外，我们还需要为 commitizen 指定一个 Adapter 比如：[cz-conventional-changelog](https://github.com/commitizen/cz-conventional-changelog) (一个符合 Angular 团队规范的 preset)。使得 commitizen 按照我们指定的规范帮助我们生成 commit message

### 全局安装（推荐）

```bash
npm install -g commitizen cz-conventional-changelog
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
```

### 项目级安装

> 如果不是 node 项目可以用 `npm init -y` 生成 `package.josn`

```bash
npm install -D commitizen cz-conventional-changelog
```

package.json 中配置：

```json
"script": {
    ...,
    "commit": "git-cz",
},
 "config": {
    "commitizen": {
      "path": "node_modules/cz-conventional-changelog"
    }
  }
```

如果全局安装过 Commitizen，那么在对应的项目中执行 `git cz` 或 `npm run commit` 都可以

### Commitlint：校验你的 message

[commitlint](https://github.com/marionebl/commitlint) 可以帮助我们 lint commit messages, 如果我们提交的不符合指向的规范, 直接拒绝提交

校验配置推荐 [@commitlint/config-conventional](https://github.com/marionebl/commitlint/tree/master/@commitlint/config-conventional)

同时需要在项目目录下创建配置文件 .commitlintrc.js，写入：

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {},
}
```

### standard-version：自动生成 CHANGELOG

通过以上工具的帮助，我们的 commit message 应该是符合 Angular 团队那套，这样也便于我们借助 standard-version 这样的工具，自动生成 CHANGELOG，甚至是 语义化的版本号(Semantic Version)

安装使用：

```bash
npm i -S standard-version
```

package.json 配置：

```json
"scirpt": {
    ...,
    "release": "standard-version"
}
```

## 最后

遵守以上提交规范的同时，**禁止**使用诸如 `git commit -am` 方式提交 Commit Message
