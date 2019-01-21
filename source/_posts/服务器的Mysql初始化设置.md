---
title: 服务器的Mysql初始化设置
date: 2018-10-09 14:49:22
tags: ["Mysql", "云服务器", "运维"]

---

继上篇博客记录了服务器的初始化安全设置之后，本篇文章会记录Mysql数据库的初始化安全设置。在操作mysql之前，最好先行备份，毕竟有“备”无患嘛。

## 一、修改root用户的口令

在Ubuntu16.04版本的服务器上，如果安装mysql的话会要求大家设置root的密码，若是没有设置过root用户的密码，可以用下面三种方法来这是

- 用mysqladmin命令来改root用户口令

```bash
mysqladmin -u root password 123456 // 设置mysql管理员root的密码为123456
```

- 用set password命令来修改密码：

```bash
mysql> set password for root@localhost=password('123456');

```

- 直接修改user表的root用户口令

```bash
mysql> use mysql;
mysql> update user set password=password('123456') where user='root';
mysql> flush privileges;
```

这就是三种修改mysql中管理员口令的操作方式。

<!--more-->

## 二、删除默认的数据库和用户

mysql初始化后会自动生成空用户和test库，但实际上这样会留有安全隐患，所以我们在这里选择全部删除的操作。我们在命令行进入mysql后执行下面这些命令。

```bash
mysql> drop database test;
mysql> use mysql;
mysql> delete from db;
mysql> delete from user where not(host="localhost" and user="root");
mysql> flush privileges;
```

## 三、修改默认的管理员名称

因为默认的管理员名称是root，为了防止别人对root这个登录名进行暴力破解，我们最好也能把root用户的名称改掉，防止穷举破解。

同样的执行下面的操作就能修改用户名，这个例子里我们把数据库管理员root的名称改为original。

```bash
mysql> use mysql;
mysql> update user set user="original" where user="root";
mysql> flush privileges;
```

## 四、禁止远程连接mysql

正常的话我们的mysql只是后端的程序来进行连接，所以我们无需开启socket进行监听，那么我们可以关闭mysql的监听功能。

- 在my.cnf文件里，[mysqld]的部分添加skip-networking参数。

- mysqld服务器中参数中添加 –skip-networking 启动参数来使mysql不监听任何TCP/IP连接，增加安全性。如果要进行mysql的管理的话,可以在服务器本地安装一个phpMyadmin来进行管理。

## 五、使用特定用户访问数据库

我们可以针对每个库，创建特定的账户，给予这些账户针对某个库有 update、select、delete、insert、drop table、create table等权限。这样就能很好的避免某个库被攻破后不牵连其他库了。而当我们设置完善之后，最好就不要在使用root用户来访问数据库了。

## 六、删除历史记录

执行以上的命令会被shell记录在历史文件里，比如bash会写入用户目录的.bash_history文件，如果这些文件不慎被读，
那么数据库的密码就会泄漏。用户登陆数据库后执行的SQL命令也会被MySQL记录在用户目录的.mysql_history文件里。
如果数据库用户用SQL语句修改了数据库密码，也会因.mysql_history文件而泄漏。所以我们在shell登陆及备份的时候
不要在-p后直接加密码，而是在提示后再输入数据库密码。 另外这两个文件我们也应该不让它记录我们的操作，以防万一。

```bash
$ rm .bash_history .mysql_history
$ ln -s /dev/null .bash_history
$ ln -s /dev/null .mysql_history
```

我自己常用的就是这六个步骤的设置，虽然不是很全，但是也勉强够用咯。