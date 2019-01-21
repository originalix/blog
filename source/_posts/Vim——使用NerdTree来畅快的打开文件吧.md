---
title: Vim——使用NerdTree来畅快的打开文件吧
date: 2017-12-19 07:11:50
tags: ["Vim"]

---


在上一章我介绍完用[Vundle](https://github.com/VundleVim/Vundle.vim)来管理Vim中所有的插件后，今天我又要强推一个Vim的文件管理插件[Nerdtree](https://github.com/scrooloose/nerdtree),相信所有使用Vim的同学都知道文件管理插件NerdTree,这个几乎是所有拥护Vim的开发人员都会使用的插件，今天就总结一下如何合理的使用NerdTree。

<!--more-->

首先我们来看一下NerdTree的官方效果图:

![Nerdtree](https://github.com/scrooloose/nerdtree/raw/master/screenshot.png)

安装的话就使用咱们上一篇讲的Vundle插件进行安装，至于安装这样的小细节咱们在此就不再赘述。

当安装完成后，我们会有疑惑，如何召唤神龙打开NerdTree的文件列表呢？

答案非常简单，在你的`.vimrc`文件中添加`map <C-n> :NERDTreeToggle<CR>`这样一行语句，那么你便能通过`ctrl+n`来开启关闭Nerdtree了。

而如果你对Nerdtree已经到达爱不释手的地步，希望只要打开了vim，就能看到Nerdtree的可爱界面，那么你可以增加自动启动的配置语句:

`autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 1 && isdirectory(argv()[0]) && !exists("s:std_in") | exe 'NERDTree' argv()[0] | wincmd p | ene | endif`

ok，把它写入到你的`.vimrc`文件中，nerdtree就会跟着你的vim自动启动了。

在增加了自动启动之后，我们也会碰到比较烦人的事情，就是有时我们编辑完文件退出后，窗口里就留下来nerdtree，还需要自己再退出一次，除非用:qall。该怎么解决这个问题呢？

`autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTree") && b:NERDTree.isTabTree()) | q | endif`

那么将上面的配置语句添加好之后，nerdtree就会在孤零零一个人的时候，悄悄地退去。

Nerdtree的基本操作，附上给你哟：

```bash
?: 快速帮助文档
o: 打开一个目录或者打开文件，创建的是buffer，也可以用来打开书签
go: 打开一个文件，但是光标仍然留在NERDTree，创建的是buffer
t: 打开一个文件，创建的是Tab，对书签同样生效
T: 打开一个文件，但是光标仍然留在NERDTree，创建的是Tab，对书签同样生效
i: 水平分割创建文件的窗口，创建的是buffer
gi: 水平分割创建文件的窗口，但是光标仍然留在NERDTree
s: 垂直分割创建文件的窗口，创建的是buffer
gs: 和gi，go类似
x: 收起当前打开的目录
X: 收起所有打开的目录
e: 以文件管理的方式打开选中的目录
D: 删除书签
P: 大写，跳转到当前根路径
p: 小写，跳转到光标所在的上一级路径
K: 跳转到第一个子路径
J: 跳转到最后一个子路径
<C-j>和<C-k>: 在同级目录和文件间移动，忽略子目录和子文件
C: 将根路径设置为光标所在的目录
u: 设置上级目录为根路径
U: 设置上级目录为跟路径，但是维持原来目录打开的状态
r: 刷新光标所在的目录
R: 刷新当前根路径
I: 显示或者不显示隐藏文件
f: 打开和关闭文件过滤器
q: 关闭NERDTree
A: 全屏显示NERDTree，或者关闭全屏
```