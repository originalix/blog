---
title: Python——爬虫入门XPath的使用
date: 2018-02-27 20:30:35
tags: ["Python", "爬虫", "XPath"]

---

Xpath即为XML路径语言（XML Path Language）。它是一种用来确定XML文档中某部分位置的语言。

XPath基于XML的树状结构，提供在数据结构树种找寻节点的能力。起初XPath的提出的初衷是将其作为一个通用的、介于XPointer与XSL间的语法模型。但是XPath很快的被开发者采用来当做小型查询语言。

由于XPath确定XML文档中定位的能力，我们在用Python写爬虫时，常常使用XPath来确定HTML中的位置，辅助我们编写爬虫，抓取数据。

<!--more-->

## 节点

在Xpath中，有七种类型的节点：元素、属性、文本、命名空间、处理指令、注释以及文档节点（或者称为根节点）。

下面举几个节点的例子来说明：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<bookstore>

<book>
  <title lang="en">Harry Potter</title>
  <author>J K. Rowling</author> 
  <year>2005</year>
  <price>29.99</price>
</book>

</bookstore>
```

在上面的XML文档中：

```xml
<bookstore>  (文档节点)
<author>J K. Rowling</author> (元素节点)
lang="en" (属性节点)
```

## 表示法

Xpath最常见的表达式就是路径表达式（XPath这一名称的另一来源）。路径表达式是从一个XML节点（当前的上下文节点）到另一个节点、或一组节点的书面步骤顺序。这些步骤以“/”字符分开，每一步有三个构成部分。

- 轴描述（用最直接的方式接近目标节点）
- 节点测试（用于筛选节点位置和名称）
- 节点描述（用于筛选节点的属性和子节点特征）

一般情况下，我们使用简写后的语法，虽然完整的轴描述是一种更加贴近人类语言，利用自然语言的单词和语法来书写的描述方式，但是相比之下也更加啰嗦。


## 实例

我们将在下面的例子中使用这个XML文档。

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<bookstore>

<book>
  <title lang="eng">Harry Potter</title>
  <price>29.99</price>
</book>

<book>
  <title lang="eng">Learning XML</title>
  <price>39.95</price>
</book>

</bookstore>
```

我们来使用路径表达式在上面的XML文档中选取节点。

节点是通过沿着路径或者step来选取的。

下面表格列举的是最有用的路径表达式:

表达式 | 描述
---- | ---
nodename | 选取此结点的所有节点
/ |  从根节点选取
// | 从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置。
. | 选取当前节点
.. | 选取当前节点的父节点
@ | 选取属性

而下面的这个表格，我已经列出了一些路径表达式以及表达式的结果

路径表达式 | 结果
--- | ---
bookstore | 选取 bookstore 元素的所有子节点
/bookstore | 选取根元素bookstore ps: 假如路径起始于正斜杠(/),则此路径始终代表到某元素的绝对路径
bookstore/book | 选取属于bookstore的子元素的所有book元素 
//book | 选取所有book子元素，而不管它们在文档中的位置
bookstore//book | 选择属于bookstore元素的后代的所有book元素，而不管它们位于bookstore之下的什么位置
//@lang | 选取名为lang的所有属性


## 通配符选用节点

XPath通配符可用来选取未知的XML元素

通配符 | 描述
--- | ---
* | 匹配任何元素节点
@* | 匹配任何属性节点 
node() | 匹配任何类型的节点

## Python中的XPath库

通过 Python 的 LXML 库利用 XPath 进行 HTML 的解析。

lxml用法源自 lxml python 官方文档，更多内容请直接参阅官方文档，本文对其进行翻译与整理。

安装lxml

```
pip install lxml
```


现在我们简单的介绍完了XPath的语法，对于爬虫的准备知识已经铺垫完毕了，从下一篇博客开始，就要进入爬虫的实战教程了。