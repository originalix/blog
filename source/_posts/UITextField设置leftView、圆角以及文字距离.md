---
title: UITextField设置leftView、圆角以及文字距离
date: 2016-03-30 01:56:37
tags: ["iOS","UITextField"]

---

今天在工作中，搭建一个登录界面，因为涉及到用户名和密码的输入，所以在iOS中我们免不了要用到UITextField这个常见的输入控件。首先来看一下美工给我的效果图，这里我仅仅截出了UITextField这个部分的效果：

<!--more-->

![UITextField效果](https://raw.githubusercontent.com/originalix/OuGeV1/master/OuGeV1/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-03-30%20%E4%B8%8A%E5%8D%881.27.27.png)

这里我们能看到这个UITextField的基本要求有如下几个：

- 输入框内有提示图片
- 之后输入的文字与输入框内的图片有间距
- 输入框有圆角

大致分为上面的三个特殊要求，那么我们一个一个来分析，首先是输入框内的提示图片，这里我们要讲UITextField里的两个属性，leftview和rightview，这两个属性分别能设置textField内的左右两边的视图，可以插入图片,我用最简单的代码来展示textField的leftview怎么实现。

```objc

 UIImageView *imgView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"github.jpg"]];
    
    UITextField *textField = [[UITextField alloc] initWithFrame:CGRectMake(0, 0, 100, 20)];
    textField.leftView =imgView;
    textField.leftViewMode = UITextFieldViewModeAlways;
    
    [self addSubview:textField];

```

上面的代码，我们能很清楚的看到首先定义一个UIImageView，之后把这个imageView设置成textField的leftview，之后设置leftview的样式，就可以很简单的定义一个leftview。

UITextFieldViewMode是一个枚举类型，有如下属性:

```objc
typedef NS_ENUM(NSInteger, UITextFieldViewMode) {
    UITextFieldViewModeNever,
    UITextFieldViewModeWhileEditing,
    UITextFieldViewModeUnlessEditing,
    UITextFieldViewModeAlways
};
```

但是这样设置的TextField我们会发现，图片是紧紧贴在输入框的边缘的，看起来特别别扭不舒服，那么该怎么设置呢？我们可以子类化一个TextField，去复写它的一个方法来设置leftView的位置

```objc
- (CGRect)leftViewRectForBounds:(CGRect)bounds
{
    CGRect iconRect = [super leftViewRectForBounds:bounds];
    iconRect.origin.x += 15; //像右边偏15
    return iconRect;
}
```

在继承与UITextField中复写这个方法，得到的结果是leftView像右偏移15，是不是很简单呢。

如果这时候我们在输入框中打字，会发现leftview确实跟最初的输入框产生的距离，但是我们打出来的字还是紧紧的黏在图片上，用户体验也极差，根据上面的思路，我们可以接着在这个子类中复写它的设置方法来实现。

```objc

//UITextField 文字与输入框的距离
- (CGRect)textRectForBounds:(CGRect)bounds{
    
    return CGRectInset(bounds, 45, 0);
    
}

//控制文本的位置
- (CGRect)editingRectForBounds:(CGRect)bounds{
    
    return CGRectInset(bounds, 45, 0);
}
```

之前的图片是20大小，加上偏移的15那么一共是35，所以我们设置偏移45的量，即为文本比leftView的图片的最右边向右15。至此，我们已经完成了textField的文本和图片设置，最后来看一下圆角。

圆角有两种实现方式，一种是在layer层处理，来渲染绘制圆角

```objc
    textField.layer.cornerRadius = 4;
    textField.layer.masksToBounds = YES;
```

第二种是设置UITextfield的样式，也能实现自带圆角，但是这个圆角的值是固定的

```objc
textField.borderStyle = UITextBorderStyleRoundedRect; 
```

写到这里，这个UITextField在界面上的要求就已经基本完成了，一般我们用到的常用属性也就是这些。