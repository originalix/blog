---
layout: post
title: 修改Host文件优化github访问速度
categories: 技术笔记
description: github 访问提速
keywords: github，github访问慢，host
---

最近可能是一些不为人知的原因，我在家里使用电脑时，github 访问的速度实在难以忍受，就连常规的提交代码都非常的慢，尝试了一下 `ping github.com`发现各种丢包。

在谷歌了一番以后，发现修改 host 的方法还是挺好使的，就记录一下，等以后网络环境再恶劣以后继续使用。

## 在命令行 ping github.com

打开我们的命令行，输入 `ping github.com`，查询 ping 的结果

如果像这样的输出，那么就可以接着往下看，这个方法也许会对你有用。

```javascript
ping github.com
PING github.com (52.74.223.119): 56 data bytes
Request timeout
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
Request timeout for icmp_seq 3
Request timeout for icmp_seq 4
Request timeout for icmp_seq 5
```

## 寻找速度最快的 github 服务器

当我们在本地的环境，访问超时后，我们需要进行如下两步的操作，

- 打开链接 [](http://ping.chinaz.com) 输入 github.com , 点击 Ping检测
- 在 ping 检测完成后，记录下一条 TTL 值最小的 ip 地址，例如：192.30.255.121

## 修改 Hosts 文件

- 在命令行输入 `sudo vi /etc/hosts` ，回车之后进入 vim 编辑器开始编辑 hosts 文件
- 在文件的最后面添加 `192.30.255.113 github.com`，保存并退出
- 再次在命令行 `ping github.com` 就能看到效果了
- 请愉快的使用 github 吧 

最后再吐槽一下 GFW，国内的开发环境真的很痛苦，想用啥都得自己各种折腾啊。
