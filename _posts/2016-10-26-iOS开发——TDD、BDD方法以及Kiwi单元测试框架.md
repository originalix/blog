---
layout: post
title: iOS开发——TDD、BDD方法以及Kiwi单元测试框架
categories: iOS开发
date: 2016-10-26 14:39:21
keywords: iOS开发, 单元测试
---

# TDD和BDD

在GitBook上看过一篇文章，一个不写单元测试的程序员不是一个好的攻城狮。坦白的说，在**Objective-C**这个领域的里，我见过的会主动写单元测试的程序员还是比较少的。当然了，在那些大的开源项目里，我还是见到过很多单元测试的应用。

于是也就促使我想总结总结自己现在对单元测试的理解。众所周知苹果在`Xcode5`中引入了`XCTest`框架替换了原来的`SenTestingKit`。这也显示了苹果一直致力于在iOS开发中集成更方便可用的测试。但是我一直觉得`XCTest`的断言可读性较差，如果是让他人来阅读这段单元测试，会比较的花费精力。

再进入讨论单元测试之前，我们来谈谈不一样测试思想

<!--more-->

- 行为驱动开发（英语：Behavior-driven development，缩写BDD）是一种敏捷软件开发的技术，BDD的重点是通过与利益相关者的讨论取得对预期的软件行为的清醒认识。它通过用自然语言书写非程序员可读的测试用例扩展了测试驱动开发方法。

- 测试驱动开发（英语：Test-driven development，缩写为TDD）是一种软件开发过程中的应用方法，由极限编程中倡导，以其倡导先写测试程序，然后编码实现其功能得名。测试驱动开发是戴两顶帽子思考的开发方式：先戴上实现功能的帽子，在测试的辅助下，快速实现其功能；再戴上重构的帽子，在测试的保护下，通过去除冗余的代码，提高代码质量。测试驱动着整个开发过程：首先，驱动代码的设计和功能的实现；其后，驱动代码的再设计和重构。

上面讲述了TDD和BDD的思想差别，看到这里，你们认为当前的iOS开发适合怎样的测试思想。不知道你们开发中的实际情况是如何，在现在大环境赶进度的开发下，一般我是采用BDD的测试方法。

而谈到BDD，我要给大家介绍一个iOS中非常有名并且好用的BDD框架 —— **Kiwi**。

# Kiwi

## Kiwi的安装

- 项目主页: [https://github.com/kiwi-bdd/Kiwi](https://github.com/kiwi-bdd/Kiwi)

**使用Cocopods 安装**

```
target :YourProjectTests do
  pod 'Kiwi'
end
```

在这里记得一定要替换`YourProject`为你的项目名。

## Kiwi的基本结构

在讲**Kiwi**的常用语法前，我们先来看一段Kiwi的Github提供的示例代码。

```objc
describe(@"Team", ^{
    context(@"when newly created", ^{
        it(@"should have a name", ^{
            id team = [Team team];
            [[team.name should] equal:@"Black Hawks"];
        });

        it(@"should have 11 players", ^{
            id team = [Team team];
            [[[team should] have:11] players];
        });
    });
});
```

我们很容易根据上下文将其提取为Given..When..Then的三段式自然语言。

> Given a team, when newly created, it should have a name, and should have 11 players

是不是非常简单易懂的语法结构。

`describe`描述需要测试的对象内容，也即我们三段式中的`Given`，`context`描述测试上下文，也就是这个测试在`When`来进行，最后`it`中的是测试的本体，描述了这个测试应该满足的条件，三者共同构成了**Kiwi**测试中的行为描述。它们是可以**nest**的，也就是一个Spec文件中可以包含多个`describe`（虽然我们很少这么做，一个测试文件应该专注于测试一个类）；一个`describe`可以包含多个`context`，来描述类在不同情景下的行为；一个`context`可以包含多个`it`的测试例。

Kiwi还有一些其他的行为描述关键字，其中比较重要的包括:

- `beforeAll(aBlock)` - 当前scope内部的所有的其他block运行之前调用一次

- `afterAll(aBlock)` - 当前scope内部的所有的其他block运行之后调用一次

- `beforeEach(aBlock)` - 在scope内的每个it之前调用一次，对于context的配置代码应该写在这里

- `afterEach(aBlock)` - 在scope内的每个it之后调用一次，用于清理测试后的代码

- `specify(aBlock)` - 可以在里面直接书写不需要描述的测试

- `pending(aString, aBlock)` - 只打印一条log信息，不做测试。这个语句会给出一条警告，可以作为一开始集中书写行为描述时还未实现的测试的提示。

- `xit(aString, aBlock)` - 和pending一样，另一种写法。因为在真正实现时测试时只需要将x删掉就是it，但是pending语意更明确，因此还是推荐pending

## Kiwi使用实例

就拿项目中一个真实的场景来说，我在写完一个适配所有iPhone机型的宽高的类之后，我用Kiwi来进行单元测试。

首先我这个类是这么描述宽高的

```objc
//CalculateLayout.h

+ (CGFloat)neu_layoutForAlliPhoneHeight:(CGFloat)height;

+ (CGFloat)neu_layoutForAlliPhoneWidth:(CGFloat)width;

//  CalculateLayout.m

+ (CGFloat)layoutForAlliPhoneHeight:(CGFloat)height type:(IPhoneType)type {
    CGFloat layoutHeight = 0.0f;
    switch (type) {
        case iPhone4Type:
            layoutHeight = ( height / iPhone6Height ) * iPhone4Height;
            break;
        case iPhone5Type:
            layoutHeight = ( height / iPhone6Height ) * iPhone5Height;
            break;
        case iPhone6Type:
            layoutHeight = ( height / iPhone6Height ) * iPhone6Height;
            break;
        case iPhone6PlusType:
            layoutHeight = ( height / iPhone6Height ) * iPhone6PlusHeight;
            break;
        default:
            break;
    }
    return layoutHeight;
}

+ (CGFloat)layoutForAlliPhoneWidth:(CGFloat)width type:(IPhoneType)type {
    CGFloat layoutWidth = 0.0f;
    switch (type) {
        case iPhone4Type:
            layoutWidth = ( width / iPhone6Width ) * iPhone4Width;
            break;
        case iPhone5Type:
            layoutWidth = ( width / iPhone6Width ) * iPhone5Width;
            break;
        case iPhone6Type:
            layoutWidth = ( width / iPhone6Width ) * iPhone6Width;
            break;
        case iPhone6PlusType:
            layoutWidth = ( width / iPhone6Width ) * iPhone6PlusWidth;
            break;
        default:
            break;
    }
    return layoutWidth;
}

```

反正大概意思就是我输入了一个宽高，他根据UI给定的设计图，返回给我一个宽高适配当前机型的宽高。

那么我们如何来写这个测试用例呢.

```objc
#import <Kiwi/Kiwi.h>
#import "CalculateLayout.h"


SPEC_BEGIN(CalculateLayoutTests)

describe(@"CalculateLayout", ^{
    context(@"when calculate width and height", ^{
        
        CGFloat width = [CalculateLayout neu_layoutForAlliPhoneWidth:375.f];
        CGFloat height = [CalculateLayout neu_layoutForAlliPhoneHeight:667.f];
        
        pending_(@"All iPhone Test", ^{
        });
        
        it(@"should layout width", ^{
            [[theValue(width) should] equal:theValue(320.f)];
        });
        
        it(@"should layout height", ^{
            [[theValue(height) should] equal:theValue(568.f)];
        });
    });
});

SPEC_END

```

我写进去的宽高数值是iPhone6的宽高数值，如果用5S的模拟器来运行，将会返回5S的宽高 320 * 568

当我们 com+U 运行这段测试用例时。

控制台的输出

```
+ 'CalculateLayout, when calculate width and height, should layout width' [PASSED]

+ 'CalculateLayout, when calculate width and height, should layout height' [PASSED]

```

可以看到，由于有`context`的存在，以及其可以嵌套的特性，测试的流程控制相比传统测试可以更加精确。我们更容易把`before`和`after`的作用区域限制在合适的地方。

实际的测试写在it里，是由一个一个的期望(Expectations)来进行描述的，期望相当于传统测试中的断言，要是运行的结果不能匹配期望，则测试失败。在`Kiwi`中期望都由`should`或者`shouldNot`开头，并紧接一个或多个判断的的链式调用，大部分常见的是`be`或者`haveSomeCondition`的形式。在我们上面的例子中我们使用了`should not be nil`和`should equal`两个期望来确保字符串赋值的行为正确。其他的期望语句非常丰富，并且都符合自然语言描述，所以并不需要太多介绍。在使用的时候不妨直接按照自己的想法来描述自己的期望，一般情况下在`IDE`的帮助下我们都能找到想要的结果。如果您想看看完整的期望语句的列表，可以参看文档的这个页面。从这一点来看，Kiwi可以说是一个非常灵活并具有可扩展性的测试框架。

来解释下上面的语法中用到的`theValue`.

`Kiwi`为我们提供了一个标量转对象的语法糖，叫做`theValue`，在做精确比较的时候我们可以直接使用例子中直接与`320.f或者568.f`做比较这样的写法来进行对比。

通过这样一个简单的例子，我们基本能掌握Kiwi的语法，以及Kiwi的使用。单元测试的门其实很好进，但是如何用心的，动脑子的去写单元测试，则是对我们程序员莫大的考验哦。

我讲的并不完善，也不详细，就算简单记录自己目前的收获吧。