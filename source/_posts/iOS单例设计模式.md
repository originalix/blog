---
title: iOS设计模式系列：单例设计模式

date: 2016-03-15 15:00:33

tags: ["单例设计模式"]
---


 单例设计模式从字面意思上来说，就是一个类在系统运行时，只创建一个实例。可以用于需要被多次调用的或者多次使用的资源中。比如我们常见的网络请求类、工具类等等。

```objc
//iOS中大量的使用了单例方法，常见的有：
[NSUserDefaults standardUserDefaults]  轻量级的本地数据存储，存储自定义的对象，比如保存登陆界面的数据、用户名、密码等。

[UIApplication sharedApplication] 处理用户事件，main函数入口

[UIScreen mainScreen] 获取全屏幕尺寸
```

<!--more-->


### 单例模式的作用

它可以保证某个类在运行过程中，只有一个实例，也就是对象实例只占用一份系统内存资源。

### 单例的要点

- 该类有且只有一个实例

- 该类必须能自行创建这个实例

- 该类必须能够向整个系统提供这个实例

### 单例的优缺点

优点：
- 提供了唯一实例的受访对象

- 因为在系统中只存在一个实例，在频繁访问和调用时，节省了系统创建和销毁资源的开销，提高系统性能。

- 因为单例化的类，控制了实例化的过程，所以能更灵活修改实例化的过程。

缺点：

- 单例模式没有抽象层，不容易扩展。

- 单例模式往往职责过重，一定程度上违背了“单一职责原则”。

### 单例类的实现

- 为单例对象创建一个静态实例，可以写成全局的，也可以在类方法中实现，并置为nil。

- 用GCD多线程的方式来实现单例，用dispatch_once_t来保证线程的安全性和单一性。

- 检查生成的静态实例是否为nil，若是则创建并返回一个本类的实例。

- 重写allocWithZone方法，用来保证其他人想通过alloc、init方法创建实例的时候，不会产生新实例。

- 适当实现copyWithZone。

代码如下：

```objc
#import "Singleton.h"

@implementation Singleton


//为单例对象创建的静态实例，置为nil，因为对象的唯一性，必须是static类型
static Singleton * _singleton = nil; 


/**
 *  重写allocWithZone方法，保证alloc或者init创建的实例不会产生新实例，因为该类覆盖了allocWithZone方法，所以只能通过其父类分配内存，即[super allocWithZone]
 *
 */
+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    static dispatch_once_t onceToken;
    
    dispatch_once(&onceToken, ^{
        _singleton = [super allocWithZone:zone];
    });
    
    return _singleton;
}

/**
 *  单例对象对外的唯一接口，用到dispatch_once在初始化时执行一次任务，且dispatch_once保证线程安全
 *
 */
+ (instancetype)shareSingleton
{
    static dispatch_once_t onceToken ;
    
    dispatch_once(&onceToken, ^{
        _singleton = [[self alloc] init];
    });
    
    return _singleton;
}

- (id)copyWithZone:(NSZone *)zone
{
    return _singleton;
}
@end
```


 到这里，一个简单的单例模式就基本完成了。
 如果有需要，还可以把整个单例模式封装成一个宏。