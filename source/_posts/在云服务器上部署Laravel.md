---
title: 在云服务器上部署Laravel
date: 2017-03-19 16:27:56
tags: ["php", "Linux", "Laravel部署"]

---


学习PHP和Laravel已经有一段时间了,但是所有的代码都是跑在本地的虚拟主机上的，于是去腾讯云申请了一个月的免费云主机，想把项目部署到云服务器上。

不得不说这里面的坑实在是有点多，让我这个初次接触服务器的小白摸不清头脑。在配置好服务器之后，部署一个Laravel项目更是费劲心思，于是乎想记录下部署Laravel项目的过程。

PS： Linux真是越用越有感觉的系统，回家在台式机上也要装个Linux敲代码用。

<!--more-->


# 环境简介

在操作系统的选择上，我选用了Linux ubuntu16.04的系统，使用的是LNMP的环境，即 Linux + Nginx + Mysql + PHP的环境。

## 删除Apache

```bash
sudo service apache2 stop
```

```bash
update-rc.d -f apache2 remove
```

```bash
sudo apt-get remove apache2
```
先用这三条命令来删除Apaceh 之后更新一下包列表

```bash
sudo apt-get update
```

## 1.安装Nginx

```bash
sudo apt-get install nginx
```

在安装完Nginx之后，要重启nginx 

```bash
sudo service nginx start
```
执行完之后，在浏览器输入云服务器分配给你的公网ip，就可以看到welcome to nginx的界面了

## 2. 安装Mysql

```bash
sudo apt-get install mysql-server mysql-client
```

过程中会提示你设置Mysql的密码，就跟平时的密码设置一样，一次输入，一次确认。密码确认完毕后基本等一会就安装好了。尝试

```bash
mysql -u root -p
```
如果登录成功，那Mysql就正确安装了。

## 3.安装PHP 

```bash
sudo apt-get install php5-fpm php5-cli php5-mcrypt
```
只有通过php5-fpm，PHP在Nginx下才能正常运行，遂，安装之。

至于php5-mcrypt，有些PHP框架会依赖于这个，比如Laravel就是，所以也把它装上了。

题外话，这里的php5我自己在部署时安装了php7 如果想尝试的也可以试试。

## 4.配置PHP

```bash
sudo vim /etc/php5/fpm/php.ini
```
打开PHP配置文件，找到cgi.fix_pathinfo选项，去掉它前面的注释分号;，然后将它的值设置为0,如下

```vim
cgi.fix_pathinfo=0
```

## 5. 启用php5-mcrypt:

```bash
sudo php5enmod mcrypt
```

## 6.重启php5-fpm:

```bash
sudo service php5-fpm restart
```

在搭建完LEMP环境之后，首先要明确两个重要目录

**Nginx的默认root文件夹**

```bash
/usr/share/nginx/html
```

**Nginx的服务器配置文件所在目录**
```bash
/etc/nginx/sites-available/
```

上面两个目录记住就好，很常用，先摆出来

# 下面一步一步在云服务器上部署Laravel

## 1.创建网站的根目录

```bash
sudo mkdir -p /var/www
```

## 2.配置nginx服务器

```bash
sudo vim /etc/nginx/sites-available/default
```

打开nginx的配置文件之后，找到server这一块，大概是长这个样子的

```bash
server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /usr/share/nginx/html;
        index index.html index.htm;

        server_name localhost;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

其中root，index ，server_name和location这几行需要稍微修改一下

**root修改**

```bash
root /var/www/laravel/public;
```

这里就是将nginx服务器的根目录指向Laravel的public文件夹下，后续的Laravel项目的代码我们会放在我们之前创建的/var/www/laravel目录下

**index修改**

```bash
index index.php index.html index.htm;
```

这里需要注意的是，将index.php排在最前面

**server_name修改**

```bash
server_name server_domain_or_IP;
```

将server_domain_or_IP修改为你的公网IP

**location修改**

```bash
location / {
        try_files $uri $uri/ /index.php?$query_string;
}
```

**修改完是这样的：**

```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /var/www/laravel/public;
    index index.php index.html index.htm;

    server_name server_domain_or_IP;

    location / {
            try_files $uri $uri/ /index.php?$query_string;
    }
}
```

最后我们还需要配置一下Nginx，让其执行PHP文件。同样是在这个文件里，在location下方添加下面的配置：

```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /var/www/laravel/public;
    index index.php index.html index.htm;

    server_name server_domain_or_IP;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri /index.php =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

注意，最下面的`location ~ \.php$`是自己加上去的：

配置完之后重启Nginx，使上面的配置项生效。

```bash
sudo service nginx restart
```

## 3.创建Laravel项目

在配置完nginx后，怎么获取Laravel的项目代码呢？有以下几种方法：

**(1).直接composer安装**

直接通过composer来安装，你可以在服务器上通过执行

```bash
cd ~
curl -sS https://getcomposer.org/installer | php
```

上面命令会安装composer

composer全局使用：

```bash
sudo mv composer.phar /usr/local/bin/composer
```

然后在/var/www目录下直接执行

```bash
sudo composer create-project laravel/laravel laravel
```

因为我们之前创建/var/www目录，你可以直接cd /var/www然后执行上面的命令。然后坐等安装完成。

**(2).直接上传代码**

使用下面命令上传

```bash
scp -r laravel root@your_IP:
```

然后在服务器上将laravel移动到/var/www目录下

```bash
sudo mv laravel/ /var/www
```

(3).使用Git和Coding平台

个人比较喜欢使用git来上传代码，可以很方便的更新代码和进行回滚，一旦版本更新出Bug我可以借助Git的强大版本管理能力来修复Bug。流程大概是这样：

本地代码－－－－>Github－－－－>云服务器
既然要使用git，那么先在云服务器上安装git：

```bash
sudo apt-get install git
```
安装完成就可以使用git了，然后在Github上创建一个私有项目laravel，里面包含所有该Laravel项目所需代码。

一旦本地代码都推送到Coding，然后在/var/www目录下直接使用

```bash
git clone your-project-git-link
```

your-project-git-link替换为你Github上的laravel项目地址

## 5.BINGO
在浏览器输入：

```html
http://server_domain_or_IP
```
至此，你可以在服务器上随意地用Laravel了,keep coding!

### 终极tips： 有了问题，页面出不来 各种错误 一定不要胡乱的调试，记得看log，非常有用。