---
layout: post
title: Python——爬虫入门Selenium的简单使用
categories: 后端技术
date: 2018-02-22 07:34:29
keywords: Python, 爬虫, Selenium
---

之前的两篇我们讲解了Python内的urllib库的使用，不知道大家有没有在爬取一些动态网站的时候，发现自己用urllib爬取到的内容是不对的，无法抓取到自己想要的内容，比如淘宝的店铺宝贝等，它会用js动态的加载内容，此时selenium这个家伙就能派上用场了。

selenium是什么？简单的概括，它的初衷就是自动化测试工具。它支持各种浏览器，包括chrome，safari，firefox等主流界面式浏览器，如果你在这些浏览器里安装一个selenium的插件，那么便可以方便的实现Web界面的测试。换句话说selenium支持这些浏览器驱动，selenium支持多种语言开发，比如Python、Java、C、Ruby等等。

<!--more-->

而在爬虫这个领域，我们则用这个自动化测试工具来模拟我们是真实的浏览器用户，用他来爬取页面非常方便，只要按照访问步骤模拟人在操作就可以了，完全不用操心cookie，session的处理，它甚至可以帮助你输入账户、密码，然后点击登录按钮，这些功能在应对一些常见的反爬虫机制时非常有用。

在我们开始示例代码之前，首先你要在Python中安装selenium库

```python
pip install selenium
```

安装好了之后，我们便开始探索抓取方法了。


那么我们就把一个淘宝店铺为示例，试着来爬取他里面的宝贝列表。你可以先用urllib来验证一下这个url，是不是爬取不到浏览器显示的dom内容。


```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

__author__ = 'Lix'

from selenium import webdriver
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.by import By
import time

def selenium_example():
    site_url = 'https://elcjstyle.taobao.com/search.htm?spm=a1z10.1-c-s.0.0.68616fccLXsimv&search=y'

    driver = webdriver.Chrome()
    driver.get(site_url)
    time.sleep(3)
    content = driver.page_source.encode('utf-8')
    print driver.title
    print content

def main():
    selenium_example()

if __name__ == "__main__":
    main()
```

执行完这段示例代码之后，不出意外会打印出店铺名字和整张页面的html代码。可以把这个代码和Chrome内调试环境下看到的html代码比较一下，是否完全一样了。

而在selenium中，更是有很多不同的策略可以定位到一个元素，实现它本身的自动化测试目的，而我们也可以配合`Beautiful Soup`或者`Xpath`来提取我们想要的内容。

一次查找多个元素 (这些方法会返回一个list列表):

```python

find_elements_by_name
find_elements_by_xpath
find_elements_by_link_text
find_elements_by_partial_link_text
find_elements_by_tag_name
find_elements_by_class_name
find_elements_by_css_selector

```

假如我们要通过name来查找一个元素:

页面元素如下所示：

```html
<html>
 <body>
  <form id="loginForm">
   <input name="username" type="text" />
   <input name="password" type="password" />
   <input name="continue" type="submit" value="Login" />
   <input name="continue" type="button" value="Clear" />
  </form>
</body>
<html>
```

name属性为 username & password 的元素可以像下面这样查找:

```python
username = driver.find_element_by_name('username')
password = driver.find_element_by_name('password')
```

通过这样的两句代码，我们就能提取到username和password的元素，所以selenium真的是一个很有用的工具呢。

关于selenium更多的用法，希望大家能认真阅读他的文档。这一定是一个会让你非常受用的工具的。
