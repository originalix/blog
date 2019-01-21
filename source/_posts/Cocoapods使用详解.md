---
title: Cocoapods使用详解
date: 2016-04-10 17:03:07
tags: ["Cocoapods教程","Cocoapods","iOS开发","iOS第三方库管理"]

---


# CocoaPods使用


[CocoaPods简介](#a1)

[CocoaPods 的安装和使用介绍](#a2)

<div id="a1"></div>

## CocoaPods简介

当你开发iOS应用时，会经常使用到很多第三方开源类库，比如JSONKit，AFNetWorking等等。可能某个类库又用到其他类库，所以要使用它，必须得另外下载其他类库，而其他类库又用到其他类库，“子子孙孙无穷尽也”，这也许是比较特殊的情况。另外一种常见情况是，你项目中用到的类库有更新，你必须得重新下载新版本，重新加入到项目中，十分麻烦。如果能有什么工具能解决这些恼人的问题，那将“善莫大焉”。所以，你需要 CocoaPods。 CocoaPods应该是iOS最常用最有名的类库管理工具了，上述两个烦人的问题，通过cocoaPods，只需要一行命令就可以完全解决，当然前提是你必须正确设置它。重要的是，绝大部分有名的开源类库，都支持CocoaPods。所以，作为iOS程序员的我们，掌握CocoaPods的使用是必不可少的基本技能了。

<!--more-->

# CocoaPods 的安装和使用介绍

### 安装

安装方式异常简单 , Mac 下都自带 ruby，使用 ruby 的 gem 命令即可下载安装：

```java
$ sudo gem install cocoapods
$ pod setup
```
如果你的 gem 太老，可能也会有问题，可以尝试用如下命令升级 gem:

```java
sudo gem update --system
```
另外，ruby 的软件源 https://rubygems.org 因为使用的是亚马逊的云服务，所以被墙了，需要更新一下 ruby 的源，使用如下代码将官方的 ruby 源替换成国内淘宝的源：

```java
gem sources --remove https://rubygems.org/
gem sources -a http://ruby.taobao.org/
gem sources -l
```
还有一点需要注意，pod setup在执行时，会输出Setting up CocoaPods master repo，但是会等待比较久的时间。这步其实是 Cocoapods 在将它的信息下载到 ~/.cocoapods目录下，如果你等太久，可以试着 cd 到那个目录，用du -sh *来查看下载进度。你也可以参考本文接下来的使用 cocoapods 的镜像索引一节的内容来提高下载速度。

### 使用 CocoaPods 的镜像索引

所有的项目的 Podspec 文件都托管在https://github.com/CocoaPods/Specs。第一次执行pod setup时，CocoaPods 会将这些podspec索引文件更新到本地的 ~/.cocoapods/目录下，这个索引文件比较大，有 80M 左右。所以第一次更新时非常慢，笔者就更新了将近 1 个小时才完成。

一个叫 akinliu 的朋友在 gitcafe 和 oschina 上建立了 CocoaPods 索引库的镜像，因为 gitcafe 和 oschina 都是国内的服务器，所以在执行索引更新操作时，会快很多。如下操作可以将 CocoaPods 设置成使用 gitcafe 镜像：

```java
pod repo remove master
pod repo add master https://gitcafe.com/akuandev/Specs.git
pod repo update
```
将以上代码中的 https://gitcafe.com/akuandev/Specs.git 替换成 http://git.oschina.net/akuandev/Specs.git 即可使用 oschina 上的镜像。

### 使用 CocoaPods

使用时需要新建一个名为 Podfile 的文件，以如下格式，将依赖的库名字依次列在文件中即可

```java
platform :ios
pod 'JSONKit',       '~> 1.4'
pod 'Reachability',  '~> 3.0.0'
pod 'ASIHTTPRequest'
pod 'RegexKitLite'
```
然后你将编辑好的 Podfile 文件放到你的项目根目录中，执行如下命令即可：

```java
cd "your project home"
pod install
```
现在，你的所有第三方库都已经下载完成并且设置好了编译参数和依赖，你只需要记住如下 2 点即可：

使用 CocoaPods 生成的 .xcworkspace 文件来打开工程，而不是以前的 .xcodeproj 文件。
每次更改了 Podfile 文件，你需要重新执行一次pod update命令。
查找第三方库

你如果不知道 cocoaPods 管理的库中，是否有你想要的库，那么你可以通过 pod search 命令进行查找，以下是我用 pod search json 查找到的所有可用的库：

```java
$ pod search json

-> AnyJSON (0.0.1)
   Encode / Decode JSON by any means possible.
   - Homepage: https://github.com/mattt/AnyJSON
   - Source:   https://github.com/mattt/AnyJSON.git
   - Versions: 0.0.1 [master repo]


-> JSONKit (1.5pre)
   A Very High Performance Objective-C JSON Library.
   - Homepage: https://github.com/johnezang/JSONKit
   - Source:   git://github.com/johnezang/JSONKit.git
   - Versions: 1.5pre, 1.4 [master repo]

// ... 以下省略若干行

```
关于 Podfile.lock

当你执行pod install之后，除了 Podfile 外，CocoaPods 还会生成一个名为Podfile.lock的文件，Podfile.lock 应该加入到版本控制里面，不应该把这个文件加入到.gitignore中。因为Podfile.lock会锁定当前各依赖库的版本，之后如果多次执行pod install 不会更改版本，要pod update才会改Podfile.lock了。这样多人协作的时候，可以防止第三方库升级时造成大家各自的第三方库版本不一致。

CocoaPods 的这篇 官方文档 也在What is a Podfile.lock一节中介绍了Podfile.lock的作用，并且指出：

```java
This file should always be kept under version control.
```

### 不更新 podspec
```java
pod install --no-repo-update
pod update --no-repo-update
```

<div id="a3"></div>

