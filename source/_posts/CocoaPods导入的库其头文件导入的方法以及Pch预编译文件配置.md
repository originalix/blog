---
title: CocoaPods导入的库其头文件导入的方法以及Pch预编译文件配置
date: 2016-08-02 10:09:13
tags: ["iOS开发", "CocoaPods", "Pch预编译文件"]

---


# CocoaPods 导入头文件

尽管CocoaPods使用十分方便,但其导入的第三方框架还是要经过几步操作,才能供项目使用

<!--more-->

> 第一步:导入库

这里要讲的配置CocoaPods以及安装第三方库，之前的文章已经讲过，这里就不再赘述。

> 第二步: 添加文件路径

- 选择工程的 Target -> Build Settings 菜单->搜索header,找到"User Header Search Paths";

![](http://upload-images.jianshu.io/upload_images/105827-cc5c5d4a7c590039.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 新增一个值"$(PODS_ROOT)",并且选择”recursive”，这样Xcode就会在项目目录中递归搜索文件且会自动找到Pods文件,头文件自动补齐功能马上就好使了.

![](http://upload-images.jianshu.io/upload_images/105827-ecb9f73eb55e5f32.png?imageMogr2/auto-orient/strip)

# Pch预编译文件配置

[该步骤相当于在项目自动"import"头文件,是不是很方便.(该步骤可不用,但使用相当方便,建议使用)].

首先说一下pch的作用：

1.存放一些全局的宏(整个项目中都用得上的宏)

2.用来包含一些全部的头文件(整个项目中都用得上的头文件)

3.能自动打开或者关闭日志输出功能


- 在工程的 TARGETS 里边 Building Setting 中搜索 Prefix Header，然后把 Precompile Prefix Header 右边的 NO 改为 Yes， 预编译后的pch文件会被缓存起来，可以提高编译速度


- 然后在 Precompile Prefix Header 下边的 Prefix Header 右边双击，添加刚刚创建的pch文件的工程路径，添加格式：`$(SRCROOT)/项目名称/pch文件名`  ，`$(SRCROOT)` 的意思就是工程根目录的意思。如果还不太清楚的话可以右键 pch 文件，然后show in finder

![](http://img.blog.csdn.net/20150428202907251?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ3JhenlaaGFuZzE5OTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

上图中黄色圈出来的就是 `$(SRCROOT)`，也就是工程的根目录，然后后边还有一个 PchText 和 pch 两个文件夹，所以完整的 pch 文件的路径就是：`$(SRCROOT)/PchText/pch`

添加完成后，点击Enter，他会自动帮你变成你工程所在的路径

可以了，编译一下程序，如果有错误检查一下添加的路径是否正确

