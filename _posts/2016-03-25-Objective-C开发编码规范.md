---
layout: post
title: Objective-C开发编码规范
categories: iOS开发
date: 2016-03-25 17:16:30
keywords: iOS开发, Objective-C编码规范, 编码规范
---

其实大多数的时间，我们写出来的代码并不仅仅是给自己看的，在协同开发中还有很多人会来Review你的代码，因此，为了不让别人吐槽自己的代码，必须要养成良好的习惯，让自己去学习一些非常好的编码风格，因此这里来罗列一下Objectiv-C常用的编码规范。

<!--more-->

【1】命名规则

- 仿照Cocoa的风格来，使用长命名风格
- 变量命名推荐的命名语素顺序是：最开头的是命名空间的简写，然后越重要、区别度越大的语素越要往前放。经典的结构是：作用范围 + 限定修饰 + 类型。

【2】 在每个方法的定义前留白一行，也就是在方法与方法之间留空一行。

【3】 功能相近的方法要放在一起，比如一些协议的方法，这里推荐用 #pragma mark - *** 的格式来导航代码，切分代码块。这样可以方便方法的查找，并且可以用快捷键control + 6来快速查找方法的位置。

【4】 在用property定义变量时，建议写全所有参数，尤其是如果想定义成只读的（那么一定要加上readonly），这也是代码安全性的一个习惯。在定义变量名时，使*号靠着变量名，不要留空格。例如：

```objc
@property (nonatomic, copy) NSString *myString;

```

【5】 定义长的变量值应该拆分成多行。尤其体现在使用数组或字典。以下也分别是快速声明数组@[] 和 字典@{}的方法。

```objc
 NSArray *array = @[@"Lin",
                       @"Hong",
                       @"is",
                       @"my",
                       @"sweet"
                       ];
    
    
    
    NSDictionary *dic = @{
                          @"name":@"LinH",
                          @"height":@"168cm",
                          @"weight":@"secret",
                          @"lover":@"Lix"
                          };


```

【6】 二元运算符和参数之间要留一个空格，如赋值号=左右两边各留一个空格。

```objc
    self.myString = @"Lin,i love u";
    
```

【7】 一元运算符和参数之间不放置空格，比如 ！非运算符，&按位运与，|按位或。

```objc

 BOOL isOpen = true;
 BOOL isClose = !isOpen;


```

【8】 强制类型转换和参数之间不放置空格。

```objc
NSString *str3 = (NSString*)self.myString;

```

【9】 尽量使用有意义的名字命名，拒绝使用i，j等无意义字符命名。命名时采用驼峰命名法，类的首字母大写，使用大驼峰命名，变量的首字母小写，使用小驼峰命名。

```objc
    NSManagedObjectContext  //类 (大驼峰)
    managedObjectContext    //变量（小驼峰）
    

```

【10】 尽量减少在代码中直接使用数字常亮，而使用宏定义等方式。如MAX_NUMBER_PHONE代替8等等。这样我们搜索和后期的修改维护代码也比较方便。

【11】 尽量减少代码中的重复使用，比如代码中多处要使用屏幕宽度，然后计算[UIScreen mainScreen].bounds.size.width很多次，很繁琐，代码也很长，不如直接宏定义。

```objc
#define SCREEN_WIDTH ([UIScreen mainScreen].bounds.size.width

```

【12】 合理使用约定俗成的缩略词：

- alloc:分配；
- alt:轮流，交替；
- app:应用程序;
- calc:计算;
- dealloc:销毁；
- func:函数、方法;
- horiz:水平的;
- info:信息;
- init:初始化;
- max:最大的;
- min:最小的;
- msg:消息;
- nib:Interface Builder;
- rect:矩形;
- temp:暂时的;
- vert:垂直的;

【13】 函数长度不要超过50行，小函数比大函数的可读性更强。函数的参数不宜过多，零元函数最好，一元函数也不错，高于三元的函数需重构。

【14】 合理范围内使用链式编程

```objc
 UIView *myView = [[UIView alloc] init];

```

但是嵌套不宜超过3层，超过3层需进行重构。

【15】 函数调用时所有参数在同一行。如果参数过多，则可以每行一个参数，每个参数以冒号对齐。

【16】 对传入参数的保护或者说是否为空的判断，尽量不要使用if(!obj),而使用NSAssert断言来处理。NSAssert是系统定义的宏。

```objc
NSAssert(myView != nil, @"myView参数为空");

```
- 如果条件判断为真，则程序继续执行
- 如果判断条件为假，则抛出异常，异常内容为后面定义的字符串


【17】 if-else超过四层的时候，就要考虑重构，多层的if-else结构很难维护。

【18】 当需要一定条件才执行某项操作时，最左边的应该是最重要的代码，不要将最重要的代码内嵌到if中。如良好的风格是:

```objc
- (void) someMethod {  
if(![someOther boolValue]) {  
   return;  
  }  
//最重要的代码写在这里；  
}

```

反面教材:

```objc
- (void) someMethod {  
if([someOther boolValue]) {   
     //重要代码；  
  }  
}

```

【19】 所有的逻辑块都使用{}花括号包围，就算只是一行代码。在写方法或者函数时，把花括号的开头放在跟方法名的同一行。

【20】 明确指定构造函数，并有适当的注释。

【21】 不要在init方法中把变量或者说属性初始化为0或者nil，因为没有必要。

【22】 UIView的子类化初始化的时候，不要进行任何的布局操作。布局操作应该在layoutSubviews里面做；需要重新布局的时候调用setNeedsLayout，而不要直接调用layoutSubviews。

【23】 保持公共API简单，也就是保持.h文件简单。放在.h中声明的函数都是会被公开的，如果根本就没必要对其他类公开，再不要在.h中声明。OC中的方法都是共有方法，没有私有方法一说。

【24】 一个文件只实现一个类，同一个文件中不要有多个类。

【25】 布局时尽量使用相对布局，比如使用子View在父View中的相对位置。

【26】 protocol单独用一个文件来创建，尽量不要与相关类混在一个文件中。

【27】 在类定义中使用到自己定义类的时候，尽量不要在头文件中引入自己定义类的头文件，使用@class替代。而在实现文件中引入头文件。

【28】 推荐方法的第一个花括号直接跟在方法体后，而不是另起一行，这样可以减少代码行。

【29】 `block`中第一行也要留空，同方法体中的第一行留空。使代码清晰。

【30】 代表类方法和实例方法的"+"加号，"-"减号后需要一个空格。这是一个非常小的细节，系统默认的方法都是这样的，我们自己声明或者实现一个方法的时候也需要这样。

【31】 不要用点语法来调用方法，只用来访问属性。这样是为了防止代码可读性问题。

【32】 一个类的**Delegate**对象通常还引用着类本身，这样很容易造成引用循环的问题，所以类的Delegate属性要设置为弱引用。

【33】 好的代码应该是"自解释的"，但仍然需要详细的注释来说明参数的意义、返回值、功能以及可能的副作用。