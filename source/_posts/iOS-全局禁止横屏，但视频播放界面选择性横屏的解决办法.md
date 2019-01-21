---
title: iOS 全局禁止横屏，但视频播放界面选择性横屏的解决办法
date: 2016-07-14 15:25:15
tags: ["iOS开发","横屏解决方案"]

---



有时我们的APP并没有适配横屏的需求，但是在个别视频播放界面，我们需要在播放视频的时候横屏，退出全屏的时候不能横屏，但是有时候并没有原生API并没有给出解决方案。

<!--more-->

### 当其他界面不支持横屏时:

这个解决方法比较容易

在 `APPDelegate.h` 文件中增加属性：是否支持横屏

```objc
/***  是否允许横屏的标记 */
@property (nonatomic,assign)BOOL allowRotation;

```

在 `APPDelegate.m` 文件中增加方法，控制全部不支持横屏

```objc
- (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window {
    if (self.allowRotation) {
        return  UIInterfaceOrientationMaskAllButUpsideDown;
    }
    return UIInterfaceOrientationMaskPortrait;
}

```

这样在其他界面想要横屏的时候，我们只要控制 `allowRotation` 这个属性就可以控制其他界面进行横屏了。


```objc
//需在上面#import "AppDelegate.h"
AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
appDelegate.allowRotation = YES;
//不让横屏的时候 appDelegate.allowRotation = NO;即可

```


### 播放界面横屏

所以这里可以使用 `UIWindow` 的通知，就可以解决

```objc
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(begainFullScreen) name:UIWindowDidBecomeVisibleNotification object:nil];//进入全屏
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(endFullScreen) name:UIWindowDidBecomeHiddenNotification object:nil];//退出全屏

```

在退出全屏时，增加逻辑让其强制编程竖屏，这样当全屏播放的时候，点击 `down("完成")` 时，就会自动变成竖屏了。

```objc
// 进入全屏
-(void)begainFullScreen
{
    AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
    appDelegate.allowRotation = YES;
}
// 退出全屏
-(void)endFullScreen
{
    AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
    appDelegate.allowRotation = NO;
    
    //强制归正：
    if ([[UIDevice currentDevice] respondsToSelector:@selector(setOrientation:)]) {
        SEL selector = NSSelectorFromString(@"setOrientation:");
        NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:[UIDevice instanceMethodSignatureForSelector:selector]];
        [invocation setSelector:selector];
        [invocation setTarget:[UIDevice currentDevice]];
        int val =UIInterfaceOrientationPortrait;
        [invocation setArgument:&val atIndex:2];
        [invocation invoke];
    }
}

```

上面的两个方案已经足够解决只有部分界面需要支持横屏的问题了，亲测可用