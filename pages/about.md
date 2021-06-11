---
layout: page
title: About
description: 李晓的博客
keywords: 李晓, Leon, Originalix
comments: true
menu: 关于
permalink: /about/
---

我是李晓，一名前端工程师。

持续学习中，致力于写出更优秀的代码。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
