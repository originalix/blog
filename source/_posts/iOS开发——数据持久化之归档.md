---
title: iOS开发——数据持久化之归档
date: 2016-05-19 20:23:49
tags: ["iOS开发","数据持久化","NSKeyedArchiver","归档"]

---

在实际的项目开发中，数据持久化是我们必须要考虑的一个事情，如何把我们需要的数据进行持久化处理。

在此之前，一些轻量级的数据类型我一般比较喜欢用NSUserDefaults来存储，因为首先这是一个单例，而且使用简便，数据之间传递自由，所以很适合用来保存简单的数据。

而昨天我在实际工作中，有一个数组NSMutableArray需要存储，但是使用时，报了一个错误 `reason: '*** -[NSUserDefaults setObject:forKey:]: attempt to insert non-property list object`，这个报错提示我试图加入一个不支持的数据类型。可是明明NSUserDefaults支持的数据类型有：NSNumber（NSInteger、float、double），NSString，NSDate，NSArray，NSDictionary，BOOL，为什么我的NSArray不能加入呢。后来发现我的数组中的对象，是Model类的对象。

<!--more-->

既然涉及到了Model类的对象，就要考虑用归档（NSKeyedArchiver）来处理它了。NSKeyedArchiver能够存储的数据范围很广，因为它对应着MVC中的Model层，即实体类。在程序中，我们会在Model层定义很多的entity，例如name、room、device等。

那么对自定义对象的归档显得重要的多，因为很多时候我们都要在程序退出时保存数据，再程序恢复时重新加载，那么，归档便是一个好的选择。

接下来我们来讲讲NSKeyedArchiver的使用。

- 要使对象可以归档，对象必须实现NSCoding协议，大部分对象都符合NSCoding的协议，一般我们可以在我们的Model类中实现NSCoding协议。

遵循NSCoding协议，我们需要实现两个方法。

- `- (void)encodeWithCoder:(NSCoder *)aCoder;`

- `- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder;`

- 同时也建议对象实现`NSCoping`协议，该协议允许复制对象，遵循该协议要实现`- (id)copyWithZone:(nullable NSZone *)zone;`方法

下面我们来看归档协议方法的实现

```objc
-(void)encodeWithCoder:(NSCoder *)aCoder{
// 这里放置需要持久化的属性
    [aCoder encodeObject:_deviceType forKey:@"deviceType"];
    [aCoder encodeObject:_roomType forKey:@"roomType"];
    [aCoder encodeObject:_brand forKey:@"brand"];
    [aCoder encodeObject:_version forKey:@"version"];
}
```

接下来是解档的协议方法实现

```objc
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder{
//  这里务必和encodeWithCoder方法里面的内容一致，不然会读不到数据
    if (self = [super init]) {
        self.deviceType = [aDecoder decodeObjectForKey:@"deviceType"];
        self.roomType = [aDecoder decodeObjectForKey:@"roomType"];
        self.brand = [aDecoder decodeObjectForKey:@"brand"];
        self.version = [aDecoder decodeObjectForKey:@"version"];
    }
    return self;
}
```

接下来是遵循了`NSCopying`协议的方法实现

```objc
#pragma mark NSCoping
- (id)copyWithZone:(NSZone *)zone {
    AddDeviceModel *copy = [[[self class] allocWithZone:zone] init];
    copy.deviceType = [self.deviceType copyWithZone:zone];
    copy.roomType = [self.roomType copyWithZone:zone];
    copy.brand = [self.brand copyWithZone:zone];
    copy.version = [self.version copyWithZone:zone];
    return copy;
}
```

- `特别注意` 如果需要归档的类是某个自定义类的子类时，就需要在归档和解档之前先实现父类的归档和解档方法。即 `[super encodeWithCoder:aCoder]` 和`[super initWithCoder:aDecoder]` 方法

`使用` ：

因为之前我提过 我们要存储一个数组，那么我们可以把数组中的数据转化成NSData类型来存储，使用`+ (NSData *)archivedDataWithRootObject:(id)rootObject;`方法。即 ：

```objc
        NSData *data = [NSKeyedArchiver archivedDataWithRootObject:_dataSource];
        [[NSUserDefaults standardUserDefaults] setObject:data forKey:kEditInfraredRepeater];
        [[NSUserDefaults standardUserDefaults]synchronize];

```

三行代码，就把NSKeyedArchiver和NSUserDefaults结合来存储数据了。

而要解档使用数据，只要使用解档`NSKeyedUnarchiver`类中的`+ (nullable id)unarchiveObjectWithData:(NSData *)data;`方法就可以实现

```objc
        NSData *data = [[NSUserDefaults standardUserDefaults] objectForKey:kEditInfraredRepeater];
        NSArray *array = [NSKeyedUnarchiver unarchiveObjectWithData:data];
        _dataSource = [NSMutableArray arrayWithArray:array];

```

同样也是三行代码实现。

以上就是最简单的归档解档数据持久化的实现方式，至于如何用runtime进行自动归档解档，就又需要日后深入研究了。