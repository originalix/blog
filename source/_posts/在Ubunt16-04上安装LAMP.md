---
title: 在Ubunt16.04上安装LAMP
date: 2018-05-09 21:47:54
tags: ["Linux", "Ubuntu 16.04", "LAMP"]

---

最近经常在自己的测试服务器上部署项目，也开了好几台测试服务器，都是用最简单的LAMP方案来建站的。毕竟是最简单易用的，LAMP即为 Linux、Web 服务器 (Apache)、 数据库服务器 (MySQL / MariaDB) 和 PHP (脚本语言)。由于我使用的都是Ubuntu 16.04的系统，所以我将记录基于Ubuntu16.04的系统中安装LAMP的过程。在这里我将默认你已经安装好Ubuntu 16.04的系统了。

<!--more-->

## Apache2 web服务器的安装:

在 Ubuntu Linux 中，web服务器是Apache2，我们可以利用如下命令安装Apache2。

```bash
$ sudo apt update

$ sudo apt install apache2 -y
```

当安装完成Apache2的包之后，Apache2的相关服务是启动的，并在重启后自动运行。在某些情况下，如果你的Apache2的服务并没有自动运行和启用，你可以利用如下命令来启用它：

```bash
$ sudo systemctl start apache2.service
$ sudo systemctl enable apache2.service
$ sudo systemctl status apache2.service
```

如果你开启了Ubuntu的防火墙（ufw）,那么你可以使用如下命令来解除web服务器的端口（80和443）限制：

```bash
$ sudo ufw status
Status: active
$ sudo ufw allow in 'Apache Full'
Rule added
Rule added (v6)

```

好了，这时候你已经可以输入你的服务器的IP地址来访问你的web服务器了，不出意外你会看到Apache2的欢迎页面。

## 数据库服务器的安装(MySQL Server 5.7):

MySQL 和 MariaDB 都是 Ubuntu 16.04 中的数据库服务器。 MySQL Server 和 MariaDB Server的安装包都可以在Ubuntu 的默认软件源中找到，我们可以选择其中的一个来安装。通过下面的命令来在终端中安装mysql服务器。

```bash
$ sudo apt install mysql-server mysql-client
```

在安装的过程中，它会要求你设置mysql服务器的root账户的密码：

![](https://dn-linuxcn.qbox.me/data/attachment/album/201606/14/203351wti8ttrbudtwb8io.jpg)

确认root账户的密码，并点击确定。

MySQL 服务器的安装到此已经结束了， MySQL 服务会自动启动并启用。我们可以通过如下的命令来校验 MySQL 服务的状态。

```bash
$ sudo systemctl status mysql.service
```

## PHP脚本语言的安装:

由于PHP7已经存在于Ubuntu的软件源中了，在终端中执行如下的命令来安装PHP7

```bash
$ sudo apt install php7.0-mysql php7.0-curl php7.0-json php7.0-cgi php7.0 libapache2-mod-php7.0
```

在`/var/www/html`的apache的根目录下创建一个简单的php页面。

```php
$ touch info.php
$ vi info.php

<?php
phpinfo();
?>
```

在vi中编辑之后保存并退出文件。

现在你可以从 web 浏览器中访问这个页面, 输入 : “http://<Server_IP>/info.php” ，你可以看到如下页面。

如果能看到紫色的PHPINFO页面，说明已经完全安装成功了。

## phpMyAdmin的安装：

phpMyAdmin 可以让我们通过它的 web 界面来执行所有与数据库管理和其他数据库操作相关的任务，这个安装包已经存在于 Ubuntu 的软件源中。

利用如下的命令来在 Ubuntu server 16.04 LTS 中安装 phpMyAdmin。

```bash
$ sudo apt install php-mbstring php7.0-mbstring php-gettext
$ sudo systemctl restart apache2.service
$ sudo apt install phpmyadmin
```

在以下的安装过程中，它会提示我们选择 phpMyAdmin 运行的目标服务器。

选择 Apache2 并点击确定。

点击确定来配置 phpMyAdmin 管理的数据库。

指定 phpMyAdmin 向数据库服务器注册时所用的密码。

确认 phpMyAdmin 所需的密码，并点击确认。

现在可以开始尝试访问 phpMyAdmin，打开浏览器并输入 : “http://Server_IP_OR_Host_Name/phpmyadmin”

使用我们安装时设置的 root 帐户和密码。
当我们点击“Go”的时候，将会重定向到如下所示的 ‘phpMyAdmin’ web界面。

如果这里出现了错误，那么记得给phpmyadmin加一个软链接，指向apache目录，

```bash
$ sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
```

到现在，LAMP 方案已经被成功安装并可以使用了，欢迎分享你的反馈和评论。



