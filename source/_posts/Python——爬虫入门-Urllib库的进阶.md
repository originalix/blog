---
title: Python——爬虫入门 Urllib库的进阶
date: 2018-02-07 21:06:30
tags: ["Python", "爬虫"]

---

上一篇文章我们简单讲解了Urllib库的基础用法，包括如何获取请求之后的页面响应，如何使用POST请求上传数据，今天我们就来讲讲Urllib库的几个进阶用法。

<!--more-->

## Headers:

我们先讨论关于请求头的使用，如何构造HTTP-Headers。我们先进入Chrome浏览器打开调试模式，

![](http://originalix.github.io/images/python-urllib.png)

在network一栏中找到Headers，在里面我们能看到Request Headers，这就是我们发送当前页面请求所用的请求头。其中User-Agent就是请求的身份，如果没有写入这个信息，那么有可能初级的反爬虫策略就会识别我们不是基于浏览器的请求，这次的请求就不会被响应了。

所以我们今天的第一段代码就是展示如何构造这个User-Agent的请求头：

```python

import urllib  
import urllib2  
 
url = 'http://originalix.github.io/#blog'
user_agent = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.140 Safari/537.36'  
headers = { 'User-Agent' : user_agent }  
request = urllib2.Request(url, None, headers)  
response = urllib2.urlopen(request)
html = response.read()
print html
```

上面的代码中的请求，我们就构造了一个携带携带User-Agent字段的请求，以后如果没有响应的页面，可要记得检查检查是不是忘记了在请求头里做文章了。

## URLError：

通常，URLError被抛出是因为网络请求出现了错误，比如服务器访问错误，或者访问的站点不存在，在这种情况下都会抛出一个URLError,这个错误是一个包含着reason和code的元组，分别对应着错误消息和错误代码。我们可以用try/except语句来捕获异常，例如:

```python

# URLError
import urllib2

req = urllib2.Request = ('http://www.lixxxxxxxx.com')
try:
    urllib2.urlopen(req)
except urllib2.URLError as e :
    print e.reason
```

在接触URLError之前，大家一定更早的接触过HTTPError，每个来自服务器的HTTP应答都会携带着一个包含数值的状态码，例如我们耳熟能详的200、404(页面丢失)、403(请求被禁止)等等。HTTPError的异常实例拥有一个整型的code属性。

```python
# 同时处理HTTPError和URLError

import urllib2

url = 'http://www.lixxxxxxxx.com'
req = urllib2.Request(url)
try:
	response = urllib2.urlopen(req)
except urllib2.HTTPError as e:
	print e.code
	print 'we can not fulfill the request \n'
except urllib2.URLError as e:
	print e.reason
	print 'we can not reach a server'
else:
	print('No problem')

```

上面的代码是一个同时处理HTTPError和URLError的例子，记得一定要把HTTPError放在前面处理，因为HTTPError是URLError的子集。

最后诸如代理什么的也就不讲解了，因为我觉得使用到这些的时候，大家可能就不会使用urllib2这个库了，有更好的轮子在等着你们。放上urllib2库的官方文档，有不懂的可以速查哟。 [urllib2官方文档任意门](https://docs.python.org/2/library/urllib2.html)

