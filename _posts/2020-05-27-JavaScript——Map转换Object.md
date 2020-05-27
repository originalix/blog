---
layout: post
title: JavaScript —— Map转换Object
categories: JavaScript
description: Map 转换成 Object
keywords: JavaScript, es6, Map, Object
---

本文记录 ES6 中新增的 Map 对象转换为 Object 的几种方式，以及测试各种方式转换的性能。

```js
const map = new Map()
map.set('🏀', 'basketball')
map.set('️⚽️', 'soccer')
map.set('⚾️', 'baseball')
map.set('🎾', 'tennis')
```

### 第一种方式

首先我们准备一个 map 对象，接下来看第一种方式：

```js
const obj = Array.from(map).reduce((obj, [key, value]) =>
  Object.assign(obj, { [key]: value} )
, {})

console.log(obj)  // { '🏀': 'basketball', '️⚽️': 'soccer', '⚾️': 'baseball', '🎾': 'tennis' }
```

但是第一种方式在数据量过大的时候，在每个迭代中创建一个新对象（使用 `Object.assign`）时，性能会受到影响，还有一点是 Map 的 key 可以是非字符串的键，转换成字面量的 object 则不可以。

### 第二种方式

于是我们来看第二种方法，来解决第一种方法可能会遇到的性能问题：

```js
const obj = Array.from(map).reduce((obj, [key, value]) => {
  obj[key] = value
  return obj
}, {})

console.log(obj)  // { '🏀': 'basketball', '️⚽️': 'soccer', '⚾️': 'baseball', '🎾': 'tennis' }
```
使用 `Array.from(map).reduce(fn, {})`, 你可以安全的在累加器中操作 object

### 第三种方式
如果你熟悉 ES6 中的写法，你也可以用第三种 ES6 的方式来替换 `Array.from(map)`:

```js
const obj = [...map.entries()].reduce((obj, [key, value]) => (obj[key] = value, obj), {})

console.log(obj)  // { '🏀': 'basketball', '️⚽️': 'soccer', '⚾️': 'baseball', '🎾': 'tennis' }
```

用 ES6 的写法，我们只需要用一行代码就可以完成任务啦，简洁控的最爱。

### 第四种方式
而最后一种方法，也是跟扩展运算符相关的一种写法：

```js
const obj = Array.from(map.entries()).reduce((main, [key, value]) => ({...main, [key]: value}), {})

console.log(obj)  // { '🏀': 'basketball', '️⚽️': 'soccer', '⚾️': 'baseball', '🎾': 'tennis' }
```

这行代码乍一看完全没问题，因为使用了扩展运算符还有点优雅的感觉，并且也是一行代码解决了战斗，但是在后面的性能测试过程中，第四种方法当真是让人大跌眼镜。

### 性能测试
现在我把四种写法放到一起，并且我创建一个拥有 10000 个 key 的 Map 来做转换，测试一下四种写法的性能。

```js
// 创建一个较大的 map
for (let i = 0; i < 10000; i++) {
  map.set(`a${i}`, i)
}

// MapConvertToObj1: 16.381ms
// MapConvertToObj2: 5.661ms
// MapConvertToObj3: 4.903ms
// MapConvertToObj4: 20344.747ms
```

看到这 4 种方式的性能输出，是不是大跌眼镜呢？并且第一种方式，果然是因为 `Object.assign()` 的用法存在性能开销，总体比第二种和第三种慢一点。

如果我们把 key 的数量减少到 1000 个，第四种方式会不会好一点呢？

```js
// MapConvertToObj1: 3.742ms
// MapConvertToObj2: 1.140ms
// MapConvertToObj3: 0.874ms
// MapConvertToObj4: 185.745ms
```

可以看到第四种方式还是没有太多起色，而多次测试下来，第三种方式是转换速度最快的，推荐大家以后 Map 转换成对象时，使用第三种方式来转换哦，又快又优雅。
