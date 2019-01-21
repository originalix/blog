---
title: 使用Block提高代码可读性
date: 2017-01-06 17:11:37
tags: ["iOS开发", "代码可读性", "Block"] 

---

最近一直在思考并持续的扩充着自己的技术栈，比如每天坚持着学习前端知识，并且时常想着在移动端这条路上，自己的技术盲区。诚然，想要在一个领域达到一定的技术高度是挺困难的一件事情，操之过急万万不可，最主要的还是保持对技术的热情，慢慢沉淀。

以前的公司并不需要高强度的加班，所以时长有时间去发掘一些新鲜玩意儿，圈内有了技术热潮，也能及时跟进观望或者学习，但是最近在一波高强度加班过后，这种业务代码与自身学习之间的冲突也让我产生了一些自己的看法。一个程序员，不论何时，不能抛掉持续学习的习惯。

最近的大面积写业务代码，当碰到两个类之间的传值问题，我习惯性的解决方案是使用`delegate`，我觉得`delegate`本身当命名得当并且功能单一时，可阅读性会比较好。在习惯了这种思维后，开发中也养成了自己的习惯。

但是在最近封装代码的时候，我发觉`delegate`对于我自己定义并编写代码来说，可读性和使用性很好，但是当他人来使用我封装的代码的时候，也许`Block`更容易被理解一点。举个简单的例子，就比如

<!--more-->

```objc
[UIView animateWithDuration:1 animations:^{
       //do something
           
}];
```

这是我们日常最常用的`Block`结构之一，需要执行的事情，只要在`Block`中交代清楚就可以了，在阅读他人的代码时，可以直接在`Block`中直接阅读到执行的事件，并不用再去关注各种`delegate`中执行了什么。大大提高了代码的可读性。

我认为，程序员首先是写人能看得懂的代码，顺便运行。

在这个理念的驱使下，我大概会在之后的开发过程中，对可读性这个概念更上心一点，能用`block`处理的事件，尽量的用`block`处理。很久以前我写过一篇博客，讲述的是`blcok`的传值，**iOS4.0**开始，苹果爸爸引入了`block`的特性，而自从`block`特性诞生之日起，似乎它就受到了苹果爸爸特殊的照顾和青睐。字面上说，`block`就是一个代码块，但是它的神奇之处在于在内联(inline)执行的时候(这和C++很像)还可以传递参数。同时block本身也可以被作为参数在方法和函数间传递，这就给予了`block`无限的可能。

在日常的`coding`里绝大时间里开发者会是各种`block`的使用者，但是当你需要构建一些比较基础的，提供给别人用的类的时候，使用`block`会给别人的使用带来很多便利。当然如果你已经厌烦了一直使用`delegate`模式来编程的话，偶尔转转写一些`block`，不仅可以锻炼思维，也能让你写的代码看起来高端洋气一些，而且因为代码跳转变少，所以可读性也会增加。

今天我用一个判断奇数偶数的例子，来说说如何在封装的方法中，根据条件来执行方法内携带的`block`。

首先看看我们这个含有`blcok`的类是如何声明的。

```objc
typedef void (^LixExcuteOperation)(NSInteger);
typedef void(^LixError)(void);

@interface LixBlock : NSObject

- (void)isOddNumber:(NSInteger)number Excute:(LixExcuteOperation)excute Lixerror:(LixError)error;

@end
```
结合下面的图片，来看看`block`是如何声明并且定义的。

![block的声明与定义](http://upload-images.jianshu.io/upload_images/783864-3ad5d92333756aa7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再来看看这个方法的内部，是如何使用`block`的，因为是示例，所以我很粗略的去判断了奇偶数，并没有考虑特殊情况，见谅，只是个栗子。

```objc
- (void)isOddNumber:(NSInteger)number Excute:(LixExcuteOperation)excute Lixerror:(LixError)error {
    BOOL isOddNum = number % 2 == 0 ? NO : YES;
    if (isOddNum && excute) {
            excute(number);
                
    }
        
    if (!isOddNum && error) {
            error();
                
    }

}
```

至于调用，就更加随意了。

```objc
    LixBlock *block = [[LixBlock alloc] init];
    [block isOddNumber:9 Excute:^(NSInteger number) {
            NSLog(@"is OddNumber %ld", number);
                
    } Lixerror:^{
            NSLog(@"is not OddNumber");
                
    }];
```

至此，一个简单的封装`block`进方法的栗子就已经讲完了。举一反三的讲，我们在对网络请求进行二次封装，执行`success`或者`error`状态的闭包时，就可以用到类似的思想了。代码的可读性是否如愿提升了呢。

简单的栗子讲到这里，`Coding`还是需要多写多思考的。

