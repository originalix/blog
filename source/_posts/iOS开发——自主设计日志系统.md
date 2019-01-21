---
title: iOS开发——自主设计日志系统
date: 2017-06-28 17:37:01
tags: ["iOS开发", "Cute-Logger", "日志系统"]

---

好像很久没有写有关iOS的文章了，其实iOS的开发一直都是在进行的，但是最近有需求拓宽知识的宽度，所以一直在接触别的知识，当然啦，移动端开发并不能丢下。

我平时开发的项目监测bug和崩溃的模块都是集成了鹅厂的`Bugly`系统，毕竟是谁用谁说好的第三方系统。而`Bugly`主要还是返回的还是崩溃之后的日志，所以如果想在平时的运行中，就能拿到客户手机中的日志怎么办呢。在这个需求的驱使下，便开始着手设计一个日志系统。

需求还是不难的，记录手机操作的内容，如

```
时间|日志级别|类名_函数名_行数|分类|Log内容
```

这样的一种日志形式。

因为不希望频繁的读写，所以希望每十条Log生成之后，读写一次。而未写入硬盘的Log保存在内存中。按照天数，每天都有一份日志，并且在客户的手机异常之后，可以将所有日志压缩上传到服务器。需求介绍完了，并不难对不对。

<!--more-->

在Log的生成方面，我的设计是枚举出日志的级别，之后利用Swift的 `#function` 和 `#line`等定义，方便的获取函数名和行数，类名我是利用一个对于`NSObject`的`extension`来完成的，类似这样：

```swift

extension NSObject {
    var className: String {
        return String(describing: type(of: self)).components(separatedBy: ".").last!
    }

    class var className: String {
        return String(describing: self).components(separatedBy: ".").last!
    }
}

```
开箱可用，准确获取类名。

生成log的核心函数例如如下这样：

```swift
public func createLog(level: DebugLevel, targetClass: AnyClass, type: OperateType, content: String,  _ line: Int = #line, _ function: String = #function)
-> String {
    let lineStr = String.init(format: "line:%d", line)
    let levelStr = levelToString(level: level)
    let separator = "|"
    let classSeparator = "_"
    let log: String = Date().toString() + separator + levelStr + separator + targetClass.className + classSeparator + function + classSeparator + lineStr + separator + content + "\n"
    print(log)
    return log
}
```

而基于这个函数做一些封装，就能封装出很简便的打印各级别日志的API了。

至此，介绍完Log的生成类： `LogGenerator`。

之后是对日志的读写，需要有一个文件读写的类，暂定名为`LogStorage`。

因为文件的读写都是常规的操作，所以代码就不贴出来了。我在这里只是贴出我在`LogStorage`类里暴露的接口方法

```swift

public protocol LogStorageProtocol {

    /// 获取日志缓存地址
    ///
    /// - Returns: String
    func getCachePath() -> String

    /// 删除文件
    ///
    /// - Parameter fileName: String
    /// - Returns: Bool
    func deleteFile(fileName: String) -> Bool

    /// 清除全部日志缓存
    ///
    /// - Returns: Bool
    func cleanCache() -> Bool

    /// 读取日志文件
    ///
    /// - Parameter fileName: String
    /// - Returns: Data
    func readFile(fileName: String) -> Data?

    /// 更新写入Log数据
    ///
    /// - Parameters:
    ///   - fileName: String
    ///   - data: Data
    /// - Returns: Data
    func updateFile(fileName: String, data: Data) -> Bool

    /// 自动根据天数创建文件名
    ///
    /// - Returns: String
    func createFileName() -> String
}
```
而这个十条一写，没有达到标准的就暂时保存在内存里，我的想法是创建一个循环队列，根据FIFO原则，当满足十条Log时，做一次写入操作，而循环队列在空间上是非常节省资源的，如果没有满足十条日志，那就都暂存在队列里，整个开销就是循环队列的一个数组，容量是11个元素，还有一个充当哨兵。

循环队列的数据结构是队列的数据结构里最基础的

```swift
protocol QueueProtocol {
    func createQueue()
    func traverseQueue()
    func isFullQueue() -> Bool
    func isEmptyQueue() -> Bool
    func Enqueue(log: String) -> Bool
    func Dequeue() -> Bool
}
```

在内部增加了一个遍历队列写入日志的函数

```
private func updateFileWhenTranverse() {
    var i = queue.front
    while (i != queue.rear) {
        let fileName = LogStorage.share.createFileName()
        let data = queue.logData[i].data(using: .utf8)
        if (LogStorage.share.updateFile(fileName: fileName, data: data!)) {
            let _ = Dequeue()
        }
        i += 1
        i = i % queue.maxsize
    }
}
```

而最后一个需求就是压缩上传了，在这里使用了`SSZipArchive`这个第三方库来压缩文件成zip格式。封装成`LogArchive`类。是不是三言两语间，整个日志系统就设计完成了，但是我是用`Swift`来写的，若是`Objective-C`调用怎么办呢。答案当然是用Objc强大的宏定义来搞定咯，几个宏定义，轻松的一行代码就调用了Log输出。

```objc
#define Cute_Debug(log) [[LogGenerator new] debugWithTargetClass:self.classForCoder content:[NSString stringWithFormat:@"%@", log] :(NSInteger)__LINE__ :[NSString stringWithFormat:@"%s", __FUNCTION__]];

#define Cute_Warning(log) [[LogGenerator new] warningWithTargetClass:self.classForCoder content:[NSString stringWithFormat:@"%@", log] :(NSInteger)__LINE__ :[NSString stringWithFormat:@"%s", __FUNCTION__]];

#define Cute_Info(log) [[LogGenerator new] infoWithTargetClass:self.classForCoder content:[NSString stringWithFormat:@"%@", log] :(NSInteger)__LINE__ :[NSString stringWithFormat:@"%s", __FUNCTION__]];

#define Cute_Error(log) [[LogGenerator new] errorWithTargetClass:self.classForCoder content:[NSString stringWithFormat:@"%@", log] :(NSInteger)__LINE__ :[NSString stringWithFormat:@"%s", __FUNCTION__]];
```
整个日志系统，我已经开源放在了我的[github](https://github.com/originalix/CuteLogger)上，具体的代码可以去github上看。
欢迎提issue，如果有用，请点个star谢谢。
