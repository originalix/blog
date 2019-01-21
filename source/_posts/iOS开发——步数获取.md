---
title: iOS开发——步数获取
date: 2016-12-09 07:12:43
tags: ["Swift", "iOS开发", "步数获取"]

---

最近半个月的开发工作，重点一直是类似于悦跑圈、咕咚这样的运动产品的功能，所以在处理iOS设备在运动中的表现也是积累了一些经验。

打算之后的文章，开始把整体的运动功能，分成简单的模块，来介绍一下。那么今天我们就来围绕iOS设备的计步功能，稍微简单的聊一聊。

<!--more-->

大家可能都看过或者知晓`HealthKit`这个框架，但是实际上，一般去研究过这个框架的，都会知道，实时的获取运动数据，并不是用这个框架的，尤其是步数，这个框架如果你在健康中没有开启步数权限的话，是获取不到的。

所以讲到了实时获取运动数据，苹果还提供了另一个框架给我们使用 —— `CoreMotion`框架。在这个框架中，我们可以获取加速度、步数等等等等运动数据，今天我们主要是讲讲步数是怎么获取的。

首先我们要去引用这个框架 `import CoreMotion`。

然后生成两个时间，分别为查询步数的起止时间，`CoreMotion`中会保存七天的运动数据,假设我们生成的时间为`startTime`,`endTime`.

分别对这两个时间进行初始化

```swift
class StepCounter: NSObject {
    
    private var startTime: NSDate?
    private var endTime: NSDate!
    
   //在这里我只是随意初始化， 你可以根据自己具体的时间周期去设置时间
   startTime = NSDate()
   endTime = NSDate()
```

`CoreMotion`框架中，专门有一个类是负责处理步数的，就是`CMPedometer`，所以在这里我们想获取到步数信息，也要创建一个这个对象,并且同时创建一个`int`对象保存步数数据

```swift
private var pedometer: CMPedometer!
lazy private var numberOfSteps = 0
```

接下来 我们来看看具体获取步数的代码。

```swift
    private func getPedonmeterData(){
        pedometer = CMPedometer()
        if CMPedometer.isStepCountingAvailable(){
            if startTime != nil{
                pedometer.queryPedometerDataFromDate(startTime!, toDate: endTime, withHandler: { (data, error) in
                    if error != nil{
                        print("\(error?.localizedDescription)")
                    }else{
                        if data != nil{
                            dispatch_async(dispatch_get_main_queue(), {
                                self.numberOfSteps = Int(data!.numberOfSteps)
                            })
                        }
                    }
                })
            }
        }
    }
```

代码是否简单易懂，先判断该设备是否支持计步功能，若是时间不为空，那么调用`public func queryPedometerDataFromDate(start: NSDate, toDate end: NSDate, withHandler handler: CMPedometerHandler)`函数去查询步数数据，传入的参数有起止时间，之后的操作在闭包中完成，分别判断是否有错误信息以及返回的数据时，就可以轻易的获取到步数。

今天的分享就到这里了，代码非常简短易懂，就不往GitHub上丢了


