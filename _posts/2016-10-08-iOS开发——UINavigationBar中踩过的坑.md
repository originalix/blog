---
layout: post
title: iOS开发——UINavigationBar中踩过的坑
categories: iOS开发
date: 2016-10-08 15:09:44
keywords: iOS开发, UINavigationBar
---

这段时间的一直在忙于编码，加上国庆假期等等时间，又有很长时间没有写过博客了。

自从升级了Xcode8，以及在做iOS10的适配工作中，我发现在NavigationBar这个控件中，有了一个小小的坑。

因为在iOS7之后，NavigationBar之后，默认有一条1px的细线，这条细线怎么去，我在这里就不赘述了，因为谷歌上以及StackOverflow上有太多的方法去除这条细线。但是我这次发现，在我升级到iOS10之后，iOS10的设备中虽然使用了以前的方法，但是还是出现了这个细线，但是iOS10以下的设备这条细线还是不存在的。

<!--more-->

于是我自己得出了这么个结论，之前去除NavigationBar的这条细线的方法失效了（这里并不是说所有方法失效，至少我使用的方法是失效的），那么在发现自己有这个问题的时候，不妨可以来换一种方法实现隐藏NavigationBar底下的这条细线。

我把我的新方法，写成了Category，这里直接贴代码出来吧。

头文件中的方法声明

```objc

/**
 * NavigationBar底部隐藏1px的线
 */
- (void)lix_hideBottomHairline;

/**
 * NavigationBar底部显示1px的线
 */
- (void)lix_showBottomHairline;

```

方法的实现

```objc
- (void)lix_hideBottomHairline {
    UIImageView *navBarHairlineImageView = [self findHairlineImageViewUnder:self];
    navBarHairlineImageView.hidden = YES;
}

- (void)lix_showBottomHairline {
    // Show 1px hairline of translucent nav bar
    UIImageView *navBarHairlineImageView = [self findHairlineImageViewUnder:self];
    navBarHairlineImageView.hidden = NO;
}

- (UIImageView *)findHairlineImageViewUnder:(UIView *)view {
    if ([view isKindOfClass:UIImageView.class] && view.bounds.size.height <= 1.0) {
        return (UIImageView *)view;
    }
    for (UIView *subview in view.subviews) {
        UIImageView *imageView = [self findHairlineImageViewUnder:subview];
        if (imageView) {
            return imageView;
        }
    }
    return nil;
}
```

简简单单，就可以随意切换NavigationBar底部线条的隐藏和显示，这样的代码可扩展性更好。

既然讲到这里了，那就干脆把NavigationBar如何变成透明的这点也讲完好了。

有时候，我们希望形成一个透明的NavigationBar，而不是像系统一样存在一个毛玻璃的效果，所以这时候我们应该如下设置NavigationBar

```objc

- (void)lix_makeTransparent {
    [self setTranslucent:YES];
    [self setBackgroundImage:[[UIImage alloc] init] forBarMetrics:UIBarMetricsDefault];
    self.backgroundColor = [UIColor clearColor];
    self.shadowImage = [UIImage new];    // Hides the hairline
    [self lix_hideBottomHairline];
}

```

如果要恢复默认，则如下设置

```
- (void)lix_makeDefault {
    [self setTranslucent:YES];
    [self setBackgroundImage:nil forBarMetrics:UIBarMetricsDefault];
    self.backgroundColor = nil;
    self.shadowImage = nil;    // Hides the hairline
    [self lix_showBottomHairline];
}

```

这里就组成了整个NavigationBar的category，希望大家能在自己的项目中灵活运用。
