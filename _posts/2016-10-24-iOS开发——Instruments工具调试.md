---
layout: post
title: iOS开发——Instruments工具调试
categories: iOS开发
date: 2016-10-24 13:25:53
keywords: iOS开发, Instruments工具, 调试
---

随着项目的进行，APP的优化必须要尽早的展开了，所以最近自己在学习很多APP的调试技巧，今天我们就来说说Xcode为我们准备的自带的调试工具。

代码性能是个避不开的话题。随着项目的扩大和功能的增多，没经过认真调试和优化的代码，要么任性地卡顿运行，要么低调地崩溃了之……结果呢，大家用着不高兴，开发者也不开心。

本篇重点讨论一下 iOS性能测试中的启动测试、内存泄露测试、CPU测试。

<!--more-->

##1.启动测试
测试工具：Instruments > TimeProfile
可在 appDelegate.m中加入一段代码，来进行测试：

```objective-c
- (void)testLaunch
{
	for(int i = 0; i < 100000;i++){
		NSLog(@"test");
	}
}
```
###1）获得启动时间
APP启动之后，中止 TimeProfile，按住 option键在监控窗口中拖拽，选中监控区域中起始点到打开 APP后的峰谷，查看APP启动所需时间，如下图：
![图1 在 TimeProfile中查看启动时间](http://img.blog.csdn.net/20150428175349386)

###2）分析可优化空间
首先，需要注意一下右侧栏中的几个给力的筛选项，如下图：
![图2 TimeProfile右边栏](http://img.blog.csdn.net/20150428180729277)

（注：我觉得非常常用的标记为『必选项』）
> * Separate by Thread //按线程聚类，『可选项』，当我们想查看每个线程中哪些方法比较耗时时，勾选它；
> * Invert Call Tree //反转调用栈信息，『必选项』，否则 main一直排在最上面，碍事；
> * Hide System Libraries //隐藏系统库，『可选项』，只查看自己应用的栈信息；
> * Top Functions //按耗时降序排列，『必选项』


Running Time列中显示运行每个方法所耗费的时间，根据耗时和占比猜测是否有代码需要优化。双击中间主窗口中的方法名进入具体的代码行查看，耗时多的代码行有颜色标记，并显示占比。
![图3 TimeProfile 代码行](http://img.blog.csdn.net/20150428182138433)

获取 APP启动时间非常简单，但分析哪些地方可以优化，则需要对代码足够了解。项目的启动时间没有一个特定的值，利用该方法可以提供一个缩小的检测范围，尽可能发现可被优化的代码。

##2.内存泄露测试
有两种方法可以采用，第一利用静态分析，第二使用Instruments工具集。
###1）静态分析
在 xcode中长按运行按钮>Analyze，可启动代码静态分析。
![启动静态分析](http://img.blog.csdn.net/20150428182542806)

![这里写图片描述](http://img.blog.csdn.net/20150428191837926)
对于 MRC项目，静态分析是必要的，对于 ARC项目，静态分析作为可选项。
这项检查只覆盖代码编译时可能存在的问题，但并不能覆盖代码运行时。这时，我们还需要结合动态分析工具。
###2）动态分析
工具: Allocations,Leaks
#### 【Allocation】
Allocations组件监控对象调用了 alloc方法申请内存后的内存使用情况，可记录对象生命周期中内存引用计数的变化，当对象被正常释放后不再继续追踪。

####【Leaks】
Leaks监控内存泄露，一般和 Allocations一起使用，在检测到内存泄露后，通过 Allocations定位到具体的代码。发现问题时，监控图会显示红条。修改代码后，再次查看，如果红色消失则表示内存泄露被修复成功了。

但 Leaks可能会『假摔』，例如每次 APP启动后，都会显示几个红条，因此 Leaks的使用过程中也需要人工判断分析。

步骤：
a）运行Profile>Allocations，启动 APP后实时查看 Allocations\Leaks图，若 Leaks中出现红条，则双击红条，切换到 Leaks视图；
![这里写图片描述](http://img.blog.csdn.net/20150430114811540)
b) 选择右侧栏查看 stack trace，点击黑色图标（非系统类），查看具体的代码实现，分析可能出现的问题，如下图：
![这里写图片描述](http://img.blog.csdn.net/20150430140252822)

例如，上面的代码中，每次初始化都会创建一个NSMutableArray 对象，可以优化为removeAllObject后重利用。

###3）CPU等指标
工具：Activity Monitor
可监控 CPU和内存指标，并可对比多次监控的结果。

步骤：
a)profile>Activity Monitor 启动 APP, 运行过程中option选择峰值查看 cpu和内存使用量。
b)对比多次监控的结果，把最差情况作为最终结果；

![ActivityMonitor](http://img.blog.csdn.net/20150430141211461)