---
title: iOS开发——全机型适配思路
date: 2016-08-25 15:57:47
tags: ["iOS开发", "全机型适配", "AutoLayout"]

---

最近一直在研究学习ReactiveCocoa，并且在给项目转型到MVVM模式打基础，所以博客也很久没有更新了。

今天打算跟大家聊聊最近研究的全机型适配思路。

<!--more-->

当前我们需要适配的iPhone机型有4s、5s、6s、6Plus四种机型。它们的尺寸分别是

```objc
 iphone4s {320, 480}     960*640
 iphone5 5s {320, 568}     1136*640
 iphone6 6s   {375, 667}    1334*750
 iphone6Plus 6sPlus {414, 736}  1920*1080

```

而一般我习惯在实际的项目开发中，使用**Masonary**来搭建**UI**界面，虽然在**Masonary**中我们能很方便的设置各个控件之间的约束，但是对于类似**4s**机型和**6s Plus**机型的很大的高度差，有时候仅仅靠一次性成型的约束还是搭建不出很合理的界面。

于是在这次搭建**UI**的过程中，我的一个思路就是按照比例，针对各个机型进行微调。思路如下:

- 美工提供的效果图是基于**iPhone6**的效果图

- 而我只需要将标注上的每个尺寸去对比**iPhone6**换算出比例，这样一些间距就能按照不同机型尺寸的比例变得不一样。


- 针对考虑交互体验的控件，在保持尺寸不变的基础上，做细节微调。

在具体的代码中，我封装出了一个类，定义了两个类方法专门去适配所有机型的高度和宽度。思路就是上述按不同机型针对于iPhone6的比例而适配。

代码我也贴一部分出来。


**头文件的定义**

```objc

#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

typedef NS_ENUM(NSInteger, IPhoneType) {
    iPhone4Type = 0,
    iPhone5Type,
    iPhone6Type,
    iPhone6PlusType
};

@interface CalculateLayout : NSObject

/**
 *  基于UI设计的iPhone6设计图的全机型高度适配
 *
 *  @param height View高度
 *
 *  @return  CGFloat 适配后的高度
 */

+ (CGFloat)neu_layoutForAlliPhoneHeight:(CGFloat)height;
/**
 *  基于UI设计的iPhone6设计图的全机型宽度适配
 *
 *  @param width 宽度
 *
 *  @return 适配后的宽度
 */
+ (CGFloat)neu_layoutForAlliPhoneWidth:(CGFloat)width;

```

**`.m`文件的部分如下:**

```objc

#define iPhone4Height (480.f)
#define iPhone4Width  (320.f)

#define iPhone5Height (568.f)
#define iPhone5Width  (320.f)

#define iPhone6Height (667.f)
#define iPhone6Width  (375.f)

#define iPhone6PlusHeight (736.f)
#define iPhone6PlusWidth  (414.f)

#pragma mark - 适配所有机型高度
+ (CGFloat)neu_layoutForAlliPhoneHeight:(CGFloat)height {
    CGFloat layoutHeight = 0.0f;
    if (iPhone4) {
        layoutHeight = ( height / iPhone6Height ) * iPhone4Height;
    } else if (iPhone5) {
        layoutHeight = ( height / iPhone6Height ) * iPhone5Height;
    } else if (iPhone6) {
        layoutHeight = ( height / iPhone6Height ) * iPhone6Height;
    } else if (iPhone6P) {
        layoutHeight = ( height / iPhone6Height ) * iPhone6PlusHeight;
    } else {
        layoutHeight = height;
    }
    return layoutHeight;
}

+ (CGFloat)neu_layoutForAlliPhoneWidth:(CGFloat)width {
    CGFloat layoutWidth = 0.0f;
    if (iPhone4) {
        layoutWidth = ( width / iPhone6Width ) * iPhone4Width;
    } else if (iPhone5) {
        layoutWidth = ( width / iPhone6Width ) * iPhone5Width;
    } else if (iPhone6) {
        layoutWidth = ( width / iPhone6Width ) * iPhone6Width;
    } else if (iPhone6P) {
        layoutWidth = ( width / iPhone6Width ) * iPhone6PlusWidth;
    }
    return layoutWidth;
}


```


代码我也已经放在了[Github](https://github.com/originalix/Layou-For-All-iPhone)上，如果这些对你有帮助，在clone代码之余能否给个star。


