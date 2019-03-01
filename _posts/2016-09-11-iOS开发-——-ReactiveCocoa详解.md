---
layout: post
title: iOS开发 —— ReactiveCocoa详解
categories: iOS开发
date: 2016-09-11 13:41:53
keywords: iOS开发, ReactiveCocoa
---

最近一直在研究**ReactiveCocoa**，现在也来讲讲**ReactiveCocoa**中一些基础类的作用。

# ReactiveCocoa作用

在我们iOS开发过程中，当某些事件响应的时候，需要处理某些业务逻辑,这些事件都用不同的方式来处理。比如按钮的点击使用`action`，`ScrollView`滚动使用`delegate`，属性值改变使用**KVO**等系统提供的方式。其实这些事件，都可以通过**RAC**处理。

<!--more-->

# **RACSiganl**

**RACSiganl:** 信号类,只是表示当数据改变时，信号内部会发出数据，它本身不具备发送信号的能力，而是交给内部一个订阅者去发出。


**RACSubscriber:** 表示订阅者的意思，用于发送信号，这是一个协议，不是一个类，只要遵守这个协议，并且实现方法才能成为订阅者。通过**create**创建的信号，都有一个订阅者，帮助他发送数据


**RACDisposable:** 用于取消订阅或者清理资源，当信号发送完成或者发送错误的时候，就会自动触发它。


```objc

 //1.创建信号
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        //block调用时刻：每当有订阅者订阅信号，就会调用block
        
        //2.发送信号
        [subscriber sendNext:@1];
        
        //如果不再发送数据，最好发送信号完成，内部会自动调用[RACDisposable disposable]取消订阅
        [subscriber sendCompleted];
        
        return [RACDisposable disposableWithBlock:^{
            //block调用时刻：当信号发送完成或者发送错误，就会自动执行这个block，取消订阅
            NSLog(@"信号被销毁");
        }];
    }];
    
    
    //3.订阅信号
    [signal subscribeNext:^(id x) {
        //block调用时刻：每当有信号发送数据，就会调用该方法
        NSLog(@"接收到的数据：%@",x);
    }];
    
```

# RACSubject与RACReplaySubject

**RACSubject:**信号提供者，自己可以充当信号，又能发送信号。**subject**可以想成是**signal**的变体，就像**NSMutableArray**相对于**NSArray**一样。它们是非**RAC**的代码和**RAC**代码之间的桥梁。

**RACReplaySubject:**重复提供信号类，**RACSubject**的子类。

**RACReplaySubject**与**RACSubject**区别:

- **RACReplaySubject**可以先发送信号，再订阅信号，**RACSubject**就不可以。

- **RACReplaySubject**可以设置**capacity**数量来限制缓存的**value**的数量,即只缓充最新的几个值。

- 如果一个信号每被订阅一次，就需要把之前的值重复发送一遍，就需要使用**RACReplaySubject**

```objc

   //1.创建信号
    RACSubject *subject = [RACSubject subject];
    
    //2.订阅信号
    [subject subscribeNext:^(id x) {
        //block调用时刻：当信号发出新值，就会调用
        NSLog(@"第一个订阅者%@",x);
    }];
    [subject subscribeNext:^(id x) {
        //block调用时刻：当信号发出新值，就会调用
        NSLog(@"第二个订阅者%@",x);
    }];
    
    //3.发送信号
    [subject sendNext:@"1"];
    
  
    
      //1.创建信号
    RACReplaySubject *replaySubject = [RACReplaySubject subject];
   // RACReplaySubject *replaySubject = [RACReplaySubject replaySubjectWithCapacity:0];
    
    //2.发送信号
    [replaySubject sendNext:@1];
    [replaySubject sendNext:@2];
    
    //3.订阅信号
    [replaySubject subscribeNext:^(id x) {
        NSLog(@"第一个订阅者%@",x);
    }];
    [replaySubject subscribeNext:^(id x) {
        NSLog(@"第二个订阅者%@",x);
    }];

```
    

## **RACSubject**替代代理

情景：跳转到另一个ViewController，TwoViewController发送通知，ViewController收到回调的通知


```objc

//ViewController里

- (IBAction)click:(UIButton *)sender {

    TwoViewController *twoVC = [[TwoViewController alloc] init];
    
    //设置代理信号
    twoVC.delegateSubject = [RACSubject subject];
    
    //订阅代理信号
    [twoVC.delegateSubject subscribeNext:^(id x) {
        NSLog(@"点击了通知按钮，%@",x);
    }];
    
    //跳转
    [self presentViewController:twoVC animated:YES completion:nil];

}

//TwoViewConrroller里

    if (self.delegateSubject) {
        //发送信号
        [self.delegateSubject sendNext:@"已跳转到TwoVC"];
    }
    
```


# 遍历数组字典、字典转模型

**RACTuple:**元组类,类似**NSArray**,用来包装值.

**RACSequence:** RAC中的集合类，用于代替**NSArray**,**NSDictionary**,可以使用它来快速遍历数组和字典。

```objc

    //第一步: 把数组转换成集合RACSequence             numbers.rac_sequence
    // 第二步: 把集合RACSequence转换RACSignal信号类    numbers.rac_sequence.signal
    // 第三步: 订阅信号，激活信号，会自动把集合中的所有值，遍历出来。    
    // 1.遍历数组
    NSArray *numbers = @[@1,@2,@3,@4];
    [numbers.rac_sequence.signal subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];

```

```objc


    // 2.遍历字典,遍历出来的键值对会包装成RACTuple(元组对象)
    NSDictionary *dict = @{@"name":@"xiaoming",@"age":@18};    
    // 解包元组，会把元组的值，按顺序给参数里面的变量赋值
    // 相当于以下写法
    //        NSString *key = x[0];
    //        NSString *value = x[1];
    [dict.rac_sequence.signal subscribeNext:^(id x) {
        RACTupleUnpack(NSString *name,NSString *age) = x;
        NSLog(@"%@ %@",name,age);
    }];
    
```

- 字典转模型

```objc

    //1.OC写法
    NSDictionary *dict1 = @{@"name":@"xiaoming",@"age":@18};
    NSDictionary *dict2 = @{@"name":@"xiaohua",@"age":@20};
    NSArray *arrs = @[dict1,dict2];
    
    NSMutableArray *items = [NSMutableArray array];
    
    for (NSDictionary *dict in arrs) {
        FlagItem *item = [FlagItem flagWithDict:dict];
        [items addObject:item];
    }
    
    //2.RAC写法
    [arrs.rac_sequence.signal subscribeNext:^(id x) {
        //遍历RAC字典
        FlagItem *item = [FlagItem flagWithDict:x];
        [items addObject:item];
    }];
    
    //3.高级RAC写法
    // map:映射的意思，目的：把原始值value映射成一个新值
    // array: 把集合转换成数组
    // 底层实现：当信号被订阅，会遍历集合中的原始值，映射成新值，并且保存到新的数组里。
    NSArray *flags = [[arrs.rac_sequence map:^id(id value) {
    
        return [FlagItem flagWithDict:value];
    
    }] array];
    NSLog(@"%@",flags);
    

```

# RACCommand

创建并订阅响应**action**的信号。 通常**command**是由**UI**触发的，像一个按钮被点击时。当**command**被触发时，控件会⾃自动被禁⽤。

有数据改变使用**RACSignal** 有事件处理需要**RACCommand**

**RACCommand**设计思想：内部**signalBlock**为什么要返回一个信号，这个信号有什么用。

- 在RAC开发中，通常会把网络请求封装到**RACCommand**，直接执行某个**RACCommand**就能发送请求。

- 当**RACCommand**内部请求到数据的时候，需要把请求的数据传递给外界，这时候就需要通过**signalBlock**返回的信号传递了。

使用场景,监听按钮点击，网络请求

```objc

   //1.创建命令
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        
        NSLog(@"执行命令");
        
        // signalBlock必须要返回一个信号，不能传nil，如果不想要传递信号，直接创建空的信号。
        //return [RACSignal empty];
        
        //2.创建信号,用来传递数据
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            
            [subscriber sendNext:@"请求数据"];
            
            //RACCommand中信号如果数据传递完，必须调用[subscriber sendCompleted]，这时命令才会执行完毕，否则永远处于执行中。
            [subscriber sendCompleted];
        
            return nil;
        }];
    }];
    
    //RACCommand需要被强引用，否则接收不到RACCommand中的信号，因此RACCommand中的信号是延迟发送的。
    _command = command;
    
    //3.订阅信号
    [command.executionSignals subscribeNext:^(id x) {
        
        [x subscribeNext:^(id x) {
            NSLog(@"%@",x);
        }];
    }];
    
    //RAC高级用法：
    // switchToLatest:用于signal of signals，获取signal of signals发出的最新信号,也就是可以直接拿到RACCommand中的信号,不需要订阅信号
    [command.executionSignals.switchToLatest subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
    
    //监听命令是否执行完毕，默认会来一次，可以直接跳过,skip表示跳过第一次命令
    [[command.executing skip:1] subscribeNext:^(id x) {
        
        if ([x boolValue] == YES) {
            NSLog(@"正在执行");
        }else{
            NSLog(@"执行完成");
        }
    }];
    
    
    //4.执行命令
    [self.command execute:nil];
    

```

# RACMulticastConnection

用于当一个信号，被多次订阅时，为了保证创建信号时，避免多次调用创建信号中的**block**，造成副作用，可以使用这个类处理。例如：当有2个**RACSignal**订阅信心的时候，就需要发送两次**RACSiagnal**的信号，执行两次**block**操作。而使用**RACMulticastConnection**连接，对**signal pulish**处理就不会多次创建。

```objc

 //1.创建信号a
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        NSLog(@"发送请求");
        
        [subscriber sendNext:@1];
        
        return nil;
    }];
    
    //2，创建连接
    RACMulticastConnection *connect = [signal publish];
    
    //3.订阅信号。即使订阅了，还没激活信号
    [connect.signal subscribeNext:^(id x) {
        NSLog(@"订阅者第一信号");
    }];
    [connect.signal subscribeNext:^(id x) {
        NSLog(@"订阅者第二信号");
    }];
    
    //4.连接，激活信号
    [connect connect];

```

# ReactiveCocoa的其他用法

 - 代替代理:`rac_signalForSelector:`用于替代代理。

- 代替KVO :`rac_valuesAndChangesForKeyPath:`用于监听某个对象的属性改变。

- 监听事件:`rac_signalForControlEvents:`用于监听某个事件。

- 代替通知:`rac_addObserverForName:`用于监听某个通知。

- 监听文本框文字改变:`rac_textSignal:`只要文本框发出改变就会发出这个信号。

- 处理当界面有多次请求时，需要都获取到数据时，才能展示界面

- `rac_liftSelector:withSignalsFromArray:Signals:`当传入的**Signals**(信号数组)，每一个**signal**都至少**sendNext**过一次，就会去触发第一个**selector**参数的方法。使用注意：几个信号，参数一的方法就几个参数，每个参数对应信号发出的数据。

```objc

// 1.代替代理
// 需求：自定义redView,监听红色view中按钮点击
// 之前都是需要通过代理监听，给红色View添加一个代理属性，点击按钮的时候，通知代理做事情
// rac_signalForSelector:把调用某个对象的方法的信息转换成信号，就要调用这个方法，就会发送信号。
// 这里表示只要redV调用btnClick:,就会发出信号，订阅就好了。
[[redV rac_signalForSelector:@selector(btnClick:)] subscribeNext:^(id x) {
    NSLog(@"点击红色按钮");
}];
// 2.KVO
// 把监听redV的center属性改变转换成信号，只要值改变就会发送信号
// observer:可以传入nil
[[redV rac_valuesAndChangesForKeyPath:@"center" options:NSKeyValueObservingOptionNew observer:nil] subscribeNext:^(id x) {
     
    NSLog(@"%@",x);
     
}];
// 3.监听事件
// 把按钮点击事件转换为信号，点击按钮，就会发送信号
[[self.btn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(id x) {
     
    NSLog(@"按钮被点击了");
}];
// 4.代替通知
// 把监听到的通知转换信号
[[[NSNotificationCenter defaultCenter] rac_addObserverForName:UIKeyboardWillShowNotification object:nil] subscribeNext:^(id x) {
    NSLog(@"键盘弹出");
}];
// 5.监听文本框的文字改变
[_textField.rac_textSignal subscribeNext:^(id x) {
     
    NSLog(@"文字改变了%@",x);
}];
// 6.处理多个请求，都返回结果的时候，统一做处理.
RACSignal *request1 = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
     
    // 发送请求1
    [subscriber sendNext:@"发送请求1"];
    return nil;
}];
RACSignal *request2 = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    // 发送请求2
    [subscriber sendNext:@"发送请求2"];
    return nil;
}];
// 使用注意：几个信号，参数一的方法就几个参数，每个参数对应信号发出的数据。
[self rac_liftSelector:@selector(updateUIWithR1:r2:) withSignalsFromArray:@[request1,request2]];
}
// 更新UI
- (void)updateUIWithR1:(id)data r2:(id)data1
{
    NSLog(@"更新UI%@  %@",data,data1);
}

```