---
title: iOS开发——系统原生二维码扫描
date: 2016-04-20 21:14:42
tags: ["iOS开发","二维码扫描","QRCode"]

---


对于现在的App应用来说，扫描二维码这个功能是再正常不过的一个功能了，在早期开发这些功能的时候，大家或多或少的都接触过ZXing和ZBar这类的第三方库，但从iOS7以后，苹果就给我们提供了系统原生的API来支持我们扫描获取二维码，ZXing和ZBar在使用中或多或少有不尽如人意的地方，再之停止更新很久了，所以今天我们就来聊聊如何用系统原生的方法扫描获取二维码。

<!--more-->

# 相机权限

众所周知，在使用App扫一扫功能的时候，获取相机权限是第一步要做的事情，而编写代码的时候也是一样，首先我们要判断用户是否已经授权能够访问相机。相机的授权是一组枚举值

- 授权枚举值

```objc
typedef NS_ENUM(NSInteger, AVAuthorizationStatus) {
	AVAuthorizationStatusNotDetermined = 0,   //当前还没有确认是否授权
	AVAuthorizationStatusRestricted,
	AVAuthorizationStatusDenied,
	AVAuthorizationStatusAuthorized
} NS_AVAILABLE_IOS(7_0) __TVOS_PROHIBITED;

```

而这样一组枚举值，首先我们要判断**AVAuthorizationStatusNotDetermined**是否已经授权，而判断授权情况的方法就是 

- 判断授权方法

```objc
AVAuthorizationStatus authorizationStatus = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];

```

- 完整的授权逻辑

```objc
    switch (authorizationStatus) {
        case AVAuthorizationStatusNotDetermined:{
            [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo completionHandler:^(BOOL granted) {
                if (granted) {
                    [self initQRScanView];
                    [self setupCapture];
                }else{
                    NSLog(@"%@",@"访问受限");
                }
            }];
            break;
        }
            
        case AVAuthorizationStatusRestricted:
        case AVAuthorizationStatusDenied: {
            NSLog(@"%@",@"访问受限");
            break;
        }
            
        case AVAuthorizationStatusAuthorized:{
           //获得权限
            break;
        }
        default:
            break;
    }

```

上面这段代码，就是在完成授权方法之后的一段完整的Switch条件判断授权的逻辑代码，而当你获得权限时，可以在里面写下你想要进一步运行的方法。

# 扫码

扫码是使用系统原生的**AVCaptureSession**类来发起的，这个类在官方文档中给出的解释是**AVFundation**框架中**Capture**类的中枢，起到管理协调的作用，而扫码是一个从摄像头（input）到 解析出字符串（output） 的过程，用AVCaptureSession 来协调。其中是通过 AVCaptureConnection 来连接各个 input 和 output，还可以用它来控制 input 和 output 的 数据流向。

- 创建扫描代码

```objc
     dispatch_async(dispatch_get_main_queue(), ^{
             AVCaptureSession * session= [[AVCaptureSession alloc] init];
        AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
        NSError *error;
        AVCaptureDeviceInput *deviceInput = [AVCaptureDeviceInput deviceInputWithDevice:device error:&error];
        if (deviceInput) {
            [session addInput:deviceInput];
            
            
            AVCaptureMetadataOutput *metadataOutput = [[AVCaptureMetadataOutput alloc] init];
            [metadataOutput setMetadataObjectsDelegate:self queue:dispatch_get_main_queue()];
            [session addOutput:metadataOutput];
            
            metadataOutput.metadataObjectTypes = @[AVMetadataObjectTypeQRCode];
            
            AVCaptureVideoPreviewLayer *previewLayer = [[AVCaptureVideoPreviewLayer alloc] initWithSession:session];
            previewLayer.videoGravity = AVLayerVideoGravityResizeAspectFill;
            previewLayer.frame = self.view.frame;
            [self.view.layer insertSublayer:previewLayer atIndex:0];
            
            CGFloat screenHeight = ScreenSize.height;
            CGFloat screenWidth = ScreenSize.width;
            
            self.scanRect = CGRectMake((screenWidth - TransparentArea([QRScanView width], [QRScanView height]).width) / 2,
                                       (screenHeight - TransparentArea([QRScanView width], [QRScanView height]).height) / 2,
                                       TransparentArea([QRScanView width], [QRScanView height]).width,
                                       TransparentArea([QRScanView width], [QRScanView height]).height);
            
            __weak typeof(self) weakSelf = self;
            [[NSNotificationCenter defaultCenter] addObserverForName:AVCaptureInputPortFormatDescriptionDidChangeNotification
                object:nil
                queue:[NSOperationQueue currentQueue]
                usingBlock:^(NSNotification * _Nonnull note) {
                [metadataOutput setRectOfInterest:CGRectMake(
                            weakSelf.scanRect.origin.y / screenHeight,
                            weakSelf.scanRect.origin.x / screenWidth,
                            weakSelf.scanRect.size.height / screenHeight,
                            weakSelf.scanRect.size.width / screenWidth)];
                            //如果不设置 整个屏幕都会扫描
                }];
            [session startRunning];
        }else{
            NSLog(@"error = %@",error);
        }
    });
}
         
```

中间CGRect的设置，是我想根据现在大多数产品的二维码扫描规则，定义个一个框扫描，这个我们在后面也会说到。

# 扫描框

扫码时 previewLayer 的扫描范围是整个可视范围的，但有些需求可能需要指定扫描的区域，虽然我觉得这样很没有必要，因为整个屏幕都可以扫又何必指定到某个框呢？但如果真的需要这么做可以设定 metadataOutput 的 rectOfInterest。

- 设置二维码扫描框  这段代码已经集成在上面的代码中，这里单独列出来只是给大家看一下，若是要复制使用的话，这段可不用复制

```objc
 CGFloat screenHeight = ScreenSize.height;
            CGFloat screenWidth = ScreenSize.width;
            
            self.scanRect = CGRectMake((screenWidth - TransparentArea([QRScanView width], [QRScanView height]).width) / 2,
                                       (screenHeight - TransparentArea([QRScanView width], [QRScanView height]).height) / 2,
                                       TransparentArea([QRScanView width], [QRScanView height]).width,
                                       TransparentArea([QRScanView width], [QRScanView height]).height);
            
            __weak typeof(self) weakSelf = self;
            [[NSNotificationCenter defaultCenter] addObserverForName:AVCaptureInputPortFormatDescriptionDidChangeNotification
                object:nil
                queue:[NSOperationQueue currentQueue]
                usingBlock:^(NSNotification * _Nonnull note) {
                [metadataOutput setRectOfInterest:CGRectMake(
                            weakSelf.scanRect.origin.y / screenHeight,
                            weakSelf.scanRect.origin.x / screenWidth,
                            weakSelf.scanRect.size.height / screenHeight,
                            weakSelf.scanRect.size.width / screenWidth)];
                            //如果不设置 整个屏幕都会扫描
                }];


```

这个**self.scanRect**是我先前定义的一个二维码扫描框的尺寸，而赋值我们在现在已经为他们设定好，现在不管适配什么机型，都会出现在屏幕的中间。

- 获取扫描的值

```objc
#pragma mark - AVCaptureMetadataOutputObjectsDelegate
- (void)captureOutput:(AVCaptureOutput *)captureOutput didOutputMetadataObjects:(NSArray *)metadataObjects fromConnection:(AVCaptureConnection *)connection{
    AVMetadataMachineReadableCodeObject *metadataObject = metadataObjects.firstObject;
    if ([metadataObject.type isEqualToString:AVMetadataObjectTypeQRCode] && !self.isQRCodeCaptured) {
        self.isQRCodeCaptured = YES;
        [self showAlertViewWithMessage:metadataObject.stringValue];
    }
}

```

获取扫描的值，必须要实现上面的这个代理方法，中间的自定义方法可以略去，直接看实现的步骤就好。

# 扫描框的外观

```objc


- (void)drawRect:(CGRect)rect{
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetRGBFillColor(context, 40/255.0, 40/255.0, 40/255.0, .5);
    CGContextFillRect(context, rect);
    NSLog(@"%@", NSStringFromCGSize(TransparentArea([QRScanView width], [QRScanView height])));
    CGRect clearDrawRect = CGRectMake(rect.size.width / 2 - TransparentArea([QRScanView width], [QRScanView height]).width / 2,
                                      rect.size.height / 2 - TransparentArea([QRScanView width], [QRScanView height]).height / 2,
                                      TransparentArea([QRScanView width], [QRScanView height]).width,TransparentArea([QRScanView width], [QRScanView height]).height);
    
    CGContextClearRect(context, clearDrawRect);
    [self addWhiteRect:context rect:clearDrawRect];
    [self addCornerLineWithContext:context rect:clearDrawRect];
}

- (void)addWhiteRect:(CGContextRef)ctx rect:(CGRect)rect {
    CGContextStrokeRect(ctx, rect);
    CGContextSetRGBStrokeColor(ctx, 1, 1, 1, 1);
    CGContextSetLineWidth(ctx, 0.8);
    CGContextAddRect(ctx, rect);
    CGContextStrokePath(ctx);
}

- (void)addCornerLineWithContext:(CGContextRef)ctx rect:(CGRect)rect{
    
    //画四个边角
    CGContextSetLineWidth(ctx, 2);
    CGContextSetRGBStrokeColor(ctx, 83 /255.0, 239/255.0, 111/255.0, 1);//绿色
    
    //左上角
    CGPoint poinsTopLeftA[] = {
        CGPointMake(rect.origin.x+0.7, rect.origin.y),
        CGPointMake(rect.origin.x+0.7 , rect.origin.y + 15)
    };
    CGPoint poinsTopLeftB[] = {CGPointMake(rect.origin.x, rect.origin.y +0.7),CGPointMake(rect.origin.x + 15, rect.origin.y+0.7)};
    [self addLine:poinsTopLeftA pointB:poinsTopLeftB ctx:ctx];
    //左下角
    CGPoint poinsBottomLeftA[] = {CGPointMake(rect.origin.x+ 0.7, rect.origin.y + rect.size.height - 15),CGPointMake(rect.origin.x +0.7,rect.origin.y + rect.size.height)};
    CGPoint poinsBottomLeftB[] = {CGPointMake(rect.origin.x , rect.origin.y + rect.size.height - 0.7) ,CGPointMake(rect.origin.x+0.7 +15, rect.origin.y + rect.size.height - 0.7)};
    [self addLine:poinsBottomLeftA pointB:poinsBottomLeftB ctx:ctx];
    //右上角
    CGPoint poinsTopRightA[] = {CGPointMake(rect.origin.x+ rect.size.width - 15, rect.origin.y+0.7),CGPointMake(rect.origin.x + rect.size.width,rect.origin.y +0.7 )};
    CGPoint poinsTopRightB[] = {CGPointMake(rect.origin.x+ rect.size.width-0.7, rect.origin.y),CGPointMake(rect.origin.x + rect.size.width-0.7,rect.origin.y + 15 +0.7 )};
    [self addLine:poinsTopRightA pointB:poinsTopRightB ctx:ctx];
    
    CGPoint poinsBottomRightA[] = {CGPointMake(rect.origin.x+ rect.size.width -0.7 , rect.origin.y+rect.size.height+ -15),CGPointMake(rect.origin.x-0.7 + rect.size.width,rect.origin.y +rect.size.height )};
    CGPoint poinsBottomRightB[] = {CGPointMake(rect.origin.x+ rect.size.width - 15 , rect.origin.y + rect.size.height-0.7),CGPointMake(rect.origin.x + rect.size.width,rect.origin.y + rect.size.height - 0.7 )};
    [self addLine:poinsBottomRightA pointB:poinsBottomRightB ctx:ctx];
    CGContextStrokePath(ctx);
}

- (void)addLine:(CGPoint[])pointA pointB:(CGPoint[])pointB ctx:(CGContextRef)ctx {
    CGContextAddLines(ctx, pointA, 2);
    CGContextAddLines(ctx, pointB, 2);
}


```

我们对于扫描框是直接采用了复写drawRect的方法来绘制的，包括我们常见的四个边框。

- 二维码扫描线的样式

对于二维码的扫描线，我给定了四种模式

```objc
typedef NS_ENUM(NSInteger, ScanLineMode) {
    ScanLineModeNone, //没有扫描线
    ScanLineModeDeafult, //默认
    ScanLineModeImge,  //以一个图为扫描线 类似一根绿色的线上下扫动
    ScanLineModeGrid, //网格状，类似于支付宝的扫一扫
};

```

所以在我封装的类里，切换不同的模式，可以实现各种二维码扫描的状态。代码稍后会传到GitHub上分享。

至此就已经完成了基本的二维码功能，今天的分享也到这里了。
