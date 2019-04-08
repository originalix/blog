---
layout: wiki
title: 管理不同 git 系统的多个 ssh-key
categories: Git
description: 多个 ssh-key 的管理问题
keywords: Git, SSH-KEY
---

今天记录一下如何管理不同 git 系统下生成的 ssh-key。比如常用的 github 有一个 key，而公司搭建的 gitlab 又是一个不同邮箱生成的 key。那么这个时候该怎么办呢？

## 生成新的 key

```bash
ssh-keygen -t rsa -C "yourmail@gmail.com" 
```

首先使用这个命令来生成对应的 ssh-key ，但是记住多个不同的 key 不可以使用以前的那种一路回车的方式，必须要将不同的 key 分开命名。

完成之后可以到 ~/.ssh 目录下查看自己的密钥和公钥，然后在该目录下生成一个 config 文件。

```bash
cd ~/.ssh
touch config
vi config
```

接下来按照如下示例配置你的 config 文件

```bash
# github
Host github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa

# gitlab
Host gitlab.example.com
    HostName gitlab.example.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/cmm_rsa

```

记得把上面的 example 替换成对应的 git 系统的 host。

在完成上述步骤后执行如下命令

```bash
ssh-agent
```

把新建的私钥都添加上

```bash
ssh-add ~/.ssh/id_rsa

ssh-add ~/.ssh/example_rsa
```

在完成之后，记得测试一下是否真的成功哦。

测试方法（以 github 为例）:

```bash
ssh -vT git@github.com
```