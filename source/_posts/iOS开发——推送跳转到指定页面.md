---
title: iOS开发——推送跳转到指定页面
date: 2017-01-24 13:54:23
tags: ["iOS开发", "推送", "组件化"]

---

眨眼2016年就这么走到了结尾，再过两天新年就要开始了。回顾从2016年开始养成的写博客的习惯，一直能延续下来，保持了一整年，还是比较欣慰的一件事情。希望2017年自己的技术能够继续稳步的提升。

今天在这2016年的最后一篇博客里，咱来聊聊推送通知的跳转。

当推送通知到达时，点击推送通知跳转到指定界面，是很多应用都会碰到的一个需求，而要实现这个功能，解决的方法也很多，若是去谷歌搜索，有一个万能跳转的文章可能会进入您的眼帘，但是我实际的去看了这个项目的源码之后，感觉这个库有一定的局限性，用**runtime**实现跳转这不假，但是在请求字段里加上了`ViewController`的类名，这其实就是局限的地方了，毕竟除了服务咱们**iOS**端，你也得考虑考虑安卓端的攻城狮不是，然而如果你换一个约定的字段再来解析，那倒不如用`URL`来作为判断依据更为方便。

<!--more-->

之前的几篇文章，我也在研究**iOS**开发的组件化的架构模式，也有的应用在走组件化的道路上使用了`URL`来跳转界面完成解耦，在实现推送时，我们也能沿用这个思路，用`URL`实现界面的跳转。关于使用哪个`Router`框架，其实真的是萝卜青菜各有所爱，很成熟的 [JLRoutes](https://github.com/joeldev/JLRoutes)、 [routable-ios](https://github.com/clayallsopp/routable-ios)、 [HHRouter](https://github.com/lightory/HHRouter)、 [MGJRouter](https://github.com/mogujie/MGJRouter)，在经过比较已经实际使用之后，我选择了`MGJRouter`这款蘑菇街开源的组件应用到项目中。为什么会选择`MGJRouter`这款组件呢，其实理由就跟他简单的介绍一样，高效、灵活。

来说一说这个基本的使用方式，首先你得跟后台约定推送的参数，比如我在跟后台的约定里，参数名就是`url`，那么我在拿到推送的`userInfo`时，就需要把`url`解析出来。在

```swift
 func application(application: UIApplication, didReceiveRemoteNotification userInfo: [NSObject : AnyObject], fetchCompletionHandler completionHandler: (UIBackgroundFetchResult) -> Void)

 func application(application: UIApplication, didReceiveRemoteNotification userInfo: [NSObject : AnyObject])
```

 这两个方法中，你可以获取到`userInfo`,例如后端给我传了这样的推送消息

 ```json
[aps: {
    alert = "\U6d4b\U8bd5\U7cfb\U7edf\U6d88\U606f";
    badge = 1;
    category = system;
    sound = "";
}, redirect: {
    data = "<null>";
    target = article;
    "target_id" = 5397;
    "target_url" = "lix://cms/articles/3333";
}, _j_msgid: 5228988433]
 ```

 很清楚的看到我们需要拿到`target_url`这个字段，至于怎么解析JSON，我就不啰嗦了，假设此时我们已经拿到了`url`，`url`其实为 `lix://cms/articles/:id`这个格式，`3333`是我们需要根据这个id跳转到的文章界面。

 ```objc
#pragma mark - 推送地址
static NSString *ARTICLES_URL = @"lix://cms/articles/:id";   //资讯详情
 ```
 在定义好`url`的情况下，我们需要先用`MGJRouter`注册我们的`url`，

 ```objc
+ (void)registerArticlePush {
    [MGJRouter registerURLPattern:ARTICLES_URL toHandler:^(NSDictionary *routerParameters) {
       //打印URL
        LixLog(@"routerParameterURL:%@", routerParameters[MGJRouterParameterURL]);
        //获取URL中的id
        NSString *id = routerParameters[@"id"];
        ArticleModel *model = [[ArticleModel alloc] init];
        model._id = id;
        ArticleDetailViewController *articleViewController = [[ArticleDetailViewController alloc] init];
        articleViewController.articleModel = model;
        //界面跳转
        [LixObjcRouter pushController:articleViewController];
    }];
}
 ```

 这段代码可以当成一个完整的业务逻辑的范例，在写好业务逻辑之后，我们需要去`AppDelegate`的` func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool`里注册`url`。

```swift
LixObjcRouter.registerArticlePush()
```

做完这些之后，咱们只要

```swift
 func application(application: UIApplication, didReceiveRemoteNotification userInfo: [NSObject : AnyObject], fetchCompletionHandler completionHandler: (UIBackgroundFetchResult) -> Void)

 func application(application: UIApplication, didReceiveRemoteNotification userInfo: [NSObject : AnyObject])
```
这两个方法里处理推送通知就好了，非常简单的一个`Api`。

```objc
[MGJRouter openURL:url];
```

到这里，推送之后的页面跳转也就差不多完成了，只要再注意`badge`的数值处理，前台时推送通知的处理等情况就可以了。

用完蘑菇街的`Router`组件，又让我想接着啰嗦上次的组件化的思考了，用完这种方式，我还是觉得，如果把这个框架引入进行组件化，那么每次启动，都必须去注册这些`url`，如果小工程也没有组件化的必要，可是大工程，管理`url`又是一件费神的事情。

所以推送跳转，我选择`MGJRouter`,因为足够方便高效，可是组件化的道路，我更看好[CTMediator](https://github.com/casatwy/CTMediator)这样彻底解耦的方式。感兴趣的自己搜索研究吧。

年前最后一篇，新年快乐，鸡年大吉吧。


