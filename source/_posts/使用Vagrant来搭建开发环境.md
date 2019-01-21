---
title: 使用Vagrant来搭建开发环境
date: 2017-05-15 16:36:31
tags: ["Vagrant", "Linux"]

---

在大半年前刚刚接触PHP的时候，因为那时候只想先熟悉PHP的语法，并且对配置服务器、Mysql等一干事情不想花费太多时间，于是在网上找到了XAMPP这个解决方案。当时那是惊为天人，感觉虚拟主机很是方便。但是随着后来自己的慢慢深入，并且也在云服务器上陆续的部署自己的小项目，这才感觉到一个很恶心的事情，就是本地和线上的开发环境不同意，导致自己在频繁的修改配置文件，并且主力开发机器是用mac，家里还有win10的台式机，线上是Liunx系统，各种不一样的环境让我想寻求一个解决方案，统一线上和线下的开发环境。

在这种想法的指引下，很快有一个解决方案进入我的视线。通过搭建Liunx虚拟机，解决线上线下开发环境不统一的情况。这个解决方案，就是`VirtualBox` + `Vagrant`。目前他能完成我的所有需求，并且提供了很快捷的打包，来实现开发环境的迁移及统一部署，非常好用。本文就来记录如何使用Vagrant这个工具，好让我在日后部署环境的时候，能够把这些命令翻出来再看看。

<!--more-->

## 安装

> 实际上Vagrant只是一个让你可以方便设置你想要的虚拟机的便携式工具，它底层支持VirtualBox、VMware甚至AWS作为虚拟机系统，本书中我们将使用VirtualBox来进行说明，所以第一步需要先安裝Vagrant和VirtualBox。

安装环境：mac 
注：windows环境下，基本一致

### 安装VirtualBox

直接来到官网 [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) 点击下载最新的virtualbox，双击安装，一路傻瓜化完成。

### 安装Vagrant

在进行完上一个步骤后，我们就要考虑我们的虚拟机需要使用什么操作系统了。因为我线上使用了`Ubuntu16.04`的操作系统，所以我线下也选择了16.04的`Ubuntu`操作系统。在以前使用vm的过程中，我们需要自己去下载镜像，然后进行相关的安装，设置系统等等操作。而vagrant的开源社区，提供了很多已经打包好的操作系统，在vagrant的世界里被称为box。在 [http://www.vagrantbox.es/](http://www.vagrantbox.es/) 这里你可以找到你想要的操作系统box，当然你也可以自己制作一个。后续教程会讲到，这里就不多说。

我的开发机是Mac，所以我建立了如下的开发环境目录：

```
$ /Users/vagrant
$ cd /Users/vagrant
```

这里注意，vagrant提供的在线安装，有可能因为天朝的网络原因，很慢或者下载失败，所以我会找到box的下载链接，用迅雷等工具下载好这个盒子。之后执行安装设置

```
$ vagrant box add {title} {url}
$ vagrant init {title}
$ vagrant up
```

`vagrant box add` 是添加box的命令 
其中｛title｝可以自行设置，我这里使用的是 `Ubuntu` ，｛url｝是下载到本地box路径。我的路径是：/Users/vagrant/ubuntu.box

box中的镜像文件被放到了：/Users/.vagrant.d/boxes/，如果在window系统中应该是放到了： C:\Users\当前用户名.vagrant.d\boxes\目录下。

```

# 如果是才add 的box，就必须执行本步骤，初始化一次后，以后启动系统，就不需要执行本步骤。
$ vagrant init Ubuntu
```

输出如下 : 

```
A `Vagrantfile` has been placed in this directory.
You are now ready to `vagrant up` your first virtual environment!
Please read the comments in the Vagrantfile as well as documentation on `vagrantup.com` for more information on using Vagrant.
```

这样就会在当前目录生成一个 Vagrantfile的文件，里面有很多配置信息，后面我在慢慢说，默认不做任何配置改动，也是可以启动系统的。

```
# 启动系统
$ vagrant up
```

```
Bringing machine 'default' up with 'virtualbox' provider...
[default] Importing base box 'base'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
...
```

## ssh链接到虚拟机

经过以上操作后，完成了虚拟机的安装，现在需要登录上虚拟机，进行操作。链接很简单，可以使用第三方（xshell等）shell工具或系统自带的，进行登录 
在系统中，如mac，可直接使用 vagrant ssh 来完成链接。或者使用第三方如xshell，ip地址是：localhost，端口，需要观察，映射的22端口是多少。一般是2200 或者2222 
用户名与密码均是： vagrant

## vagrant的命令详解

命令 | 作用
----|----
vagrant box add	| 添加box的操作
vagrant init	| 初始化box的操作，会生成vagrant的配置文件Vagrantfile
vagrant up | 启动本地环境
vagrant ssh	| 通过 ssh 登录本地环境所在虚拟机
vagrant halt	| 关闭本地环境
vagrant suspend	| 暂停本地环境
vagrant resume	| 恢复本地环境
vagrant reload	| 修改了 Vagrantfile 后，使之生效（相当于先 halt，再 up）
vagrant destroy	 | 彻底移除本地环境
vagrant box list | 显示当前已经添加的box列表
vagrant box remove	| 删除相应的box
vagrant package	| 打包命令，可以把当前的运行的虚拟机环境进行打包
vagrant plugin	| 用于安装卸载插件
vagrant status	| 获取当前虚拟机的状态
vagrant global-status	| 显示当前用户Vagrant的所有环境状态


## 后记

配置好Vagrant只是开始，而之后在Linux配置环境，可以参考我之前的一篇文章，在《云服务器上部署Laravel》这篇文章，来配置自己的`LNMP`环境。
