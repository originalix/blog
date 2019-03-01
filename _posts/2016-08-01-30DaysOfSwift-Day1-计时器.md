---
layout: poss
title: 30DaysOfSwift - Day1 计时器
categories: iOS开发
date: 2016-08-01 17:17:49
keywords: iOS开发, 30DaysOfSwift
---


前几天逛Github，偶然看到一个**Swift**的项目 —— **30DaysOfSwift**,作者一共用30个小项目，来熟悉**Swift**语言，而我正好也学习了一段时间的**Swift**语言，准备仿照这样的模式，来更加深入的了解UI部分

今天做的是一个计时器项目，大概效果如下 :

![](https://raw.githubusercontent.com/originalix/Lix30DaysOfSwift/master/Project-01/Simple%20Stop%20Watch.gif)


<!--more-->

作者在这个项目中，使用**AutoLayout**来完成自动布局，使用**StoryBoard**完成UI创建。

而我一直都是喜欢用纯代码布局，UI的搭建也是使用代码完成。所以我在写这个小Demo之前在我的项目里集成了**SnapKit**，使用类似**Objective-C**中常用的**masonry**框架来完成自动布局。

这里我还发现一个**Swift**中的小问题，使用**cocoadPods**集成第三方库，引用不到头文件的解决方法和**Objective-C**不一样。

这是第一个**Swift**小Demo，很简单，也很好的帮助熟悉UI.

```swift

import UIKit
import SnapKit

let SCREEN_WIDTH = UIScreen.mainScreen().bounds.size.width
let SCREEN_HEIGHT = UIScreen.mainScreen().bounds.size.height

let kTopViewHeight = SCREEN_HEIGHT * 0.4 //倒计时试图高度
let kButtonHeight = SCREEN_HEIGHT * 0.6  //开始暂停按钮高度
let kPauseButtonWidth = SCREEN_WIDTH * 0.4 //暂停按钮宽度
let kStartButtonWidth = SCREEN_WIDTH * 0.6 //开始按钮高度

var counter = 0.0
var timer = NSTimer()
var isPlaying = false


class ViewController: UIViewController {

    //MARK: - 懒加载
    
    
    //倒计时Label
    private lazy var showLabel: UILabel = {
        let label = UILabel(frame: CGRect.zero)
        label.text = String(counter)
        label.textColor = UIColor.yellowColor()
        label.font = UIFont.systemFontOfSize(100)
        label.textAlignment = NSTextAlignment.Center
        return label
    }()
    
    //顶部背景试图
    private lazy var topBackgroundView: UIView = {
        let view = UIView(frame: CGRect(x: 0, y: 0, width: SCREEN_WIDTH, height: kTopViewHeight))
        view.backgroundColor = UIColor.blackColor()
        return view
    }()
    
    //Reset按钮
    private lazy var resetButton: UIButton = {
        let button = UIButton(type: (UIButtonType.Custom))
        button.frame = CGRectZero
        button.setTitle("Reset", forState: UIControlState.Normal)
        button.setTitleColor(UIColor.whiteColor(), forState: UIControlState.Normal)
        button.setTitleColor(UIColor.blackColor(), forState: UIControlState.Highlighted)
        button.titleLabel?.font = UIFont.systemFontOfSize(15)
        button.backgroundColor = UIColor.clearColor()
        button.addTarget(self, action: "buttonDidClick:", forControlEvents: UIControlEvents.TouchUpInside)
        button.tag = 101
        return button
    }()
    
    //暂停按钮
    private lazy var pauseButton: UIButton = {
        let button = UIButton(type: (UIButtonType.Custom))
        button.frame = CGRectZero
        button.setImage(UIImage(named: "pause"), forState: UIControlState.Normal)
        button.backgroundColor = UIColor.greenColor()
        button.addTarget(self, action: "buttonDidClick:", forControlEvents: UIControlEvents.TouchUpInside)
        button.tag = 102
        return button
    }()
    
    //开始按钮
    private lazy var startButton: UIButton = {
        let button = UIButton(type: (UIButtonType.Custom))
        button.frame = CGRectZero
        button.setImage(UIImage(named: "play"), forState: UIControlState.Normal)
        button.backgroundColor = UIColor.blueColor()
        button.addTarget(self, action: "buttonDidClick:", forControlEvents: UIControlEvents.TouchUpInside)
        button.tag = 103
        return button
    }()
    
    //MARK: - 创建UI界面
    
    func setupUI() {
        //顶部的背景试图
        self.view.addSubview(self.topBackgroundView)
        // 显示倒计时的Label
        self.topBackgroundView.addSubview(self.showLabel)
        self.showLabel.snp_makeConstraints { make in
            make.width.equalTo(SCREEN_WIDTH)
            make.height.equalTo(137)
            make.centerX.equalTo(self.topBackgroundView.snp_centerX)
            make.centerY.equalTo(self.topBackgroundView.snp_centerY)
        }
        //Reset按钮
        self.topBackgroundView.addSubview(self.resetButton)
        self.resetButton.snp_makeConstraints { make in
            make.top.equalTo(self.topBackgroundView.snp_top).offset(20)
            make.right.equalTo(self.topBackgroundView.snp_right).offset(-20)
            make.height.equalTo(20)
            make.width.equalTo(60)
        }
        // 暂停按钮
        self.view.addSubview(self.pauseButton)
        self.pauseButton.snp_makeConstraints { make in
            make.top.equalTo(self.topBackgroundView.snp_bottom).offset(0)
            make.left.equalTo(self.view).offset(0)
            make.height.equalTo(kButtonHeight)
            make.width.equalTo(kPauseButtonWidth)
        }
        
        //开始按钮
        self.view .addSubview(self.startButton)
        self.startButton.snp_makeConstraints { make in
            make.top.equalTo(self.topBackgroundView.snp_bottom).offset(0)
            make.left.equalTo(self.pauseButton.snp_right).offset(0)
            make.height.equalTo(kButtonHeight)
            make.width.equalTo(kStartButtonWidth)
        }
    }
    //MARK: - 设置状态栏
    override func preferredStatusBarStyle() -> UIStatusBarStyle {
        return UIStatusBarStyle.LightContent
    }
    
    //MARK: - ButtonClick
    func buttonDidClick(sender: AnyObject) {
        switch sender.tag {
            //reset
        case 101 :
            timer.invalidate()
            isPlaying = false
            counter = 0.0
            self.showLabel.text = String(counter)
            self.startButton.enabled = true
            self.pauseButton.enabled = false
            
            //暂停
        case 102 :
            self.startButton.enabled = true
            self.pauseButton.enabled = false
            isPlaying = false
            timer.invalidate()
            
            //开始
        case 103 :
            if (isPlaying) {
                return
            }
            self.startButton.enabled = false
            self.pauseButton.enabled = true
            timer = NSTimer.scheduledTimerWithTimeInterval(0.1, target: self, selector: Selector("updateTimer"), userInfo: nil, repeats: true)
            isPlaying = true
            
        default :
            break
        }
    }
    
    //MARK: - 计时器方法
    func updateTimer() {
        counter = counter + 0.1
        self.showLabel.text = String(format: "%.1f", counter)
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.backgroundColor = UIColor.whiteColor()
        setupUI()
    }
    
    

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }


}

```

[代码已经上传到GitHub上](https://github.com/originalix/Lix30DaysOfSwift/tree/master/Project-01)

