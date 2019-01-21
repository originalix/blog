---
title: Swift中的警卫队
date: 2017-02-22 19:12:03
tags: ["Swift", "guard"]

---


大半个月没有更新自己的博客了，最近在忙一个新项目时间非常紧张，所以最近的博客更新进度就要稍微放缓一点了。

这个项目是纯粹的Swift项目，所以最近的博客会结合自己在使用Swift这门语言的过程中，对Swift的总结和感悟。今天就来介绍一下能让你在Swift中如虎添翼的警卫队成员 —— `guard`。

我们在编写业务代码中，经常会遇到的一种情况就是一大串的**if else*，一层又一层的甚至还有嵌套，看的眼花缭乱，有时候根本不知道跑到了哪一层了，代码的可读性非常差。而在Swift里有比**if else**更优雅的写法，那就是`guard`。

<!--more-->

## 使用 guard 来判断 nil

传统的Objective-C在判断**nil**的时候，会用下面的写法:

```swift
if (something == nil) {
    // doSomething
} else {
    // throw error
}
```

而若是我们改用**guard**来实现的话会是下面的情况:

```swift
guard (something == nil) else {
    // throw error
    return // guard 裡面一定要有 return
}

// doSomething
```

改成这样，判断并不会一层一层的累加下去，代码看上去也比之前美观易懂了很多，但是要注意的是不要搞错逻辑，免得直接**return**了。

而这样的用法也符合[Swift编码规范](https://github.com/github/swift-style-guide)所强调的那样，尽早的`Return`或者`break`。

## 使用 guard 来判断类型

介于Swift里的Optional类型的增加，我们有时候也可以使用**guard**来判断一个属性的类型。例如如下代码所做的那样:

```swift
// 声明的function
func doSomething(input: Bool, handler: (obj: AnyObject?) -> Void) -> Void {
    let someDict: [Int: String] = [1: "One", 2: "Two", 3: "Three"]

    if input {
        handler(obj: someDict)
    } else {
        handler(obj: [1,2,3])
    }
}

// 执行函数
doSomething(true) { (obj) -> Void in
    // 判断是否为 [Int:String] 的类型(Dictionary)
    guard let someDict = obj as? [Int:String] else {
        print("obj not match dictionary")
        return
    }
    // 将 Dictionary 里面每个 value 打印出来
    for (key, value) in someDict {
        print("Dictionary key \(key) -  Dictionary value \(value)")
    }
    // 自行输入 key 但会印出 Optional
    print( "Value of key = 1 is \(someDict[1])" )
    print( "Value of key = 2 is \(someDict[2])" )
    print( "Value of key = 3 is \(someDict[3])" )
}
```

通过上面的🌰，我们已经可以看到一个如何对可选类型解包的操作了。许多人在初入Swift的大门时，经常被可选类型搞的程序Crash，所以学以致用，用上优雅的警卫队吧。让他来帮助你实现更加优雅健壮的代码。