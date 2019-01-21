---
title: iOS开发——Carthage安装和使用教程
date: 2016-09-22 09:53:26
tags: ["iOS开发", "Carthage"]

---

# Carthage是什么？

Carthage 使用于 Swift 语言编写，只支持动态框架，只支持 iOS8+的Cocoa依赖管理工具。

与现在流行的 CocoaPods 不同，Carthage编译你的依赖，并提供框架的二进制.framework文件，但你仍然保留对项目的结构和设置的完整控制，Carthage不会自动的修改你的项目文件或编译设置。是一个去中心化的Cocoa依赖管理工具

<!--more-->

# 如何下载和安装Carthage？

## 使用Brew安装(建议)

1. 安装Mac OSX流行的的软件包管理工具Homebrew之前要检查Mac中是否有Ruby环境,目前的版本基本都内置了Ruby,终端输入

```ruby
ruby -v
```

显示类似 ruby 2.0.0p648 (2015-12-16 revision 53162) [universal.x86_64-darwin15]

```
brew -v
```

显示类似文本 Homebrew 0.9.9 (git revision 2f20; last commit 2016-05-15) 说明已经安装brew不需要再次安装

2. 如果电脑中没有Homebrew,终端执行脚本安装即可

```ruby
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

3. 每次使用 Homebrew 进行安装Carthage 或者其他软件之前,习惯性的先对Homebrew进行更新一下, 不然可能会安装到比较老版本的Carthage等软件

```ruby
brew update
```

提示 Already up-to-date.......  更新到最新啦!!

4. 假如你在本地已经安装好Homebrew环境，那么下载和安装carthage将十分简单，只需要一行命令。

```ruby
brew install carthage
```

## PKG文件安装

直接下载pkg文件：[https://github.com/Carthage/Carthage/releases](https://github.com/Carthage/Carthage/releases) 进行安装即可

# 如何使用Carthage？

包管理工具,不管是CocoaPods,还是Node 的NPM,配置依赖管理都是在工程目录,建立相应的配置文件,Carthage的配置文件即 Cartfile文件

## 添加 Cartfile 文件 (需要提交到 Git)

建立添加Cartfile（配置文件）文件在

通过终端或者文本编辑器 进入到项目所在的文件夹建立一个 空的Cartfile文件

现在只支持GitHub库(GitHub.com和GitHub企业),指定GitHub的关键字:

```ruby
github "ReactiveCocoa/ReactiveCocoa" # GitHub.com
github "https://enterprise.local/ghe/desktop/git-error-translations" # GitHub Enterprise
```

或者是其他git源,指定git关键词:

```r
git "https://enterprise.local/desktop/git-error-translations2.git"
```

其他可能的源在未来也会会被添加

## 版本指定

Carthage 支持以下几种版本指定方法:

1. `>= 1.0` 代表 “最低 1.0版本”
2. `~> 1.0` 代表 “表示使用版本1.0以上但是低于2.0的最新版本，如1.5, 1.9”
3. `== 1.0` 代表 “必须是 1.0 版本”
4. `"some-branch-or-tag-or-commit"`指定一个 Git 对象 (任何被 `git rev-parse` 允许的)

如果没有版本要求,任何版本的依赖是允许的。

版本好的兼容性是根据语语义化版本控制决定的。这意味着任何大于或等于1.5.1版本,但小于2.0,将认为与1.5.1“兼容”。

## Cartfile示例

```vim
# Require version 2.3.1 or later 最低2.3.1版本
github "ReactiveCocoa/ReactiveCocoa" >= 2.3.1

# Require version 1.x   必须1.x版本
github "Mantle/Mantle" ~> 1.0    # (大于或等于 1.0 ，小于 2.0)

# Require exactly version 0.4.1 必须0.4.1版本
github "jspahrsummers/libextobjc" == 0.4.1

# Use the latest version  使用最新版本
github "jspahrsummers/xcconfigs"

# Use the branch  使用git分支
github "jspahrsummers/xcconfigs" "branch"

# Use a project from GitHub Enterprise  使用一个企业项目，在 "development" 分支
github "https://enterprise.local/ghe/desktop/git-error-translations"

# Use a project from any arbitrary server, on the "development" branch  使用一个私有项目，在 "development" 分支
git "https://enterprise.local/desktop/git-error-translations2.git" "development"

# Use a local project   使用一个本地的项目
git "file:///directory/to/project" "branch"
```

## 安装依赖 just do it

执行以下命令 拉取指定版本代码并编译为 .Framework 文件

内部工作流程即  carthage update => carthage checkout => checkout build

```vim
carthage update
```

只编译iOS平台的类库

```vim
carthage update --platform iOS
```

结果如下,(PS我只留了一个Mantle依赖)

```vim
Jakey-Pro:test jakey$ carthage update
*** Fetching Mantle
*** Checking out Mantle at "1.5.7"
*** xcodebuild output can be found in /var/folders/sm/b5fssgjx147b722vsgx20mg00000gn/T/carthage-xcodebuild.b3IHTG.log
*** Building scheme "Mantle Mac" in Mantle.xcworkspace

...balabala
```

工程目录多了以下文件

![](http://www.skyfox.org/wp-content/uploads/2016/05/C888B12C-6500-4B6F-B7BE-075379FD5E95.jpg)

## Cartfile.resolved (需要提交到 Git)

在执行 carthage update 命令后会在根目录创建一个 Cartfile.resolved 文件，这个文件是生成后的依赖关系，不能修改。

Cartfile.resolved 文件确保提交的项目可以使用完全相同的配置与方式运行启用。 跟踪项目当前所用的依赖版本号，保持多端开发一致,出于这个原因,强烈建议提交这个文件到版本控制中。

## 自动生成的Carthage目录 (不需要提交到 Git)

Carthage文件夹用来存放:

carthage checkout 从git拉取的依赖库源文件(Checkouts)

carthage build编译后的文件(Build),包含Mac 与 iOS对应的.framework

## 引入 .Framework 动态库的方法

1 . 手动拖拽Build中的所有依赖.framework到你的工程,本人的建议当然是在工程根目录建立"Vendor"类似文件夹,创建"Vendor" folder/group到工程,所有第三方 .Framework都拷贝到此目录下,然后继续以下操作

![](http://www.skyfox.org/wp-content/uploads/2016/05/A5A69D07-3A4F-49EC-A1BB-8BF087C669D6.jpg) 

- 打开项目，点击project，选择target, 再选择上方的General，将需要的framework文件拖到 Embedded  Binaries(动态库)内

注意:动态库拷贝到Embedded  Binarie会同时自动加入到Linked Frameworks and Libraries,但是错误的拖入到Linked Frameworks and Libraries是不会自动增加到Embedded  Binarie中的,会导致动态库加载失败

2 . 在对应 Target 中的 Build Setting 中的 Framework Search Path 项加入以下路径，Xcode 便会自动搜索目录下的 Framework： 

```vim
$(PROJECT_DIR)/Carthage/Build/iOS
```

# Git 中忽略不需要提交到版本库的文件与文件夹

则修改 .gitignore 文件，增加忽略 Carthage 文件夹就行了：

```vim
#Carthage
Carthage
```

# 总结

本人在实际项目中迟迟没有使用CocoaPods的原因就是,啰里啰嗦...对原有工程破坏性大(建立workspace,增加一堆乱七八糟的文件),侵入性太强,耦合太高,Carthage的出现的确是茉莉花香扑鼻而至!

