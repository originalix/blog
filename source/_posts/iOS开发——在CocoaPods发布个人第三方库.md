---
title: iOS开发——在CocoaPods发布个人第三方库
date: 2017-01-17 17:14:24
tags: ["iOS开发", "CocoaPods"]

---

> 在进行日常的 iOS 开发的时候，我们肯定会用到各种第三方库，每次使用一个库的时候，各种依赖配置总是让人很头痛。幸好我们有了 CocoaPods，CocoaPods 为我们节省了大量集成第三方库的时间。在我们便捷地使用他人制作的 Pod 的时候，一定也希望能够将自己的代码制作成 Pod 供别人使用，来为开源社区做一点贡献。接下来，我们就来尝试制作一个属于自己的 Pod 库，并发布供他人使用。

备注：假设你已经拥有了一个完善的用于制作 Pod 的 iOS 工程。

<!--more-->

# 创建 Pod 库依赖的文件
## 1. podspec 文件

podspec 文件是 Pod 库的描述文件，每个 Pod 库必须有且仅有一个这样的文件。文件名需要与我们将要创建的 Pod 库的名称一致，如我将创建的 Pod 库名称为 LixFoundation， 那么我的 podspec 文件就是 LixFoundation.podspec。

## 创建 podspec 文件

在工程目录下，执行(注意把名称换成你自己的名称)

```
pod spec create LixFoundation
```

创建完成后，在你的工程目录下就会出现对应的文件。以下是我创建出来的文件（为了方便阅读，我把所有注释和不需要的选项都删去了）：

```
Pod::Spec.new do |s|

  s.name         = "LixFoundation"
  s.version      = "0.0.1"
  s.summary      = "Objective-C编程基础工具类"

  s.homepage     = "https://github.com/originalix/LixFoundation"

  #s.license      = "MIT"
  s.license      = { :type => "MIT", :file => "LICENSE" }


  s.author             = { "Lix" => "xiao.liunit@gmail.com" }

  s.platform     = :ios
  s.platform     = :ios, "7.0"

  s.source       = { :git => "https://github.com/originalix/LixFoundation.git", :tag => "#{s.version}" }

  s.source_files  = "Lix_Foundation", "Lix_Foundation/**/*.{h,m}"
  s.frameworks = "Foundation", "UIKit"
  s.dependency 'LixMacro', '~> 0.0.3'

end
```

该文件虽然是 Ruby 文件，但是里面的条目的意思都明显，就算没有学过 Ruby 也不影响。
几个需要说明的参数：

1. s.homepage: 一般就是 Github 仓库的主页）。
2. s.license: 指的是 Pods 依赖库使用的 license 类型。
3. s.source: 表示 git 仓库的地址，注意是指向 tag 的，所以需要给你的 git 仓库打上 tag。
4. s.source_files : 表示源文件的路径，是相对于创建的 podspec 文件而言的。

## 其他应该具备的文件

### 1. license 文件

`CocoaPods` 强制要求所有的 `Pod` 依赖库都必须有 `license` 文件，否则验证不会通过。`license` 的类型很多，你可以在创建 `Github `仓库的时候一并创建，也可以在后来添加进去。具体的 `license` 类型和适合你的 `license` 请自行 `google`。我在 Github 创建项目的时候，选择了 `MIT license` 一并创建

### 2. 主文件

`Pod` 的根本目的就是将自己创建的类供他人使用，比如我的 Pod 的作用就是将 `LixFoundation` 类分享给别人使用，方便他们快捷的使用常用扩展和宏定义。所以我将 `LixFoundation` 放入到 `LixFoundation` 文件夹中。

### 3. Demo 工程

`Demo` 工程是为了方便向其他使用者展示该 `Pod` 的使用。我将 `Demo`工程放在了 `LixFoundationDemo` 文件夹中。

### 4. README 文件

该文件一般使用 `Markdown` 语言标记，是对仓库的详细说明。作为一个共享给他人使用的 `Pod` 库，`README` 文件是必不可少的，这样对方才能很快的了解你的 `Pod` 具体是干嘛用的。

## 提交修改的文件`

### 1. 提交修改到 Github

依次执行以下命令：

- 将新添加的文件加入到 `git` 管理，并提交一个 `commit`
- 打上 `tag` 为 `0.0.1`（为了 `Pod` 指向）
- 提交 `tag`
- 提交到 `Github` 仓库

```
git add -A && git commit -m "Release 0.0.1"
git tag '0.0.1'
git push --tags
git push origin master
```
如果一切正常，就可以在 `Github` 上看到刚才的改动

### 2. pod 验证

执行以下命令，为 `Pod` 添加版本号，并打上 `tag`：

```
set the new version to 0.0.1
set the new tag to 0.0.1
```

接下来是验证

```
pod lib lint
```

如果一切正常，终端中会输出：

```
 -> LixFoundation (0.0.1)

LixFoundation passed validation.
```

到此，pod 验证就成功。

##Trunk 你的 `Pod`

以上工作都就绪后，就可以将我们的 `Pod ` 提交给 `CocoaPods` 了，CocoaPods 使用 `trun` 服务让我们来提交 `Pod`。
`
### 1. 注册 Trunk

使用以下命令进行 `trunk` 注册：

```
pod trunk register  youremail@gmail.com 'ihomway' --description='macbook pro @ home' --verbose
```

上面的命令是我注册时使用的，你需要把邮箱和名字以及描述替换成你的，加上 `--verbose` 可以输出详细 `debug` 信息，方便出错时查看。

注册后 `CocoaPods` 会给你的邮箱发送验证链接，点击后就注册成功了，可以用 `pod trunk me` 命令查看自己的注册信息，如我的注册信息是：


```
  - Name:     **********
  - Email:    **********
  - Since:    **********
  - Pods:
    - Lix_ModelSqliteKit
    - LixMacro
    - LixFoundation
  - Sessions:
    - January 99th, 99:99 - May 25th, 03:02. IP: ********
    Description: macbook pro @ home
```

### 2. 部署你的Pod

使用以下命令，通过 `trunk` 部署你的 `Pod`:

```
pod trunk push LixFoundation.podspec
```

将 `podspec` 的文件名换成自己的文件名。

如果你出现报错，请根据提示寻找错误的原因。

之后你可以运行 `pod setup` 来更新你的 Pod 依赖库后，再使用 `pod search LixFoundation` 命令来查找刚刚加入的名字叫 `LixFoundation` 的 `Pod`。