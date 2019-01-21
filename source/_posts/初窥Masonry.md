---
title: 初窥Masonry
date: 2016-03-24 18:01:23
tags: ["Masonry","iOS","AutoLayout","自动布局"]

---


在早期，iPhone尺寸比较固定，都是4英寸屏幕的时候，在计算App的尺寸时，只要稍微根据Window的size稍微计算一下就可以了，但是前年iPhone6以及iPhone6Plus的推出，作为开发者就会警觉为了多机型的适配，使用AutoLayout是势在必行的一件事情了，但是说实话，我在用了AutoLayout之后真的觉得挺不方便的。

一直以来可能是一个coder的矫情情怀，我喜欢用纯代码来搭建界面，因为那样思路清晰，而且日后维护的时候也能很清楚的知道问题究竟出在哪里。于是，一个第三方框架Masonry就自然而然的进入了视线，Masonry是一个轻量级的布局框架，拥有自己的描述语法，采用更优雅的链式语法来封装自动布局，简洁明了，具有高的可读性。

于是我今天就从Masonry最简单的demo入手，来学习Masonry的使用。<!--more-->


### 居中显示一个View

```objc

 UIView *view = [[UIView alloc] init];
    view.backgroundColor = [UIColor grayColor];
    [self addSubview:view];
    
    [view mas_makeConstraints:^(MASConstraintMaker *make) {
        make.center.equalTo(self);
        make.size.mas_equalTo(CGSizeMake(300 , 300));
    }];

```

运行结果:

![居中显示View](https://raw.githubusercontent.com/originalix/MasonryDemo/master/basic.png)

我们可以看到View已经按我们的要求设置成了预期大小，并且居中，那么我们回过头来看看代码。

```objc

//因为Masonry有设置尺寸的功能，以后基本能抛弃initWithFrame方法了
 UIView *view = [[UIView alloc] init];
    view.backgroundColor = [UIColor grayColor];

//这里是个重点，要记得设置autoLayout之前一定要把view添加到父视图上，不然会报错
    [self addSubview:view];
    
//mas_makConstraints就是Masnory的添加AutoLayout函数了 把内容加入到中间的block块中就好了
    [view mas_makeConstraints:^(MASConstraintMaker *make) {

//将View居中到父视图上 很好理解吧
        make.center.equalTo(self);

//将size设置成300*300
        make.size.mas_equalTo(CGSizeMake(300 , 300));
    }];

```

看完上面的这段注释后的代码，一定觉得很简单对不对。

这里要注意的一点就是Masonry中能够添加AutoLayout的一共有三个函数。

```objc
- (NSArray *)mas_makeConstraints:(void(^(MASConstraintMaker *make))block;
- (NSArray *)mas_updateConstraints:(void(^(MASConstraintMaker *make))block;
- (NSArray*) mas_remakeConstraints(void(^(MASConstraintMaker *make))block;


/* mas_makeConstraints 只负责新增约束 Autolayout不能同时存在两条针对于同一对象的约束 否则会报错  

   mas_updateConstraints 针对上面的情况 会更新在block中出现的约束 不会导致出现两个相同约束的情况

   mas_remakeConstraints 则会清除之前的所有约束 仅保留最新的约束  三种函数善加利用 就可以应对各种情况了
*/

```

### 让一个View略小于SuperView

这里我们假定让一个View小于它的SuperView每个边界的距离都是10，那么代码可以这么写

```objc

    [view mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(self).with.insets(UIEdgeInsetsMake(10, 10, 10, 10));
        
        //等价于
        make.top.equalTo(self).with.offset(10);
        make.left.equalTo(self).with.offset(10);
        make.right.equalTo(self).with.offset(-10);
        make.bottom.equalTo(self).with.offset(-10);
        
        
        //也等价于
        make.top.left.bottom.and.right.equalTo(self).with.insets(UIEdgeInsetsMake(10, 10, 10, 10));
        
    }];


```

 上面的例子给出了三种实现方式，他们能实现的效果是相同的。我个人的意见是使用第一种，毕竟一句话能完成的代码何必用四句话呢。

那么为什么bottom和right里的offset是负数呢？因为这里的计算是绝对的数值，计算bottom需要小于superView的高度，所以要-10，同理用于right。


### 让两个高度为150的view垂直居中且等宽等间隔排列 间隔为10（自动计算其宽度）

```objc

    int padding1 = 10;
    
    UIView *view1 = [[UIView alloc]init];
    
    view1.backgroundColor = [UIColor blackColor];
    
    [self addSubview:view1];
    
    UIView * view2 = [[UIView alloc]init];
    view2.backgroundColor = [UIColor grayColor];
    [self addSubview:view2];
    
    //让两个高度为150的View垂直居中且等宽且等间隔排列 间隔为10（自动计算其宽度）
    [view1 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.centerY.mas_equalTo(self.mas_centerY);
        make.left.equalTo(self.mas_left).with.offset(padding1);
        make.right.equalTo(view2.mas_left).with.offset(-padding1);
        make.height.mas_equalTo(@150);
        make.width.equalTo(view2);
    }];
    
    [view2 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.centerY.mas_equalTo(self.mas_centerY);
        make.left.equalTo(view1.mas_right).with.offset(padding1);
        make.right.equalTo(self.mas_right).with.offset(-padding1);
        make.height.mas_equalTo(@150);
        make.width.equalTo(view1);
    }];

```

效果如图:

![两个View等间隔排列](https://raw.githubusercontent.com/originalix/MasonryDemo/master/padding.png)

这里我们在两个子View之间相互约束，可以看到他们的宽度在约束下被计算出来。

### 在UIScrollView顺序排列一些View并自动计算contentSize

```objc

//在UIScrollView顺序排列一些View并自动计算contentSize
    UIScrollView *scrollView = [[UIScrollView alloc]init];
    [self addSubview:scrollView];
    [scrollView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(self).with.insets(UIEdgeInsetsMake(10, 10, 10, 10));
    }];
    
    UIView *container = [[UIView alloc]init];
    [scrollView addSubview:container];
    
    [container mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(scrollView);
        make.width.equalTo(scrollView);
    }];
    
    int count = 10;
    
    UIView *lastView = nil;
    
    for (int i = 1; i <= count; ++i) {
        UIView *subView = [[UIView alloc]init];
        [container addSubview:subView];
        subView.backgroundColor = [UIColor colorWithHue:(arc4random()%256 / 256.0)
                                             saturation:(arc4random()%128 / 256.0) + 0.5
                                             brightness:(arc4random()%128 / 256.0) + 0.5
                                                  alpha:1];
        
        [subView mas_makeConstraints:^(MASConstraintMaker *make) {
            make.left.and.right.equalTo(container);
            make.height.mas_equalTo(@(20*i));
            
            if (lastView) {
                make.top.mas_equalTo(lastView.mas_bottom);
            }else{
                make.top.mas_equalTo(container.mas_top);
            }
        }];
        
        lastView = subView;
    }
    
    [container mas_makeConstraints:^(MASConstraintMaker *make) {
        make.bottom.equalTo(lastView.mas_bottom);
    }];

```

效果图:

![ScrollView1](https://raw.githubusercontent.com/originalix/MasonryDemo/master/ScrollView1.png)

![ScrollView2](https://raw.githubusercontent.com/originalix/MasonryDemo/master/ScrollView2.png)

从scrollView的scrollIndicator可以看出 scrollView的内部已如我们所想排列好了

这里的关键就在于container这个view起到了一个中间层的作用 能够自动的计算UIScrollView的contentSize

### 横向或者纵向排列等间隙的一组view

Masonry并没有向我们提供这样的方法，所以为了等间隙的排列，我们首先对UIView类扩展一个类目

```objc

@implementation UIView (Masonry_Lix)

- (void)distributeSpacingHorizontallyWith:(NSArray *)views
{
    NSMutableArray *spaces = [NSMutableArray arrayWithCapacity:views.count+1];
    
    for (int i = 0; i < views.count+1; i++) {
        
        UIView *view = [[UIView alloc] init];
        [spaces addObject:view];
        [self addSubview:view];
        
        [view mas_makeConstraints:^(MASConstraintMaker *make) {
            make.width.equalTo(view.mas_height);
        }];
    }
    
    UIView * view0 = spaces[0];
    
    [view0 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.mas_left);
        make.centerY.equalTo(((UIView*)views[0]).mas_centerY);
    }];
    
    UIView *lastSpace = view0;
    for (int i = 0; i < views.count; i++) {
        UIView *obj = views[i];
        UIView *space = spaces[i+1];
        
        [obj mas_makeConstraints:^(MASConstraintMaker *make) {
            make.left.equalTo(lastSpace.mas_right);
        }];
        
        [space mas_makeConstraints:^(MASConstraintMaker *make) {
            make.left.equalTo(obj.mas_right);
            make.centerY.equalTo(obj.mas_centerY);
            make.width.equalTo(view0);
        }];
        
        lastSpace = space;
    }
    
    [lastSpace mas_makeConstraints:^(MASConstraintMaker *make) {
        make.right.equalTo(self.mas_right);
    }];
    
}

- (void)distributeSpacingVerticallyWith:(NSArray *)views
{
    NSMutableArray *spaces = [NSMutableArray arrayWithCapacity:views.count+1];
    
    for (int i = 0; i < views.count+1; i++) {
        UIView * view = [[UIView alloc] init];
        [spaces addObject:view];
        [self addSubview:view];
        
        [view mas_makeConstraints:^(MASConstraintMaker *make) {
            make.width.equalTo(view.mas_height);
        }];
        
    }
    
    UIView *view0 = spaces[0];
    
    [view0 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(self.mas_top);
        make.centerX.equalTo(((UIView *)views[0]).mas_centerX);
    }];
    
    UIView *lastSpace = view0;
    for (int i = 0; i < views.count; i++) {
        UIView *obj = [[UIView alloc] init];
        UIView *space = spaces[i+1];
        
        [obj mas_makeConstraints:^(MASConstraintMaker *make) {
            make.top.equalTo(lastSpace.mas_bottom);
        }];
        
        [space mas_makeConstraints:^(MASConstraintMaker *make) {
            make.top.equalTo(obj.mas_bottom);
            make.centerX.equalTo(obj.mas_centerX);
            make.height.equalTo(view0);
        }];
        
        lastSpace = space;
    }
    
    [lastSpace mas_makeConstraints:^(MASConstraintMaker *make) {
        make.bottom.equalTo(self.mas_bottom);
    }];
    
}
@end

```

category写好之后 我们来随便写几个view测试一下

```objc


    UIView *sv1 = [[UIView alloc] init];
    UIView *sv2 = [[UIView alloc] init];
    UIView *sv3 = [[UIView alloc] init];
    UIView *sv14 = [[UIView alloc] init];
    UIView *sv15 = [[UIView alloc] init];
    
    
    sv1.backgroundColor = [UIColor redColor];
    sv2.backgroundColor = [UIColor redColor];
    sv3.backgroundColor = [UIColor redColor];
    sv14.backgroundColor = [UIColor redColor];
    sv15.backgroundColor = [UIColor redColor];
    
    [self addSubview:sv1];
    [self addSubview:sv2];
    [self addSubview:sv3];
    [self addSubview:sv14];
    [self addSubview:sv15];
    
    
    //给予每个视图不同大小 测试效果
    
    [sv1 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.centerY.equalTo(@[sv2,sv3]);
        make.centerX.equalTo(@[sv14,sv15]);
        make.size.mas_equalTo(CGSizeMake(40, 40));
    }];
    
    [sv2 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.size.mas_equalTo(CGSizeMake(70, 20));
    }];
    
    [sv3 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.size.mas_equalTo(CGSizeMake(50, 50));
    }];
    
    [sv14 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.size.mas_equalTo(CGSizeMake(50, 20));
    }];
    
    [sv15 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.size.mas_equalTo(CGSizeMake(40, 70));
    }];
    
    [self distributeSpacingHorizontallyWith:@[sv1,sv2,sv3]];
    
    [self distributeSpacingVerticallyWith:@[sv1,sv14,sv15]];
    
```

效果图：

![view等间隙排列](https://raw.githubusercontent.com/originalix/MasonryDemo/master/spaceView.png)

在这个效果图上我们已经能看到我们写的view按照我们设定的水平或者垂直方向排列了，所以这个类目也可以保留下来以后使用。

### 小结

通过上面5个demo的学习，我感觉已经把masonry常用的操作搞清楚了，如果你觉得还不清楚 也可以[在这里下载](https://github.com/originalix/MasonryDemo) Demo来学习。

总而言之Masonry是一个非常优秀的AutoLayout库，能够节省大量的开发时间，适合我这种喜欢纯代码的iOSer。