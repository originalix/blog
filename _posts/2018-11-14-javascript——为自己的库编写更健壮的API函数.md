---
layout: post
title: javascript——为自己的库编写更健壮的API函数
categories: 前端圈 读书笔记
date: 2018-11-14 14:05:25
keywords: javaScript, API设计
---

最近在看书的时候，阅读了关于使用JavaScript在代码库的设计时需要注意的文章，对我的启发很大，于是决定记录一些其中的知识点，一是分享自己获取到的知识，二是辅助记忆，让我以后更注意地去编写更健壮的JavaScript函数。

首先我们要记住的一个规则就是**使用undefined来代替没有值的情况**。

<!--more-->

我们来看下面的这个例子，有一个对象，有宽高的属性，我们传入宽高属性并用构造函数创建对象。

```js
function Element(width, height) {
  this.width = width || 320; // wrong test
  this.height = height || 240; // wrong test
  // ...
}

var c1 = new Element(); // width: 320, height: 240
```

如c1这个对象一样，他创建并且使用了我们预先设置的宽高的默认值。乍一看是不是没有什么问题，可是这里就隐藏了一个bug。如果我们想创建一个宽高都为0的对象，那么这个写法就会产生问题了。


```js
var c2 = new Element(0, 0);
c2.width; // 320
c2.height; // 240
```

可见如果目标属性可以接受0作为参数，那么我们就不能使用或来取值。而对于String类型的对象的话，使用或还是可行的。那么对于能接受0作为值的参数，我们应该如何编写代码呢？答案很简单，使用undefined来代替没有值的情况就可以了。

```
function Element(width, height) {
  this.width = width === undefined ? 320 : width;
  this.height = height === undefined ? 240 : height;
  // ...
}

var c1 = new Element(0, 0);
c1.width; // 0
c1.height; // 0

var c2 = new Element();
c2.width; // 320
c2.height; // 240
```

当我们用undefined来判断是否有值的情况时，这个bug也就自然而然的没有了。

第二个需要我们记住的规则是**函数有时应该接受关键字对象作为参数**。

现在我们假设我们要设计一个第三方的弹窗库，我们有一个弹窗的对象Alert。那么我们先来看看这个构造函数:

```js
var alert = new Alert(100, 75, 300, 200, 'Error', message, 'blue', 'white', 'black', 'error', true);
```

这就是我们设计的构造函数，需要把每个参数对应的传入。如果你在阅读到这样的代码时，是不是会觉得非常难受，因为你并不知道每个参数对应着什么意思，尤其是最后一个`true`，到底代表的是什么布尔值？

而如果这时我们使用关键字对象来作为参数又会是什么表现呢？

```js
var alert = new Alert({
  x: 100,
  y: 75,
  width: 300,
  height: 200,
  title: 'Error',
  message: 'this is message',
  titleColor: 'blue',
  bgColor: 'white',
  textColor: 'black',
  icon: 'error',
  modal: 'true'
});
```

看看这个新的构造函数，是不是舒服很多了，所有的参数意义一目了然。

但是这样的设计也存在一个问题，如果有的必传参数，漏传了怎么办？那么程序就会运行错误了。所以我们可以把一些必传的参数提取出来，放入构造函数的参数内。

```js
var alert = new Alert(app, message, {
 width: 150, height: 100,
 title: "Error",
 titleColor: "blue", bgColor: "white", textColor: "black",
 icon: "error", modal: true
});
```

如果提取出必须要传入的参数的话，构造函数就是这样了，这样看还是比较清晰的呢。那么再结合我们的第一条规则，将undefined作为没有值的情况，完整的构造函数代码，就像下面这样了：

```js
function Alert(parent, message, opts) {
  opts = opts || {};
  this.width = opts.width === undefined ? 320 : opts.width;
  this.height = opts.height === undefined ? 200 : opts.height;
  this.x = opts.x === undefined
         ? (parent.width / 2) - (this.width / 2)
         : opts.x;
  this.y = opts.y === undefined
         ? (parent.height / 2) - (this.height / 2)
         : opts.y;
  this.title = opts.title || 'Alert';
  this.titleColor = opts.titleColor || 'blue';
  this.bgColor = opts.bgColor || 'white';
  this.textColor = opts.textColor || 'black';
  this.icon = opts.icon || 'info';
  this.modal = !!opts.modal;
  this.message = message;
}
```

再往后优化的话，还可以使用一些库里的extend方法了，由于并不是标准库的方法，我在这里也就不讲下去了。希望这些分享能给初学JavaScript的同学一点帮助。