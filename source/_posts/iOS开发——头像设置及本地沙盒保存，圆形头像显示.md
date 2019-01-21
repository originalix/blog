---
title: iOS开发——头像设置及本地沙盒保存，圆形头像显示
date: 2016-07-19 15:03:44
tags: ["iOS开发","头像设置","圆形头像"]

---

今天来讲一讲iOS实际开发中，对于头像的应用。

现在的APP中，对于头像的设置，我们大多采用圆形头像，并且需要支持从照相机获取或者从相册中选择用户需要的头像，并且保存在本地或者服务器中。

<!--more-->


在设置完头像之后，后期如果用户想查看头像，一般有设置手势，点击将头像按我们的设想放大。这个功能，我计划放在后面的一篇文章里讲。

本文主要讲解对于头像的设置，圆形头像的设置、并且头像的本地获取已经本地化保存。

因为头像的唯一性，所以我想大家都会考虑在头像中使用单例设计模式。这里我们把头像定义为 `HeadsPicture` 类。

类中我们的代码可以定义如下:

```objc
#import <UIKit/UIKit.h>
#import <Foundation/Foundation.h>

@interface HeadsPicture : NSObject

+(instancetype)sharedHeadsPicture;
/**
 *  设置头像
 *
 *  @param image 图片
 */
-(void)setImage:(UIImage *)image forKey:(NSString *)key;

/**
 *  读取图片
 *
 */
-(UIImage *)imageForKey:(NSString *)key;

@end


```

我们在类中 使用了 `sharedHeadsPicture` 这个单例方法,也定义了一个读取头像图片、以及存储头像图片的方法。暂时我还是把代码保存到了沙盒文件里，代码中大家也可以很方便的把存储在服务器里的头像图片集成进来。

在 `HeadsPicture.m` 中，代码如下。

```objc
#import "HeadsPicture.h"

@interface HeadsPicture()

@property (nonatomic, strong) NSMutableDictionary *dictionary;

-(NSString *)imagePathForKey:(NSString *)key;

@end

@implementation HeadsPicture

+(instancetype)sharedHeadsPicture{
    
    static HeadsPicture *instance = nil;
    //确保多线程中只创建一次对象,线程安全的单例
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[self alloc] initPrivate];
    });
    return instance;
}

-(instancetype)initPrivate{
    
    self = [super init];
    if (self) {
        
        _dictionary = [[NSMutableDictionary alloc] init];
        //注册为低内存通知的观察者
        NSNotificationCenter *nc = [NSNotificationCenter defaultCenter];
        [nc addObserver:self
               selector:@selector(clearCaches:)
                   name:UIApplicationDidReceiveMemoryWarningNotification
                 object:nil];
    }
    return self;
}

-(void)setImage:(UIImage *)image forKey:(NSString *)key{
    
    [self.dictionary setObject:image forKey:key];
    //获取保存图片的全路径
    NSString *path = [self imagePathForKey:key];
    //从图片提取JPEG格式的数据,第二个参数为图片压缩参数
    NSData *data = UIImageJPEGRepresentation(image, 0.5);
    //以PNG格式提取图片数据
    //NSData *data = UIImagePNGRepresentation(image);
    
    //将图片数据写入文件
    [data writeToFile:path atomically:YES];
}

-(UIImage *)imageForKey:(NSString *)key{
    //return [self.dictionary objectForKey:key];
    UIImage *image = [self.dictionary objectForKey:key];
    if (!image) {
        NSString *path = [self imagePathForKey:key];
        image = [UIImage imageWithContentsOfFile:path];
        if (image) {
            
            [self.dictionary setObject:image forKey:key];
        }else{
            
            NSLog(@"Error: unable to find %@", [self imagePathForKey:key]);
        }
    }
    return image;
}

-(NSString *)imagePathForKey:(NSString *)key{
    
    NSArray *documentDirectories = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentDirectory = [documentDirectories firstObject];
    return [documentDirectory stringByAppendingPathComponent:key];
}

-(void)clearCaches:(NSNotification *)n{
    
    NSLog(@"Flushing %ld images out of the cache", (unsigned long)[self.dictionary count]);
    [self.dictionary removeAllObjects];
}
@end

```

在上面，我们已经完成了头像的设置与读取。

回到界面上，我们先定义一个头像显示的试图。

`@property (weak, nonatomic) IBOutlet UIImageView *avatarImage;`


```objc
/**
 *  设置圆形头像属性
 */
- (void)setCirclePhoto{
  [self.avatarImage.layer setCornerRadius:CGRectGetHeight([self.avatarImage bounds]) / 2];
  self.avatarImage.layer.masksToBounds = true;
  //可以根据需求设置边框宽度、颜色
  self.avatarImage.layer.borderWidth = 1;
  self.avatarImage.layer.borderColor = [[UIColor blackColor] CGColor];
  //设置图片；
  self.avatarImage.layer.contents = (id)[[UIImage imageNamed:@"avatar.png"] CGImage];
    self.avatarImage.userInteractionEnabled = YES;
}

```

之后完成圆形头像的属性设置。

最后来写 `设置头像` 按钮背后的选择照片的逻辑代码。

因为是从 `照相机` 或者 `相册` 中来读取照片，需要使用 `UIImagePickerController"图像选择器" ` 。

`UIImagePickerController` 是一种导航控制器，使用它，用户可以打开系统的图片选取器或者打开相机进行拍照。实现协议 `UIImagePickerDelegate`中定义的委托方法可以对选定后的结果进行操作，或是没有选择取消的操作。 

具体代码如下:

- 首先我们先要确定、用户需要使用相册还是摄像头来直接拍摄头像。

```objc
- (IBAction)selectPhoto:(id)sender {

  UIImagePickerController *imagePicker = [[UIImagePickerController alloc] init];
  imagePicker.editing = YES;
  imagePicker.delegate = self;
  /*
   如果这里allowsEditing设置为false，则下面的UIImage *image = [info valueForKey:UIImagePickerControllerEditedImage];
   应该改为： UIImage *image = [info valueForKey:UIImagePickerControllerOriginalImage];
   也就是改为原图像，而不是编辑后的图像。
   */
  //允许编辑图片
  imagePicker.allowsEditing = YES;

  /*
   这里以弹出选择框的形式让用户选择是打开照相机还是图库
   */
  //初始化提示框；
  UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"请选择打开方式" message:nil preferredStyle:  UIAlertControllerStyleActionSheet];
  [alert addAction:[UIAlertAction actionWithTitle:@"照相机" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {

    imagePicker.sourceType = UIImagePickerControllerSourceTypeCamera;

    [self presentViewController:imagePicker animated:YES completion:nil];

  }]];

  [alert addAction:[UIAlertAction actionWithTitle:@"相册" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
    imagePicker.sourceType = UIImagePickerControllerSourceTypePhotoLibrary;
    [self presentViewController:imagePicker animated:YES completion:nil];
  }]];
    
  [alert addAction:[UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleDestructive handler:^(UIAlertAction * _Nonnull action) {
    //取消；
  }]];
  //弹出提示框；
  [self presentViewController:alert animated:true completion:nil];
}

```

- 之后实现 实现协议 `UIImagePickerDelegate`中定义的委托方法可以对选定后的结果进行操作。

```objc

-(void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary<NSString *,id> *)info{
  //通过info字典获取选择的照片
  UIImage *image = [info valueForKey:UIImagePickerControllerEditedImage];
  //以itemKey为键，将照片存入ImageStore对象中
  [[HeadsPicture sharedHeadsPicture] setImage:image forKey:@"HeadsPicture"];
  //将照片放入UIImageView对象
  self.avatarImage.image = image;
  //把一张照片保存到图库中，此时无论是这张照片是照相机拍的还是本身从图库中取出的，都会保存到图库中；
  UIImageWriteToSavedPhotosAlbum(image, self, nil, nil);
  //压缩图片,如果图片要上传到服务器或者网络，则需要执行该步骤（压缩），第二个参数是压缩比例，转化为NSData类型；
  NSData *fileData = UIImageJPEGRepresentation(image, 1.0);
     //关闭以模态形式显示的UIImagePickerController
  [self dismissViewControllerAnimated:YES completion:nil];

}

``` 

至此，我们已经完成了头像的设置和本地的沙盒保存，以及圆形头像的显示。

至于头像的触摸放大等手势功能，等下篇文章再来讲解。