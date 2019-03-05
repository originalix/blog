---
layout: post
title: 使用Vundle管理Vim插件
categories: 技术笔记
date: 2017-12-13 20:47:27
keywords: Vim
---

编辑器流派的划分在程序员群体中一直存在着，而我也在尝试了SublimeText、VSCode和Atom这些代表着现代时尚功能的编辑器之后试着使用Vim，其实说到学习使用Vim那已经是很早之前的事情了，但是每次看到各种配置，各种插件，总感觉把折腾Vim的配置是一件很要命的事情，如果把时间用在看书上或许会提高的更多，在这种阿Q的想法下，我始终没有认真的去研究过Vim的配置。

但是最近也算突然心血来潮，一直在敲前端和PHP的代码，因为有了一点嫌弃移动鼠标的麻烦，开始折腾起了Vim，也想配置出一个自己用得顺手的编辑器。虽然网上有很多完善强大的Vim配置文件，但是总感觉不是自己配置的，再怎么用也不舒服。所以一点一点折腾，一点一点爬坑，总算配置出了一个自己满足自己敲敲前端代码和日常编辑文本功能的编辑器。

<!--more-->

而折腾完才发现，Vim的代码补全，编译及错误跳转等插件功能其实还是足够使用的，尤其是我经常在家中和办公室切换电脑使用，各种系统之间跨平台支持做的非常的好，希望是一个可以终生使用的工具。

对于Vim中如此众多的插件，一个好的插件管理工具是必不可少的，所以今天在这里，我们来讲解一下Vundle这款插件管理器的使用。首先如果你不适用插件管理工具的话，那么你对插件的安装、配置和管理相对会麻烦很多，曾经没使用Vundle的时候，我经常遇到无法安装一些vim插件，但是使用Vundle后你只要在文件中添加一行你的插件名就ok了。

首先我们要去[Vundle的github库](https://github.com/VundleVim/Vundle.vim)下载安装Vundle。

```bash
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

执行上面的命令，将Vundle库下载到本地的`~/.vim/bundle/`目录下。


之后编辑我们的`.vimrc`文件，将下面的文本拷贝到你的`.vimrc`文件中，并且你可以选择性的删除你暂时不想安装的插件。

```bash
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from http://vim-scripts.org/vim/scripts.html
" Plugin 'L9'
" Git plugin not hosted on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Install L9 and avoid a Naming conflict if you've already installed a
" different version somewhere else.
" Plugin 'ascenator/L9', {'name': 'newL9'}

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
```

类似于`Plugin 'tpope/vim-fugitive'`这样的一行语句，就是安装一个新的插件，例如此行为安装`vim-fugitive`插件，例如这是一个Vim的Git相关插件，利用他可以很方便的查看对于文件的改动，还是很推荐安装使用的。

之后在我们的终端键入`vim`跳进vim编辑器的乌干达主界面，并且输入`:PluginInstall`，则会开始自动的执行插件安装过程，我们说的毫不费劲便是在此体现，一行语句对应一个插件。

在执行完毕后输入`qall`退出我们的插件安装界面，至此插件便安装完毕了。

至于怎么移除插件呢，同样是在`.vimrc`文件中删除对应的语句，并且在vim编辑器的界面，输入`PluginClean`就完成插件的清理了。
