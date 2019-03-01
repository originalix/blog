---
layout: post
title: iOS开发 —— Runtime
categories: iOS开发
date: 2016-07-20 15:04:01
keywords: iOS开发, Runtime
---

# Runtime

因为Objc是一门动态语言，所以它总是想办法把一些决定工作从编译连接推迟到运行时。也就是说只有编译器是不够的，还需要一个运行时系统 (runtime system) 来执行编译后的代码。这就是 Objective-C Runtime 系统存在的意义，它是整个Objc运行框架的一块基石。

Runtime基本是用C和汇编写的，可见苹果为了动态系统的高效而作出的努力。你可以在这里下到苹果维护的开源代码。苹果和GNU各自维护一个开源的runtime版本，这两个版本之间都在努力的保持一致。

<!--more-->

# 消息传递

在很多语言中，比如C，调用一个方法其实就是跳到内存中的某一点并开始执行一段代码。没有任何动态的特性，因为这在编译时就决定好了。而在Objective-C中，类似 `[Receiver message]` 这种语法并不会立即执行 `message` 这个方法的代码。 它是在运行时给 `Receiver` 发送一条叫 `message` 的消息。 这个消息，也许会被 `Receiver` 来处理，也许会被转发给另一个对象，或者没有对象处理最终被报错导致程序崩溃。

事实上，在编译时，我们所编写的 `Objective-C` 函数都会被翻译成一个C语言的函数调用。 ` - objc_msgSend(id self, SEL op, ...)`。

比如我们创建一个类 名字叫 `Family`  创建一个数组来存储我们生成的 `Family` 对象。

```objc
 Family *family = [[Family alloc] init];

 NSMutableArray *array = [NSMutableArray array];
    
```

接下去的两行代码是等价的

```obj-c
[array insertObject:family atIndex:0];
    
objc_msgSend(array, @selector(insertObject:atIndex:), family, 0);

```

在调用 `objc_msgSend` 这个方法时，我们先要包含 `#import <objc/message.h>` 这个头文件。 而且在编写 `objc_msgSend` 这个函数时，先去 `TARGETS` - `Build settings` - `Apple LLVM 7.0` - `Preprocessing` - `Enable Strict Checking of objc_msgSend Calls` 设置为NO。

从这里我们就能得到一个结论 `objc_msgSend` 能确定消息名以及参数数目。

消息传递的关键藏于 `objc_object` 中的 `isa` 指针和 `objc_class` 中的 `class dispatch table` 。


# 消息传递的过程

在 `Objective-C 中`，类、对象和方法都是一个 C 的结构体，从 `objc/objc.h` 头文件中，我们可以找到他们的定义。

具体的定义分析可以在我的另一篇 解释 `Runtime` 中各种方法、属性的定义中去学习。

从这些定义中可以看出发送一条消息也就 `objc_msgSend` 做了什么事。举 `objc_msgSend(array, @selector(insertObject:atIndex:), family, 0);` 这个例子来说：

1. 首先，通过array的isa指针找到它的class；（从之前的代码 得知是 `NSMutableArray` 类）

2. 在 `class` 的 `method list` 中找 `insertObject:atIndex:` ；

3. 如果 `class` 中 没有找到 `insertObject:atIndex:` 这个方法, 继续前往它的 `SuperClass` 中去找。

4. 一旦找到 `insertObject:atIndex:` 这个函数，就去执行它的实现IMP。 

但这种实现有个问题，效率低。但一个 `class` 往往只有 20% 的函数会被经常调用，可能占总调用次数的 80% 。每个消息都需要遍历一次 `objc_method_list` 并不合理。如果把经常被调用的函数缓存下来，那可以大大提高函数查询的效率。这也就是 `objc_class` 中另一个重要成员 `objc_cache` 做的事情 - 再找到 `insertObject:atIndex:` 之后，把 `insertObject:atIndex:` 的 `method_name` 作为  **key** ，`method_imp` 作为 **value** 给存起来。当再次收到 `insertObject:atIndex:` 消息的时候，可以直接在 `cache` 里找到，避免去遍历 `objc_method_list`。

# 方法的动态解析和转发

如果在 `Family` 类中，我运行了类中没有存在的 `- familyName:` 这个方法时。 会出现怎样的结果， 相信接下来的语句 每一个做过开发人员都应该见过  **unrecognized selector sent to instance 0x7ffee1e90f00** 。

但是在这个异常抛出之前，我们的 `Objective-C` 尝试过用三种方法来拯救我们的程序。

1. Method resolution
2. Fast forwarding
3. Normal forwarding

# Method Resolution

首先，**Runtime** 会调用 `+ (BOOL)resolveInstanceMethod: ` 这个方法，让你有机会提供一个函数实现。如果你添加了函数并返回YES，那么 **Runtime** 系统就会重新启动一次消息发送的过程。

以刚才我想实现的 `- familyName:` 这个方法为例。

我在 `Family` 类中这样重写了 `+ (BOOL)resolveInstanceMethod: `  方法:

```obj-c

/**
 *  里边有三个类型别名,在这儿先解释一下
 
      SEL selector的简写,俗称方法选择器,实质存储的是方法的名称
      IMP implement的简写,俗称方法实现,看源码得知它就是一个函数指针
      Method 对上述两者的一个包装结构.
 *
 */
 
void lostInstanceMethod(){
    NSLog(@"this InstanceMethod was lost");
}


+ (BOOL) resolveInstanceMethod:(SEL)sel{
    if (sel == @selector(familyName:)) {
        class_addMethod([self class], sel, (IMP)lostInstanceMethod, nil);
        return YES;
    }
    return [super resolveClassMethod:sel];
}

```

在当我执行了  `objc_msgSend(family, @selector(familyName:), nil);`  这句语句时，程序不出意外的打印出了 `lostInstanceMethod()`
函数中的那句话。

```obj-c
 /**
     *  这里当我们运行了类中没有存在的 familyName 这个方法时.
     *  Objective_C 运行时 会调用 + (BOOL)resolveInstanceMethod:(SEL)sel
     */
    
    /**
     *  但是当我注销掉 + (BOOL)resolveInstanceMethod:(SEL)sel 这个方法后
     *  编译通过之后程序崩溃 并不会进入 + (BOOL)resolveClassMethod:(SEL)sel 这个方法
     */
    
  objc_msgSend(family, @selector(familyName:), nil);

```


# Fast forwarding

如果resolve方法返回了NO， 运行时就会移到下一步， 消息转发 （Message Forwarding）。

如果目标对象实现了 `- (id)forwardingTargetForSelector:(SEL)aSelector` ， **Runtime** 这时就会调用这个方法，给你把这个消息转发给其他对象的机会。

如果这是我运行这样一行代码:

```obj-c
 objc_msgSend(family, @selector(familyHouse:), nil);
```

我的 `Family` 类中并没有实现 `- familyHouse:` 这个方法，所以如果运行下去肯定会抛出异常代码 。

但是我新建了一个 `FamilyHouse` 类， 我在 `FamilyHouse` 这个类中实现了 `-familyHouse: ` 。

```obj-c
#import "FamilyHouse.h"

@implementation FamilyHouse

- (void)familyHouse:(NSString *)house{
      NSLog(@"this is my house");
}
@end


```

并且 我回到 `Family` 类中，重写 `- (id)forwardingTargetForSelector:(SEL)aSelector` 方法 。 

首先定义一个 `House` 的对象，并将它初始化，之后重写方法。

```objc
- (id)forwardingTargetForSelector:(SEL)aSelector{
    if (aSelector == @selector(familyHouse:)) {
        return _house;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

这时当 **Runtime**在当前类找不到 `- familyHose:` 这个方法时，就会调用 `- forwardingTargetForSelector:` 这个方法，将消息分发到属于 `FamilyHouse` 类的对象 `house` 去执行这个方法。 

只要这个方法返回的不是 `nil` 和 `self`，整个消息发送的过程就会被重启，当然发送的对象会变成你返回的那个对象。否则，就会继续 **Normal Fowarding** 。

这里叫 Fast ，只是为了区别下一步的转发机制。因为这一步不会创建任何新的对象，但下一步转发会创建一个 `NSInvocation` 对象，所以相对更快点。


# Normal forwarding

这一步是 **Runtime**最后一次给你机会挽救程序。 首先它会发送 `- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector` 这个消息来获取你转发消息的对象的函数参数和方法名。 

如果 `- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector` 这个方法返回  `nil` ，**Runtime** 则会抛出 `- (void)doesNotRecognizeSelector:(SEL)aSelector` 这个消息，当然，你的程序也就华丽丽的挂掉了。 

如果是 `- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector ` 返回了一个函数签名，  **Runtime** 就会创建一个 `NSInvocation` 对象，并执行 `- (void)forwardInvocation:(NSInvocation *)anInvocation` 这个方法，转发消息给目标对象。

接着上面的 **Fast forwarding** 部分叙述，假如我在第二步，并没有重写 `- (id)forwardingTargetForSelector:(SEL)aSelector` 这个方法我该怎么在最后一步完成拯救。

首先是完成函数签名这个方法,注意这里的 `_house` 对象，是我已经初始化好的 `FamilyHouse` 类的对象。

```objc
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector{
    NSMethodSignature* signature = [super methodSignatureForSelector:aSelector];
    
    if (!signature){
        signature = [_house methodSignatureForSelector:aSelector];
    }
        return signature;
}
```

这步完成后，就会返回给程序一个函数签名，**Runtime** 就会创建一个 **NSInvocation** 对象并发送 `-forwardInvocation:` 消息给目标对象。

`NSInvocation` 实际上就是对一个消息的描述 ，包括方法名以及参数等信息。所以我们可以在 `- (void)forwardInvocation:(NSInvocation *)anInvocation` 里修改传进来的 `NSINvocation` 对象， 然后发送  `-invokeWithTarget:`  的消息给他，传进去一个新的对象

```objc
- (void)forwardInvocation:(NSInvocation *)anInvocation{
    SEL sel = anInvocation.selector;
    
    if ([_house respondsToSelector:sel]) {
        [anInvocation invokeWithTarget:_house];
    }else {
        [self doesNotRecognizeSelector:sel];
    }
}

```

这个代码的运行结果 就跟之前一样了。

# 总结

`Objective-C` 中给一个对象发送消息会经过以下几个步骤：

1. 在对象类的 `dispatch table` 中尝试找到该消息。如果找到了，跳到相应的函数IMP去执行实现代码；

2. 如果没有找到，**Runtime** 会执行 `+ (BOOL)resolveInstanceMethod:(SEL)sel` 方法，尝试着去 **resolve**这个方法。

3. 如果 **resolve** 方法返回 **NO**，**Runtime** 就发送 `-forwardingTargetForSelector:` 允许你把这个消息转发给另一个对象；

4. 如果没有新的目标对象返回， **Runtime** 就会发送 `-methodSignatureForSelector:` 和 `-forwardInvocation:` 消息。你可以发送 `-invokeWithTarget:` 消息来手动转发消息或者发送 `-doesNotRecognizeSelector:` 抛出异常。

利用 **Objective-C** 的 **Runtime** 特性，我们可以自己来对语言进行扩展，解决项目开发中的一些设计和技术问题。