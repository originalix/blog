---
title: 使用KVC来自定义UISearchBar的外观
date: 2016-04-16 22:18:42
tags: ["iOS开发","UISearchBar","UISearchController"]

---


以前我们在项目中使用搜索框的时候，如果用系统自带的控件则是使用UISearchDisplayController，而自从iOS8之后，系统重新给我们提供了一个搜索控件：UISearchController。在UISearchController中我们无需再自己初始化UISearchBar，只需要提供searchResult展示的视图。然而在开发中，我们往往需要根据项目的风格来改变UISearchBar的外观，通过继承的方式，我们可以完全定制符合项目风格的外观，然而有些情况下我们很难短时间内完成全部的外观定制工作，譬如我们项目用的好几个旧框架，代码中充斥着各种写好的UISearchBar的展示，而改动底层框架并不是一个较好地实践。于是我开始搜索并总结出了几个不通过继承的方式来更改UISearchBar外观的方法。

<!--more-->

#获取子View

我们在UISearchController或者是UISearchDisplayController中都可以直接获取到UISearchBar的实例，我们可以从这里改变一些UISearchBar的属性来改变外观显示。同时我们也可以直接获取UISearchBar的subViews,UISearchBar的subView是一个UIView的实例，这个UIView包含了所有在UISearchBar上可以展示的子视图，iOS SDK提供的UISearchBar，在iOS7之前是分为UISearchBarBackground、UISearchBarTextField、UIButton这几个类的实例组成，而在iOS7之后，是将UIButton转换为了UINavigationButton的实例。

* 我们可以通过循环遍历出UISearchBar上所有展示出来的子视图

```objc
for(UIView*viewin[[[_searchController.searchBar
 subviews]lastObject]subviews] ) {

if([viewisKindOfClass:NSClassFromString(@"UISearchBarBackground")]) {}

if([viewisKindOfClass:NSClassFromString(@"UISearchBarTextField")]) {}

if([viewisKindOfClass:NSClassFromString(@"UINavigationButton")]) {}

}


```

* 通过KVC获取子视图

```objc
UIView*backgroundView = [_searchController.searchBar valueForKey:@"_background"];

UITextField*searchField = [_searchController.searchBar valueForKey:@"_searchField"];

UIButton*cancelButton = [_searchController.searchBar valueForKey:@"_cancelButton"];

```

* 当我们获取cancelButton时，一定要确保cancelButton包含在了UISearchBar中，必要时可以提前调用：

```objc
[_searchController.searchBar setShowsCancelButton:YES animated:NO];

```

* 去掉搜索框背景

```objc
for(UIView*viewin[[[_searchController.searchBar subviews]lastObject]subviews] ) {

if([viewisKindOfClass:NSClassFromString(@"UISearchBarBackground")]) {

[view removeFromSuperview];

    }

}
```

* 去掉搜索框边框

```objc
[_searchController.searchBar setBackgroundImage:[UIImage new]];

```

* 改变输入框文本

```objc
//提示文本颜色

UITextField*searchField = [_searchController.searchBar valueForKey:@"_searchField"];

[searchFieldsetTextColor:[UIColorblackColor]];

[searchFieldsetValue:[UIColorgrayColor]forKeyPath:@"_placeholderLabel.textColor"];

[searchFieldsetFont:[UIFontsystemFontOfSize:14]];

[searchFieldsetBackgroundColor:[UIColorwhiteColor]];

```

* 改变取消按钮的title

```objc
UIButton*cancelButton = [_searchController.searchBar valueForKey:@"_cancelButton"];

[cancelButtonsetTitle:@"Close"forState:UIControlStateNormal];

```

以上就是基于KVC模式来自定义UISearchBar的外观，至于怎样使用UISearchController来搜索，以及谓词的使用，下一篇文章再更新。