---
title: iOS开发——定制UITextField
date: 2016-04-02 21:07:27
tags: ["iOS开发","UITextField"]

---


在iOS中UITextField这个控件作为文本输入控件一定是使用率最高的几个控件之一，而iOS提供的默认的原始TextField的造型肯定在开发时很难满足我们的要求，原因很简单，不够美观，实在太单调。所以今天我们从一些简单的复写UITextField方法开始，来讲一讲如何定制一个属于自己的UITextField。

<!--more-->

之前的文章我们讲过UITextField中，如何设置leftView，圆角以及控制文字输入时的距离。今天我们主要从UITextField的键盘收起、placeholder的设置以及自定义距离、字体，以及控制输入文本时，距离UITextField边框的距离和UITextField中一些常用的方法和枚举变量等方面来阐述如何定制自己的UITextField。

### 键盘的收起

首先我们先来看UITextField的键盘弹出和回收，UITextField在默认的情况下，键盘在输入完成后是不会自动回收的，这里我们讲解如何在按下Return键时，键盘自动回收。首先我们要遵循<UITextFieldDelegate>协议，之后在后面写入

```objc
- (BOOL)textFieldShouldReturn:(UITextField *)textField
{
    [textField resignFirstResponder];
    return YES;
}
```

通过写入这个方法，来实现按下Return按钮回收键盘。

### placeholder的设置

在一些特定功能的文本输入框，我们常常要设置placeholder属性来指明当期UITextField的功能，例如：请在此处输入密码。可是placeholde的默认属性是紧贴文本输入框的，而且字体以及字体大小也不美观，于是我们可以这么来设置placeholder

```objc

//控制placeHolder的位置，左右缩20
-(CGRect)placeholderRectForBounds:(CGRect)bounds
{
        return CGRectInset(bounds, 20, 4);
}

//控制左视图位置
- (CGRect)leftViewRectForBounds:(CGRect)bounds
{

    return CGRectInset(bounds,0,0);
}

//控制编辑文本的位置
-(CGRect)editingRectForBounds:(CGRect)bounds
{
    return CGRectInset( bounds, 20, 0);
}

//控制显示文本的位置
-(CGRect)textRectForBounds:(CGRect)bounds
{
    return CGRectInset(bounds, 20, 0);
}
```

我们可以先如上面的代码一样，设置placeholder的位置，同时要注意的一点是，在设置了placeholder的位置之后，我们也要相应的调整文本显示的位置，以及在编辑完成后，文本显示在输入框的位置。

至于placeholder的字体和字体大小设置 可以用如下方法设置，记住这个方法写在子类化的UITextField中是没有效果的，一定要写在创建UITextField的过程中。

```objc
    [TextField setValue:[UIFont fontWithName:@"Arial" size:12]   forKeyPath:@"_placeholderLabel.font"];
```

### UITextField中一些常用的属性以及枚举变量

####UITextFieldBorder 边框设置

 设置TextField的边框效果，一定要设置了才有效果，类型如下

```objc

typedef NS_ENUM(NSInteger, UITextBorderStyle) {
    UITextBorderStyleNone,
    UITextBorderStyleLine,
    UITextBorderStyleBezel,
    UITextBorderStyleRoundedRect
};

```

####UITextFieldViewMode

此属性用来定义我们之前讲的leftView和rightView的存在时机

```objc
typedef NS_ENUM(NSInteger, UITextFieldViewMode) {
    UITextFieldViewModeNever,
    UITextFieldViewModeWhileEditing,
    UITextFieldViewModeUnlessEditing,
    UITextFieldViewModeAlways
};
```

#### UIReturnKeyType返回按钮类型 

在键盘上的返回按键，系统也给我们提供了一些常用的类型

```objc
typedef NS_ENUM(NSInteger, UIReturnKeyType) {
    UIReturnKeyDefault,
    UIReturnKeyGo,
    UIReturnKeyGoogle,
    UIReturnKeyJoin,
    UIReturnKeyNext,
    UIReturnKeyRoute,
    UIReturnKeySearch,
    UIReturnKeySend,
    UIReturnKeyYahoo,
    UIReturnKeyDone,
    UIReturnKeyEmergencyCall,
    UIReturnKeyContinue NS_ENUM_AVAILABLE_IOS(9_0),
};
```

#### UIKeyboardType键盘类型

```objc
typedef NS_ENUM(NSInteger, UIKeyboardType) {
    UIKeyboardTypeDefault,       

    UIKeyboardTypeASCIICapable,      

    UIKeyboardTypeNumbersAndPunctuation, 

    UIKeyboardTypeURL,    

    UIKeyboardTypeNumberPad,  

    UIKeyboardTypePhonePad,             

    UIKeyboardTypeNamePhonePad, 

    UIKeyboardTypeEmailAddress,
  
    UIKeyboardTypeDecimalPad ,

    UIKeyboardTypeWebSearch ,

    UIKeyboardTypeAlphabet = UIKeyboardTypeASCIICapable, 
};
```

```objc
//输入框中是否有个叉号，在什么时候显示，用于一次性删除输入框中的内容
  text.clearButtonMode = UITextFieldViewModeAlways;

//每输入一个字符就变成点 用语密码输入
  text.secureTextEntry = YES;

//是否纠错
  text.autocorrectionType = UITextAutocorrectionTypeNo;
 
typedef enum {
    UITextAutocorrectionTypeDefault, 默认
    UITextAutocorrectionTypeNo,   不自动纠错
    UITextAutocorrectionTypeYes,  自动纠错
} UITextAutocorrectionType;
 
//再次编辑就清空
  text.clearsOnBeginEditing = YES; 

//设置为YES时文本会自动缩小以适应文本窗口大小.默认是保持原来大小,而让长文本滚动  
  textFied.adjustsFontSizeToFitWidth = YES;
 
//首字母是否大写
  text.autocapitalizationType = UITextAutocapitalizationTypeNone;
 
typedef enum {
    UITextAutocapitalizationTypeNone, 不自动大写
    UITextAutocapitalizationTypeWords,  单词首字母大写
    UITextAutocapitalizationTypeSentences,  句子的首字母大写
    UITextAutocapitalizationTypeAllCharacters, 所有字母都大写
} UITextAutocapitalizationType;


 //键盘外观
textView.keyboardAppearance=UIKeyboardAppearanceDefault；
typedef enum {
UIKeyboardAppearanceDefault， 默认外观，浅灰色
UIKeyboardAppearanceAlert，     深灰 石墨色
 
} UIReturnKeyType;
 
```

大体属性已经罗列完毕，以后想到再来补充