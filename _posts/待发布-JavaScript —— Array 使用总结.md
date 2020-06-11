<!-- ---
layout: post
title: JavaScript —— Array 使用汇总
categories: JavaScript 数组，Array
description: js数组使用
keywords: js, javascript，array，数组, 数组使用
--- -->

作为一名前端工程师，数组应该是我们写的最频繁的一种数据结构。所以弄懂 js 中的数组用法，是非常有必要的一件事情。今天我就准备按分类，总结一下数组的各种用法。

## Property

| 属性名 | 说明 |
|---|---|
| length | 数组的 length 属性，值为 0 |
| constructor | 数组实例都继承这个属性，表明了所有数组都是由 Array 构造出来的 |

由于 length 和 prototype 两个属性比较通用，所以这里不过多的介绍。

## Method

| 方法名 | 说明 |
|---|---|
| from | 从类数组的对象或者可迭代对象中，创建一个新的数组实例 |
| isArray | 判断变量是否是一个数组 |
| of | 根据参数来创建新的数组实例，参数数量和类型任意 |

#### Array.from()

对于 Array.from 有以下几个点要注意
    - 可以通过伪数组对象(有 length 属性)、可迭代对象(可以获取对象中的元素，如 Map 和 Set 等)来创建数组对象。
    - 该函数第二个参数是一个可选参数 —— mapFn
        - 如果指定了该参数，则在生成数组之后，会对数组执行 map 方法后再返回。
```js
// 常规使用
Array.from(foo)
// expected output: Array ['f', 'o', 'o']

// 传入 mapFn 时
Array.from([1, 2, 3], x => x + 1)
// expected output: Array [2, 3, 4]
```
`Array.from(obj, mapFn)` 相当于 `Array.from(obj).map(mapFn)`。

#### Array.isArray()

根据值是否是数组，返回 true 或 false

```js
Array.isArray(['⚽️', '🏀']) // true
Array.isArray({ foo: 'bar' }) // false
Array.isArray('foo') // false
```

#### Array.of()
Array.of() 和 Array 构造函数之间的区别在于处理整数参数，`Array.of(7)`创建具有单个元素 7 的数组，而 `Array(7)` 创建一个长度为 7 的空数组。

```js
Array.of('🥎') // ['🥎']

Array.of('🏓', '🏐', '🏉') // ['🏓', '🏐', '🏉']
```

## 实例方法

对于数组的实例方法，我们将它们分为修改器方法、访问方法以及迭代方法。

### 修改器方法 —— 调用后会改变自身的值

| 方法名 | 说明 | 返回值 |
|---|---|---|
| `copyWithin()` | 在数组内部，将一段元素序列拷贝到另一段元素序列上，覆盖原有的值 | 改变之后的数组 |
| `fill()` | 将数组中指定区间的所有元素的值，替换成某个固定值 | 修改后的数组 |
| pop() | 删除数组的最后一个元素 | 返回弹出的元素 |
| push() | 在数组的末尾增加一个或多个元素 | 返回数组的新长度 |
| reverse() | 颠倒数组中元素的排列顺序 | 颠倒后的数组 |
| shift() | 删除数组中的第一个元素 | 返回被删除的元素 |
| unshift() | 在数组的开头增加一个或多个元素 | 返回数组的新长度 |
| sort() | 对数组元素进行排序 | 返回排序后的数组 |
| splice() | 在任意的位置，给数组添加或删除任意的元素 | 返回被删除元素组成的数组|

关于修改器方法，对于传参的索引值有个要注意的地方。对于 `fill()` 、 `copyWithin()`、 `splice()` 等方法，传参索引的基底为 0，如果是负数，那么一般是从数组的末尾开始计算的。

#### copyWithin()

```js
const array = [0, 1, 2, 3, 4, 5]
console.log(array.copyWithin(0, 3, 4)) // [3, 1, 2, 3, 4, 5]
console.log(array.copyWithin(1, 3)) // [3, 3, 4, 5, 4, 5]
```

该函数接收三个参数 target、start、end，如果 end 被忽略，则会一直复制到数组底部。

#### fill()

```js
[1, 2, 3].fill(4) // [4, 4, 4]
[1, 2, 3].fill(4, 1) // [1, 4, 4]
[1, 2, 3].fill(4, 1, 2) // [1, 4, 3]
```

fill 方法接受三个参数 value、start 以及 end。start 和 end 参数是可选的。

fill 方法是一个通用方法，不要求 this 是数组对象

#### push() && pop()

我们将 push() 和 pop() 放在一起看，因为这两个方法的操作是相对的，可以将这个操作理解成压栈和出栈，符合先进后出原则，以方便理解。这里要注意的是这两个方法的返回值，pop() 返回出栈的元素，而 push() 返回新数组的长度。

```js
const array = ['🏓', '🏀']

array.push('⚽️', '🥊', '🏈') // ['🏓', '🏀', '⚽️', '🥊', '🏈']

const ball = array.pop()

console.log(array) // ['🏓', '🏀', '⚽️', '🥊']
console.log(ball) // '🏈'
```

#### shift() && unshift()

我们将 shift() 和 unshift() 放在一起看，因为这两个操作也是相对的，可以将这个操作理解成队列，符合先进先出原则，方便理解。这里要注意的是他们的返回值，shift() 返回被删除的元素，而 unshift() 返回数组的新长度。

```js
const array = ['🏓', '🏀']

console.log(array.shift()) // '🏀'

array.unshift('🎾') // ['🎾', '🏀']
```

#### reverse()

这个函数颠倒数组中元素的排列顺序，返回一个颠倒数组的引用

```js
const array = ['⚽️', '🏀', '🏈']

console.log(array) // ['⚽️', '🏀', '🏈']

array.reverse()
console.log(array) // ['🏈', '🏀', '⚽️']
```

#### sort()

sort()方法用原地算法对数组的元素进行排序，并返回数组。默认排序顺序是将元素转换为字符串，比较各个字符串的 Unicode 位点进行排序。

我们可以将自己的排序函数，作为 compareFunction 传入 sort 函数

```js
const numbers = [4, 3, 2, 1, 5, 6, 0]
numbers.sort((a, b) => a - b)

console.log(numbers) // [0, 1, 2, 3, 4, 5, 6]
```

#### splice()

splice() 方法通过删除或替换现有元素或者原地添加新的元素来修改数组，并以数组形式返回被修改的内容。此方法也会改变原数组。

```js
const array = ['⚽️', '🏀', '🏈', 🏐]

const removed = array.splice(2, 1, '🥎')

console.log(array) // ['⚽️', '🏀', '🥎', '🏐']
console.log(removed) // ['🏈']
```

至此，我们列举完了所有的修改器方法，还是要强调一遍，以上的修改器方法，调用之后都会修改数组自身的值。

### 访问方法 —— 绝对不会改变调用它们对象的值，只会返回一个新的数组或者返回一个其他的期望值

| 方法名 | 说明 | 返回值 |
|---|---|---|
| `concat()` | 将当前数组和其他数组结合 | 结合之后的新数组 |
| `slice()` | 抽取当前数组中的一段元素，组合成一个新数组 | 组合之后的新数组 |
| `includes()` | 判断当前数组是否包含某指定的值 | `true` or `false` |
| `indexOf()` | 返回数组中第一个与指定值相等的元素的索引 | 找到的元素 or -1 |
| `lastIndexOf()` | 返回数组中最后一个与指定值相等的元素的索引 | 找到的元素 or -1 |
| `join()` | 连接所有数组元素，组成一个字符串 | 连接后的字符串 |
| `toSource()` | 返回一个表示当前数组字面量的源代码 | 数组字面量字符串 |
| `toString()` | 返回一个表示当前数组字面量的字符串 | 数组字面量字符串 |
| `toLocaleString()` | 返回一个由所有数组元素组合而成的本地化后的字符串 | 本地化后的字符串 |

#### concat()

```js
const ball = ['⚽️', '🏀', '🏈']
const animal = ['🐳', '🐠', '🦄']

ball.concat(animal)
// result in ['⚽️', '🏀', '🏈', '🐳', '🐠', '🦄']
```

新生成数组的参数，依次是该参数的元素或参数本身。它不会递归到嵌套数组的参数中。


#### slice()

slice() 方法返回的，是由 begin 和 end 参数决定的原数组的浅拷贝。

如果 begin 被忽略，则从索引 0 开始拷贝。如果 end 参数被忽略，则提取到数组终止处的索引

```js
// 用 slice 来拷贝数组
const array = ['🐨', '🐯', '🦁', '🐮']
const newArray = array.slice(2)

console.log(newArray) // ['🦁', '🐮']
```

#### includes()

includes() 是 ES6 的中新标准，用来判断一个数组是否包含一个指定的值，如果包含则返回 true ，否则返回 false。

includes()接收一个参数，fromIndex，可以从 fromIndex 的索引处开始查找。

```js
['🐯', '🦁', '🐮'].includes('🦁') // true

['🐯', '🦁', '🐮'].includes('🐷') // false
```

#### indexOf() && lastIndexOf()

这两个方法，也推荐放在一起看，他们都是返回在数组中找到给定元素的索引，区别是一个从左边寻找，一个从右边寻找。如果找不到，则返回 -1 。

与 includes() 方法一样，都接收 fromIndex 参数，可以从指定索引处开始查找。

```js
const array = ['🐨', '🐯', '🦁']

array.indexOf('🦁') // 2
array.indexOf('🐷') // -1

array.lastIndexOf('🐨') // 0
array.lastIndexOf('🐷') // -1
```

#### toSource() && toString() && toLocaleString()

这三个函数都是返回数组字面量的字符串，但是各有区别

- toSource() 返回一个字符串，代表该数组的源代码，但是这个特性是非标准的，尽量不要在生产环境使用
- toString() 返回是是由数组中所有元素组成的字符串，以逗号分隔
- toLocaleString() 返回的是所有元素组成的特定语言环境的字符串

```js
// toSource
const array = new Array('a', 'b', 'c')
array.toSource() // ['a', 'b', 'c']

// toString()
array.toString() // 'a,b,c'

// toLocaleString()
const array1 = [1, 'a', new Date('21 Dec 1997 14:12:00 UTC')];
const localeString = array1.toLocaleString('en', { timeZone: 'UTC' });

console.log(localeString);
// expected output: "1,a,12/21/1997, 2:12:00 PM",
```

### 迭代方法

| 方法名 | 说明 | 返回值 |
|---|---|---|
| `forEach()` | 为数组中的每个元素执行一次回调元素 | `undefined` |
| `every()` | 数组中每个函数都满足测试函数，则返回 | `true` or `false` |
| `some()` | 数组中至少有一个元素满足测试函数，则返回 | `true` or `false` |
| `map()` | 对数组中所有元素执行一次回调函数 | 回调函数返回值组成的新数组 |
| `filter()` | 为每个元素执行一次测试函数，将返回值为 true 的元素返回 | 所有符合测试函数条件的元素组成的新数组 |
| `entries()` | 返回一个数组迭代器对象 | 返回的对象，包含数组元素的键值对 |
| `reduce()` | 从左到右的为每一个元素执行回调函数，并把每次执行的返回值放入暂存器中，传给下次的回调函数 | 返回最后一次回调函数的返回值 |
| `reduceRight()` | 从右到左的为每一个元素执行回调函数，并把每次执行的返回值放入暂存器中，传给下次的回调函数 | 返回最后一次回调函数的返回值 |
| `find()` | 找到第一个满足测试函数的元素 | 返回找到元素的值，找不到返回 `undefined` |
| `findindex()` | 找到第一个满足测试函数的元素 | 返回找到元素的索引，找不到返回 -1 |
| `keys()` | 返回一个包含所有数组元素的键的迭代器 | 迭代器 |
| `values()` | 返回一个包含所有数组元素的值的迭代器 | 迭代器 |

在这些众多遍历方法中，有很多方法都需要指定一个回调函数作为参数。在每一个数组元素都分别执行完回调函数之前，数组的 length 都会被缓存在某个地方，所以在回调函数中动态的为数组添加新属性，这些新属性是不会被遍历到的。此外如果在回调函数中对数组进行了其他修改，比如改变某个元素的值或删掉某个元素，那么随后的遍历操作可能会受到未预期的影响。

所以为了代码的可读性和可维护性，不要在迭代方法的回调函数中对原数组进行操作。

#### forEach()

 forEach() 方法对数组中的每个函数执行一次给定的函数。它遍历的范围在第一次调用 callback 前就会确定。之后添加到数组的项不会被 callback 访问到；已存在的值如果被改变，则传递给 callback 的值是遍历到这些值的那一刻的值；如果数组在迭代时被修改了，则其他的元素会被跳过。

 另外要注意的一点是，除了抛出异常，否则是没有办法中止或跳出 forEach() 循环的。这里要与 `for 循环` 和 `for..of` 、`for...in` 做个比较。

 ```js
//  用 forEach 来代替 for 循环
const animals = ['🐨', '🐯', '🦁']
const copy = []

// for 循环
for (let i = 0; i < animals.length; i++) {
    copy.push(animals[i])
}

// 使用 forEach
animals.forEach(animal => copy.push(animal))
```

#### every() && some()

如果是要判断数组中，元素是否满足给定条件，那么建议使用 every() 或者 some() 方法，而不要使用 filter() 方法来过滤，原因是当前者碰到一个会使 callback 返回 false 或者 true 的值时，会立即返回 false, 而 filter() 会持续的遍历整个数组。

```js
// every
[12, 4, 8, 130, 44].every(x => x > 10) // false
[12, 43, 34, 130, 44].every(x => x > 10) // true

// some
[12, 2, 32, 49].some(x => x < 1) // false
[12, 10, 3, 43].some(x => x >= 0) // false
```

#### map() && filter()

这两个函数放在一起说的原因是因为我经常看到有弄不明白这两个具体用法的开发者，将这两个函数作为循环来使用，因为这两个函数都会遍历数组中的所有元素，当你不打算使用新返回的数组而使用 map() 或 filter() 是违背设计初衷的。

```js
// map
const numbers = [1, 3, 5]
const plusNumbers = numbers.map(x => x + 3)

console.log(plusNumbers) // 4, 6, 9

// filter
const origin = [5, 10, 14, 16]
const filterArray = origin.filter(x => x > 10)

console.log(filterArray) // [14, 16]
```

#### entries() && keys() && values()

这三个方法放在一起说，是因为这三个方法都会返回一个数组迭代器对象。

- entries() 该迭代器会包括数组的键值对
- keys() 该迭代器会包含所有数组元素的键
- values() 该迭代器会包含所有数组元素的值

#### find() && findIndex()

 这两个方法是是查找元素的方法
 - find() 方法会返回找到第一个满足测试函数的元素，如果找不到，则返回 `undefined`
 - findIndex() 方法会返回找到第一个满足测试函数的索引，如果找不到，则返回 -1

 ```js
const array = [5, 12, 8, 130, 44]
const found = array.find(element => element > 10)
const foundIndex = array.findIndex(element => element > 10)

console.log(found) // 12
console.log(foundIndex) // 1
```

#### reduce() && reduceRight()

reduce() 方法对数组中的每个元素执行一个由你提供的 reduce 函数，将其结果汇总为单个返回值。

如果没有提供初始值，则将使用数组中的第一个元素作为初始值。在没有初始值的空数组上调用 reduce() 将报错。

reduceRight() 与 reduce() 的区别是累加的过程是从右向左执行。

```js
const sum = [0, 1, 2, 3, 4].reduce((acc, curr) => acc + curr)
const sum1 = [0, 1, 2, 3, 4].reduceRight((acc, curr) => acc + curr)

console.log(sum) // 10
console.log(sum1) // 10
```

## 总结

至此我们已经将数组中所有的属性和方法都总结了一遍，并且给出了最基本的用法示例。为了方便记忆，我们将数组的实例方法分为修改器方法、访问器方法、迭代方法，通过这几类方法的特性、返回值、以及对原数组的影响进行分类，方便记忆。相信在这样的一个合理的分类下，大家都会对数组的使用方法有更深刻的印象，也会在写代码的过程中，更合理的使用更具语义化和可读性的 API，提升代码的质量。

如果这篇文章能加深你对 JavaScript 中数组的理解，请点个赞支持一下哦。

最后放上我的 [github 博客地址](https://github.com/originalix/originalix.github.io)，欢迎 star。
