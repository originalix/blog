---
title: iOS开发 —— Swift版地址选择器
date: 2016-11-25 10:44:06
tags: ["iOS开发", "地址选择器", "Swift"]

---

已经有二十多天没有更新自己的博客了，这段时间经历了很多事情，离开了生活了六七年的杭州，从离职再入职，忙的是一塌糊涂。

现在这个公司的项目使用了Swift开发，我一直想在自己的项目中也运用Swift，但是一直也没有机会，所以这次能够使用Swift正儿八经的开发，我也是超级兴奋的。

所以从以后开始，我的iOS系列的文章会逐渐的与Swift语言越来越相关。不得不说只有实际开发才能发现Swift中等着我要去踩的坑还有很多。没辙了，爱他就拼命的去填坑吧。

刚入职的第一周写了个简单的页面来熟悉公司项目代码，并且了解下业务。做了一个电商方面相关的收货地址的选择。

今天就来讲讲Swift版本的地址选择器的构建。

<!--more-->

## 构建思路

刚开始领导丢给我了一个数据库包含着中国地区的省市区关系，但是以前处理这个问题常用Plist文件来搞定，所以我也就偷懒懒得再去写Sql语句了，直接用一个Plist文件来处理。

之前OC写的很多省市选择器，都是封装的不够完善，直接调用存在很多问题。并且在处理省市联动的问题上，常常是通过拆分省市区为三个数组，当其中一个数据变化时，再根据 `index`来处理之后的数据联动。

所以这次的类就本着提高复用性的想法，对地址选择界面做了比较全面的封装，在之后的任何地方调用就非常方便。

首先把`UIPickerView`这个类的两个代理方法在自己的类里实现，以后调用的时候不用再去实现`UIPickerView`的两个**Delegate Method**，之后我们再提供一个协议，用最简单的方式来完成数据的获取。

至于省市区的结构，我们用结构体来处理，将省市区写成两个**Struct**，再之后就是简单的数据处理了。将数据加载并且传入这个Struct中。

最后，因为有时候不是省市区三个一起调用，有可能只是单个，或者两个。所以再用枚举声明三种类型，包括了**省、省市、省市区**三种情况，我想这样就可以满足所有情况的使用了。

## 简单调用

贴上一个简单调用的方法吧，最直接的调用，非常的简单。

```swift
class ViewController: UIViewController, LixAreaPickerDelegate {
    @IBOutlet weak var dataLabel: UILabel!
    @IBOutlet weak var subdivisionsPicker: LixAreaPickerView!

    @IBAction func selectPickerType(sender: UISegmentedControl) {
        switch sender.selectedSegmentIndex {
        case 0:
            subdivisionsPicker.pickerType = .Province
        case 1:
            subdivisionsPicker.pickerType = .City
        case 2:
            subdivisionsPicker.pickerType = .District
        default:
            break
        }
    }

    func areaPickerDidUpdate(sender: LixAreaPickerView) {
        dataLabel.text = (subdivisionsPicker.province ?? "") + " " + (subdivisionsPicker.city ?? "") + " " + (subdivisionsPicker.district ?? "")
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        subdivisionsPicker.pickerDelegate = self
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }

}

```

## 源码地址

源码我已经放在我的[Github](https://github.com/originalix/LixAreaPickerView)上面了，欢迎使用。
如果您觉得好用，麻烦Star

谢谢。
