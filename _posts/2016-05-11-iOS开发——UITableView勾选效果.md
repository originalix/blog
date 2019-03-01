---
layout: post
title: iOS开发——UITableView勾选效果
categories: iOS开发
date: 2016-05-11 22:05:31
keywords: iOS开发, UITableView勾选
---

今天整理了一下项目中常用的一些控件功能，整理一些UI效果来总结一下。

如今的APP开发中，`UITableView`是最常用的控件之一，而`UITableView`中有个很常见的效果就是勾选效果，这个效果是由`UITableViewCell`中的`accessoryType`属性来决定的。


`accessoryType`中的变量是一个枚举值`UITableViewCellAccessoryType`，让我们来看一下其中包含的东西。

```objc
typedef NS_ENUM(NSInteger, UITableViewCellAccessoryType) {
    UITableViewCellAccessoryNone,                                                      // 不显示任何效果
    UITableViewCellAccessoryDisclosureIndicator,                                       // 常用的V形引导图标
    UITableViewCellAccessoryDetailDisclosureButton //信息提示+V形引导图标
    UITableViewCellAccessoryCheckmark,                                                 // 勾选效果
    UITableViewCellAccessoryDetailButton //信息图标
};

```

这几个效果我已经罗列在上面了，今天我们要探讨的，就是`UITableViewCellAccessoryCheckmark`勾选效果。我们要实现的，就是单选一个列表中的信息。

<!--more-->

有以下几个注意点：

- 首先在`- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath` 方法中实现判断被选中的单元格的功能。记录下之前选择的单元格，并且实时更新。

- 其次，解决单元格的复用问题。不然当单元格复用时，会显示多个勾选的BUG。看了一下网上分享的很多的方法，都没有解决单元格复用的问题，或者问的很笼统。

好了，解释的差不多，下面来上代码。

首先我们先声明一个变量，用来存储被选择的行数的标志

```objc
@property (nonatomic, strong) NSIndexPath *selectPath; //存放被点击的哪一行的标志

```

之后我们实现`- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath`这个代理方法

```objc
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
    int newRow = (int)[indexPath row];
    int oldRow = (int)(_selectPath != nil) ? (int)[_selectPath row]:-1;
    if (newRow != oldRow) {
        UITableViewCell *newCell = [tableView cellForRowAtIndexPath:indexPath];
        newCell.accessoryType = UITableViewCellAccessoryCheckmark;
        
         UITableViewCell *oldCell = [tableView cellForRowAtIndexPath:_selectPath];
        oldCell.accessoryType = UITableViewCellAccessoryNone;
        _selectPath = [indexPath copy];
    }
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
}

```

最后看一下怎么在`- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath`中添加一段代码，解决复用问题

```objc
if (_selectPath == indexPath) {
        cell.accessoryType = UITableViewCellAccessoryCheckmark;
    }else{
        cell.accessoryType = UITableViewCellAccessoryNone;
    }
    cell.roomType = _dataSource[indexPath.row];

```

至此，单选效果就已经完成，并且不会有单元格复用的问题，代码很简单，也不想过多解释了,不清楚的可以接着问.