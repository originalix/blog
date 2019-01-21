---
title: iOS开发——带有暂停功能的计时器
date: 2016-12-23 07:39:35
tags: ["Swift", "iOS开发", "计时器"]

---

上篇博客我跟大家分享了如何在iOS系统中使用原生框架获取步数，又是大半个月过去了，运动模块的全部功能也总算完成了，也打算有始有终的把如何做一个跑步类App跟大家分享了。

运动类应用中，有一个很重要的模块就是计时器，当然，这个计时器不算复杂，只要有简单的开始、暂停以及复位功能即可。那么今天我们从**Model**层来看看这个计时器的逻辑实现。

我们先自己创建一个时间的**Model**

```swift
class RunningTimer: NSObject {
//MARK: var property
    private var timeLabel: UILabel!
    private var timer: NSTimer?
    //开始和结束时间列表
    lazy private var startTimes = [NSDate]()
    lazy private var endTimes = [NSDate]()
    
    internal var timeNumber = 0 {
        didSet {
            timeString = getTimeStringFromSecond(timeNumber)
        }
    }
    
    internal private(set) var timeString = "00:00:00" {
        didSet {
            timeLabel.text = timeString
        }
    }


}
```

<!--more-->

先从这段声明变量的代码分析开来，首先是定义了一个**timeLabel**，这个变量主要是为了在初始化时，直接将**View**层要显示的**Label**绑定进来，**timer**即为一个计时器，顺便定义了两个数组，用来记录时间，因为在真实环境中，可能有若干次暂停，所以用数组来存储。**timeNumber**即为计时器中的总秒数，用**Swift**的**didSet**特性来监听属性的变化，当秒数发送变化时，讲秒数转化成时间的标准格式，并且赋值给**timeString**，同理，**timeString**也在属性发送变化时，将自己的值赋值给**Label**的**text**属性用以显示。

到这里我们的变量讲解完毕，接着往下看功能的实现。

```swift
    //MARK: - 初始化
    init(timeLabel: UILabel) {
        self.timeLabel = timeLabel
        timeLabel.text = timeString
    }
```

这是这个**Model**的初始化，用意一目了然，传入一个外部**Label**用以显示时间。

```swift
//计时开始
    func timingStart(){
        startTimes.append(NSDate())
        timer = NSTimer.scheduledTimerWithTimeInterval(1, target: self, selector: #selector(self.count), userInfo: nil, repeats: true)
    }
    
    //暂停计时
    func timingPause(){
        endTimes.append(NSDate())
        timer?.invalidate()
    }
    
    //暂停后继续计时
    func timingContinue(){
        timingStart()
    }
    
    //重置Timer
    func resetToStart() {
        startTimes = []
        endTimes = []
        timer?.invalidate()
        timeNumber = 0
    }

```

这里定义了四个方法，对应我们**UI**界面会出现的**Button**功能，**Start**、**Pause**、**Continue**、**resetToStart**。代码很简单，当start时添加当前时间至数组里，并且启动定时器，暂停时，销毁定时器，添加暂停的时间进入暂停数组。继续和重置同理。那么我们来看定时器启动时，对应的**selector**做了哪些事情。

```swift
//MARK: - 计时器
    private func timeCount(){
        if startTimes.count == 1 {
            let currentTime = NSDate()
            timeNumber = Int(CFDateGetTimeIntervalSinceDate(currentTime, startTimes[0]))
        }else{
            if startTimes.count - endTimes.count == 1 {
                endTimes.append(NSDate())
            }
            let index = startTimes.count - 1
            endTimes[index] = NSDate()
            var timeCount = 0
            for startTime in startTimes{
                timeCount += Int(CFDateGetTimeIntervalSinceDate(endTimes[startTimes.indexOf(startTime)!],startTime))
            }
            timeNumber = timeCount
        }
    }
    
    @objc private func count(){
        timeCount()
    }
```

当计时器的`count()`方法运行时、调用`timeCount()`方法。
当我们第一次运行计时器时，获取的秒数就是开始时间与当前时间比对的差值。
而之后，就是跟暂停之后启动时间的对比了。
这里面使用`public func CFDateGetTimeIntervalSinceDate(theDate: CFDate!, _ otherDate: CFDate!) -> CFTimeInterval
`函数获取两个时间之间的时间戳差值。
最后再把前面那个秒数转格式化时间的方法也贴出来吧。

```swift
//从以秒计时的时间里获得表示时间的字符串用于显示
func getTimeStringFromSecond(seconds: Int) -> String {
    
    let secondNumber = seconds % 60
    let minuteNumber = (seconds / 60) % 60
    let hourNumber = (seconds / (60*60)) % 24
    
    let secondText = secondNumber < 10 ? "0\(secondNumber)" : "\(secondNumber)"
    let minuteText = minuteNumber < 10 ? "0\(minuteNumber)" : "\(minuteNumber)"
    let hourText = hourNumber < 10 ? "0\(hourNumber)" : "\(hourNumber)"
    
    return "\(hourText):\(minuteText):\(secondText)"
}
```
