---
layout: post
title: iOS开发——GCD在Swift中的变脸
categories: iOS开发
date: 2016-11-02 08:34:25
keywords: iOS开发, 多线程, Swift
---

Xcode8正式发布后，Swift3也随即发布，为了跟上苹果这艘大船的脚步，赶紧逼着自己看文档哦。在看文档的过程中，发现GCD的变化跟OC相比简直都要不认识了，赶紧写个文章总结下，顺手复习下GCD中死锁的概念，死锁的总结发布在另一篇文章里了。

# 取消过去的接口

说起 GCD， 大家肯定回想起类似 `dispatch_async` 这样的语法。 GCD 的这个语法模式无论是和 Objc 还是 Swift 的整体风格都不太搭调。 所以 Swift 3 中对它的语法进行了彻底的改写。

<!--more-->

比如最常用的，在一个异步队列中读取数据， 然后再返回主线程更新 UI， 这种操作在新的 Swift 语法中是这样的：

```swift
DispatchQueue.global().async {
    
    DispatchQueue.main.async {
        
        self.label?.text = "finished"
        
    }
    
}
```

# DispatchQueue

首先，dispatch 的全局函数不再写为下划线风格的名称了，它变成了一个更符合 Swift 风格的 DispatchQueue 的样子。

# main thread

同样的，你不需要在去用  `dispatch_get_main_queue()` 来获取主线程，而是 `DispatchQueue.main` ，那么要放到主线程的代码怎么执行呢？只需要在线程后边使用  `.async {}` 即可，也就是说，大概是这样：

```swift
DispatchQueue.main.async { [weak self] in
      your code runs in main thread
}
```

# 优先级

无论从代码长度，已经语法含义上都清晰了一些呢。 另外， 这次对 `GCD` 的改进还包括优先级的概念。 以往我们使用 `Global Queue` 的时候，可以使用`DISPATCH_QUEUE_PRIORITY_DEFAULT` 或 `DISPATCH_QUEUE_PRIORITY_BACKGROUND` 等，来指定队列的优先级。 而新的 `GCD` 引入了 `QoS (Quality of Service)` 的概念，体现在代码上面就是优先级所对应的名称变了， 对应关系如下：

```swift
* DISPATCH_QUEUE_PRIORITY_HIGH:         .userInitiated
* DISPATCH_QUEUE_PRIORITY_DEFAULT:      .default
* DISPATCH_QUEUE_PRIORITY_LOW:          .utility
* DISPATCH_QUEUE_PRIORITY_BACKGROUND:   .background
```
举个例子，如果想以最高优先级执行这个队列， 那么就可以这样：
 
```swift

DispatchQueue.global(qos: .userInitiated).async {
            
            
}
```

所以这个优先级概念的变化， 大家也需要留意一下。

# 获取一个队列

我们使用  `DispatchQueue.global()` 获取一个系统的队列，这样的话获取的就是默认  `.default` 优先级的队列了，如果要获取其他优先级的队列，就使用 `DispatchQueue.global(qos: .userInitiated)` ，最后，我们使用 `.async {}` 来执行代码。

# 创建一个队列

直接用`DispatchQueue` 的初始化器来创建一个队列。最简单直接的办法是这样：

```swift
let queue = DispatchQueue(label: "myBackgroundQueue")

```

复杂一点？你可以指定优先级以及队列类别：

```swift
let queue = DispatchQueue(label: "myBackgroundQueue", qos: .userInitiated, attributes: .concurrent)
```

然后把代码放进去即可：

```swift
queue.async {
    print("aaa")
}
```

# 组队列

对于组，现在你可以使用这样的语法直接创建一个组：

```swift
let group = DispatchGroup()
```

至于使用，则是这样的：

```swift
let group = DispatchGroup()
 
let queue = DispatchQueue(label: "myBackgroundQueue")
 
queue.async(group:group) {
    print("background working")
}
```

那么，如果有多个并发队列在同一个组里，我们需要它们完成了再继续呢？

```swift
	
group.wait()
```

# dispatch_time_t

还有一个是对 dispatch_time_t 的改进：

```swift
let delay = DispatchTime.now() + .seconds(60)
DispatchQueue.main.after(when: delay) {
    // Do something
}
```

语法使用起来更加简单。DispatchTime.now() 是当前事前， 然后加上 .seconds(60) 代表 60秒。 再使用 DispatchQueue.main.after 让这个操作在 60 秒后执行。 相比于之前的 GCD 语法，那就容易理解很多了。

顺手儿把 GCD 以前获取当前时间的语法贴出来对比一下：

```swift
let dispatch_time = dispatch_time(DISPATCH_TIME_NOW, Int64(60 * NSEC_PER_SEC))
```

这样一比， 立竿见影哈。 至少上面新的 GCD 语法， 我大概看了一眼，就能明白它是什么意思了。 而下面这个老的语法，如果不查阅相关文档的话，第一眼恐怕没那么容易看懂了。

# 结尾

Swift 3 对 GCD 的语法改进还是很大的。 新的语法结构还是很不错的， 当然大多数朋友应该都习惯了以前的写法，也包括我~ 所以肯定需要一点时间适应。 希望这篇文章能帮你节省查阅文档的时间， 在闲暇时刻了解一些技术点。

