---
layout: post
title: "理解prototype、getPrototypeOf和\__proto\__之间的不同"
categories: 前端圈
date: 2018-11-21 16:26:31
keywords: JavaScript
---

在学习JavaScript的过程中，原型是如何也绕不过去的一个知识点。虽然在现在ES6已经非常普及的现在，许多js的程序员都已经不再用原型的知识点来编写代码了，但是充分的理解原型也是很有必要的，尤其是在阅读他人优秀的js代码时，理解原型能帮助我们更好的理解早期代码。而原型包括三个访问器，这三个访问器有时功能重叠，所以准确的理解并区分他们还是很有必要的。

<!--more-->

这三个访问器就是`prototype`、`getPrototypeOf`和`__proto__`，从名字上可见这三个访问器都是对prototype这个单词做了一些变化，生成这样的属性方法名。为了测试这三个方法的输出，我们先来模拟创建一个存储用户数据User的类。

```js
function User(name, passwordHash) {
  this.name = name;
  this.passwordHash = passwordHash;
}

User.prototype.toString = function() {
  return '[User ' + this.name + ']';
}

User.prototype.checkPassword = function(password) {
  return hash(password) === this.passwordHash;
}

```

这里我们创建的这个User类的构造函数，接收两个参数，一个是用户名name，一个是密码的hash值，并且类中有两个方法`toString`以及`checkPassword`用来输出用户信息和检查密码。

如果这个时候我们打印这三个原型方法的日志会得到一样的结果

```js
var u = new User('Lix', '123456');

console.log(Object.getPrototypeOf(u)); // User { toString: [Function], checkPassword: [Function] }

console.log(u.__proto__); // User { toString: [Function], checkPassword: [Function] }

console.log(User.prototype); // User { toString: [Function], checkPassword: [Function] }
```

既然他们的输出都一样，那么他们是否作用一样呢，我们可以来比较测试一下。

```js
Object.getPrototypeOf(u) === User.prototype; // true
```

```js
u.__proto__ === User.prototype; // true
```

既然这两个方法都跟我们User对象的原型相等，那么这三个属性的区别究竟是什么呢？别急，接下来就把结论告诉大家。

- `C.prototype`用于建立由 `new C()` 创建的对象的原型。
- `Object.getPrototype(obj)`是ES5中用来获取obj对象的原型对象的标准方法。
- `obj.__proto__`是获取obj对象的原型对象的非标准方法。

所以一般我们是不会直接访问`C.prototype`去获取原型对象的，在ES5的环境中，我们使用`Object.getPrototype(obj)`来获取原型对象，而在不支持ES5的环境中，我们可以考虑用`__proto__`这样的非标准方法来当做权宜之计，希望各位不明白的同学能牢记这些区别。