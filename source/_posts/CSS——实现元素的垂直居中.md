---
title: CSS——实现元素的垂直居中
date: 2018-05-14 20:41:02
tags: ["CSS", "前端"]

---

在写CSS的过程中，我常常谷歌一个东西，就是如何实现元素的垂直居中，水平居中难度还不是很大，但是垂直居中我这个烂记性是写一次忘一次，于是本着好记性不如烂笔头的想法，写下一篇博客记录下来。那么今天就记录下三种垂直居中的方法，各位看官按需使用。

<!-- more -->

## 通用情况

首先我们先介绍一种通用情况下的垂直居中，这个方法不需要设置自己的高度，也不需要父容器设置高度，利用绝对定位只需要三行代码就能实现。

首先html代码长这个样子

```html
<div id="outter1">
    <div class="inner1"></div>
    <div class="inner2">
        <div>这里的子元素自适应，不设置高度</div>
    </div>
</div>
```

那么来看css代码如何完成垂直居中：

```css
#outter1{
    position:relative;
    background:black;
}

#outter1 .inner1{
    display:inline-block;
    width:400px;
    height:400px;
    background:red;
}

#outter1 .inner2{
    background:blue;
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
}
```


## 父容器设置了高度，父容器下只有一个元素的情况

若是父容器设置了高度，父容器里只有一个元素，那么使用相对定位即可完成垂直居中。

html代码:


```html
<div id="outter2">
    <div class="inner2">
        <div>父亲设置了高度，且只有我一个子元素</div>
    </div>
</div>
```

css代码:

```css
#outter2{
    position:relative;
    background:black;
    height:400px;
}

#outter2 .inner1{
    display:inline-block;
    width:200px;
    height:200px;
    background:red;
}

#outter2 .inner2{
    display:inline-block;
    background:blue;
    position: relative;
    top: 50%;
    transform: translateY(-50%);
}
```

## Flex布局搞定垂直居中

如果不用考虑老式浏览器兼容的话，直接用flex布局来搞定就是非常简单的了，三行代码搞定垂直居中。

```html
<div id="outter3">
    <div class="inner1">
        <div>子元素1</div>
        <div>子元素2</div>
        <div>子元素3</div>
    </div>
</div>
```

```css
#outter3.inner1 {
  display:flex;
  display: -webkit-flex;
  align-items:center;
}
```

这就是三种CSS里垂直居中的方法了，希望写下这篇文章的我，在遇到垂直居中的问题时，再也不用谷歌了。