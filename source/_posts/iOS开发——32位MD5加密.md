---
title: iOS开发——32位MD5加密
date: 2016-04-24 16:12:16
tags: ["iOS开发","MD5加密"]

---



在IOS开发过程中，为了保证数据的安全，我们通常要采取一些加密方法，常见的加密有Base64加密和MD5加密。Base64加密是可逆的，MD5加密目前来说一般是不可逆的。我们在开发一款App过程中，对于发的请求，其中有个“sign”的字段，这个key对应的value是MD5加密的字段，旁边的安卓同事问php后台，说MD5加密是32位的还是16位的，由于以前未曾注意过，所以就搜索了下，现在稍微总结下：
 
<!--more-->

MD5即Message-Digest Algorithm 5（信息-摘要算法5），用于确保信息传输完整一致。是计算机广泛使用的杂凑算法之一（又译摘要算法、哈希算法），主流编程语言普遍已有MD5实现。MD5的作用是让大容量信息在用数字签名软件签署私人密钥前被"压缩"成一种保密的格式（就是把一个任意长度的字节串变换成一定长的十六进制数字串）。（引用自百度百科）

 

注意生成“一定长”，这个“一定长”到底是多长呢！看了好多资料，包括维基百科和一些论坛，说MD5其实进过算法产生的是固定的128bit，即128个0和1的二进制位，而在实际应用开发中，通常是以16进制输出的，所以正好就是32位的16进制，说白了也就是32个16进制的数字。

下面就来说说在Objective-C中我们怎么进行MD5加密，首先我们需要包含头文件是

```objc
#import <CommonCrypto/CommonDigest.h>
```

接下来，我们有两种方式来生成32位的16进制小写的md5串，下面直接贴代码。

```objc
/**
 *  32位md5加密算法
 *
 *  @param str 传入要加密的字符串
 *
 *  @return NSString
 */
- (NSString *)md5:(NSString *)str
{
    const char *cStr = [str UTF8String];
    unsigned char result[16];
    CC_MD5(cStr, strlen(cStr), result); // This is the md5 call
    return [NSString stringWithFormat:@"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x",
            result[0], result[1], result[2], result[3],
            result[4], result[5], result[6], result[7],
            result[8], result[9], result[10], result[11],
            result[12], result[13], result[14], result[15]
            ];
}

```

而另一种方法:

```objc
- (NSString *)md5HexDigest:(NSString*)password
{
    const char *original_str = [password UTF8String];
    unsigned char result[CC_MD5_DIGEST_LENGTH];
    CC_MD5(original_str, strlen(original_str), result);
    NSMutableString *hash = [NSMutableString string];
    for (int i = 0; i < CC_MD5_DIGEST_LENGTH; i++)
    {
        /*
         %02X是格式控制符：‘x’表示以16进制输出，‘02’表示不足两位，前面补0；
         */
        [hash appendFormat:@"%02X", result[i]];
    }
    NSString *mdfiveString = [hash lowercaseString];
    return mdfiveString;
}

```

其中%02x是格式控制符：‘x’表示以16进制输出，‘02’表示不足两位，前面补0；如‘f’输出为0f，‘1f3’则输出1f3;本来一般的都会介绍到这里就完了，我想多介绍一下代码中result是个字符数组，那为什么是[16]呢，这是因为MD5算法最后生成的是128位，而在计算机的最小存储单位为字节，1个字节是8位，对应一个char类型，计算可得需要16个char。所以result是[16]。那么为什么输出的格式一定是%02x呢，而不是其它呢。这也是有原因的：因为约定MD5一般是以16进制的格式输出，那么其实这个问题就转换为把128个0和1以16进制来表示，每4位二进制对应一个16进制的元素，则需要32个16进制的元素，如果元素全部为0，放到char的数组中，正常是不会输出，如00001111，以%x输出，则是f,那么就会丢失0；但如果以%02x表示则输出结果是0f，正好是转换的正确结果。

 

所以以上就是char[16]和%02x的来历。

 

至于人们说的16位MD5加密，其实是这样的：举例如果产生的MD5加密字符串是：01234567abcdefababcdefab76543210，则16位的MD加密字符是abcdefababcdefab，也就是只是截取了中间的16位。实际上这个操作已经不是MD5加密算法所包括的，而应当是对MD5加密算法结果的二次处理。其它的64位和大小写什么的，都属于对MD5算法结果的二次处理。因为MD5算法产生的结果就是128bit，128个二进制数字。