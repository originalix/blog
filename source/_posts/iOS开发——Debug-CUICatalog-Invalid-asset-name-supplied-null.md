---
title: 'iOS开发——Debug CUICatalog: Invalid asset name supplied: (null)'
date: 2016-10-13 10:48:40
tags: ["iOS开发", "Debug"]

---


今天看到了一个Xcode log出了一个错误 `CUICatalog:Invalid asset name supplied: (null)`, Google了一下在StackOverflow上各位大神说应该是`[UIImage imageNamed:]`调用的时候, name为nil. 虽然在运行的时候界面一切正常, 但是看到这个log还是想干掉它，或许每个程序猿都是一个重度强迫症患者。

<!--more-->

需要解决的问题是查找所有`[UIImage imageNamed:]`调用的时候, 找到name是nil的地方, 但是整个项目一搜 `imageNamed` 显示 `267 results in 117 files`, 人工查找就算了吧,麻烦不说还显得蠢.

一开始想到的是用Method Swizzle来修改`[UIImage imageNamed:]`的实现, 在name为nil的时候用断言, 查看调用栈. 但是想想写了debug之后还得删掉, 比较麻烦.

于是机智的我想到了用`Symbolic Breakpoint`.

- 在Xcode的`Breakpoint Navigator`点击加号, 选择` Add Symbolic Breakpoint.`

- 右键选择`Breakpoint`选择 `Edit Breakpoint` , 在Symbol填入`[UIImage imageNamed:]` , 在Condition填入 `[(NSString *)$arg3 length] == 0 `或者` $arg3 == nil`. 可以自己尝试`po $arg1`, `po $arg2`试试看.

![](http://i.stack.imgur.com/ATz38.png)

- 运行程序, 直到程序进入断点. 打开`Debug Navigator`观察调用栈, 最顶部的一定是`[UIImage imageNamed:] `, 点击调用栈下一条, 能够看到有调用到`imageNamed`的代码, 就是`name`为`nil`的地方.

![](http://philcai.com/img/Debug2.png)