---
layout: post
title: Python——爬虫入门 Urllib库的使用
categories: 后端技术
date: 2018-01-28 20:44:16
keywords: Python, 爬虫
---

最近在系统的学习Python爬虫，觉得还是比较有意思的，能够干很多的事情，所以也写点文章记录一下学习过程，帮助日后回顾。

网上关于Python的爬虫文章、教程特别多，尤其是喜欢刷知乎的用户，我总是感觉其他语言都是讨论xx框架如何，xx如何进阶，而Pythoner一开专栏，保准是xx爬虫入门教学，于是想零基础的入门Python爬虫也是非常容易的一件事，但是事实上会和达到工作要求，之间要学习补充的东西还是非常非常多的。

我在初学爬虫这段时间内，对于爬虫的流程，简单的概括为下面四步骤：网页 -> 网页源代码 -> 正则表达式 -> 需要的内容。而针对这四个方面展开则有许多许多可以提升并完善的细节，例如一个网页，我们就可以从在浏览器的地址栏敲入url之后说起。当然，这里我是不准备展开来说的。

而目前的文章我打算从爬虫的入门开始写，看看是否可以写出一个系列来记录，完成品是争取带大家爬出一个动态js加载的妹子页面。

爬虫的功能往简单的说就是把网页源代码想办法爬下来，然后分析我们需要的内容，总结起来就是两个部分：

- 爬！

- 提取。

<!-- more -->

所以整个过程里需要掌握的技能就是如何爬，以及爬到之后如何处理。

所以今天的入门文章里，我们就不去介绍第三方库的工具如何使用，我们来看看Python自带的标准库——Urllib库。

# Urllib

这个自带的标准库提供了诸如网页请求、响应获取、代理和cookie设置、异常处理、URL解析等等功能。

一个爬虫所需要的功能，基本上在urllib中都能找到，学习这个标准库，可以更加深入的理解之后要用到的第三方库，包括提高对于爬虫框架的理解。

那我们就从第一个网页的爬取入手，现在我们首先打开我们的编辑器，创建一个Python文件，并且在里面写入如下代码：

```python
import urllib2

response = urllib2.urlopen("http://www.baidu.com")
print(response.read().decode('utf-8'))
```

是的，就是这么简短的代码，现在我们来运行一下这个py文件。在命令行中能看到爬取到的百度网站的一堆html代码。是的，你没有看错，想抓到百度的html页面，只要这么简单的两行代码，看到命令行里反馈回来的这么多代码，是不是心里一阵痛快！

好，我们我们开始从头分析我们的三行代码，第一行，我们import了我们的urllib2的库。

第二行代码，我们使用urlopen的api，传入了url参数，执行urlopen方法后，就返回了一个response对象，我们打印的返回信息便保存在里面。

第三行代码，我们打印了response响应对象里的内容，并且转换成了utf-8编码，你可以试试如果去掉了utf-8的编码，是不是会乱码呢？

而我们请求之前，也可以根据urllib2提供的request的类，在发送请求前构造一个request的对象，然后通过urllib的urlopen函数来发送请求。例如上面请求百度的代码也可以写成这样：

```python
import urllib2

url = r'http://www.baidu.com'
req = urllib2.Request(url)
html = urllib2.urlopen(req).read()
print html
```

我们先用`req = urllib2.Request(url)`实例化了一个request对象，之后再请求打开这个网页。

而我们常用的HTTP协议里还有GET、POST等方法的请求，我们该如何实现呢，以Post请求为例，我们怎么发送一个携带着数据的请求呢。

```python
url = 'http://www.baidu.com/'
params = {'name' : 'Lix', 'location' : 'China'}
data = urllib.urlencode(params)
req = urllib2.Request(url,data)
response = urllib2.urlopen(req)
the_page = response.read()

print the_page
```

这样就可以完成一个post请求，至于GET方法，我们举例的第一段代码就是get类型的哦。

根据现在讲解的一些基本知识，我们就可以抓取到一些简单的页面的数据了，之后更深的内容我们在之后的文章里接着分析哦。

