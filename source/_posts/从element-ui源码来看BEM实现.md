---
title: 从element-ui源码来看BEM实现
date: 2018-07-23 19:50:58
tags: ["前端开发", "css", "源码"]

---

以前看过一些CSS的编码规范，也照着规范编写代码，但是还是觉得自己的CSS编码风格不是很好，在平时开发中使用各个知名的组件库的时候，发现现在挺多组件库都是BEM的命名风格了，于是找了比较知名的element饿了么前端的组件库的源码，想从看饿了么组件库的代码入手，学习学习大厂的的CSS BEM规范代码的书写风格。

<!--more-->

BEM代表“块（block），元素（element），修饰符（modifier）”，开发组件的过程中经常使用这三个实体，而在选择器中，这三个实体由以下符号来表示扩展关系：

```
-  中划线：仅作为连字符使用，表示某个块或者某个子元素的多单词之间的连接记号。
__ 双下划线：双下划线用来连接块和块的子元素。
_  单下划线：单下划线用来描述一个块或者块的子元素的一种状态。

type-block__element_modifier
```

以上的描述是从腾讯的前端规范库中找到的，简单的来说理解了块、元素、修饰符三个分类之后，就能大致理解BEM代码是什么样的结构了，可是了解结构是一方面，如何写好代码又是另一方面。我在element组件库中的`mixins.scss`文件中找到了想要的答案。

接下来我要讲的就是如何利用sass，编写具有可读性和可维护性的BEM规则的css代码。

首先来看sass代码中对于block的定义:

```css
$namespace: 'el';
$element-separator: '__';
$modifier-separator: '--';
$state-prefix: 'is-';


@mixin b($block) {
  $B: $namespace+'-'+$block !global;

  .#{$B} {
    @content;
  }
}
```

为了方便大家理解代码，我现在开头贴上配置文件中定义的变量，而这时就能很清楚的看到block的生成就是基于BEM规范中，块是设计或布局的一部分，具有唯一地意义，利用命名空间`el`加上中划线，以及传入的block的名字，构建出block的样式，例如`alert`组件，在渲染完成后是`el-alert`，体现出它的唯一性。而在块的内部，再来编写跟这个块关联的其他样式代码。

块的构建非常的简单，接下来来看稍微有点复杂的元素的定义:

```css
@mixin e($element) {
  $E: $element !global;
  $selector: &;
  $currentSelector: "";
  @each $unit in $element {
    $currentSelector: #{$currentSelector + "." + $B + $element-separator + $unit + ","};
  }

  @if hitAllSpecialNestRule($selector) {
    @at-root {
      #{$selector} {
        #{$currentSelector} {
          @content;
        }
      }
    }
  } @else {
    @at-root {
      #{$currentSelector} {
        @content;
      }
    }
  }
}
```

元素选择器的实现中，我们应该把关注点放在if和eles分支上，为什么会出现`hitAllSpecialNestRule`函数判断的分支，原因是在修饰符或者其他mixin中嵌套一个元素element，会出现修饰符在前，而元素在后的编译结果，所以我们用`hitAllSpecialNestRule`函数来判断是否存在特殊的嵌套，如果存在的话，将我们的元素字符串写在前面，而修饰符放在后面，如果不存在，则原样输出。

而`hitAllSpecialNestRule`的实现则在下面的代码中：

```css
@function selectorToString($selector) {
  $selector: inspect($selector);
  $selector: str-slice($selector, 2, -2);
  @return $selector;
}

@function containsModifier($selector) {
  $selector: selectorToString($selector);

  @if str-index($selector, $modifier-separator) {
    @return true;
  } @else {
    @return false;
  }
}

@function containWhenFlag($selector) {
  $selector: selectorToString($selector);

  @if str-index($selector, '.' + $state-prefix) {
    @return true
  } @else {
    @return false
  }
}

@function containPseudoClass($selector) {
  $selector: selectorToString($selector);

  @if str-index($selector, ':') {
    @return true
  } @else {
    @return false
  }
}

@function hitAllSpecialNestRule($selector) {

  @return containsModifier($selector) or containWhenFlag($selector) or containPseudoClass($selector);
}

```

第一个函数`selectorToString`，就是将我们的选择器转换成一个字符串，而接下来的三个函数，分别判断了是否存在修饰符、flag例如（.isCenter）、伪类的情况。最后综合在一起返回结果，避免嵌套。

最后则是`modifier`修饰符的实现了，代码如下:

```css
@mixin m($modifier) {
  $selector: &;
  $currentSelector: "";
  @each $unit in $modifier {
    $currentSelector: #{$currentSelector + & + $modifier-separator + $unit + ","};
  }

  @at-root {
    #{$currentSelector} {
      @content;
    }
  }
}
```
在看懂的元素的实现之后，修饰符的实现就是一目了然的了。利用刚刚介绍的函数，以及块、元素、修饰符的实现代码，在sass中已经能非常高效率并且优雅的基于BEM规范的代码了。

贴一段示例代码，如何利用上面的代码编写BEM规范的css代码：

```css
@import "mixins/mixins";
@import "common/var";

@include b(test) {
    width: 100%;
    background-color: $--color-white;

    @include when(center) {
        justify-content: center;
    }

    @include m(success) {
        background-color: $--color-black;
        font-size: 12px;

        .lix-test__description {
            color: $--color-white;
        }
    }

    @include m(warning) {
        background-color: $--color-black;
        font-size: 12px;

        .lix-test__description {
            color: $--color-white;
        }
    }

    @include e(content) {
        display: table-cell;
        padding: 0 8px;
    }

    @include e(title) {
        font-size: 8px;
        line-height: 10px;
        
        @include when(bold) {
            font-weight: bold;
        }
    }
}

.lix-test-fade-enter,
.lix-test-fade-leave-active {
    opacity: 0;
}

```
编译之后的样子：

```css
.lix-test {
  width: 100%;
  background-color: #fff; }
  .lix-test.is-center {
    justify-content: center; }
  .lix-test--success {
    background-color: #000;
    font-size: 12px; }
    .lix-test--success .lix-test__description {
      color: #fff; }
  .lix-test--warning {
    background-color: #000;
    font-size: 12px; }
    .lix-test--warning .lix-test__description {
      color: #fff; }
  .lix-test__content {
    display: table-cell;
    padding: 0 8px; }
  .lix-test__title {
    font-size: 8px;
    line-height: 10px; }
    .lix-test__title.is-bold {
      font-weight: bold; }

.lix-test-fade-enter,
.lix-test-fade-leave-active {
  opacity: 0; }
```

瞧见了么，合理的利用b、m、e的mixin就可以风格良好的css代码了。上面的所有代码都在[这里](https://github.com/originalix/Original/tree/master/sass/BEM)哦，有需要的可以直接看完整代码学习。