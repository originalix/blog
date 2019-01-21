---
title: Nodejs中编写异步的单元测试代码
date: 2018-12-18 20:54:36
tags: ["Node.js", "单元测试", "异步"]

---

在Nodejs的开发过程中，异步这个话题是无论如何都躲不过去的，关于异步的文章已经有过许多篇了，我也不打算写在开发Web应用的过程中，该如何在Nodejs中处理异步代码。在前些日子，我跟单元测试覆盖率这个指标杠上了，因为自己在写一个Nodejs的工程，我希望这个工程的测试代码量不要太少，目标是100%的行覆盖率，所以最近写了许多的单元测试代码。使用的测试框架是Mocha，断言库是Chai，那么今天我们就来聊聊在单元测试中，处理异步代码的各种姿势。

<!--more-->

## 处理promise


```js
const { query } = require('../app/utils/async-db');
const { should } = require('chai');
const mysql = require('mysql');
should();

/**
 * 测试数据库连接的正确状态
 */
describe('mysql connect success state', function() {
  it('should return an array', function(done) {
    let sql = 'SELECT * FROM `Users`';
    query(sql)
      .then((rows) => {
        rows.should.be.an('array');
        done();
      })
      .catch(err => {
        done(err);
        throw err;
      });
  });
});


```

先来看看今天的例子，这段代码就是测试数据库连接状态的库，在断言库中我偏向于使用should类型的，因为更加的语义化，更符合TDD的阅读习惯。而这段代码看似没有问题，但是运行起来会报错：

```bash
Error: Timeout of 2000ms exceeded. For async tests and hooks, ensure "done()" is called; if returning a Promise, ensure it resolves.
```

为什么呢，原因是在第二行、第四行。

```js
const { should } = require('chai');
...
should();

```

在这样引用了should之后，是无法像刚才代码中那样使用should的，为什么我会写出这样的语法呢？我承认我当时偷懒随便看了篇博客就照猫画虎了，以后一定要跟着官方文档来！！！所以我们这里先纠正错误，正确的代码如下：

```js
const { query } = require('../app/utils/async-db');
const should = require('chai').should();
const mysql = require('mysql');

/**
 * 测试数据库连接的正确状态
 */
describe('mysql connect success state', function() {
  it('should return an array did not have done', function(done) {
    let sql = 'SELECT * FROM `Users`';
    query(sql)
      .then((rows) => {
        rows.should.be.an('array');
        done();
      })
      .catch(err => {
        done(err);
        // throw err;
      });
  });
});
```

这样，在promise中，在then里直接写断言，之后再跟上done，表示测试完成，就可以成功的完成异步测试，这种方式是done回调的方式。

而还有直接返回promise的方式，写法如下：

```js
/**
 * 测试数据库连接的正确状态
 */
describe('mysql connect success state', function() {
  it('should return an array did not have done', function() {
    let sql = 'SELECT * FROM `Users`';
    return query(sql)
      .then((rows) => {
        rows.should.be.an('array');
      });
  });
});

```

直接说一下写法的区别吧，在第二行代码的it块内，回调的function中不要再加入done回调的，不然测试程序会一直等待你的done回调，当超时之后就会报错了。而去除done回调之后，直接写返回结果就好了，如果catch到了error，那么直接会被抛出，测试失败。

这两种方法写完，应该还有很多同学觉得这样写非常啰嗦吧，那么我们来看一个chai断言库的中间件，这个中间件可以大大简化promise相关的断言，这个库就是`chai-as-promised`。这个库中提供了一个最重要的Api就是`should.eventually`，直接按字面意思去理解这个链式api吧，意味着它会等待promise的最终执行结果，来测试断言。还是刚才的例子，用这个小插件改写完之后是这样的。

```js
const { query } = require('../app/utils/async-db');
const chai = require('chai');
const chaiAsPromised = require('chai-as-promised');
chai.use(chaiAsPromised);
chai.should();

/**
 * chai-as-promised库的简单使用
 */
describe('Mysql connect', function() {
  // 记得使用chai-as-promised的时候 这里的function不要加 done 参数
  it('should return an array', function() {
    let sql = 'SELECT * FROM `Users`';
    return query(sql).should.eventually.be.an('array');
  });
});

```
瞬间测试的代码块内只剩下两行代码了，是不是看着分外的爽？稍微学习一下这样的用法，相信异步的单元测试，从此以后对同学们来说就是小菜一碟咯。