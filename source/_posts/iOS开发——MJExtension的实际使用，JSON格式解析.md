---
title: iOS开发——MJExtension的实际使用，JSON格式解析
date: 2016-05-25 19:40:42
tags: ["iOS开发","MJExtension","JSON解析"]

---

现在的iOS在跟服务器进行交互式，采取的常用数据格式是XML和JSON，而今天就探讨一个JSON解析框架 —— MJExtension。

MJExtension是李明杰老师写的一个开源框架，是一个转换速度快，使用简单方便的字典转模型框架。

<!--more-->

它可以完成的数据转换是有如下方式的:

* `JSON` --> `Model`、`Core Data Model`
* `JSONString` --> `Model`、`Core Data Model`
* `Model`、`Core Data Model` --> `JSON`
* `JSON Array` --> `Model Array`、`Core Data Model Array`
* `JSONString` --> `Model Array`、`Core Data Model Array`
* `Model Array`、`Core Data Model Array` --> `JSON Array`

在MJExtension的GitHub上，文档中已经很清楚的写明了这个框架的简单用法，所以我就不赘述这个框架最基本的使用了。今天我打算举一个很简单的例子，来告诉大家，实际项目中该如何使用MJExtension框架来处理Model数据。

首先我们先来看一个JSON数据格式。

```objc
{
    "group": [
        {
            "roomgroup": [
                {
                    "device_name": "海尔",
                    "address": "客厅电视机",
                    "device_id": "c6f7cd61-245f-46fc-8e81-1e32a0f38cd6",
                    "device_status": "0"
                },
                {
                    "device_name": "乐视",
                    "address": "餐厅电视机",
                    "device_id": "7c756f4a-d3cf-492a-817b-92311f6ea34b",
                    "device_status": "0"
                }
            ],
            "room": "厨房"
        },
        {
            "roomgroup": [
                {
                    "device_name": "海尔",
                    "address": "客厅电视机",
                    "device_id": "784ec8cd-723d-44bc-a6cc-3ae78022e6e9",
                    "device_status": "0"
                }
            ],
            "room": "客厅"
        },
        {
            "roomgroup": [
                {
                    "device_name": "乐视",
                    "address": "餐厅电视机",
                    "device_id": "3d141e71-95ce-4e04-845a-9f0a455d37c2",
                    "device_status": "0"
                }
            ],
            "room": "阳台"
        }
    ],
    "message": {
        "code": "200",
        "message": "操作成功！"
    },
    "rows": null,
    "total": 0
}
```

观察这个JSON数据，我们能发现它里面装着数组属性，而数组中又装着其他模型。这里主要就是要提取`group`这个数组中的数据。

* 那么首先我们定义一个模型，我把它命名为`GroupRoomModel`。

```objc
@property (nonatomic, copy) NSMutableArray *roomGroup;
@property (nonatomic, copy) NSMutableArray *group; 
@property (nonatomic, copy) NSString *room; 
@property (nonatomic, copy) NSMutableArray *deviceDetail;

```

* 接着再定义一个模型，我把它命名为`RoomModel`,用来存储`roomgroup`中的数据

```objc
@property (nonatomic, copy) NSString *deviceName;   
@property (nonatomic, copy) NSString *deviceStatus; 
@property (nonatomic, copy) NSString *deviceID;     
@property (nonatomic, copy) NSString *address;      
```


* 接下来我们要重新映射`Model`中的键值与`JSON`数据中的对应。

```objc
    [GroupRoomModel mj_setupReplacedKeyFromPropertyName:^NSDictionary *{
        return @{
            @"roomGroup":@"roomgroup",
            };
    }];
    
        [RoomModel mj_setupReplacedKeyFromPropertyName:^NSDictionary *{
        return @{
                             @"deviceID":@"device_id",
                             @"deviceName":@"device_name",
                             @"deviceStatus":@"device_status",
                 };
    }];
    
```

* 之后先解析我们拿到的`data`，这里可以直接解析出group

```objc
GroupRoomModel *groupRoomModel = [GroupRoomModel mj_objectWithKeyValues:data];
```

一行代码搞定，是不是很简单。

之后我们解析`group`这个数组中的数据，把`room`中的字符串提取出来存在`Model`的`room`里，把`roomgroup`里的字典分别提取出来，存在`RoomModel`类型的Model里，并且把`RoomModel`添加到`GroupRoomModel`中的`deviceDetail`这个可变数组中。

具体的代码如下

```objc
  NSMutableArray *modelArray = [NSMutableArray array];
    _dataSource = [NSMutableArray array];
    for (NSDictionary *dic in groupRoomModel.group) {
        GroupRoomModel *model = [GroupRoomModel mj_objectWithKeyValues:dic];
        NSMutableArray *deviceArray = [NSMutableArray array];
        for (NSDictionary *deviceDic in model.roomGroup) {
            RoomModel *roomModel = [RoomModel mj_objectWithKeyValues:deviceDic];
            [deviceArray addObject:roomModel];
        }
        [modelArray addObject:model];
        model.deviceDetail = [NSMutableArray arrayWithArray:deviceArray];
        [_dataSource addObject:model];
    }


```

这样我们就完成了多层的JSON数据解析。

其实MJExtension的使用非常简单，多看看文档，很容易掌握。