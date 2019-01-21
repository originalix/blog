---
title: 腾讯云上免费部署HTTPS
date: 2018-05-22 20:15:05
tags: ["HTTPS", "Linux"]

---


最近在写微信小程序的时候，微信小程序需要所有的请求接口都部署在https协议上，于是就研究了一下怎么在腾讯云上部署https环境，发现还是比较简单的，首先我的服务器环境是Ubuntu 16.04, LAMP的环境。

## 获取SSL证书

腾讯云的SSL证书服务中，域名型的（DV）SSL证书是免费的，那么我们这次主要是申请这个证书，如需其他类型证书，也请付费申请。

进入SSL证书管理控制台，点击申请证书

<!--more-->

能看到如图所示的申请表单:

![](http://img.yzl1030.com/510586EC-C2CF-49EF-988E-5AF9044B5BCF.png)

填上申请信息后，等待大概一个小时左右，证书就能申请下来。接着在云解析里配置上申请的二级域名:

![](http://img.yzl1030.com/561179-20161123132020581-1776760709.png)

把二级域名解析好之后，待证书申请好。

在证书申请通过后，下载证书。

![](http://img.yzl1030.com/55E31E55-8095-4A55-A273-D6DD71F0ADC1.png)

## 上传SSL证书

将下载好之后的证书，解压，可以看到里面有Apache, IIS, Nginx, Tomcat等证书，这里根据自己的服务器环境选择对应的证书。这里根据我使用的是Apache环境，使用FileZilla将证书文件上传到Apache目录下，我上传的路径是`/etc/apache2/ctr`，ctr是我自己创建存储证书的文件夹。

## 添加HTTPS的Apache配置

待证书上传完成后，我在路径`/etc/apache2/sites-available`下创建一个文件，名为`vhostssl.conf`，在这个文件里写我这个站点的https配置信息。

```bash
Listen 443
<VirtualHost *:443>
    ServerName www.example.com:443
    DocumentRoot "/var/www/html/example"
    ServerAlias www.example.com
    SSLEngine on
    SSLCertificateFile "/etc/apache2/ctr/examplecom/Apache/2_example.com.crt"
    SSLCertificateKeyFile "/etc/apache2/ctr/examplecom/Apache/3_example.com.key"
    SSLCertificateChainFile "/etc/apache2/ctr/examplecom/Apache/1_root_bundle.crt"
</VirtualHost>
```

在`vhostssl.conf`文件内写入上述的配置信息，其中注意将`example`替换为你自己的域名，并且修改成正确的证书路径。

配置文件完成后，进入`/etc/apache2/sites-enabled/`路径，

```bash
ln -s ../sites-available/vhostssl.conf
```

执行这个命令，添加一个软链至`sites-available`目录。

在这些工作都做完后，执行

```bash
$ service apache2 restart
```

重启Apache服务器，然后在你配置的域名前输入https，就能看到一把小绿锁了，至此https的配置也就结束了。

在完成一遍配置后，会觉得特别简单是么？