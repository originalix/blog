---
title: iOS开发——制作圆形头像
date: 2016-03-30 02:11:16
tags: ["iOS开发","圆形头像","UI"]

---

在iOS7之后，我们能发现许多应用都开始使用圆形来作为用户头像的形状，代表App就是腾讯QQ了，QQ的头像就是圆形的。

在今天看到美工给的登陆效果图时，我发现也是要求做一个圆形的头像显示效果，在晚上琢磨之后，我打算把这段经验记录一下，因为以后肯定会用到的次数也很多，为此我也专门Category一个类目以便日后使用。效果图如下 :

<!--more-->

![圆形头像效果图](https://raw.githubusercontent.com/originalix/OuGeV1/master/OuGeV1/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-03-30%20%E4%B8%8A%E5%8D%881.22.18.png)

这里可能看得不是特别清楚，实际的效果，在圆形头像的外部还有一个外框，用暗淡的阴影显示。

制作这个圆形头像，我的大体思路就是直接用Core graphic直接绘制，将原本的非圆形图片直接裁剪为圆形，之后再绘制上外面的阴影。

如果对外边框没有要求的同学，可以直接用最简单的方式来设置，我把简单的方法先贴出来:

```objc
UIImage * image = [UIImage imageNamed:@"icon_huo"]; 
UIImageView * imageV = self.imageView; 
imageV.layer.masksToBounds = YES; 
imageV.layer.cornerRadius =imageV.frame.size.width / 2 ;

 /**如果需要边框，请把下面2行注释去掉*/

// imageV.layer.borderColor = [UIColor purpleColor].CGColor;

// imageV.layer.borderWidth = 10; 

imageV.image= image;

```

为了之后代码的复用，以及提高开发效率，我把这个方法做了封装，我直接把封装好的代码贴出来，注释很全，很容易理解，对照着上面的效果图一起看吧。

```objc
/**
 *  圆形头像的绘制
 *
 *  @param icon 头像文件名
 *
 *  @return image
 */
+ (instancetype)imageWithIconName:(NSString *)icon{
   
    //边框大小
    CGFloat border = 113.0;
    
    //这里不用管实现的方法，只要你设置一张你想使用的边框图片就可以了
    UIImage *borderImg = [self createImageWithColor:[UIColor colorWithRed:53 green:53 blue:68 alpha:0.32]];
    
    //头像图片
    UIImage *image = [UIImage imageNamed:icon];
    //设置头像白色边框 像素6px
    CGSize size = CGSizeMake(image.size.width + border, image.size.height + border);
    
    //创建图片上下文
    UIGraphicsBeginImageContextWithOptions(size, NO, 0);
    
    //绘制边框的圆
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextAddEllipseInRect(context, CGRectMake(0, 0, size.width, size.height));
    
    //剪切可视范围
    CGContextClip(context);
    
    //绘制边框图片
    [borderImg drawInRect:CGRectMake(0, 0, size.width, size.height)];
    
    //设置头像
    CGFloat iconX = border/2;
    CGFloat iconY = border/2;
    CGFloat iconW = image.size.width;
    CGFloat iconH = image.size.height;
    
    //绘制圆形头像范围
    CGContextAddEllipseInRect(context, CGRectMake(iconX, iconY, iconW, iconH));
    //剪切可视范围
    CGContextClip(context);
    //绘制头像
    [image drawInRect:CGRectMake(iconX, iconY, iconW, iconH)];
    //取出整个头像上下文的图片
    UIImage *iconImage = UIGraphicsGetImageFromCurrentImageContext();
    
    return iconImage;
}
```