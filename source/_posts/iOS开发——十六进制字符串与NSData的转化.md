---
title: iOS开发——十六进制字符串与NSData的转化
date: 2017-07-07 14:58:42
tags: ["iOS开发", "Objective-C"]

---

最近在完成一个需求时，遇到了`NSData`类型转换为十六进制的字符串这个需求的函数，在`stackoverflow`中翻找的时候，给出的答案基本上是如下的:


```objc
NSString* newStr = [[NSString alloc] initWithData:theData encoding:NSUTF8StringEncoding];

// If the data is null-terminated, you should instead use -stringWithUTF8String: to avoid the extra \0 at the end.

NSString* newStr = [NSString stringWithUTF8String:[theData bytes]];

// (Note that if the input is not properly UTF-8-encoded, you will get nil.)

```

<!--more-->

Swift的写法

```swift
let newStr = String(data: data, encoding: .utf8)
// note that `newStr` is a `String?`, not a `String`.
// If the data is null-terminated, you could go though the safe way which is remove the that null character, or the unsafe way similar to the Objective-C version above.

// safe way, provided data is \0-terminated
let newStr1 = String(data: data.subdata(in: 0 ..< data.count - 1), encoding: .utf8)
// unsafe way, provided data is \0-terminated
let newStr2 = data.withUnsafeBytes(String.init(utf8String:))
```

但是在实际的测试中，并不能完成将NSData转换为NSData中存储的十六进制字符串的功能，所以在最终找到答案之后，决定记录下来，以便下次使用可以快速查找。

```objc
- (NSData *)convertHexStrToData:(NSString *)str {
    if (!str || [str length] == 0) {
        return nil;
    }
    
    NSMutableData *hexData = [[NSMutableData alloc] initWithCapacity:8];
    NSRange range;
    if ([str length] % 2 == 0) {
        range = NSMakeRange(0, 2);
    } else {
        range = NSMakeRange(0, 1);
    }
    for (NSInteger i = range.location; i < [str length]; i += 2) {
        unsigned int anInt;
        NSString *hexCharStr = [str substringWithRange:range];
        NSScanner *scanner = [[NSScanner alloc] initWithString:hexCharStr];
        
        [scanner scanHexInt:&anInt];
        NSData *entity = [[NSData alloc] initWithBytes:&anInt length:1];
        [hexData appendData:entity];
        
        range.location += range.length;
        range.length = 2;
    }
    
    NSLog(@"hexdata: %@", hexData);
    return hexData;
}
```
传入参数字符串`@"400"`时，打印出来的是 `hexdata: <0400>`。十六进制的400就是10进制的1024。

```objc
- (NSString *)convertDataToHexStr:(NSData *)data {
    if (!data || [data length] == 0) {
        return @"";
    }
    NSMutableString *string = [[NSMutableString alloc] initWithCapacity:[data length]];
    
    [data enumerateByteRangesUsingBlock:^(const void *bytes, NSRange byteRange, BOOL *stop) {
        unsigned char *dataBytes = (unsigned char*)bytes;
        for (NSInteger i = 0; i < byteRange.length; i++) {
            NSString *hexStr = [NSString stringWithFormat:@"%x", (dataBytes[i]) & 0xff];
            if ([hexStr length] == 2) {
                [string appendString:hexStr];
            } else {
                [string appendFormat:@"0%@", hexStr];
            }
        }
    }];
    
    return string;
}
```
将上一个🌰的NSData作为参数传入时，返回的字符串为`400`。转换完成。
