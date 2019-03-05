---
layout: post
title: 在Mac上使用ssh-key免密码登录服务器
categories: 技术笔记
date: 2018-04-25 20:42:50
keywords: Linux
---

从很早之前开始，在搭建测试服务器的时候，就不停的谷歌怎么免密登录服务器，每次配置好免密登录后，到搭建新的服务器时，又忘记了具体的命令，所以决定把这个方法记下来，方便之后日后查找。

通常的来说，我们会使用 `ssh user@host -p port`这个命令，之后输入密码来登录服务器，才能ssh登录到服务器进行操作。如果一天需要登录很多遍服务器，就会输入很多次密码，偷懒的我当然不愿意这么干。而今天我们就要偷懒的进行免密码登录服务器的操作。

<!--more-->

Unix系的操作系统提供了各种ssh支持，我们可以通过这些来实现ssh登录。

首先我们要在我们的mac上电脑上生成公钥和私钥，在终端中输入以下命令:

```bash
cd ~/.ssh
```
首先进入`~/.ssh`目录，之后:

```bash
ssh-keygen -t rsa
```

之后就可以一路回车，一般都不设置密码，即可在~/.ssh目录中生成私钥文件(id_rsa)和公钥文件（id_rsa.pub）。如果熟悉git ssh-key配置的朋友，可能已经很熟悉这个步骤了，所以我们只要把公钥上传到我们的服务器的~/.ssh目录就好了。

所以我们可以用接下来的命令上传我们的公钥文件:

```bash
scp ~/.ssh/id_rsa.pub ssh foo@8.8.8.8 -p 2222:~/.ssh/
```

接下来我们登录到服务器中，将~/.ssh目录下的id_rsa.pub文件改名为authorized_keys:

```bash
mv id_rsa.pub authorized_keys
```

接着修改文件权限

```bash
chmod 700 ~/.ssh/
chmod 600 ~/.ssh/authorized_keys
```
现在，我们就可以正常的在mac 终端中使用ssh来登录服务器了，无需输入密码。

我们可以在`bash_profile`中设置一个`alias`，更能方便登录服务器的操作。