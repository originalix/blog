---
title: UINavigationBar使用总结
date: 2016-04-05 20:28:40
tags: ["iOS开发","UINavigationBar"]

---

UINavigationBar是一个我们在开发中必定会碰到的控件，用好它能帮助我们自定义导航栏的样式，所以今天讲解一下UINavigationBar的用法。

<!--more-->

### 设置导航栏的标题

这个直接是很简单的设置，一行代码搞定

```objc
self.navigationItem.title = @"导航栏标题";
```

### 设置导航栏背景颜色

导航栏的背景颜色，也是很简单的 自己替换代码中的颜色即可

```objc
self.navigationBar.barTintColor =[UIColor blackColor];
```

### 设置导航栏的背景图片

这里虽然一行代码很简单，但是要来简单的说一下BarMetrics这个枚举值

```objc
[self.navigationBar setBackgroundImage:[UIImage imageNamed:@"123.jpg"] forBarMetrics:UIBarMetricsDefault];
```

```objc
//表示横屏竖屏都显示
UIBarMetricsDefault,

//表示在只横屏下才显示，和UIBarMetricsLandscapePhone功效一样，不过iOS8已经弃用了
UIBarMetricsCompact,

UIBarMetricsDefaultPrompt和UIBarMetricsCompactPrompt

```

### 更改顶部状态栏的颜色

```objc
typedef NS_ENUM(NSInteger, UIStatusBarStyle) {
   UIStatusBarStyleDefault = 0, // Dark content, for use on light backgrounds
    UIStatusBarStyleLightContent NS_ENUM_AVAILABLE_IOS(7_0) = 1, // Light content, for use on dark backgrounds
    
```

这个一个是默认的，黑色颜色，用于亮色背景，一个是白色用于深色背景

### 设置返回按钮

有时候我们会发现，我们设置的返回按钮都是蓝色的默认颜色，那么到底该怎么更改这些按钮的颜色呢

- 设置返回按钮的颜色，只设置tintColor的颜色就好了

```objc
self.navigationController.navigationBar.tintColor = [UIColor whiteColor];
```

- 只设置返回按钮的图片

```objc
- (void)goToBack {
    [self.navigationController popViewControllerAnimated:YES];
}

- (void)setBackButtonWithImage {
    UIImage *leftButtonIcon = [[UIImage imageNamed:@"LeftButton_back_Icon"]imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
    
    UIBarButtonItem *leftButton = [[UIBarButtonItem alloc] initWithImage:leftButtonIcon
                                                                   style:UIBarButtonItemStyleBordered
                                                                  target:self
                                                                  action:@selector(goToBack)];
    self.navigationItem.leftBarButtonItem = leftButton;

    //修复navigationController侧滑关闭失效的问题
    self.navigationController.interactivePopGestureRecognizer.delegate = (id)self;
}
```

这里需要注意的地方有三点：

```objc
需要自己实现返回按钮的事件。
特别的解释下UIImage的imageWithRenderingMode:方法，参数UIImageRenderingModeAlwaysOriginal 表示总是用原图渲染，如果不这么设置，返回按钮将会显示tintColor的颜色(默认为蓝色)。UITabbarItem也存在同样地问题。
我们自己设置返回按钮，会导致系统的侧滑关闭效果失效。添加上面代码中最后一句代码即可修复。
```

- 仅仅设置返回按钮的文字

```objc
- (void)setBackButtonTitle {
    UIBarButtonItem *leftButton = [[UIBarButtonItem alloc] initWithTitle:NSLocalizedString(@"取消", nil)
                                                                   style:UIBarButtonItemStylePlain
                                                                  target:self action:@selector(goToBack)];
    leftButton.tintColor = [UIColor whiteColor];
    self.navigationItem.leftBarButtonItem = leftButton;
}

```

- 自定义返回按钮

如果你对返回按钮实在不满意，你可以自定义一个按钮，并把它设置为navigation的leftButton

```objc
- (void)setCustomLeftButton {
    UIView* leftButtonView = [[UIView alloc]initWithFrame:CGRectMake(0, 0, 60, 40)];
    UIButton* leftButton = [UIButton buttonWithType:UIButtonTypeSystem];
    leftButton.backgroundColor = [UIColor clearColor];
    leftButton.frame = leftButtonView.frame;
    [leftButton setImage:[UIImage imageNamed:@"LeftButton_back_Icon"] forState:UIControlStateNormal];
    [leftButton setTitle:@"返回" forState:UIControlStateNormal];
    leftButton.tintColor = [UIColor redColor];
    leftButton.autoresizesSubviews = YES;
    leftButton.contentHorizontalAlignment = UIControlContentHorizontalAlignmentLeft;
    leftButton.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleLeftMargin;
    [leftButton addTarget:self action:@selector(goToBack) forControlEvents:UIControlEventTouchUpInside];
    [leftButtonView addSubview:leftButton];
    UIBarButtonItem* leftBarButton = [[UIBarButtonItem alloc] initWithCustomView:leftButtonView];
    self.navigationItem.leftBarButtonItem = leftBarButton;
}
```

### 设置导航栏底部线条的颜色 

有了上面的基础，设置导航栏线条的颜色就变得很简单了。
首先，我做了个UIImage的分类：通过颜色转成UIImage；
然后，用上面的方案来设置导航栏底部线条。

颜色转图片的代码：

```objc
@implementation UIImage (ColorImage)

+ (UIImage *)imageWithColor:(UIColor *)color
{
    CGRect rect = CGRectMake(0.0f, 0.0f, 1.0f, 1.0f);
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();

    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, rect);

    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();

    return image;
}

@end
```

设置导航栏底部线条颜色的代码:

```objc
UINavigationBar *navigationBar = self.navigationController.navigationBar;
    [navigationBar setBackgroundImage:[[UIImage alloc] init]
                       forBarPosition:UIBarPositionAny
                           barMetrics:UIBarMetricsDefault];
    //此处使底部线条颜色为红色
    [navigationBar setShadowImage:[UIImage imageWithColor:[UIColor redColor]]];
```

关于navigation的用法，就先写到这里，以后碰到更多的问题再接着更新。