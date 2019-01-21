---
title: iOS开发——字典的升序排列以及字符串拼接
date: 2016-04-22 17:03:23
tags: ["iOS开发","字典排序","字符串拼接"]

---

在调用SDK包的时候，很多时候我们自己开发的第三方应用想要取得数据的时候得获得登陆令牌以及其他很多信息，比如accessToken等登陆令牌信息，而第三方SDK包往往会要求我们完成签名算法，我今天在项目中集成海康威视的网络摄像头时，就要求我必须完成签名算法才能获得信息，而签名算法的要求是这样子的。

<!--more-->

- 签名算法要求如下：
- 第一步： 算出“签名原始串”= params中参数集合字符串（将所有字段按升序排列后，依次连接所有字段名及对应值）+ method（接口名字）+ time(UTC时间戳) +请求源secret。样式如下：“<attr>:<val>,…,<attr>:<val>, method:<val>,time:<val>,secret:<val>”
- 第二步： 将“签名原始串”进行MD5校验，并转化为16进制的32位小写字符串，作为签名值sign。（注：编码格式为UTF-8）


所以这里我们分析，我们要完成的步骤如下，首先我们先讲集合内的字符串以升序排列，第二步我们依次按照规定的样式拼接字符串，最后我们把拼接好的字符串进行MD5校验，转化为16进制的32位小写字符串。

下面我们先从字典的升序排列开始说起，我先假定一个字典。

```objc
  NSDictionary *params = @{
                             @"name":@"LinH",
                             @"hometown":@"Dongying",
                             @"userID":@"330909199301271234",
                             @"phone":@"18814868888"
                             };

```

这个参数字典中一共有4个key:name、hometown、userID、phone。
我们该怎么样把这四个字符串按升序排列呢？

首先我们定义一个数组，存储字典中的所有key值:

```objc
NSArray *keyArray = [params allKeys];
```

接下来我们定义一个排序数组，存储排序好之后的key值

```objc
    NSArray *sortArray = [keyArray sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return [obj1 compare:obj2 options:NSNumericSearch];
    }];
```

**sortedArrayUsingComparator**方法是苹果为我们提供的一个数组排序方法，利用block语法来完成排序功能。

而这时，我们排序好的key值，已经按顺序存储在**sortArray**数组中，这时我们再创建一个数组，来按升序存储key对应的Value，通过遍历**sortArray**的方法。

```objc
    NSMutableArray *valueArray = [NSMutableArray array];
    for (NSString *sortString in sortArray) {
        [valueArray addObject:[params objectForKey:sortString]];
    }
```

现在我们有两个数组，分别对应升序排序的key和value，所以再创建一个keyValue的数组来存储每一个key和value的格式。

```objc
    NSMutableArray *signArray = [NSMutableArray array];
    for (int i = 0; i < sortArray.count; i++) {
        NSString *keyValueStr = [NSString stringWithFormat:@"%@:%@",sortArray[i],valueArray[i]];
        [signArray addObject:keyValueStr];
    }
```

最后的一步，就是用**“,”**把每个字符串拼接起来，很简单。

```objc
    NSString *sign = [signArray componentsJoinedByString:@","];
    return sign;
```

这时，字符串sign里存储的就是要求我们完成的，没有进行MD5校验的签名字符串了，可以打印出来看一下。

```objc
[24923:3184830] hometown:Dongying,name:LinH,phone:18814868888,userID:330909199301271234

```

可以看到hometown,name,phone,userID已经按首字母排序并且完成拼接。所以字典的排序我们就讲到这里，MD5加密下一篇再来讲述。