---
title: iOS设计模式之简单工厂模式

date: 2016-03-18 17:20:37
tags: ["简单工厂模式","设计模式"]
---


最近在看关于设计模式的书籍，开始觉得在设计程序架构之时，能够灵活运用这些设计模式，代码将变得非常具有美感。一个好的设计模式使得程序更加的灵活，容易修改，易于使用。

从最简单的简单工厂模式开始学起，举一个实现计算器的例子，来完成简单工厂模式。

<!--more-->

### 普通的简易计算器代码示例
一个简单计算器，用四则运算来考虑的话，加减乘除，那么初学者会觉得很简单，用if条件来进行判断，判断好了之后就可以完成要求，而稍微有经验点的 可能会选择switch case的判断方式，例如下面的代码:

```objc
Operation运算方法的逻辑

- (void)operationWithnumberA:(double )numberA Withoperator:(char)operator WithnumberB:(double )numberB
{
/**
 *  封装了一个传递值的方法
 *
 *  @param numberA  数字A
 *  @param operator 运算符
 *  @param numberB  数字B
 */
    double result = 0;
    
    switch (operator) {
        case 'a':
        result = numberA + numberB;
            break;
            
        case 'b':
            result = numberA - numberB;
            break;
            
        case 'c':
            result = numberA * numberB;
            break;
            
            case 'd':
            if (numberB == 0) {
                NSLog(@"除数不能为0 请重新输入");
            }else{
                result = numberA / numberB;
            }
            
            case 'e':
            NSLog(@"退出");
            break;
            
        default:
            break;
    }

```

而客户端方面的代码 我们可以这么写

```objc
/**
 *  四则运算
 */
- (void)operation

{
    char a ;
    
    double numberA;
    NSLog(@"请输入数字A");
    scanf("%lf",&numberA);
    double numberB;
    NSLog(@"请输入数字B");
    scanf("%lf",&numberB);
    
    NSLog(@"加法请输入a");
    NSLog(@"减法请输入b");
    NSLog(@"乘法请输入c");
    NSLog(@"除法请输入d");
    NSLog(@"退出请输入e");
    
    scanf("%c",&a);
    
    [self operationWithnumberA:numberA Withoperator:a WithnumberB:numberB];  
}
```
在我们得到需要的数值之后，调用运算方法做判断，算出结果。

这样写就会比if的判断清晰，因为我们已经把业务逻辑和界面显示的部分完全分离了，在任何需要用到的地方，我们就可以直接复制这段代码，完成运算。

但是假如，我有一天的运算需求不满足于四则运算，而是希望加上开根号或者平方的运算方法，该怎么办。难道我们还要回头，去switch语句里再加判断条件，之后在界面上增加提示么？

之前的代码，我们只用到了面向对象的三个特性之一，就是封装，而解决我上一段话提出的疑问，我们可以用到另外两个特性，多态和继承来实现。

### 简单工厂模式的代码示例

为了实现之前的要求，在不改动其他代码的情况下，能够增加更多的运算方法，或者修改出问题的运算方法。那么我们首先先把四则运算，封装成四个类，即为加法类、减法类、乘法类、除法类。

```objc

@implementation AddOperation
/**
 *   加法
 */
+ (double)addOperationWithNumberA:(double)numberA WithNumberB:(double)numberB
{
    double result = 0;
    
    result = numberA + numberB ;
    
    NSLog(@"%f",result);

    return result;
}




@implementation SubOperation

/**
 *   减法
 */
+ (double)subOperationWithNumberA:(double)numberA WithNumberB:(double)numberB
{
    double result = 0;
    
    result = numberA - numberB ;
    
     NSLog(@"%f",result);
    
    return result;
}


@implementation MulOperation

/**
 *   乘法
 */
+ (double)mulOperationWithNumberA:(double)numberA WithNumberB:(double)numberB
{
    double result = 0;
    
    result = numberA * numberB ;
    
     NSLog(@"%f",result);
    
    return result;
}

@implementation DivOperation

/**
 *   除法
 */
+ (double)divOperationWithNumberA:(double)numberA WithNumberB:(double)numberB
{
    double result = 0;
    
    if (numberB == 0) {
        NSLog(@"除数不能为0 请重新输入");
    }else{
        result = numberA / numberB;
    }
    
     NSLog(@"%f",result);
    
    return result;
}
```
这样我们就已经把四则运算，封装成了四个类。因为偷懒，我并没有设计界面模型，只是把结果输出来，所以每段输出结果的NSLog请不要介意。

接下来，我们在简单工厂的Operation类中，把调用这四个类的运算方法实现。

```objc

/**
 *  封装了一个运算方法
 *
 *  @param numberA  数字A
 *  @param operator 运算符
 *  @param numberB  数字B
 */
+ (void)operationWithnumberA:(double )numberA Withoperator:(char)operator WithnumberB:(double )numberB
{
    
    switch (operator) {
        case 'a':
            [AddOperation addOperationWithNumberA:numberA WithNumberB:numberB];
            break;
            
        case 'b':
            [SubOperation subOperationWithNumberA:numberA WithNumberB:numberB];
            break;
            
        case 'c':
            [MulOperation mulOperationWithNumberA:numberA WithNumberB:numberB];
            break;
            
        case 'd':
            [DivOperation divOperationWithNumberA:numberA WithNumberB:numberB];
            break;
        case 'e':
            NSLog(@"退出");
            break;
            
        default:
            break;
    }  
}

```

以上就是在简单工厂的类中，调用四个运算方法的类，来实现运算，并且成功解耦合，有利于以后的维护和扩展。客户端方面的代码也就非常简单。

```objc

/**
 *  四则运算
 */
- (void)operation

{
    char a = 'a';
    
    double numberA = 10;
    
    double numberB = 20;
    
    
//    NSLog(@"加法请输入a");
//    NSLog(@"减法请输入b");
//    NSLog(@"乘法请输入c");
//    NSLog(@"除法请输入d");
//    NSLog(@"退出请输入e");
    
    
    [Operation operationWithnumberA:numberA Withoperator:a WithnumberB:numberB];

}
```

客户端的代码还是偷懒，没有设计UI部分，所以也直接把数据代入进去了，但是大体的思路就是这样。直接用面向对象的三大特性来解决问题，在设计代码时，一定要本着可维护、可复用、可扩展、灵活性好的设计思路来设计。尤其要注意，这里的可复用，可不是可复制哦。今天的学习笔记就写到这里。

简单工厂模式的Demo我已经上传到Github上，如果觉得对您有帮助，请自行下载。

[Operation Factory Demo](https://github.com/originalix/OperationFactoryDemo.git)