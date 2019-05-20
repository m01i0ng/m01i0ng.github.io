---
title: Mac 系统优化+通用开发环境搭建
date: 2019-05-16 15:19:41
tags:
  - Mac
categories: Mac
---

## 系统配置优化

### 触摸板控制优化，开启轻按点击功能

```bash
defaults write com.apple.AppleMultitouchTrackpad Clicking -int 1
defaults -currentHost write NSGlobalDomain com.apple.mouse.tapBehavior -int 1
defaults write NSGlobalDomain com.apple.mouse.tapBehavior -int 1

# 开启三指拖拽
defaults write com.apple.driver.AppleBluetoothMultitouch.trackpad TrackpadThreeFingerDrag -bool true
defaults write com.apple.AppleMultitouchTrackpad TrackpadThreeFingerDrag -bool true
```

### 将 F1-F12 用做标准功能键

```bash
defaults write -globalDomain com.apple.keyboard.fnState -int 1
```

### 关闭 SIP

重启，按住 `cmd+r`，进入恢复模式，实用工具-终端，执行

```bash
csrutil disable
```

### 关闭第三方程序验证

我们或多或少会下载某些破解版的应用，此时直接打开很可能被系统拒绝，或者报错：无法打开已损坏的安装包。我们可以通过命令行关闭这一保（限）护（制）：

```bash
sudo spctl --master-disable
defaults write com.apple.LaunchServices LSQuarantine -bool false
```
### 关闭镜像验证

在打开 .dmg 格式的安装文件时，默认会先验证镜像，如果文件本身很大，验证的时间会很长，可以输入以下命令关闭验证：

```bash
defaults write com.apple.frameworks.diskimages skip-verify -bool true
defaults write com.apple.frameworks.diskimages skip-verify-locked -bool true
defaults write com.apple.frameworks.diskimages skip-verify-remote -bool true
```

### 完全键盘控制

很多操作都会弹出系统的对话框，要求我们确认或者取消，如果没有开启完全键盘控制，我们只能按回车键确认，或者移动鼠标选择取消。如果开启了完全键盘控制，只要按下空格键，就相当于选中蓝色边框的按钮。按下 Tab 键可以在多个按钮之间切换。

```bash
defaults write NSGlobalDomain AppleKeyboardUIMode -int 3
```

### Finder 快速预览增强

> 需要先安装 brew

```bash
brew cask install qlcolorcode qlgradle qlmarkdown qlstephen qlvideo quicklook-json quicklookapk webpquicklook
```

## Brew

*nix 都有自带的包管理，用于管理软件包之间的依赖，如 RHEL 的 `yum`、Debian 的 `apt`，Mac 本身没有自带的包管理，单有这个第三方的实现 —— `HomeBrew`，一行命令安装：

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装过程中需要输入密码并下载 `Command Line Tools`，视网络情况快慢不一

### 更换镜像源

Brew 自带的镜像源位于国外，不挂代理可能会很慢，这里推荐更换国内的阿里云镜像源：

```bash
# 替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
# 替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git
# 应用生效
brew update
# 替换homebrew-bottles:
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```

### 安装命令行工具

基本的命令行工具可以直接用 `brew install` 安装，更多子命令直接执行 `brew` 即可查看：

```bash
brew update
brew install vim wget curl git 
```

### 安装一般图形化软件

如微信、QQ 或是 Intellij Idea 等开发工具，也可以用 Brew 安装：

```bash
brew cask install wechat qq intellij-idea
```

### 查找

执行 `brew search xxx` 即可从软件源中查找，之后从结果列表中取名字安装即可，其中 `Formulae` 子项下的 `brew install xxx`，`Casks` 子项下的 `brew cask install xxx`

## Oh My Zsh

此处引官方描述：

```
Oh My Zsh is a delightful, open source, community-driven framework for managing your Zsh configuration. It comes bundled with thousands of helpful functions, helpers, plugins, themes, and a few things that make you shout...
```

简单说是对默认终端的一个功能增强，还是极大的增强，可以添加各种自定义主题插件等，一行命令安装：

```bash
brew install zsh
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

> 注意使用 zsh 之后需要将以前 ~/.bash_profile ~/.bashrc 设置的环境变量等转移到 ~/.zshrc 中，例如上述的 echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.zshrc

更多介绍和使用方法[点击](https://github.com/robbyrussell/oh-my-zsh/)查看

这里推荐几个很好用的插件：[Zsh Users](https://github.com/zsh-users)

## 其他开发环境

### Node

使用 [nvm.sh](http://nvm.sh) 管理 Node 版本

`npm i -g mirror-config-china` 配置所有中文镜像

### Python

使用 [pyenv](https://github.com/pyenv/pyenv) 管理 Python 版本

> pyenv install 时会遇到个报错，这里是其解决方式：

### Java、Kotlin、Scala、Maven、Gradle

[sdkman](https://sdkman.io/install)

Maven 使用阿里云镜像：

全局

```bash
vim ~/.m2/settings.xml
```

```xml
<settings>
    <mirrors>
        <mirror>
            <id>aliyunmaven</id>
            <mirrorOf>*</mirrorOf>
            <name>阿里云公共仓库</name>
            <url>https://maven.aliyun.com/repository/public</url>
        </mirror>
    </mirrors>
</settings>
```

项目级

`pom.xml`

```xml
<repositories>
    <repository>
        <id>aliyunmaven</id>
        <name>阿里云公共仓库</name>
        <url>https://maven.aliyun.com/repository/public</url>
    </repository>
</repositories>
```

Gradle 使用阿里云镜像：

全局

```bash
vim ~/.gradle/init.gradle
```

```groovy
allprojects {
    repositories {
        def ALIYUN_REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public'
        def ALIYUN_JCENTER_URL = 'http://maven.aliyun.com/nexus/content/repositories/jcenter'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository){
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                    remove repo
                }
                if (url.startsWith('https://jcenter.bintray.com/')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_JCENTER_URL."
                    remove repo
                }
            }
        }
        maven {
            url ALIYUN_REPOSITORY_URL
            url ALIYUN_JCENTER_URL
        }
    }
}
```

项目级

`build.gradle`

```groovy

buildscript {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }        
}

allprojects {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
    }
}
```

### Ruby

[rbenv](https://github.com/rbenv/rbenv)
